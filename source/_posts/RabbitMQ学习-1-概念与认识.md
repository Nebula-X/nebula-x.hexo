---
title: "RabbitMQ学习(1)--概念与认识"
catalog: true
date: 2018-11-21 10:11:54
subtitle: 
header-img: "Demo.png"
tags:
- RabbitMQ 消息队列
catagories:
- RabbitMQ
---
# RabbitMQ学习(1)--概念与认识

## 消息队列

### 解决了什么问题
1.异步处理
>以前一般写业务是串行模式，发短信是耗费时间的操作，串行有可能断开HTTP连接还没有做完

![异步处理](异步处理.png)
2.应用解耦
>即多个子系统有关联，例如库存与订单系统，如果使用串行的模式，有可能在运行的过程中，由于系统间的通讯异常,造成数据丢失，这样会影响整个流程不能正常的走通

![系统解耦](系统解耦.png)
3.流量削锋
>使用MQ之后,如果不满足的请求直接丢弃，这样可以把我们大量请求处理进行隔绝，保证业务不会因海量的访问而崩溃

![秒杀抢购](秒杀抢购.png)
4.日志处理
>下面例子用于统计UV,PV等日志数据例子,一般来说，日志不会直接写入数据库，而是先写入文件，再由日志收集工具(logstash)进行统一的处理

>流计算：就是实时计算，生成显示数据
离线计算：对数据进行持久化

![日志收集与处理](日志收集与处理.png)

5. 等等

### RabbitMQ安装 (不做详细的介绍)
>自行参考网址: https://www.cnblogs.com/zzpblogs/p/8168763.html

### RabbitMQ的简单使用

#### 用户管理
1.添加用户
![用户管理-添加用户](用户管理-添加用户.png)

#### virtual hosts管理
virtual hosts 相当于mysql的db(即数据库)，一般以"/"开头
创建完数据库后，就要对数据库进行授权，
![vhost](vhost.png)
![v-host授权](v-host授权.png)
![v-host添加](v-host添加.png)
![v-host授权2](v-host授权2.png)
![v-host授权后](v-host授权后.png)
