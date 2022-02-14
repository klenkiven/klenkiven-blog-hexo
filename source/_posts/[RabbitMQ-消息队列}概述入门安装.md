---
title: '[RabbitMQ] 概述 | 入门 | 安装'
sidebar:
  - toc
date: 2022-02-13 23:50:34
tags: [分布式, 消息队列]
categories: [消息队列]
---
## 简介

RabbitMQ是一个消息中间件。它是一个数据服务器，它接受消息并用它们做两个主要的事情，它根据任意标准将它们路由到不同的消费者，当消费者不能足够快地接受它们时，它会将它们缓冲在内存或磁盘上。

用途：

- 异步处理

  很多能够异步处理的事件，不仅能够使用多线程来实现，也可以加一层消息队列，更加高效的实现异步事件。

  ![image-20220212200341246](https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220212200341246.png)

- 应用解耦

  例如，订单系统对库存系统有影响，产生一个订单的时候，在库存系统应该减少对应的库存量。常规的做法是远程调用库存的接口，实现这个功能。如果库存接口，频繁升级修改（现实中并不会这样），就会导致订单系统也需要频繁修改代码。这样增加系统的耦合性，违反开闭原则。如果中间加一层消息队列，就可以一定程度上解决这种问题。

  ![image-20220212200434875](https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220212200434875.png)

- 流量控制
  
  例如，要做秒杀系统，突然大量的用户请求涌来，系统无法承受这么大的流量，如果中间没有一个缓冲措施，很有可能系统就会发生崩溃。中间加一层消息队列，将用户请求一个个存储起来，慢慢处理，这样就能避免因为突发的大量流量使系统崩溃的问题。

  ![image-20220212200448208](https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220212200448208.png)

###  概述

消息中间件提高异步通信，扩展解耦的能力

消息代理（Message Broker）和目的地（Destination）

目的地主要有两个：队列（Queue）和主题（Topic）

1. **点对点模式**：消息唯一的发送者和很多接受者，但是每个消息只能接受一次

2. **发布订阅模式**：多个订阅者，多个监听，消息到来，全部订阅者得到消息

3. JMS / AMQP协议

   ![image-20220212201124938](https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220212201124938.png)

### 优势

1. 标准化支持多种标准协议，JMS，AMQP等等
2. 互联网公司使用比较多

### Spring 整合

1. spring-jms
2. spring-rabbit
3. JmsAutoCOnfiguration
4. RabbitAutoConfiguration

## RabbitMQ（AMQP）基本概念

{% image https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/img/AMQ-Architecture.png AMQ Main Architecture %}

1. Message

   消息头+消息体：消息体不透明，消息头透明

   - route-key：在一般情况下，Exchange检查消息的属性、头字段和消息内容，并使用这些以及可能来自其他来源的数据来决定如何路由消息。 

2. Queue
   
   队列是位于 RabbitMQ 中的邮箱的名称。尽管消息流经 RabbitMQ 和您的应用程序，但它们只能存储在队列中。  队列仅受主机的内存和磁盘限制，它本质上是一个大的**消息缓冲区**。许多生产者可以发送去一个队列的消息，许多消费者可以尝试从一个队列接收数据。

3. Publisher

    生产无非就是发送。发送消息的程序是生产者
   
4. Consumer

    消费与接收具有相似的含义。消费者是一个主要等待接收消息的程序

5. Exchange

    交换器接受来自生产者应用程序的消息，并根据预先安排的标准将这些消息**路由**到消息队列

6. Binding

   Exchanger ---Binding---> Queue

   这些标准称为“绑定（Binding）”。Exchange是匹配和路由引擎。也就是说，他们检查消息并使用它们的绑定表，决定如何将这些消息转发到消息队列或其他交换。 Exchange从不存储消息。

7. Broker

    消息代理也就是RabbitMQ

8. Connection: RabbitMQ使用TCP长连接

9. Channel：连接可以产生许多信道，真正的操作都是在信道上进行的

10. Virtual Host: 隔离不同的业务，例如，"/java", "/python"

## 安装RabbitMQ

- 4369, 25672：Erlang发现 & 集群端口
- 5672, 5671：AMQP端口
- 15672：WEB管理后台端口
- 61613, 61614：STOMP协议端口
- 1883, 8883：MQTT协议端口

参考：https://www.rabbitmq.com/networking.html

```bash 安装RabbitMQ
sudo docker pull rabbitmq:management
sudo docker run --name rabbitmq -d \
-p 5671:5671 -p 5672:5672 \
-p 4369:4369 \
-p 25672:25672 \
-p 15671:15671 -p 15672:15672 \
--restart=always \
rabbitmq:management
```

## 参考资料

1. [Bilibili] [尚硅谷RabbitMQ教程丨快速掌握MQ消息中间件](https://www.bilibili.com/video/BV1cb4y1o7zz)
2. [Bilibili] [Java项目《谷粒商城》Java架构师 | 微服务 | 大型电商项目](https://www.bilibili.com/video/BV1np4y1C7Yf)
3. [RabbitMQ] [Advanced Message Queuing Protocol - Protocol Specification](https://www.rabbitmq.com/resources/specs/amqp0-9-1.pdf)