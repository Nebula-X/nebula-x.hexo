---
title: 微服务API网关框架(2)--Nginx的使用
catalog: true
date: 2018-12-19 16:52:03
subtitle:
header-img: "Demo.png"
tags:
- 微服务API网关
catagories:
- 微服务API网关
---

# Nginx

## Nginx命令

### Nginx的基本配置
    Nginx的主配置文件是：nginx.conf，nginx.conf主要组成如下：
    
    # 全局区   有一个工作子进程，一般设置为CPU数 * 核数
    worker_processes  1; 
    events {
        # 一般是配置nginx进程与连接的特性
        # 如1个word能同时允许多少连接，一个子进程最大允许连接1024个连接
         worker_connections  1024;
    }
     # 配置HTTP服务器配置段
    http {
        # 配置虚拟主机段
        server {
             # 定位，把特殊的路径或文件再次定位。
             location  {
                       
             } 
        }
         server {
                       
         } 
    }

### nginx启动
    指令：nginx程序   -c   nginx配置文件
    # /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
    
### nginx重启
    #cd /usr/local/nginx/sbin
    ##重启
    # ./nginx -s reload   
    进入nginx可执行程序的目录
    # cd /usr/local/nginx/sbin/
    # ./nginx -s reload
    nginx: [error] invalid PID number "" in "/usr/local/nginx/logs/nginx.pid"
    重启是建立在nginx服务需要启动
    
### nginx停止
    # ./nginx -s stop 
    # ./nginx -s quit
    
    quit 是一个优雅的关闭方式，Nginx在退出前完成已经接受的连接请求
    stop 是快速关闭，不管有没有正在处理的请求。
    
### 重新打开日志
    # 用于日志切割   
    # ./nginx -s reopen   
    
### nginx检查配置文件
    检查配置文件是否正确
    第一种
    进入nginx可执行程序的目录
    # cd /usr/local/nginx/sbin/
    # ./nginx -t
    
    第二种
    # /usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf

## Nginx信号控制
    Nginx支持2种进程模型Single和Master-Worker
    Single是单进程，一般不适用，
    Master-Worker是主进程和工作进程模型运行，主进程对工作进程管理。
    Nginx允许我们通过信号来控制主进程，用信号的方式可以达到不影响现有连接的目的。 

### 信号类型
    INT，TERM		快速关闭信号
    QUIT			从容关闭信号
    HUP				从容重启信号，一般用于修改配置文件后，重启
    USR1			重读日志，一般用于日志的切割
    USR2			平滑升级信号
    WINCH			从容关闭旧进程

    具体语法:
    kill    -信号选项    nginx的主进程号
    例：
    # kill -INT 26661 
    # kill -HUP 4873

#### Nginx的优雅停止
    1）nginx停止
    #  ps -ef | grep nginx 获得进程号
    
    第1种从容“优雅”停止
    # kill -QUIT master进程号
    # Nginx服务可以正常地处理完当前所有请求再停止服务
    # 步骤：首先会关闭监听端口，停止接收新的连接，然后把当前正在处理的连接全部处理完，最后再退出进程。
    
    第2种快速停止
    # kill -TERM master进程号
    # kill -INT master进程号
    # 快速停止服务时，worker进程与master进程在收到信号后会立刻跳出循环，退出进程。
    第3种强制停止
    # pkill -9 nginx
    # 系统强杀nginx进程
    
    2）重启nginx
    # kill -HUP master进程号

