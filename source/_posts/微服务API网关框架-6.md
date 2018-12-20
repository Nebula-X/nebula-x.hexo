---
title: 微服务API网关框架(6)--OpenResty介绍与简单使用
catalog: true
date: 2018-12-20 11:40:48
subtitle:
header-img: "Demo.png"
tags:
- 微服务API网关
catagories:
- 微服务API网关
---

# OpenResty

## 介绍
    OpenResty
        是一个基于 Nginx 与 Lua 的高性能 Web 平台，其内部集成了大量精良的 Lua 库、第三方模块以及大多数的依赖项。
        用于方便地搭建能够处理超高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关。
        通过汇聚各种设计精良的 Nginx 模块（主要由 OpenResty 团队自主开发），从而将 Nginx 有效地变成一个强大的通用 Web 应用平台。
        这样，Web 开发人员和系统工程师可以使用 Lua 脚本语言调动 Nginx 支持的各种 C 以及 Lua 模块，快速构造出足以胜任 10K 乃至 1000K 以上单机并发连接的高性能 Web 应用系统。
        OpenResty的目标是让你的Web服务直接跑在 Nginx 服务内部，充分利用 Nginx 的非阻塞 I/O 模型，不仅仅对 HTTP 客户端请求,
        甚至于对远程后端诸如 MySQL、PostgreSQL、Memcached 以及 Redis 等都进行一致的高性能响应。

## 安装
    1）下载安装
    centos系统
    # yum install readline-devel pcre pcre-devel openssl openssl-devel gcc curl GeoIP-devel
    下载源码包
    https://github.com/openresty/openresty/releases
    选择最新版本v1.13.6.1
    解压安装
    # tar -xzvf openresty-1.13.6.1.tar.gz
    # cd openresty-1.13.6.1/
    
    ##选择模块 ./configure --help
    
    # ./configure --with-luajit --with-pcre --with-http_gzip_static_module --with-http_realip_module --with-http_geoip_module --with-http_ssl_module  --with-http_stub_status_module 
    
    --with-http_gzip_static_module #静态文件压缩
    --with-http_stub_status_module #监控nginx状态
    --with-http_realip_module #通过这个模块允许我们改变客户端请求头中客户端IP地址值(例如X-Real-IP 或 X-Forwarded-For)，意义在于能够使得后台服务器记录原始客户端的IP地址
    --with-pcre #设置PCRE库（pcre pcre-devel）
    --with-http_ssl_module #使用https协议模块。（openssl openssl-devel）
    --with-http_geoip_module #增加了根据ip获得城市信息，经纬度等模块 （GeoIP-devel）
    
    # make && make install
    
    2）安装成功后，默认会在/usr/local/openresty/
    目录下
    luajit 是采用C语言写的Lua代码的解释器 ----just in time   即时解析
    lualib 是编辑好的lua类库
    nginx，其实我们openResty就是nginx，只是做了一些模块化工作；所以启动openResty就是启动nginx，我们可以到 cd nginx/sbin/，直接运行  ./nginx
    
    3）设置环境变量
    # vi /etc/profile
    export NGINX_HOME=/usr/local/openresty/nginx
    export PATH=$PATH:$NGINX_HOME/sbin
    # source /etc/profile ##生效

