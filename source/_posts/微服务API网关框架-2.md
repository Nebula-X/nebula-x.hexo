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
            旧的主进程号收到WINCH信号后，将旧进程号管理的旧的工作进程优雅的关闭。即一段时间后旧的工作进程全部关闭，只有新的工作进程在处理请求连接。
            这时，依然可以恢复到旧的进程服务，因为旧的进程的监听socket还未停止。
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
              #MIME(Multipurpose Internet Mail Extensions)多用途互联网邮件扩展类型。是设定某种扩展名的文件用一种应用程序来#打开的方式类型，
              当该扩展名文件被访问的时候，浏览器会自动使用指定应用程序来打开
             include       /etc/nginx/mime.types;
             default_type  application/octet-stream;
             #设定日志格式
            log_format  main  '[$remote_addr] - [$remote_user] [$time_local] "$request" '
                           '$status $body_bytes_sent "$http_referer" '
                           '"$http_user_agent" "$http_x_forwarded_for"';
             access_log    /var/log/nginx/access.log;
     
             #sendfile 开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，
             如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
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
    虚拟主机使用的是特殊的软硬件技术，它把一台运行在因特网上的服务器主机分成一台台“虚拟”的主机，每台虚拟主机都可以是一个独立的网站，可以具有独立的域名，
    具有完整的Intemet服务器功能（WWW、FTP、Email等），同一台主机上的虚拟主机之间是完全独立的。从网站访问者来看，每一台虚拟主机和一台独立的主机完全一样。
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
    配置完成之后，重启nginx。因为www.server1.com是模拟的，需要在访问的客户端配置一下域名映射，老顾访问的客户端用的是windows系统，
    所以要到C:\Windows\System32\drivers\etc\目录下，编辑hosts文件，增加个映射
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
    
    
    这样第二个虚拟主机也配置完成。说明一下 再配置server模块是，监听的端口 listen 和 server_name 组合起来是唯一的，
    如果server_name一样，那么listen监听的端口就不一样；如端口一样，server_name就不一样。这是很好理解的，虚拟主机的请求映射系统才能够判别。

## Nginx日志与日志切割

### 日志文件的格式配置
    nginx服务器在运行的时候，会有各种操作，操作的信息会记录到日志文件中，日志文件的记录是有格式的。那我们如何设置日志文件的格式呢？
    使用log_format指令进行配置文件格式
    nginx的log_format有很多可选的参数用于指示服务器的活动状态，默认的是：
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '    '$status $body_bytes_sent "$http_referer" '    '"$http_user_agent" "$http_x_forwarded_for"';
    
    192.168.31.247 - - [11/Mar/2018:16:26:43 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64) 
    AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3350.0 Safari/537.36" "-"

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
    
    nginx日志默认情况下统统写入到一个文件中，文件会变的越来越大，非常不方便查看分析。
    以日期来作为日志的切割是比较好的，通常我们是以每日来做统计的。下面来说说nginx日志切割。
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