#### Nginx平滑升级
    原理:把服务器从低版本升级为高版本，强行停止服务器，会影响正在运行的进程。
         平滑升级不会停掉正在进行中的进程，这些进程会继续处理请求。但不会再接受新请求，这些老的进程在处理完请求之后 会停止。此平滑升级过程中，新开的进程会被处理。

    
    一）平滑升级
        进入nginx可执行程序的目录
             #  cd /usr/local/nginx/sbin/
             # ./nginx -V  #查看nginx版本
    
        1）下载高版本nginx http://nginx.org/download/nginx-1.13.1.tar.gz
        执行指令
        #  ./configure
        # make    #不能执行 make install
        # cd objs
        此目录下 有高版本的nginx
        备份低版本的nginx
        cp nginx nginx.old
        执行强制覆盖
        cp -rfp objs/nginx /usr/local/nginx/sbin
    
        测试一下新复制过来文件生效情况：
        # /usr/local/nginx/sbin/nginx -t
        # ps -ef | grep nginx
    
        2）执行信号平滑升级
            # kill -USR2 `cat /usr/local/nginx/logs/nginx.pid`  更新配置文件
            给nginx发送USR2信号后，nginx会将logs/nginx.pid文件重命名为nginx.pid.oldbin，然后用新的可执行文件启动一个新的nginx主进程和对应的工作进程，并新建一个新的nginx.pid保存新的主进程号
            
            # ps -ef | grep nginx
            
            # ll logs/
        
        
        3）kill -WINCH 旧的主进程号
            旧的主进程号收到WINCH信号后，将旧进程号管理的旧的工作进程优雅的关闭。即一段时间后旧的工作进程全部关闭，只有新的工作进程在处理请求连接。这时，依然可以恢复到旧的进程服务，因为旧的进程的监听socket还未停止。
            处理完后，工作进程会自动关闭
            # ps -ef | grep nginx
        
        4）# kill -QUIT `cat /usr/local/nginx/logs/nginx.pid.oldbin` 优雅的关闭
            给旧的主进程发送QUIT信号后，旧的主进程退出，并移除logs/nginx.pid.oldbin文件，nginx的升级完成。
        
        升级完成了，最后在看一下升级后的版本
    
        查看
        ./nginx -V
        已经平滑升级成功
    
    二）中途停止升级，回滚到旧的nginx
        在步骤(3)时，如果想回到旧的nginx不再升级
        
        (1)给旧的主进程号发送HUP命令，此时nginx不重新读取配置文件的情况下重新启动旧主进程的工作进程。
            kill -HUP 9944 --旧主进程号
            重启工作进程
        (2)优雅的关闭新的主进程
            kill -QUIT 10012  --新主进程号

#### Nginx 用户
    #以哪个用户，运行nginx应用
    #nobody是个低权限用户，为了安全
    #user nobody;

#### Nginx 进程数设置    
    worker_processes：表示开启nginx的worker进程的个数，nginx启动会开两种进程，master进程用来管理调度，worker进程用来处理请求；
    上面表示两种设置方法，比如
    方法一：worker_processes auto;
    　　表示设置服务器cpu核数匹配开启nginx开启的worker进程数    
    　　查看cpu核数：cat /proc/cpuinfo    
    方法二：nginx设置cpu亲和力
    　　worker_processes 8;    
    　　worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;    
    　　00000001表示启用第一个CPU内核，00000010表示启用第二个CPU内核，以此类推
    
    worker_cpu_affinity：表示开启八个进程，第一个进程对应着第一个CPU内核，第二个进程对应着第二个CPU内核，以此类推。
        这种设置方法更高效，因将每个cpu核提供给固定的worker进程服务，减少cpu上下文切换带来的资源浪费
    
    如果服务器cpu有限
        比如：2核CPU，开启2个进程，设置如下 
        worker_processes     2;   
        worker_cpu_affinity 01 10;  
        比如：4核CPU,开启4个进程，设置如下
        worker_processes     4;
        worker_cpu_affinity 0001 0010 0100 1000;
    
    8核cpu , worker_processes=8
    1个worker进程 能够最大打开的文件数（线程数）worker_connections=65535 （参考worker_rlimit_nofile  ---->  linux  ulimit -n）
    最大的客户端连接数 max_clients = （多少个工作进程数）worker_processes * （1个工作线程的处理线程数）worker_connections    8*65535
    