## 简单例子 -- Hello World
    1）ngx_lua模块的hello world
    编辑nginx下conf配置文件nginx.conf
    # vi nginx.conf
    在server模块加上
    location /helloworld {
    	default_type text/html;
        content_by_lua 'ngx.say("hello world")';
    }
    
    检查配置文件是否正确
    # /usr/local/openresty/nginx/sbin/nginx -t -c /usr/local/openresty/nginx/conf/nginx.conf
    
    重启nginx
    # ./nginx -s reload
    访问http://192.168.31.138/helloworld  输出 hello world
    
    2）nginx的内部变量
    
    名称 说明
    $arg_name 请求中的name参数
    $args 请求中的参数
    $binary_remote_addr 远程地址的二进制表示
    $body_bytes_sent 已发送的消息体字节数
    $content_length HTTP请求信息里的"Content-Length"
    $content_type 请求信息里的"Content-Type"
    $document_root 针对当前请求的根路径设置值
    $document_uri 与$uri相同; 比如 /test2/test.php
    $host 请求信息中的"Host"，如果请求中没有Host行，则等于设置的服务器名
    $hostname 机器名使用 gethostname系统调用的值
    $http_cookie cookie 信息
    $http_referer 引用地址
    $http_user_agent 客户端代理信息
    $http_via 最后一个访问服务器的Ip地址。
    $http_x_forwarded_for 相当于网络访问路径
    $is_args 如果请求行带有参数，返回“?”，否则返回空字符串
    $limit_rate 对连接速率的限制
    $nginx_version 当前运行的nginx版本号
    $pid worker进程的PID
    $query_string 与$args相同
    $realpath_root 按root指令或alias指令算出的当前请求的绝对路径。其中的符号链接都会解析成真是文件路径
    $remote_addr 客户端IP地址
    $remote_port 客户端端口号
    $remote_user 客户端用户名，认证用
    $request 用户请求
    $request_body 这个变量（0.7.58+）包含请求的主要信息。在使用proxy_pass或fastcgi_pass指令的location中比较有意义
    $request_body_file 客户端请求主体信息的临时文件名
    $request_completion 如果请求成功，设为"OK"；如果请求未完成或者不是一系列请求中最后一部分则设为空
    $request_filename 当前请求的文件路径名，比如/opt/nginx/www/test.php
    $request_method 请求的方法，比如"GET"、"POST"等
    $request_uri 请求的URI，带参数; 比如http://localhost:88/test1/
    $scheme 所用的协议，比如http或者是https
    $server_addr 服务器地址，如果没有用listen指明服务器地址，使用这个变量将发起一次系统调用以取得地址(造成资源浪费)
    $server_name 请求到达的服务器名
    $server_port 请求到达的服务器端口号
    $server_protocol 请求的协议版本，"HTTP/1.0"或"HTTP/1.1"
    $uri 请求的URI，可能和最初的值有不同，比如经过重定向之类的
    
    
    写个配置文件，测试一下$uri变量
    location /test_url {
    	echo "url:$uri";
    }
    
    location /test_url {
    			echo "url:$uri";
    			echo "full url : $host$request_uri";
    		}
    		
    重启nginx，访问
    
    再此基础上测试$args变量
    location /test_url {
    	echo "url:$uri----args:$args";
    }
    
    再此基础上测试$arg_name变量
    location /test_url {
    	echo "url:$uri----args:$args--------arg_name:$arg_name";
    }
    
    说明一下，$arg_name表示取名为name的参数，如果想取其他名称的参数可以对应的取该参数名
    location /test_url {
    	echo "url:$uri --- args:$args --- arg_name:$arg_name <br/>";
    	echo "arg_user:$arg_user --- arg_age:$arg_age<br/>";
    	echo "arg_test:$arg_test";
    }
    
    test参数名不存在，则为空。
    自定义变量
    location /test_def {
    	set $name "rainbow";
    	echo $name;
    }
    
    set 设置的变量 为局部变量，其他请求无法获取此$name的值
    location /test_def {
    	set $name "rainbow";
    	echo_exec /test_def2;  ##内部跳转，可以传递变量
    }
    location /test_def2 {
    	echo $name;
    }