## Nginx的Location
    nginx的location配置详解
    1）语法规则： location [=|~|~*|^~] /uri/ { … }
    构成：
    指令                  前缀                  uri
    location          [=|~|~*|^~]           /uri
    
    路由匹配规则，正则匹配，正则表达式
    
    
    
    2）location区分普通匹配和正则匹配
    
    用前缀 “~” 和 “~*”修饰的为正则匹配
    ~   前缀表示区分大小写的正则匹配
    ~*  前缀表示不区分大小写的正则匹配
    
    除上面修饰的前缀（“=” 和 “^~”，或没有前缀修饰）都为普通匹配
    =   前缀表示精确匹配
    ^~  前缀表示uri以某个常规字符串开头，可以理解为url的普通匹配
    
    location作用于server模块,且支持多个location模块
    server {
           .........
            location /p {
                root   html/p;
                index  index.html index.htm;
            }
            location = /50x.html {
                root   html;
            }
            location / {
                root   html/server1;
                index  index.html index.htm;
            }
    }
    在多个location情况下，是按照什么原则进行匹配的呢？
    
    3）匹配的原则
    普通匹配：优先原则---->最大前缀匹配原则; 顺序无关
    如：
    server {
    	location /prefix/ {
    		#规则A
    	}
    	location /prefix/mid/ {
    		#规则B
    	} 
    }
    
    请求url为：/prefix/mid/t.html 
    
    此请求匹配的是 规则B，是以最大的匹配原则进行的，跟顺序无关
    
    ------------------------
    正则匹配：为顺序匹配，优先原则：谁在前面 就匹配谁；顺序相关
    如：
    server {
    	location ~ \.(gif|jpg|png|js|css)$ {
    	   		#规则C
    	}
    	location ~* \.png$ {
    	   		#规则D
    	}
    }
    请求http://localhost/1.png,匹配的是规则C，因为规则C在前面，即叫做顺序匹配
    
    ----------------
    如果location有普通匹配也有正则匹配，那匹配的原则为
    匹配模式及顺序
    ----带前缀普通匹配 最优先，=前缀优先级最高
    location = /uri 　　　=开头表示精确匹配，只有完全匹配上才能生效。
    location ^~ /uri 　　^~ 开头对URL路径进行前缀匹配，并且在正则之前。
    ----正则匹配
    location ~ pattern 　~开头表示区分大小写的正则匹配。
    location ~* pattern 　~*开头表示不区分大小写的正则匹配。
    ---不带前缀匹配
    location /uri 　　　　不带任何修饰符，也表示前缀匹配，但是在正则匹配之后。
    
    location / 　　　　　通用匹配，任何未匹配到其它location的请求都会匹配到，相当于switch中default。 
    
    首先匹配 =，其次匹配^~, 其次是按文件中顺序的正则匹配，不带前缀普通匹配，最后是交给 / 通用匹配。
    当有匹配成功时候，停止匹配，按当前匹配规则处理请求。
    
    例子，有如下匹配规则：
    location = / {
                return 200 '规则A';
            }
    location = /login {
                return 200 '规则B';
            }
    location ^~ /static/ {
                return 200 '规则C';
            }
    location ~ \.(gif|jpg|png|js|css)$ {
                return 200 '规则D';
            }
    location ~* \.js$ {
                return 200 '规则E';
            }	
    location / {
                return 200 '规则F';
    }
    那么产生的效果如下：
    访问根目录/， 比如http://localhost/ 将匹配规则A
    访问 http://localhost/login 将匹配规则B，
    http://localhost/register 则匹配规则F
    http://localhost/static/a.html 将匹配规则C
    http://localhost/a.css, 匹配规则D
    http://localhost/b.js则优先匹配到 规则D，不会匹配到规则E
    http://localhost/static/c.js 则优先匹配到 规则C
    http://localhost/a.JS 则匹配规则E， 而不会匹配规则D，因为规则E不区分大小写。
    访问 http://localhost/category/id/1111 则最终匹配到规则F，因为以上规则都不匹配，
    
    
    4）在实际场景中，通常至少有三个匹配规则定义，如下：
    #直接匹配网站根，通过域名访问网站首页比较频繁，使用这个会加速处理。
    #这里是直接转发给后端应用服务器了，也可以是一个静态首页
    # 第一个必选规则
    location = / {
        .....
    }
    # 第二个必选规则是处理静态文件请求，这是nginx作为http服务器的强项
    # 有两种配置模式，目录匹配或后缀匹配,任选其一或搭配使用
    location ^~ /static {
        root /webroot/static/;
    }
    location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {
        root /webroot/res/;
    }
    #第三个规则就是通用规则，用来转发动态请求到后端应用服务器
    #非静态文件请求就默认是动态请求，自己根据实际把握
    #毕竟目前的一些框架的流行，带.php,.jsp后缀的情况很少了
    location / {
        .....
    }

## Nginx负载均衡
    当一台服务器单位时间内访问量很大的时候，服务器压力就会很大，当达到这台服务器的极限，就会崩溃；怎么解决？
    可以通过nginx的反向代理设置，添加几台同样功能的服务器 分担压力。
    
    nginx实现负载均衡原理，用户访问首先访问到nginx服务器，然后nginx服务器再从应用服务器集群中选择压力比较小的服务器，然后将该访问请求引向该服务器。
    如应用服务器集群中某一台服务器崩溃，那么从待选择服务器列表中将该服务器删除，也就是说一个服务器崩溃了，那么nginx服务器不会把请求引向到该服务器。
    
    upstream mypro {
        server 192.168.5.140:8080;
        server 192.168.5.141:8080;
        xxxxx
        xxxx
    }
    
    server {
        listen 80;
        server_name xxxx;
        location / {
            proxy_pass http://mypro;
        } 
     }
    
### 负载均衡方案
    
#### 随机轮询
    upstream mypro {
        server 192.168.5.140:8080;
        server 192.168.5.141:8080;
    }
    
#### 权重
    upstream mypro {
        server 192.168.5.140:8080 weight=5;
        server 192.168.5.141:8080 weight=10;
    }
    
