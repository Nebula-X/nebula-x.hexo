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