#### Nginx的并发量计算
    #nginx作为http服务器
    #请求模型   client <---> nginx
    #max_clients = worker_processes * worker_connections/2
    
    #nginx作为反向代理服务器的时候
    #请求模型   client <---> nginx  <----> web server
    #max_clients = worker_processes * worker_connections/4
    (
    为什么除以2：该公式基于http 1.1协议，一次请求大多数浏览器发送两次连接，并不是request和response响应占用两个线程（很多人也是这么认为，实际情况：请求是双向的，连接是没有方向的，由上面的图可以看出来)
    为什么除以4：因nginx作为方向代理，客户端和nginx建立连接，nginx和后端服务器也要建立连接
    )
    
    由此，我们可以计算nginx作为http服务器最大并发量(作为反向代理服务器自己类推)，可以为压测和线上环境的优化提供一些理论依据：
    单位时间（keepalive_timeout）内nginx最大并发量C
    C=worker_processes * worker_connections/2=8*65535/2
    
    而每秒的并发量CS
    CS=worker_processes * worker_connections/(2*65)    
    
    
#### Nginx全局错误日志
    #全局错误日志 
        #nginx的error_log类型如下（从左到右：debug最详细 crit最少）： 
        #[ debug | info | notice | warn | error | crit ] 
    
    #例如：error_log logs/nginx_error.log  crit; 
    #解释：日志文件存储在nginx安装目录下的 logs/nginx_error.log ，错误类型为 crit ，也就是记录最少错误信息;
        error_log  logs/error.log;
        error_log  logs/notice.log  notice;
        error_log  logs/info.log  info;
    
#### Nginx主进程ID
    #PID文件，记录当前启动的nginx的进程ID
        pid        logs/nginx.pid; # 主进程ID存放位置
        
#### Nginx的worker进程
    worker_rlimit_nofile 65535;
    
    #这个参数表示worker进程最多能打开的文件句柄数，基于liunx系统ulimit设置
    #查看系统文件句柄数最大值：ulimit -n
    #Linux一切皆文件，所有请求过来最终目的访问文件，所以该参数值设置等同于liunx系统ulimit设置为优
    #可以通过linux命令设置  最大的文件句柄数65535

#### Nginx工作模式与连接数上限
    events {
       #网络模型高效(相当于建立索引查找结果)，nginx配置应该启用该参数
       #但是仅用于linux2.6以上内核,可以大大提高nginx的性能
       use   epoll;             
       #该参数表示设置一个worker进程最多开启多少线程数
       #优化设置应该等同于worker_rlimit_nofile设置值，表明一个线程处理一个http请求，同时可以处理一个文件数，各个模块之间协调合作不等待。
       worker_connections  65535;
    }
                