#### ip_hash
    upstream mypro {
        ip_hash;
        server 192.168.5.140:8080;
        server 192.168.5.141:8080;
    }
    
#### 使用方式
    server {
        listen       80;
        server_name  192.168.5.138;
         location / {
            proxy_pass http://mypro;
        }
    }
    
## Nginx拓展模块安装
    查看nginx安装的现有模块指令
    /usr/local/nginx/sbin/nginx -V （大写的V）
    nginx version: nginx/1.13.2
    built by gcc 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC) 
    configure arguments:
    
    1、下载需要的echo模块
    https://github.com/openresty/echo-nginx-module/tags
    # wget https://github.com/openresty/echo-nginx-module/archive/v0.61.tar.gz
    # tar zxvf v0.61.tar.gz
    # mv echo-nginx-module-0.61 nginx-tools/
    
    2、重新编译nginx，安装echo-nginx模块
    进入nginx源文件，重新编译
    # ./configure --add-module=nginx安装目录下面/echo-nginx-module-0.61 #安装echo模块(文件夹名echo-nginx-module-0.61)
    # make #开始编译，但别安装 （make install会直接覆盖安装）
    
    3、平滑升级 nginx
    
    注意先备份一下之前老的，手动安装一下。
    # mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.old
    # cp -f objs/nginx /usr/local/nginx/sbin/nginx
    
    这里是平滑升级，如是全新安装请执行：make install
    # make upgrade 
    # make clean （清除编译产生的文件，可以忽略）
    
    
    
    location /module {
        echo 'hello world'; 
    }
    --------------------
    location /module {
                echo_exec /set;
    }
            
    location /set {
        set $foo 'hello world';     #自定义变量
        echo "$request_uri";      #显示nginx全局变量的内容
        echo $foo;
    }
    
    
    lua模块：可以在nginx服务中执行lua脚本
    
    
    nginx全局变量
    $args ：                     #这个变量等于请求行中的参数，同$query_string
    $content_length ：    #请求头中的Content-length字段。
    $content_type ：       #请求头中的Content-Type字段。
    $document_root ：   #当前请求在root指令中指定的值。
    $host ：                     #请求主机头字段，否则为服务器名称。
    $http_user_agent ：  #客户端agent信息
    $http_cookie ：          #客户端cookie信息
    $limit_rate ：              #这个变量可以限制连接速率。
    $request_method ：   #客户端请求的动作，通常为GET或POST。
    $remote_addr ：         #客户端的IP地址。
    $remote_port ：          #客户端的端口。
    $remote_user ：         #已经经过Auth Basic Module验证的用户名。
    $request_filename ： #当前请求的文件路径，由root或alias指令与URI请求生成。
    $scheme ：                #HTTP方法（如http，https）。
    $server_protocol ：    #请求使用的协议，通常是HTTP/1.0或HTTP/1.1。
    $server_addr ：         #服务器地址，在完成一次系统调用后可以确定这个值。
    $server_name ：       #服务器名称。
    $server_port ：          #请求到达服务器的端口号。
    $request_uri ：          #包含请求参数的原始URI，不包含主机名，如：”/foo/bar.php?arg=baz”。
    $uri ：                        #不带请求参数的当前URI，$uri不包含主机名，如”/foo/bar.html”。
    $document_uri ：      #与$uri相同

