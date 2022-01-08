---
title: '[分布式]分布式锁以及Redission'
sidebar:
  - toc
date: 2022-01-08 23:50:34
tags: [分布式, 并发编程]
categories: [分布式]
---

随着高并发场景的扩大，使用人数激增，单台服务器的单个服务已经难以满足日益增长的需要了。这时候就需要使用一个集群来处理问题。暂且先不谈引入集群的各种问题，就数据的并发场景的安全性而言，又是一个挑战。例如，多个相同的服务访问Redis，尽管每个服务内部都是线程安全的，但是多个服务之间却缺少一种约束，那么在这种条件下，使用一个“锁机制”就很有必要。在这种场景下使用的锁就叫做分布式锁。

<!-- more -->

## 为什么要使用分布式锁？

当一个服务正在使用而且不存在并发场景的时候，很显然是不需要锁的，也自然不存在并发的安全问题。但是如果存在高并发场景的时候，这不仅对性能上是一个挑战，而且在线程安全的问题上提出了挑战。

那么类似的，随着高并发场景的扩大，使用人数激增，单台服务器的单个服务已经难以满足日益增长的需要了。这时候就需要使用一个集群来处理问题。暂且先不谈引入集群的各种问题，就数据的并发场景的安全性而言，又是一个挑战。例如，多个相同的服务访问Redis，尽管每个服务内部都是线程安全的，但是多个服务之间却缺少一种约束，那么在这种条件下，使用一个“锁机制”就很有必要。在这种场景下使用的锁就叫做分布式锁。

## 如何设计一个分布式锁？

1. 最简单的想法就是，在Redis上面设置一个键值对，值可以是任意的。如果其他的服务发现这个键已经存在了，那么就自旋等待。获得锁的服务就可以执行业务，最后释放这个锁。这就是最简单的一个分布式锁的模型。
    {% image https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/img/分布式锁阶段一.png 分布式锁阶段一 %}
2. 为了解决服务宕机之后不会造成死锁问题，那么就需要增加一个过期时间。
    {% image https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/img/分布式锁阶段一.png 分布式锁阶段二 %}
3. 设置过期时间也会带来很多问题，如果在设置过期时间之前，服务器宕机就还会发生死锁的问题。那么这就需要设置锁和设置过期时间这个操作是原子性的。
    {% image https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/img/分布式锁阶段三.png 分布式锁阶段三 %}
4. 删除锁的不安全问题
    {% image https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/img/分布式锁阶段四.png 分布式锁阶段四 %}
5. 删除保证原子性
    {% image https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/img/分布式锁阶段五.png 分布式锁阶段五 %}

## 像使用Java Lock一样使用 分布式锁

### 读写锁

读锁是一个共享锁，写锁是一个派他锁。写锁的时候，读和写都是不被允许的，只有在读的时候所有线程可以进行读取操作。

```java 读写锁示例
@GetMapping("write")
@ResponseBody
public String write() {
    String s = "";
    Lock lock = redissonClient.getReadWriteLock("rw-lock").writeLock();
    try {
        lock.lock();
        s = UUID.randomUUID().toString();
        redisTemplate.opsForValue().set("rw-test", s);
        Thread.sleep(10 * 1000);
    } catch (Exception e) {
        // Do nothing
    } finally {
        lock.unlock();
    }
    return s;
}

@GetMapping("read")
@ResponseBody
public String read() {
    String s = "";
    Lock lock = redissonClient.getReadWriteLock("rw-lock").readLock();
    try {
        lock.lock();
        s = redisTemplate.opsForValue().get("rw-test");
    } finally {
        lock.unlock();
    }
    return s;
}
```

{% image https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/img/分布式锁-读写锁.png  分布式锁-读写锁 %}

不同场景下读写锁情况：

* 读 + 读：并发场景下同时加锁成功
* 写 + 写：阻塞等待写锁释放
* 写 + 读：阻塞等待写锁释放
* 读 + 写：等待读锁释放

### 信号量

基于Redis的Redisson的分布式信号量（[Semaphore](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RSemaphore.html)）Java对象`RSemaphore`采用了与`java.util.concurrent.Semaphore`相似的接口和用法。同时还提供了[异步（Async）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RSemaphoreAsync.html)、[反射式（Reactive）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RSemaphoreReactive.html)和[RxJava2标准](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RSemaphoreRx.html)的接口。