## Openresty中使用Lua
    openresty 引入 lua
    
    一）openresty中nginx引入lua方式
    
      1）xxx_by_lua   --->字符串编写方式
      2) xxx_by_lua_block ---->代码块方式
      3) xxx_by_lua_file  ---->直接引用一个lua脚本文件
    
    我们案例中使用内容处理阶段，用content_by_lua演示
    
    -----------------编辑nginx.conf-----------------------
    
    第一种：content_by_lua
    
    location /testlua {
      content_by_lua "ngx.say('hello world')";
    }
    
    输出了hello world
    
    content_by_lua 方式，参数为字符串，编写不是太方便。
    
    ----------------------------------------
    
    第二种：content_by_lua_block
    location /testlua {
      content_by_lua_block {
           ngx.say("hello world");
      } 
    }
    
    content_by_lua_block {}  表示内部为lua块，里面可以应用lua语句
    
    ----------------------------------------
    
    第三种：content_by_lua_file
    
    location /testlua {
      content_by_lua_file /usr/local/lua/test.lua;
    }
    
    content_by_lua_file 就是引用外部lua文件
    
    # vi  test.lua
    ngx.say("hello world");
    
    
    二）openresty使用lua打印输出案例
    
      location /testsay {
        content_by_lua_block {
            --写响应头  
            ngx.header.a = "1"  
            ngx.header.b = "2" 
            --输出响应  
            ngx.say("a", "b", "<br/>")  
            ngx.print("c", "d", "<br/>")  
            --200状态码退出  
            return ngx.exit(200) 
        } 
      }
    
      ngx.header：输出响应头；
      ngx.print：输出响应内容体；
      ngx.say：通ngx.print，但是会最后输出一个换行符；
      ngx.exit：指定状态码退出。
    
    三）介绍一下openresty使用lua常用的api
    
    1）ngx.var ： 获取Nginx变量 和 内置变量
    
    nginx内置的变量
    
    $arg_name 请求中的name参数
    $args 请求中的参数
    $binary_remote_addr 远程地址的二进制表示
    $body_bytes_sent  已发送的消息体字节数
    $content_length HTTP请求信息里的"Content-Length"
    $content_type 请求信息里的"Content-Type"
    $document_root  针对当前请求的根路径设置值
    $document_uri 与$uri相同; 比如 /test2/test.php
    $host 请求信息中的"Host"，如果请求中没有Host行，则等于设置的服务器名
    $hostname 机器名使用 gethostname系统调用的值
    $http_cookie  cookie 信息
    $http_referer 引用地址
    $http_user_agent  客户端代理信息
    $http_via 最后一个访问服务器的Ip地址。
    $http_x_forwarded_for 相当于网络访问路径
    $is_args  如果请求行带有参数，返回“?”，否则返回空字符串
    $limit_rate 对连接速率的限制
    $nginx_version  当前运行的nginx版本号
    $pid  worker进程的PID
    $query_string 与$args相同
    $realpath_root  按root指令或alias指令算出的当前请求的绝对路径。其中的符号链接都会解析成真是文件路径
    $remote_addr  客户端IP地址
    $remote_port  客户端端口号
    $remote_user  客户端用户名，认证用
    $request  用户请求
    $request_body 这个变量（0.7.58+）包含请求的主要信息。在使用proxy_pass或fastcgi_pass指令的location中比较有意义
    $request_body_file  客户端请求主体信息的临时文件名
    $request_completion 如果请求成功，设为"OK"；如果请求未完成或者不是一系列请求中最后一部分则设为空
    $request_filename 当前请求的文件路径名，比如/opt/nginx/www/test.php
    $request_method 请求的方法，比如"GET"、"POST"等
    $request_uri  请求的URI，带参数; 比如http://localhost:88/test1/
    $scheme 所用的协议，比如http或者是https
    $server_addr  服务器地址，如果没有用listen指明服务器地址，使用这个变量将发起一次系统调用以取得地址(造成资源浪费)
    $server_name  请求到达的服务器名
    $server_port  请求到达的服务器端口号
    $server_protocol  请求的协议版本，"HTTP/1.0"或"HTTP/1.1"
    $uri  请求的URI，可能和最初的值有不同，比如经过重定向之类的
    
    ngx.var.xxx
    
    location /var {
        set $c 3;
    
        #处理业务
        content_by_lua_block {
          local a = tonumber(ngx.var.arg_a) or 0
          local b = tonumber(ngx.var.arg_b) or 0
          local c = tonumber(ngx.var.c) or 0
          ngx.say("sum:", a + b + c )
        }
    }
    
    注意：ngx.var.c 此变量必须提前声明；
    另外对于nginx location中使用正则捕获的捕获组可以使用ngx.var[捕获组数字]获取；
    
    location ~ ^/var/([0-9]+) {
       content_by_lua_block {
        ngx.say("var[1]:", ngx.var[1] )
      }
    }
    
    2）ngx.req请求模块的常用api
    
       ngx.req.get_headers：获取请求头，
       获取带中划线的请求头时请使用如headers.user_agent这种方式；如果一个请求头有多个值，则返回的是table；
    
    -----------test.lua-------------------
    
    local headers = ngx.req.get_headers()  
    ngx.say("============headers begin===============", "<br/>")  
    ngx.say("Host : ", headers["Host"], "<br/>")  
    ngx.say("headers['user-agent'] : ", headers["user-agent"], "<br/>")  
    ngx.say("headers.user_agent : ", headers.user_agent, "<br/>") 
    ngx.say("-------------遍历headers-----------", "<br/>") 
    for k,v in pairs(headers) do  
        if type(v) == "table" then  
            ngx.say(k, " : ", table.concat(v, ","), "<br/>")  
        else  
            ngx.say(k, " : ", v, "<br/>")  
        end  
    end  
    ngx.say("===========headers end============", "<br/>")  
    ngx.say("<br/>")  
    
    
    
    3）获取请求参数
      ngx.req.get_uri_args：获取url请求参数，其用法和get_headers类似；
      ngx.req.get_post_args：获取post请求内容体，其用法和get_headers类似，
                             但是必须提前调用ngx.req.read_body()来读取body体
                             （也可以选择在nginx配置文件使用lua_need_request_body on;开启读取body体，
                               但是官方不推荐）；
    
      ngx.req.get_body_data：为解析的请求body体内容字符串。
    
    ---------------test.lua---------------
    
    --get请求uri参数  
    ngx.say("===========uri get args begin==================", "<br/>")  
    local uri_args = ngx.req.get_uri_args()  
    for k, v in pairs(uri_args) do  
        if type(v) == "table" then  
            ngx.say(k, " : ", table.concat(v, ", "), "<br/>")  
        else  
            ngx.say(k, ": ", v, "<br/>")  
        end  
    end  
    ngx.say("===========uri get args end==================", "<br/>") 
      
    --post请求参数  
    ngx.req.read_body()  
    ngx.say("=================post args begin====================", "<br/>")  
    local post_args = ngx.req.get_post_args()  
    for k, v in pairs(post_args) do  
        if type(v) == "table" then  
            ngx.say(k, " : ", table.concat(v, ", "), "<br/>")  
        else  
            ngx.say(k, ": ", v, "<br/>")  
        end  
    end  
    ngx.say("================post args end=====================", "<br/>")  
      
    
    4) ngx.req其他常用的api
    --请求的http协议版本  
    ngx.say("ngx.req.http_version : ", ngx.req.http_version(), "<br/>")  
    --请求方法  
    ngx.say("ngx.req.get_method : ", ngx.req.get_method(), "<br/>")  
    --原始的请求头内容  
    ngx.say("ngx.req.raw_header : ",  ngx.req.raw_header(), "<br/>")  
    --请求的body内容体  
    ngx.say("ngx.req.get_body_data() : ", ngx.req.get_body_data(), "<br/>")  
    ngx.say("<br/>")  
    
    ngx.req.raw_header()这个函数返回值为字符串
    
    5）编码解码
    
    ngx.escape_uri/ngx.unescape_uri ： uri编码解码；
    
    ngx.encode_args/ngx.decode_args：参数编码解码；
    
    ngx.encode_base64/ngx.decode_base64：BASE64编码解码；
    
    -------test.lua
    
    --未经解码的请求uri  
    local request_uri = ngx.var.request_uri;  
    ngx.say("request_uri : ", request_uri, "<br/>"); 
    
    --编码
    local escape_uri = ngx.escape_uri(request_uri)
    ngx.say("escape_uri : ", escape_uri, "<br/>"); 
    
    --解码  
    ngx.say("decode request_uri : ", ngx.unescape_uri(escape_uri), "<br/>");
    
    --参数编码
    local request_uri = ngx.var.request_uri;
    local question_pos, _ = string.find(request_uri, '?')
    if question_pos>0 then
      local uri = string.sub(request_uri, 1, question_pos-1)
      ngx.say("uri sub=",string.sub(request_uri, question_pos+1),"<br/>");
      
      --对字符串进行解码
      local args = ngx.decode_args(string.sub(request_uri, question_pos+1))
      
      for k,v in pairs(args) do
        ngx.say("k=",k,",v=", v, "<br/>");
      end
      
      if args and args.userId then
        args.userId = args.userId + 10000
        ngx.say("args+10000 : ", uri .. '?' .. ngx.encode_args(args), "<br/>");
      end
    end
    
    6）md5加密api
    --MD5  
    ngx.say("ngx.md5 : ", ngx.md5("123"), "<br/>")  
    
    7）nginx获取时间
    
    之前介绍的os.time()会涉及系统调用，性能比较差，推荐使用nginx中的时间api
    
    ngx.time()  --返回秒级精度的时间戳
    ngx.now()   --返回毫秒级精度的时间戳
    
    就是通过这两种方式获取到的只是nginx缓存起来的时间戳，不是实时的。
    所以有时候会出现一些比较奇怪的现象，比如下面代码：
    
    local t1 = ngx.now()
    for i=1,1000000 do
    end
    local t2 = ngx.now()
    print(t1, ",", t2) -- t1和t2的值是一样的，why？
    ngx.exit(200)
    
    正常来说，t2应该大于t1才对，但由于nginx没有及时更新（缓存的）时间戳，所以导致t2和t1获取到的时间戳是一样的。
    那么怎样才能强迫nginx更新缓存呢？调用多一个ngx.update_time()函数即可：
    
    local t1 = ngx.now()
    for i=1,1000000 do
    end
    ngx.update_time()
    local t2 = ngx.now()
    print(t1, ",", t2) 
    ngx.exit(200)
    
    8）ngx.re模块中正则表达式相关的api
    
    ngx.re.match
    ngx.re.sub
    ngx.re.gsub
    ngx.re.find
    ngx.re.gmatch
    
    我们这里只简单的介绍 ngx.re.match，详细用法可以自行去网上学习
    
    ngx.re.match
    只有第一次匹配的结果被返回，如果没有匹配，则返回nil；或者匹配过程中出现错误时，
    也会返回nil，此时错误信息会被保存在err中。
    
    当匹配的字符串找到时，一个Lua table captures会被返回，
    captures[0]中保存的就是匹配到的字串，
    captures[1]保存的是用括号括起来的第一个子模式（捕获分组）的结果，
    captures[2]保存的是第二个子模式（捕获分组）的结果，依次类似。
    
    ---------------------
    
    local m, err = ngx.re.match("hello, 1234", "[0-9]+")
    if m then
      ngx.say(m[0])
    else
      if err then
        ngx.log(ngx.ERR, "error: ", err)
        return
      end
    
      ngx.say("match not found")
    end
    
    上面例子中，匹配的字符串是1234，因此m[0] == "1234"，
    --------------
    
    local m, err = ngx.re.match("hello, 1234", "([0-9])[0-9]+")
    ngx.say(m[0],"<br/>")
    ngx.say(m[1])
    
    
    ---------------------------------------------------------
    
    
    备注：有没有注意到，我们每次修改都要重启nginx，这样太过于麻烦，我们可以用
    content_by_lua_file 引入外部lua，这样的话 只要修改外部的lua，就可以了，不需要重启nginx了。
    注意需要把lua_code_cache 设置为off
    
    语法：lua_code_cache on | off
    默认： on
    适用上下文：http、server、location、location if
    这个指令是指定是否开启lua的代码编译缓存，开发时可以设置为off，以便lua文件实时生效，
    如果是生产线上，为了性能，建议开启。
    最终nginx.conf修改为
    
    以后我们只要修改test.lua 文件就可以了。
    
    9）标准日志输出
    
    ngx.log(log_level, ...) 
    
    日志输出级别
    
    ngx.STDERR     -- 标准输出
    ngx.EMERG      -- 紧急报错
    ngx.ALERT      -- 报警
    ngx.CRIT       -- 严重，系统故障，触发运维告警系统
    ngx.ERR        -- 错误，业务不可恢复性错误
    ngx.WARN       -- 告警，业务中可忽略错误
    ngx.NOTICE     -- 提醒，业务比较重要信息
    ngx.INFO       -- 信息，业务琐碎日志信息，包含不同情况判断等
    ngx.DEBUG      -- 调试
    
    -------------------------------------
    
    #user  nobody;
    worker_processes  1;
    
    error_log  logs/error.log error;    # 日志级别
    #pid        logs/nginx.pid;
    
    events {
        worker_connections  1024;
    }
    
    http {
        server {
            listen    80;
            location / {
                content_by_lua_block {
                    local num = 55
                    local str = "string"
                    local obj
                    ngx.log(ngx.ERR, "num:", num)
                    ngx.log(ngx.INFO, " string:", str)
                    print([[i am print]])
                    ngx.log(ngx.ERR, " object:", obj)
                }
            }
        }
    }
    
    日志输出级别使用的 error，只有等于或大于这个级别的日志才会输出
    
    ngx.DEBUG
    ngx.WARN
    
    对于应用开发，一般使用 ngx.INFO 到 ngx.CRIT 就够了。生产中错误日志开启到 error 级别就够了
    
    10）重定向 ngx.redirect
    
    -----重定向
    
    location = /bar {
      content_by_lua_block {
        ngx.say([[I am bar]])
      }
    }
    
    location = /foo {
      rewrite_by_lua_block {
        return ngx.redirect('/bar');
      }
    }
    
    
    11）不同阶段共享变量
    
    ngx.ctx 全局共享变量
    
    在 OpenResty 的体系中，可以通过共享内存的方式完成不同工作进程的数据共享，
    本地内存方式 去让不同的工作进程共享数据
    
    openresty有不同处理阶段，后面的课程会介绍。在不同的处理阶段，如何共享数据
    
    可以通过 Lua 模块方式完成单个进程内不同请求的数据共享。如何完成单个请求内不同阶段的数据共享呢？
    
    ngx.ctx 表就是为了解决这类问题而设计的。参考下面例子：
    
    location /test {
         rewrite_by_lua_block {
             ngx.ctx.foo = 76
         }
         access_by_lua_block {
             ngx.ctx.foo = ngx.ctx.foo + 3
         }
         content_by_lua_block {
             ngx.say(ngx.ctx.foo)
         }
     }
    
     ngx.ctx.xxxxx
    
    首先 ngx.ctx 是一个表，所以我们可以对他添加、修改。它用来存储基于请求的 Lua 环境数据，
    其生存周期与当前请求相同 (类似 Nginx 变量)。它有一个最重要的特性：
    单个请求内的 rewrite (重写)，access (访问)，和 content (内容) 等各处理阶段是保持一致的。
    
    额外注意，每个请求，包括子请求，都有一份自己的 ngx.ctx 表。例如：
    
     location /sub {
         content_by_lua_block {
             ngx.say("sub pre: ", ngx.ctx.blah)
             ngx.ctx.blah = 32
             ngx.say("sub post: ", ngx.ctx.blah)
         }
     }
    
     location /main {
         content_by_lua_block {
             ngx.ctx.blah = 73
             ngx.say("main pre: ", ngx.ctx.blah)
             local res = ngx.location.capture("/sub")
             ngx.print(res.body)
             ngx.say("main post: ", ngx.ctx.blah)
         }
     }
    
    ngx.ctx 表查询需要相对昂贵的元方法调用，这比通过用户自己的函数参数直接传递基于请求的数据要慢得多。
    所以不要为了节约用户函数参数而滥用此 API，因为它可能对性能有明显影响。
    
    由于 ngx.ctx 保存的是指定请求资源，所以这个变量是不能直接共享给其他请求使用的。
    
    
    更多api使用  https://www.nginx.com/resources/wiki/modules/lua/#nginx-api-for-lua
    
    操作指令  说明
    ngx.arg 指令参数，如跟在content_by_lua_file后面的参数
    ngx.var 变量，ngx.var.VARIABLE引用某个变量
    ngx.ctx 请求的lua上下文
    ngx.header  响应头，ngx.header.HEADER引用某个头
    ngx.status  响应码
    
    API 说明
    ngx.log 输出到error.log
    print 等价于 ngx.log(ngx.NOTICE, ...)
    ngx.send_headers  发送响应头
    ngx.headers_sent  响应头是否已发送
    ngx.resp.get_headers  获取响应头
    ngx.timer.at  注册定时器事件
    ngx.is_subrequest 当前请求是否是子请求
    ngx.location.capture  发布一个子请求
    ngx.location.capture_multi  发布多个子请求
    ngx.exec   
    ngx.redirect   
    ngx.print 输出响应
    ngx.say 输出响应，自动添加'n'
    ngx.flush 刷新响应
    ngx.exit  结束请求
    ngx.eof  
    ngx.sleep 无阻塞的休眠（使用定时器实现）
    ngx.get_phase  
    ngx.on_abort  注册client断开请求时的回调函数
    ndk.set_var.DIRECTIVE  
    ngx.req.start_time  请求的开始时间
    ngx.req.http_version  请求的HTTP版本号
    ngx.req.raw_header  请求头（包括请求行）
    ngx.req.get_method  请求方法
    ngx.req.set_method  请求方法重载
    ngx.req.set_uri 请求URL重写
    ngx.req.set_uri_args   
    ngx.req.get_uri_args  获取请求参数
    ngx.req.get_post_args 获取请求表单
    ngx.req.get_headers 获取请求头
    ngx.req.set_header   
    ngx.req.clear_header   
    ngx.req.read_body 读取请求体
    ngx.req.discard_body  扔掉请求体
    ngx.req.get_body_data  
    ngx.req.get_body_file  
    ngx.req.set_body_data  
    ngx.req.set_body_file  
    ngx.req.init_body  
    ngx.req.append_body  
    ngx.req.finish_body  
    ngx.req.socket   
    ngx.escape_uri  字符串的url编码
    ngx.unescape_uri  字符串url解码
    ngx.encode_args 将table编码为一个参数字符串
    ngx.decode_args 将参数字符串编码为一个table
    ngx.encode_base64 字符串的base64编码
    ngx.decode_base64 字符串的base64解码
    ngx.crc32_short 字符串的crs32_short哈希
    ngx.crc32_long  字符串的crs32_long哈希
    ngx.hmac_sha1 字符串的hmac_sha1哈希
    ngx.md5 返回16进制MD5
    ngx.md5_bin 返回2进制MD5
    ngx.sha1_bin  返回2进制sha1哈希值
    ngx.quote_sql_str SQL语句转义
    ngx.today 返回当前日期
    ngx.time  返回UNIX时间戳
    ngx.now 返回当前时间
    ngx.update_time 刷新时间后再返回
    ngx.localtime  
    ngx.utctime  
    ngx.cookie_time 返回的时间可用于cookie值
    ngx.http_time 返回的时间可用于HTTP头
    ngx.parse_http_time 解析HTTP头的时间
    ngx.re.match   
    ngx.re.find  
    ngx.re.gmatch  
    ngx.re.sub   
    ngx.re.gsub  
    ngx.shared.DICT  
    ngx.shared.DICT.get  
    ngx.shared.DICT.get_stale  
    ngx.shared.DICT.set  
    ngx.shared.DICT.safe_set   
    ngx.shared.DICT.add  
    ngx.shared.DICT.safe_add   
    ngx.shared.DICT.replace  
    ngx.shared.DICT.delete   
    ngx.shared.DICT.incr   
    ngx.shared.DICT.flush_all  
    ngx.shared.DICT.flush_expired  
    ngx.shared.DICT.get_keys   
    ngx.socket.udp   
    udpsock:setpeername  
    udpsock:send   
    udpsock:receive  
    udpsock:close  
    udpsock:settimeout   
    ngx.socket.tcp   
    tcpsock:connect  
    tcpsock:sslhandshake   
    tcpsock:send   
    tcpsock:receive  
    tcpsock:receiveuntil   
    tcpsock:close  
    tcpsock:settimeout   
    tcpsock:setoption  
    tcpsock:setkeepalive   
    tcpsock:getreusedtimes   
    ngx.socket.connect   
    ngx.thread.spawn   
    ngx.thread.wait  
    ngx.thread.kill  
    coroutine.create   
    coroutine.resume   
    coroutine.yield  
    coroutine.wrap   
    coroutine.running  
    coroutine.status   
    ngx.config.debug  编译时是否有 --with-debug选项
    ngx.config.prefix 编译时的 --prefix选项
    ngx.config.nginx_version  返回nginx版本号
    ngx.config.nginx_configure  返回编译时 ./configure的命令行选项
    ngx.config.ngx_lua_version  返回ngx_lua模块版本号
    ngx.worker.exiting  当前worker进程是否正在关闭（如reload、shutdown期间）
    ngx.worker.pid  返回当前worker进程的pid
       
    常量说明
    ngx.OK (0)
    ngx.ERROR (-1)
    ngx.AGAIN (-2)
    ngx.DONE (-4)
    ngx.DECLINED (-5)
    ngx.nil
    
    
    HTTP 请求方式
    ngx.HTTP_GET
    ngx.HTTP_HEAD
    ngx.HTTP_PUT
    ngx.HTTP_POST
    ngx.HTTP_DELETE
    ngx.HTTP_OPTIONS  
    ngx.HTTP_MKCOL    
    ngx.HTTP_COPY      
    ngx.HTTP_MOVE     
    ngx.HTTP_PROPFIND 
    ngx.HTTP_PROPPATCH 
    ngx.HTTP_LOCK 
    ngx.HTTP_UNLOCK    
    ngx.HTTP_PATCH   
    ngx.HTTP_TRACE  
    
    
    HTTP 返回状态
    ngx.HTTP_OK (200)
    ngx.HTTP_CREATED (201)
    ngx.HTTP_SPECIAL_RESPONSE (300)
    ngx.HTTP_MOVED_PERMANENTLY (301)
    ngx.HTTP_MOVED_TEMPORARILY (302)
    ngx.HTTP_SEE_OTHER (303)
    ngx.HTTP_NOT_MODIFIED (304)
    ngx.HTTP_BAD_REQUEST (400)
    ngx.HTTP_UNAUTHORIZED (401)
    ngx.HTTP_FORBIDDEN (403)
    ngx.HTTP_NOT_FOUND (404)
    ngx.HTTP_NOT_ALLOWED (405)
    ngx.HTTP_GONE (410)
    ngx.HTTP_INTERNAL_SERVER_ERROR (500)
    ngx.HTTP_METHOD_NOT_IMPLEMENTED (501)
    ngx.HTTP_SERVICE_UNAVAILABLE (503)
    ngx.HTTP_GATEWAY_TIMEOUT (504) 