## Nginx的Lua模块

    Nginx的优势
        Nginx设计为一个主进程多个工作进程的工作模式，每个进程是单线程来处理多个连接，而且每个工作进程采用了非阻塞I/O来处理多个连接，
        从而减少了线程上下文切换，从而实现了公认的高性能、高并发;因此在生成环境中会通过把CPU绑定给Nginx工作进程从而提升其性能;另外因为单线程工作模式的特点，内存占用就非常少了。 
        Nginx更改配置重启速度非常快，可以毫秒级，而且支持不停止Nginx进行升级Nginx版本、动态重载Nginx配置。  
        Nginx模块也是非常多，功能也很强劲，不仅可以作为http负载均衡，Nginx发布1.9.0版本还支持TCP负载均衡，还可以很容易的实现内容缓存、web服务器、反向代理、访问控制等功能。
        nginx模块：rewrite 经常用到的
    
    什么是ngx_lua
        ngx_lua是Nginx的一个模块，将Lua嵌入到Nginx中，从而可以使用Lua来编写脚本，这样就可以使用Lua编写应用脚本，部署到Nginx中运行，即Nginx变成了一个Web容器;
        这样开发人员就可以使用Lua语言开发高性能Web应用了。
        
        网站开发，也有一个重要的脚步语言，javascript，js文件；；；客户端脚步语言；；；（node.js服务器语言）
        网站页面进行渲染的时候，可以通过javascript脚本语言 进行一些业务处理
        <script>
        function hello(){
        	alert("hello");
        }
        </script>
        脚本文件，还需要一个javascript引擎---解析javascript脚本语言
        全部的浏览器 都包含 javascript引擎
        
        
        lua -- javascript  一样的脚本语言
        lua脚本语言 应用  ----》游戏开发中
        
        
        Lua是一种轻量级、可嵌入式的脚本语言，这样可以非常容易的嵌入到其他语言中使用。另外Lua提供了协程并发，
        即以同步调用的方式进行异步执行，从而实现并发，比起回调机制的并发来说代码更容易编写和理解，排查问题也会容易。Lua还提供了闭包机制，
        函数可以作为First Class Value 进行参数传递，另外其实现了标记清除垃圾收集。
        
        因为Lua的小巧轻量级，可以在Nginx中嵌入Lua VM，请求的时候创建一个VM，请求结束的时候回收VM。
        
        ngx_lua模块的原理：
        ngx_lua将Lua嵌入Nginx，能够让Nginx运行Lua脚本，而且高并发、非堵塞的处理各种请求。Lua内建协程。这样就能够非常好的将异步回调转换成顺序调用的形式。ngx_lua在Lua中进行的IO操作都会托付给Nginx的事件模型。从而实现非堵塞调用。
        开发人员能够採用串行的方式编敲代码，ngx_lua会自己主动的在进行堵塞的IO操作时中断。保存上下文；然后将IO操作托付给Nginx事件处理机制。在IO操作完毕后，ngx_lua会恢复上下文，程序继续运行，这些操作都是对用户程序透明的。
        每一个NginxWorker进程持有一个Lua解释器或者LuaJIT实例，被这个Worker处理的全部请求共享这个实例。
        
        每一个请求的Context会被Lua轻量级的协程切割，从而保证各个请求是独立的。 ngx_lua採用“one-coroutine-per-request”的处理模型。对于每一个用户请求，ngx_lua会唤醒一个协程用于执行用户代码处理请求，当请求处理完毕这个协程会被销毁。
        
        每一个协程都有一个独立的全局环境（变量空间），继承于全局共享的、仅仅读的“comman data”。所以。被用户代码注入全局空间的不论什么变量都不会影响其它请求的处理。而且这些变量在请求处理完毕后会被释放，
        这样就保证全部的用户代码都执行在一个“sandbox”（沙箱），这个沙箱与请求具有同样的生命周期。 得益于Lua协程的支持。ngx_lua在处理10000个并发请求时仅仅须要非常少的内存。依据測试，ngx_lua处理每一个请求仅仅须要2KB的内存，
        假设使用LuaJIT则会更少。所以ngx_lua非常适合用于实现可扩展的、高并发的服务。
        
        ngx_lua 模块提供的指令和API
        
        三、ngx_lua安装
        echo模块
            ngx_lua安装能够通过下载模块源代码，编译Nginx。可是推荐採用openresty。Openresty就是一个打包程序，包括大量的第三方Nginx模块，比方HttpLuaModule，HttpRedis2Module，HttpEchoModule等。省去下载模块。而且安装很方便。        
            OpenResty将Nginx核心、LuaJIT、许多有用的Lua库和Nginx第三方模块打包在一起;这样开发人员只需要安装OpenResty，不需要了解Nginx核心和写复杂的C/C++模块就可以，只需要使用Lua语言进行Web应用开发了。        
        
        OpenResty提供了一些常用的ngx_lua开发模块：如        
            lua-resty-memcached       
            lua-resty-mysql        
            lua-resty-redis        
            lua-resty-dns        
            lua-resty-limit-traffic        
            lua-resty-template       
        nginx + lua 就可以开发出 一些系统。龙果学院中 有一门课程 就专门应用了这个技术  
        这些模块涉及到如mysql数据库、redis、限流、模块渲染等常用功能组件;另外也有很多第三方的ngx_lua组件供我们使用，对于大部分应用场景来说现在生态环境中的组件已经足够多了;如果不满足需求也可以自己去写来完成自己的需求。
        openresty.org/cn官网
        
        应用场景
        应用的公司：奇虎360、京东、百度、魅族、知乎、优酷、新浪这些互联网公司都在使用。
        业务场景： WAF、有做 CDN 调度、广告系统、消息推送系统，API server 网关
    
