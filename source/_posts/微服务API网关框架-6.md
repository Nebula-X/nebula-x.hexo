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

