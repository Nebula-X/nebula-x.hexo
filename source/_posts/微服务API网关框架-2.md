---
title: 微服务API网关框架(2)--Nginx的使用
catalog: true
date: 2018-12-08 13:13:15
subtitle:
header-img: "Demo.png"
tags:
- 微服务API网关
catagories:
- 微服务API网关
---
# Nginx

## Nginx的命令

### 启动
    指令: nginx -c xxxxx[xxxxx为nginx的配置文件]

### 重启
    nginx -s reload

### 停止
    nginx -s stop
    nginx -s quit
 
    注意:quit是一个优雅的关闭方式，Nginx在退出前完成已经接受的连接请求
         stop是一个快速关闭，不管有没有正在处理的请求
     
### 重新打开日志
    nginx -s reopen

### nginx检查配置文件
    检查配置文件是否正确
    nginx -t 
    nginx -t -c xxxxx [这个指令是先检查，然后运行nginx服务]
    
## Nginx的信号启动
    Nginx支持两种进程模型Single和Master-Worker
    Single是单进程，一般不适用
    Master-Worker是主进程和工作进程模型运行，主进程对工作进程管理
    Nginx允许我们通过信号来控制主进程，用信号的方式可以达到不影响现有连接的目的

### Nginx的信号类型
    INT,TERM                    快速关闭信号
    QUIT                        从容关闭信号
    HUP                         从容重启信号，一般用于修改配置文件后，重启
    USR1                        重读日志，一般用于日志的切割
    USR2                        平滑升级信号
    WINCH                       从容关闭旧进程
        
### Nginx具体语法
    kill -信号量  nginx的主进程号
    例如,
    kill -INT 26661
    kill -HUP 487
    
    (1)nginx停止
       ps -ef | grep nginx 获得的进程号    
    
    第一种从容"优雅"停止
    kill -QUIT master进程号
    Nginx服务可以正常地处理完当前所有请求再停止服务
    步骤：首先会先关闭监听端口，停止接收新的连接，然后把当前正在处理的连接全部处理完，最后在退出进程
    
    第二种快速停止
    kill -TERM master进程号
    kill -INT master进程号
    快速停止服务时，worker进程与master进程在收到信号后会立即跳出循环，退出进程
    
    第三种强制停止
    pkill -9 nginx
    系统强杀nginx进程
    
    重启nginx
    kill -HUP master进程号
    
## Nginx平滑升级
![平滑升级](平滑升级.png) 
![平滑升级2](平滑升级2.png) 
![平滑升级中途退出](平滑升级中途退出.png) 
  
## Nginx配置文件
 ![基本配置](基本配置.png)    

### 配置文件的注意事项 
 ![配置文件的注意事项](配置文件的注意事项.png)    
 ![配置文件的注意事项-2](配置文件的注意事项-2.png)    
 ![配置文件的注意事项-3](配置文件的注意事项-3.png)    
 ![配置文件的注意事项-4](配置文件的注意事项-4.png)    
 ![配置文件的注意事项-5](配置文件的注意事项-5.png)    
 ![配置文件的注意事项-6](配置文件的注意事项-6.png)    
 ![配置文件的注意事项-7](配置文件的注意事项-7.png)    
 ![配置文件的注意事项-8](配置文件的注意事项-8.png)    