#### Nginx负载均衡
     #设定http服务器，利用它的反向代理功能提供负载均衡支持
         http {
              #设定mime类型,类型由mime.type文件定义
              #MIME(Multipurpose Internet Mail Extensions)多用途互联网邮件扩展类型。是设定某种扩展名的文件用一种应用程序来#打开的方式类型，当该扩展名文件被访问的时候，浏览器会自动使用指定应用程序来打开
             include       /etc/nginx/mime.types;
             default_type  application/octet-stream;
             #设定日志格式
            log_format  main  '[$remote_addr] - [$remote_user] [$time_local] "$request" '
                           '$status $body_bytes_sent "$http_referer" '
                           '"$http_user_agent" "$http_x_forwarded_for"';
             access_log    /var/log/nginx/access.log;
     
             #sendfile 开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
             sendfile        on;
             tcp_nopush     on; #防止网络阻塞
             tcp_nodelay        on; #防止网络阻塞
     
             #连接超时时间
             #keepalive_timeout  0;  
             keepalive_timeout  65; #长连接超时时间，单位是秒
            
     
             #开启gzip压缩
            gzip  on;
     		gzip_disable "MSIE [1-6]\."; # IE6及以下禁止压缩 
            gzip_min_length 1k; #最小压缩文件大小
     		gzip_buffers 4 16k; #压缩缓冲区
     		gzip_http_version 1.0; #压缩版本（默认1.1，前端如果是squid2.5请使用1.0）
     		gzip_comp_level 2; #压缩等级
     		gzip_types text/plain application/x-javascript text/css application/xml; #压缩类型
     		gzip_vary on; #给CDN和代理服务器使用，针对相同url，可以根据头信息返回压缩和非压缩副本
     
     
             #设定请求缓冲
             client_header_buffer_size    1k;   #上传文件大小限制
             large_client_header_buffers  4 4k;  #设定请求缓存
     
     
             #设定负载均衡的服务器列表
             upstream mysvr {
     	        #weigth参数表示权值，权值越高被分配到的几率越大
     	        server 192.168.8.1x:3128 weight=5;
     	        server 192.168.8.2x:80  weight=1;
     	        server 192.168.8.3x:80  weight=6;
             }
     
             upstream mysvr2 {
     	        #weigth参数表示权值，权值越高被分配到的几率越大
     	        server 192.168.8.x:80  weight=1;
     	        server 192.168.8.x:80  weight=6;
             }
     
             #虚拟主机的配置
            server {
                 #侦听80端口
                 listen       80;
                 #设置编码
      	          #charset koi8-r;
     
                 #定义使用www.xx.com访问 域名可以有多个，用空格隔开
                 server_name  www.xx.com;
     
                 #设定本虚拟主机的访问日志
                 access_log  logs/www.xx.com.access.log  main;
     
             #默认请求
             location / {
                   root   /root;      #定义服务器的默认网站根目录位置
                   index index.php index.html index.htm;   #定义首页索引文件的名称
     
     
                  proxy_pass  http://mysvr ;#请求转向mysvr 定义的服务器列表
     
                   client_max_body_size 10m;    #允许客户端请求的最大单文件字节数
                   client_body_buffer_size 128k;  #缓冲区代理缓冲用户端请求的最大字节数，	
     
                  #以下是一些反向代理的配置可删除.
     
                   proxy_redirect off;
     
                   #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
                   proxy_set_header Host $host;
                   proxy_set_header X-Real-IP $remote_addr;
                   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                   proxy_connect_timeout 90;  #nginx跟后端服务器连接超时时间(代理连接超时)
                   proxy_send_timeout 90;        #后端服务器数据回传时间(代理发送超时)
                   proxy_read_timeout 90;         #连接成功后，后端服务器响应时间(代理接收超时)
                   proxy_buffer_size 4k;             #设置代理服务器（nginx）保存用户头信息的缓冲区大小
                   proxy_buffers 4 32k;               #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
                   proxy_busy_buffers_size 64k;    #高负荷下缓冲大小（proxy_buffers*2）
                   proxy_temp_file_write_size 64k;  #设定缓存文件夹大小，大于这个值，将从upstream服务器传
     
             }
     
             # 定义错误提示页面
             error_page   500 502 503 504 /50x.html; 
                 location = /50x.html {
                 root   /root;
             }
     
             #本地动静分离反向代理配置
     		#所有jsp的页面均交由tomcat或resin处理
     		location ~ .(jsp|jspx|do)?$ {
     			proxy_set_header Host $host;
     			proxy_set_header X-Real-IP $remote_addr;
     			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
     			proxy_pass http://127.0.0.1:8080;
     		}
     
             #静态文件，nginx自己处理
             location ~ ^/(images|javascript|js|css|flash|media|static)/ {
                 root /var/www/virtual/htdocs;
                 #过期30天，静态文件不怎么更新，过期可以设大一点，如果频繁更新，则可以设置得小一点。
                 expires 30d;
             }
     
             #设定查看Nginx状态的地址
             location /NginxStatus {
                 stub_status            on;
                 access_log              on;
                 auth_basic              "NginxStatus";
                 auth_basic_user_file  conf/htpasswd;
                 #htpasswd文件的内容可以用apache提供的htpasswd工具来产生。
             }
             #禁止访问 .htxxx 文件
             location ~ /\.ht {
                 deny all;
             }
     
             }
         }
       
