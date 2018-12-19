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