信号量如果失败使用`tryAcquire()` 不会发生阻塞，但是如果使用`acquire()` 会发生阻塞等待。利用这个属性，可以进行分布式的流量控制。

```java 信号量的使用
RSemaphore semaphore = redisson.getSemaphore("semaphore");
semaphore.acquire();
//或
semaphore.acquireAsync();
semaphore.acquire(23);
semaphore.tryAcquire();
//或
semaphore.tryAcquireAsync();
semaphore.tryAcquire(23, TimeUnit.SECONDS);
//或
semaphore.tryAcquireAsync(23, TimeUnit.SECONDS);
semaphore.release(10);
semaphore.release();
//或
semaphore.releaseAsync();
```

例子：停车抢车位。acquire就是抢车位，release就是释放车位，如果抢不到车位就阻塞等待，或者直接开走。

### 闭锁

基于Redisson的Redisson分布式闭锁（[CountDownLatch](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RCountDownLatch.html)）Java对象`RCountDownLatch`采用了与`java.util.concurrent.CountDownLatch`相似的接口和用法。

```java 闭锁的使用
RCountDownLatch latch = redisson.getCountDownLatch("anyCountDownLatch");
latch.trySetCount(1);
latch.await();

// 在其他线程或其他JVM里
RCountDownLatch latch = redisson.getCountDownLatch("anyCountDownLatch");
latch.countDown();
```

例子：放假锁门，只有在所有的窗户都关上了才能锁教室的门。

`latch.trySetCount(1);` 设置有多少个窗户需要被关闭，然后`latch.await();` 等着关窗户。

`latch.countDown();` 关一个窗户就减少一个，最后全部关闭以后，就可以锁教室门口了。

## Redisson 以及 Redisson和Spring的整合

### 什么是Redisson

Redission是一个Redis客户端，但是不只是如此，前面所说的分布式锁，在Redisson中也有实现。

### Spring整合Redission

在配置类中配置RedissionClient

```java Spring整合Redission
/**
 * Redisson - Redis Client with Redis Distribute Lock
 */
@Configuration
public class RedissonConfig {

    @Value("${spring.redis.host}")
    private String redisHost;

    @Value("${spring.redis.port}")
    private String redisPort;

    /**
     * Redisson Config in Single Node Mode
     */
    @Bean(destroyMethod = "shutdown")
    public RedissonClient redissonClient () {
        Config config = new Config();
        config.useSingleServer().setAddress("redis://" + redisHost + ":" + redisPort);
        return Redisson.create(config);
    }
}
```

### 使用Redission的锁功能

```java 使用Redission的锁
@GetMapping("product/hello")
@ResponseBody
public String hello() {
    // 1. Get Distribute Lock
    RLock lock = redissonClient.getLock("klenkiven-redisson-lock");
    // 2. Lock
    // 在这里加锁的代码  

    try {
        // 下面的程序模式一个长时间的业务流程
        System.out.println("Do Business by Thread-" + Thread.currentThread().getId());
        Thread.sleep(30 * 1000);
    } catch (Exception e) {
        System.out.println("Business Exception by Thread-" + Thread.currentThread().getId());
    } finally {
        System.out.println("Release Lock by Thread-" + Thread.currentThread().getId());
        lock.unlock();
    }
    return "hello";
}
```

1. 普通的加锁

    ```java
    lock.lock(); // 阻塞等待。默认等待30s
    // 这个锁使用看门狗程序进行自动锁的过期时间延长，防止锁因为业务时间过长导致失效。
    // 如果服务崩溃，看门狗不会自动续期最后锁失效。
    ```

2. 使用带有计时的锁

    ```java
    lock.lock.lock(10, TimeUnit.SECONDS); // 阻塞等待，10秒过期，删除锁
    ```

    1. 如果传递了锁的超时时间，就发送给Redis执行脚本，进行占锁的操作，默认就是指定的时间
    2. 默认传递时间是-1，如果未指定时间，就是用30s（LockWatchDogTimeout看门狗的默认超时时间）

        只要占锁成功，就会启动一个定时任务：重新给锁设置过期时间，新的过期时间就是看门狗的默认过期时间。每过1/3看门狗超时时间，就重新设置超时时间。

#### 最佳实践

使用手动设置的带有计时器的锁，这样的锁可以节约续期操作。