#### Nginx虚拟主机
    1）虚拟主机
    虚拟主机使用的是特殊的软硬件技术，它把一台运行在因特网上的服务器主机分成一台台“虚拟”的主机，每台虚拟主机都可以是一个独立的网站，可以具有独立的域名，具有完整的Intemet服务器功能（WWW、FTP、Email等），同一台主机上的虚拟主机之间是完全独立的。从网站访问者来看，每一台虚拟主机和一台独立的主机完全一样。
    利用虚拟主机，不用为每个要运行的网站提供一台单独的Nginx服务器或单独运行一组Nginx进程。虚拟主机提供了在同一台服务器、同一组Nginx进程上运行多个网站的功能。
    
    2）配置虚拟主机
    我们先配置在一个nginx中配置一个虚拟主机，编辑nginx.conf配置文件，在http模块中，配置server模块，一个server模块就针对一个虚拟主机。
    我们模拟一个独立的网站，此网站域名访问为www.server1.com；网站的根目录放到nginx目录下html/server1目录，我们创建一个首页index.html到server1中，编辑index.html
    <!DOCTYPE html>
    <html>
    <head>
    <title>server1 首页</title>
    </head>
    <body>
    <h1>server1 首页</h1>
    </body>
    </html>
    下面我们回到nginx.conf配置文件中，配置server模块
    server {
    listen 80; #监听80端口
    server_name www.server1.com; #虚拟主机名，可以为域名或ip地址
    location / { #默认请求路由，以后文章中会重点介绍
    root html/server1; #网站的根目录
    index index.html index.htm; #默认首页文件名
    }
    }
    配置完成之后，重启nginx。因为www.server1.com是模拟的，需要在访问的客户端配置一下域名映射，老顾访问的客户端用的是windows系统，所以要到C:\Windows\System32\drivers\etc\目录下，编辑hosts文件，增加个映射
    192.168.5.150 www.server1.com
    192.168.5.150是老顾的nginx服务器地址，注意编辑hosts要用管理员身份编辑，要不然会报无权限修改错误。
    打开浏览器，访问www.server1.com
    
    
    3）第一个虚拟主机配置完成，我们再配置一个server2，与server1配置类似，先给server2网站创建一个根目录，nginx目录下html/server2目录，编辑index.html
    <!DOCTYPE html>
    <html>
    <head>
    <title>server2</title>
    </head>
    <body>
    <h1>server2 首页</h1>
    </body>
    </html>
    再编辑nginx.conf，再增加个server模块，监听还是80端口，但服务名改为www.server2.com
    server {
    listen 80; #监听80端口
    server_name www.server2.com; #虚拟主机名，可以为域名或ip地址
    location / { #默认请求路由，以后文章中会重点介绍
        root html/server2; #网站的根目录
        index index.html index.htm; #默认首页文件名
    }
    }
    重启nginx，不要忘了把hosts再增加个域名映射
    192.168.5.150 www.server2.com
    打开浏览器访问www.server2.com,运行结果
    
    
    这样第二个虚拟主机也配置完成。说明一下 再配置server模块是，监听的端口 listen 和 server_name 组合起来是唯一的，如果server_name一样，那么listen监听的端口就不一样；如端口一样，server_name就不一样。这是很好理解的，虚拟主机的请求映射系统才能够判别。

## Nginx日志与日志切割

