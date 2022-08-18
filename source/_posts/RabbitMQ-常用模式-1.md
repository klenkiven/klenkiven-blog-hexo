---
title: RabbitMQ-常用工作模式
sidebar:
  - toc
date: 2022-02-18 15:51:41
tags: [消息队列, 分布式]
categories: [消息队列]
cover: https://www.rabbitmq.com/img/tutorials/python-two.png
---

RabbitMQ常用的模式在官网上主要有六种模式：简单队列，工作队列模式，发布订阅模式，路由模式，主题模式，RPC模式。本篇文章主要讨论RabbitMQ的其中前三种工作模式。

<!--more-->

## Introduction

首先介绍一下整个POST的标志的含义，以及消息队列主要的参与角色：

1. 生产者（Producer）
   
   生产者主要的工作就是产生消息，发送消息。

   ![Producer](https://www.rabbitmq.com/img/tutorials/producer.png)

2. 队列（Queue）

   队列其实就是一个在RabbitMQ中的邮箱名称。尽管消息流经 RabbitMQ 和应用程序，但它们只能存储在队列中。队列仅受主机的内存和磁盘限制，它本质上是一个大的消息缓冲区。许多生产者可以发送到一个队列的消息，许多消费者可以尝试从一个队列接收数据。

   ![Queue](https://www.rabbitmq.com/img/tutorials/queue.png)

3. 消费者（Consumer）
   
   消费者就是接收处理消息的角色。

   ![Consumer](https://www.rabbitmq.com/img/tutorials/consumer.png)

## 简单队列模式

消息队列本质上就是一个队列，最基础的功能，自然就是简单的队列模式。

![简单队列模式](https://www.rabbitmq.com/img/tutorials/python-one.png)

发送端：

发送端需要声明一个队列，以便于消费端接收消息。

```JAVA 发送端核心代码
public class Send {

    private final static String QUEUE_NAME = "hello";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            String message = "Hello World!";
            channel.basicPublish("", QUEUE_NAME, null, 
                message.getBytes(StandardCharsets.UTF_8));
            System.out.println(" [x] Sent '" + message + "'");
        }
    }
}
```

接收端：

接收端的`channel.basicConsume()`是一个异步处理过程（新建一个监听线程），其中`DeliverCallback`主要用于处理接收的消息。这样就可以做到，不断监听消息，不影响主线程的继续进行。

```JAVA 接收端核心代码
public class Recv {

    private final static String QUEUE_NAME = "hello";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        // 消息处理回调以及channel.basicConsume
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println(" [x] Received '" + message + "'");
        };
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, consumerTag -> { });
    }
}
```

## 工作队列模式

前面一个模式，主要用于简单的队列模式，为消息提供一个很大的缓冲空间，存储来不及处理的消息。这个模式主要用于耗时较长的消息处理，一个消费者消费速度比较慢，就可以增加消费端来处理消息，增加消息处理效率。

这种工作模式的核心思想就是，避免同步去做`资源密集型`任务，这种任务交由后台调度处理。尤其是对于HTTP处理，用户要做一个对于后端处理时间较长的操作时，就可以将这个请求作为消息发送到消息队列中，然后将成功消息发送给用户，用户就可以获得很好的反馈体验。

![工作队列模式](https://www.rabbitmq.com/img/tutorials/python-two.png)

### 轮询调度

非抢占的轮询调度，保证每个消费端，都可以获得任务，并行地完成任务，这样的做法，增加了消费端的可扩展性。如果一个消费端消费速度难以跟上生产者的生产速度，那么就可以简单的增加消费者的数量来完成任务。

开启两个消费者端

```BASH 开启消费者端
# shell 1
java -cp $CP Worker
# => [*] Waiting for messages. To exit press CTRL+C

# shell 2
java -cp $CP Worker
# => [*] Waiting for messages. To exit press CTRL+C
```

发送消息

```BASH 发送消息代码
# shell 3
java -cp $CP NewTask First message.
# => [x] Sent 'First message.'
java -cp $CP NewTask Second message..
# => [x] Sent 'Second message..'
java -cp $CP NewTask Third message...
# => [x] Sent 'Third message...'
java -cp $CP NewTask Fourth message....
# => [x] Sent 'Fourth message....'
java -cp $CP NewTask Fifth message.....
# => [x] Sent 'Fifth message.....'
```

按照轮询机制，其中一个消费者SHELL将会处理1，3，5的消息，另一个将会处理2，4的消息。这是RabbitMQ默认的发送原则，按照顺序一个一个将消息发送到每一个消费端。这样的操作将会带来一些问题，后面将会讨论到。

### 消息确认机制

试想在消息在发送到接收端以后，就删除这个消息，一般情况下是没有问题的，而且可以一定程度上增加消息的处理效率。但是，如果出现消费端意外终止的问题，就会导致任务丢失，这样的结果很难以承受，所以需要一种确认机制来保证消息的正确投递。

RabbitMQ采用[消息确认](https://www.rabbitmq.com/confirms.html)来保证消息的不丢失。消费者发回一个确认，告诉 RabbitMQ 一个特定的消息已经被接收、处理并且 RabbitMQ 可以自由地删除它。

如果消费者宕机（Channel或者Connection关闭）没有发送ACK，那么RabbitMQ就可以理解消息没有呗正确消费，这种情况下，消息就需要重新排队，重新分配给其他消费者处理消息。这样就可以保证消费者宕机后，消息依然不会丢失，可以正确被处理。

默认情况下，手动消息确认是打开的。在前面的示例中，我们通过`autoAck=true`标志明确地关闭了它们。一旦我们完成了一项任务，是时候将此标志设置为`false`并从工作人员那里发送适当的确认。

```JAVA 消息确认机制
channel.basicQos(1); // accept only one unack-ed message at a time (see below)

DeliverCallback deliverCallback = (consumerTag, delivery) -> {
  String message = new String(delivery.getBody(), "UTF-8");

  System.out.println(" [x] Received '" + message + "'");
  try {
    doWork(message);
  } finally {
    System.out.println(" [x] Done");
    channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
  }
};
boolean autoAck = false;
channel.basicConsume(TASK_QUEUE_NAME, autoAck, deliverCallback, consumerTag -> { });
```

### 消息的持久化

现在通过消费端的确认机制，保证消息不会在消费过程中丢失，那么还有一种可能丢失消。当RabbitMQ本身崩溃的时候，就会将内存中的数据丢失，导致消息的大量丢失。或者RabbitMQ重启，内存中的数据也会发生丢失的问题，因此需要一种机制来保证数据即使RabbitMQ崩溃也可以存储。

这就是消息的持久化。

```JAVA 消息的持久化设置
boolean durable = true;
channel.queueDeclare("task_queue", durable, false, false, null);
```

{% grid 关于消息持久化的注意事项 %}
消息的持久化，并不是意味着消息不会丢失。尽管消息被RabbitMQ存储了下来，但是还是有很短的一段时间的消息没有被存储下来可以参考数据库的日志机制。新接收的消息会被缓存起来，隔一段时间才会把数据刷进去，或者消息放入磁盘缓冲区，由操作系统决定何时刷入磁盘。这因为这样并不能保证消息真的被刷进去。需要更强的保证，可以参考[发布者确认机制](https://www.rabbitmq.com/confirms.html)。
{% endgrid %}

### 公平调度

前面提到的[轮询调度](#轮询调度)有个问题：如果有两台消费端，奇数的任务处理时间长，偶数任务处理时间短，这就会造成一个问题，消费偶数消息的处理机干等干活少，另一个累死。这是因为RabbitMQ没办法评估任务的量，所以盲目的把第n个消息给第n个消费者，造成了这样的问题。

为了避免这种情况，就需要更高效的调度机制 -- 公平调度。

使用`basicQos()`和`prefetchCount = 1`的设置来达到公平调度的目的。这样相当于告诉RabbitMQ不要一次把对应序号的消息全部给消费者，而是最多给一条消息。在消费端消费完成之前的任务并且确认后，再将新的消息派发给消费端。这样，消息就不会因为预先分配好，导致工作量不同的问题了。

```JAVA “公平”调度
int prefetchCount = 1;
channel.basicQos(prefetchCount);
```

### 源代码

发送端：

```JAVA 发送端核心代码
public class NewTask {

  private static final String TASK_QUEUE_NAME = "task_queue";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    try (Connection connection = factory.newConnection();
         Channel channel = connection.createChannel()) {
        channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);

        String message = String.join(" ", argv);

        channel.basicPublish("", TASK_QUEUE_NAME,
                MessageProperties.PERSISTENT_TEXT_PLAIN,
                message.getBytes("UTF-8"));
        System.out.println(" [x] Sent '" + message + "'");
    }
  }

} 
```

接收端：

```JAVA 接收端核心代码
public class Worker {

  private static final String TASK_QUEUE_NAME = "task_queue";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    final Connection connection = factory.newConnection();
    final Channel channel = connection.createChannel();

    channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);
    System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

    channel.basicQos(1);

    DeliverCallback deliverCallback = (consumerTag, delivery) -> {
        String message = new String(delivery.getBody(), "UTF-8");

        System.out.println(" [x] Received '" + message + "'");
        try {
            doWork(message);
        } finally {
            System.out.println(" [x] Done");
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        }
    };
    channel.basicConsume(TASK_QUEUE_NAME, false, deliverCallback, consumerTag -> { });
  }

  private static void doWork(String task) {
    for (char ch : task.toCharArray()) {
        if (ch == '.') {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException _ignored) {
                Thread.currentThread().interrupt();
            }
        }
    }
  }
}
```

## 发布/订阅模式

发布/订阅模式，主要依靠`Exchange`来将生产者的消息发送到多个队列中去。示例中使用相对较为简单的`fanout`作为广播Exchange。

![发布/订阅模式](https://www.rabbitmq.com/img/tutorials/python-three-overall.png)

### 临时队列

当使用一个队列的时候，队列的名称是很重要的，但是并不是所有时候，例如想要一个临时队列的时候，并不关心它叫什么名字，并且希望在使用队列结束以后能够删除这个队列，这个时候，就需要临时队列。

{% grid %}
In the Java client, when we supply no parameters to queueDeclare() we create a **non-durable**, **exclusive**, **autodelete** queue with a **generated name**:
{% endgrid %}

```JAVA 临时队列
String queueName = channel.queueDeclare().getQueue();
```

### 绑定（Binding）

使用扇出的交换机构，那么就需要绑定一个队列来转发消息。可以使用下面的方法绑定。

```JAVA 绑定
channel.queueBind(queueName, "logs", "");
```
### 源代码

发送端：

```JAVA 发送端核心代码
public class EmitLog {

  private static final String EXCHANGE_NAME = "logs";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    try (Connection connection = factory.newConnection();
         Channel channel = connection.createChannel()) {
        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");

        String message = argv.length < 1 ? "info: Hello World!" :
                            String.join(" ", argv);

        channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes("UTF-8"));
        System.out.println(" [x] Sent '" + message + "'");
    }
  }
}
```

接收端：

```JAVA 接收端核心代码
public class ReceiveLogs {
  private static final String EXCHANGE_NAME = "logs";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
    String queueName = channel.queueDeclare().getQueue();
    channel.queueBind(queueName, EXCHANGE_NAME, "");

    System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

    DeliverCallback deliverCallback = (consumerTag, delivery) -> {
        String message = new String(delivery.getBody(), "UTF-8");
        System.out.println(" [x] Received '" + message + "'");
    };
    channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
  }
}
```

## 参考资料

1. [RabbitMQ] [RabbitMQ Tutorials](https://www.rabbitmq.com/getstarted.html)
2. [RabbitMQ] [Consumer Acknowledgements and Publisher Confirms](https://www.rabbitmq.com/confirms.html)
3. [RabbitMQ] [Advanced Message Queuing Protocol - Protocol Specification](https://www.rabbitmq.com/resources/specs/amqp0-9-1.pdf)