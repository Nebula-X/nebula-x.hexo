---
title: "RabbitMQ学习(2)--队列模型"
catalog: true
date: 2018-11-21 14:11:54
subtitle: 
header-img: "Demo.png"
tags:
- RabbitMQ 消息队列
catagories:
- RabbitMQ
---

# RabbitMQ学习(1)--队列模型

## 概念
RabbitMQ存在六种模型
>1.简单队列
>2.工作队列
>3.发布订阅
>4.路由
>5.Topic
>6.RPC

## 模型
### 简单队列
![简单队列](简单队列.png)
>模型解析
P: 消息生产者
红色方块集合: 消息队列
C: 消息消费者

#### Golang实现简单队列

官方依赖库: go get github.com/streadway/amqp

向队列发送消息具体流程
>1.获取MQ的连接
>2.创建频道,从连接中获取通道
>3.创建队列声明
>4.创建消息，并发送消息
>5.关闭连接

>具体代码:
```
package main
import (
	"github.com/streadway/amqp"
	"log"
)

// We also need an helper function to check the return value for each amqp call
func failOnError(err error, msg string) {
	if err != nil {
		log.Fatalf("%s: %s", msg, err)
	}
}

func main() {
	// 连接到RabbitMQ Server
	conn, err := amqp.Dial("amqp://user_mmr:123456@localhost:5672/")
	failOnError(err, "Failed to connect to RabbitMQ")
	defer conn.Close()
	// 创建channel，channel为驻留程序
	ch, err := conn.Channel()
	failOnError(err, "Failed to Open a channel")
	defer ch.Close()
	// 我们需要声明发送队列，那么我们可以发送消息到这个队列上
	// 队列的创建时幂等的，只有当不存在，才会创建队列，且队列名是唯一的
	q, err := ch.QueueDeclare(
		"hello", // name
		false,   // durable
		false,   // delete when unused
		false,   // exclusive
		false,   // no-wait
		nil,     // arguments
	)
	failOnError(err, "Failed to declare a queue")
	body := "Hello World222!"
	err = ch.Publish(
		"",     // exchange
		q.Name, // routing key
		false,  // mandatory
		false,  // immediate
		amqp.Publishing{
			ContentType: "text/plain",
			Body:        []byte(body),
		})
	failOnError(err, "Failed to publish a message")
}
```


从队列接收消息具体流程
>1.获取MQ的连接
>2.创建频道,从连接中获取通道
>3.定义队列的监听者
>4.监听队列(开一个协程来监听处理)

>具体代码:
```
package main

import (
	"fmt"
	"github.com/streadway/amqp"
	"log"
)

// We also need an helper function to check the return value for each amqp call
func failOnError(err error, msg string) {
	if err != nil {
		log.Fatalf("%s: %s", msg, err)
	}
}

func main() {
	conn, err := amqp.Dial("amqp://user_mmr:123456@localhost:5672/")
	failOnError(err, "Failed to connect to RabbitMQ")
	defer conn.Close()

	ch, err := conn.Channel()
	failOnError(err, "Failed to open a channel")
	defer ch.Close()

	q, err := ch.QueueDeclare(
		"hello", // name
		false,   // durable
		false,   // delete when usused
		false,   // exclusive
		false,   // no-wait
		nil,     // arguments
	)
	failOnError(err, "Failed to declare a queue")
	msgs, err := ch.Consume(
		q.Name, // queue
		"",     // consumer
		true,   // auto-ack
		false,  // exclusive
		false,  // no-local
		false,  // no-wait
		nil,    // args
	)
	failOnError(err, "Failed to register a consumer")
	fmt.Println("aaaaaa")
	forever := make(chan bool)

	go func() {
		for d := range msgs {
			log.Printf("Received a message: %s", d.Body)
		}
	}()

	log.Printf(" [*] Waiting for messages. To exit press CTRL+C")
	<-forever
}

```

#### 简单队列的不足
耦合性高，生产者一一对应消费者(如果我想有多个消费者消费队列中的消息，这种情况就不行了)
队列名变更，这个时候也得同时变更

### 工作队列
![工作队列](工作队列.png)

####为什么会出现工作队列
Simple 队列是一一对应的，而且我们实际开发，生产者发送消息是毫不费力的，而消费者一般是要跟业务相结合的，消费者接收到消息之后就需要处理，可能需要花费时间,即耗时操作，由于只有一个消费者，此时队列就会挤压了很多的消息等待处理，所以工作队列能够解决这个问题

从上面的代码中可以发现,消费者1和消费者2处理的消息是一样的,
消费者1: 偶数
消费者2：奇数
这个方式叫做轮训分发(round-robin),结果就是不管谁忙者谁清闲，都不会多给一个消息，消息任务总是你一个我一个