### 日志文件的格式配置
    nginx服务器在运行的时候，会有各种操作，操作的信息会记录到日志文件中，日志文件的记录是有格式的。那我们如何设置日志文件的格式呢？
    使用log_format指令进行配置文件格式
    nginx的log_format有很多可选的参数用于指示服务器的活动状态，默认的是：
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '    '$status $body_bytes_sent "$http_referer" '    '"$http_user_agent" "$http_x_forwarded_for"';
    
    192.168.31.247 - - [11/Mar/2018:16:26:43 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3350.0 Safari/537.36" "-"

#### 日志相关参数
    参数                      说明                                         示例
    $remote_addr             客户端地址                                    211.28.65.253
    $remote_user             客户端用户名称                                --
    $time_local              访问时间和时区                                18/Jul/2012:17:00:01 +0800
    $request                 请求的URI和HTTP协议                           "GET /article-10000.html HTTP/1.1"
    $http_host               请求地址，即浏览器中你输入的地址（IP或域名）     www.wang.com 192.168.100.100
    $status                  HTTP请求状态                                  200
    $upstream_status         upstream状态                                  200
    $body_bytes_sent         发送给客户端文件内容大小                        1547
    $http_referer            url跳转来源                                   https://www.baidu.com/
    $http_user_agent         用户终端浏览器等信息                           "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; SV1; GTB7.0; .NET4.0C;
    $ssl_protocol            SSL协议版本                                   TLSv1
    $ssl_cipher              交换数据中的算法                               RC4-SHA
    $upstream_addr           后台upstream的地址，即真正提供服务的主机地址     10.10.10.100:80
    $request_time            整个请求的总时间                               0.205
    $upstream_response_time  请求过程中，upstream响应时间                    0.002
    $http_x_forwarded_for    是反向代理服务器转发客户端地址的参数
    
    假设将Nginx服务器作为Web服务器，位于负载均衡设备、Squid、Nginx反向代理之后，不能获取到客户端的真实IP地址了。
    原因是经过反向代理后，由于在客户端和Web服务器之间增加了中间层，因此Web服务器无法直接拿到客户端的IP。
    通过$remote_addr变量拿到的将是反向代理服务器的IP地址。
    但是，反向代理服务器在转发请求的HTTP头信息中，可以增加X-Forwarded-For信息，用以记录原有的客户端IP地址和原来客户端请求的服务器地址。
    这时候，要用log_format指令设置日志格式，让日志记录X-Forearded-For信息中的IP地址，即客户的真实IP。
    日志文件路径配置

#### access_log指令
    语法: access_log path [format [buffer=size [flush=time]]];
    access_log path format gzip[=level] [buffer=size] [flush=time];
    access_log off;
    默认值: access_log logs/access.log combined;
    配置段: 
    gzip压缩等级。
    buffer设置内存缓存区大小。
    flush保存在缓存区中的最长时间。
    不记录日志：access_log off;
    使用默认combined格式记录日志：access_log logs/access.log 或 access_log logs/access.log combined;
    值得注意的是，Nginx进程设置的用户和组必须对日志路径有创建文件的权限，否则，会报错。
    此外，对于每一条日志记录，都将是先打开文件，再写入日志，然后关闭。可以使用open_log_file_cache来设置日志文件缓存(默认是off)。

#### 日志文件切割
    server1.log   ---->  server1-2018-03-11.log  ---> server1-2018-03-12.log
                  ----》 server1-2018-03-10.log 
    
    通过mv命令 把当前log文件重命令
    再用信号控制指令 发送重读日志指令  产生了新的日志log文件
    
    nginx日志默认情况下统统写入到一个文件中，文件会变的越来越大，非常不方便查看分析。以日期来作为日志的切割是比较好的，通常我们是以每日来做统计的。下面来说说nginx日志切割。
    我们先手动完成日志文件切割
    到logs目录中，先备份日志文件，在重新生成日志文件
    mv access.log access_20180124.log
    kill -USR1 pid进程号   #向 Nginx 主进程发送 USR1 信号。USR1 信号是重新打开日志文件

#### 系统自动切割
    利用sh脚本的方式执行刚才的手动操作，在每天凌晨执行一个计划任务 调用sh脚本，就完成的系统自动切割日志文件
    
    编写脚本
    在nginx目录下logs目录
    # touch cutlog.sh脚本
    # vi cutlog.sh 
    #!/bin/bash
    LOGS_PATH=/usr/local/nginx/logs
    YESTERDAY=$(date -d "yesterday" +%Y-%m-%d)
    mv ${LOGS_PATH}/access.log ${LOGS_PATH}/access_${YESTERDAY}.log
    kill -USR1 $(cat /usr/local/nginx/logs/nginx.pid)   # 向 Nginx 主进程发送 USR1 信号。USR1 信号是重新打开日志文件
    
    注意：执行 sed -i 's/\r$//' cutlog.sh
    原因：
    
        这个文件在Windows 下编辑过，在Windows下每一行结尾是\n\r，而Linux下则是\n
    
        sed -i 's/\r$//' cutlog.sh 会把cutlog.sh中的行尾的\r替换为空白
    
    
    设置定时任务
    # vi  /etc/crontab
    0 0 * * * root /usr/local/nginx/logs/cutlog.sh  
    表示配置一个定时任务，定时每天00:00以root身份执行脚本/usr/local/nginx/logs/cutlog.sh，实现定时自动分割Nginx日志

