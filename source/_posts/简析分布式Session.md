---
title: 简析分布式Session
sidebar:
  - toc
date: 2022-02-10 11:08:57
tags: [分布式, 计算机网络]
categories: [分布式]
cover: https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/img/分布式Session-01.png
---

在多个微服务协作的过程中，单个服务产生的SESSION并不能被其他服务所共享（两服务同域）；在单个服务的集群中也会出现这样的问题，用户通过网关访问集群，**负载均衡**到某个服务器产生了一个Session，但是其他集群内的服务器并不能同步或者共享这个Session内容。

综上，种种原因，需要一种解决方案来解决这种困难，解决方案就是 —— 分布式Session。

<!-- more -->

## 问题分析

上面有两个问题：子域不同，同父域之间的服务Session共享、同域的服务集群共享Session。

对于第一个问题，可以修改在浏览器（用户代理）中，SessionID的作用域，将作用域提高到父域即可。

```java 设置CookieSerializer

```

但是第二个问题，就会相对复杂一些，下面会讨论关于Session共享的问题。

## 分布式Session解决方案有哪些？

### 客户端存储

简单来说，就是在浏览器（用户代理）中存储相关的信息。

优点就是，简单，能够在cookie中存储用户信息，所有服务都可以解析这个信息，达到记录信息的效果。

缺点也很明显：
- 在客户端存储的Cookie大小限制为4KB，存储空间有限
- 数据存储在cookie中，如果一次请求cookie过大，会给网络增加更大的开销
- 最致命的问题就是，Cookie是明文传输，所以会造成信息的泄露

### Session会话保持（黏滞会话）

集群内为什么Session会不同步？就是因为负载均衡策略，使得用户的请求到达了不同的服务器。那么有没有办法，让用户始终到同一个服务器，这样不就不会造成Session的不一致了吗？这种方案就是会话保持（粘滞会话）。

我们利用nginx的反向代理和负载均衡，之前是客户端会被分配到其中一台服务器进行处理，具体分配到哪台服务器进行处理还得看服务器的负载均衡算法(轮询、随机、ip-hash、权重等)，但是我们**可以基于nginx的ip-hash策略**，可以对客户端和服务器进行绑定，同一个客户端就只能访问该服务器，无论客户端发送多少次请求都被同一个服务器处理

优点：便于服务器水平拓展，配置简单

缺点：服务器重启丢失Session、存在单点负载很高的风险、单点故障问题

### Session复制

在小型企业中，通常配置一个Tomcat集群，集群中的几台服务器，通过广播的方式在**服务器之间同步Session对象**，这样，每台服务器都有全量的Session数据，这样就不会造成Session数据的丢失。

优点：
- 带来局域网内开销。因为Session之间的同步是通过局域网内广播的机制，进行同步消息的，如果网站访问量增加，局域网内开销就会特别大。
- 带来内存开销。如果登录用户量增加，每台服务器就会存储一个这个用户的Session信息。这样显然造成了很大的冗余，如果集群非常庞大，这个内存占用量是难以接受的。

### Session服务器集群

使用Redis存储Session的方式。其实说白了就是中间再来一层在不改变原有负载军和策略的前提下，减少耦合，减少数据的冗余存储。

{% image https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/img/分布式Session-01.png Redis共享Session %}

这样的解决方案，优点总是多于缺点，所以被很多企业所采用

- 能适应各种负载均衡策略
- 服务器重启或宕机不会造成Session丢失
- 扩展能力强
- 适合集群数量大时使用

缺点：
- 在访问Session会增加一次网络查询，增加网络开销。
- 对应用有入侵，需增加相关配置
- 序列化反序列化消耗CPU性能

可以将存储Session的Redis集群和服务器放置在同意内网环境，减少网络带来的开销。

## Spring Session的实现原理

Spring Session的主要原理是对Http请求和响应进行拦截，将请求和响应对象进行包装，并且将HttpSession的实现交由Spring的Session支持（由HttpSessionAdapter作为**适配器**）。最终实现由Spring来接管Session。

### SpringHttpSessionConfiguration：有哪些关键配置？

{% noteblock %}
1. SessionRepository：最终能够实现Session存储的Repository
2. SessionRepositoryFilter：拦截请求，封装请求体和响应体（**装饰器模式**）
{% endnoteblock %}


`SpringHttpSessionConfiguration`主要配置了一个`SessionRepositoryFilter`，这个类最重要的参数就是`SessionRepository<S> sessionRepository`，最终要使用Redis来存储Session，需要一个Redis实现的SessionRepository，以及一个Redis实现的Session。

```java SpringHttpSessionConfiguration#springSessionRepositoryFilter
@Bean
public <S extends Session> SessionRepositoryFilter<? extends Session> springSessionRepositoryFilter(
		SessionRepository<S> sessionRepository) {
	SessionRepositoryFilter<S> sessionRepositoryFilter = new SessionRepositoryFilter<>(sessionRepository);
	sessionRepositoryFilter.setHttpSessionIdResolver(this.httpSessionIdResolver);
	return sessionRepositoryFilter;
}
```

### SessionRepositoryFilter是怎么做到拦截？

Java WEB中的拦截器，就是一个很好的拦截载体，只要将拦截到的请求进行封装。重写其`getSession()`就可以获得由Spring支持的Session。

因为`SessionRepositoryFilter`继承自`OncePerRequestFilter`。所以，在请求来的时候，就会进行拦截，优先级最高，这样也保证了拦截器可以尽快将Session与HttpSession进行适配，使HttpSesion由Spring的Session支持。

{% image https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/img/Session.png Session和HttpSessionAdapter %}

```java SessionRepositoryFilter
@Override
rotected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
	throws ServletException, IOException {
    // 将SessionRepository放进HttpServletRequest中
    request.setAttribute(SESSION_REPOSITORY_ATTR, this.sessionRepository);
    // 包装HttpServletRequest
	SessionRepositoryRequestWrapper wrappedRequest = new SessionRepositoryRequestWrapper(request, response);
    // 包装HttpServletResponse
	SessionRepositoryResponseWrapper wrappedResponse = new SessionRepositoryResponseWrapper(wrappedRequest,
				response);

  try {
    // 继续接下来的请求拦截
		filterChain.doFilter(wrappedRequest, wrappedResponse);
	}
	finally {
		wrappedRequest.commitSession();
	}
}
```

### SessionRepository：为什么支持多种存储类型？

SessionRepository的实现在Spring Session中有很多，所以，SessionRepository支持多种存储类型。极大增加了Session存储的灵活性

## 参考资料

1. [程序员大本营] [Session复制的解决方案，你知道几种？](https://www.pianshen.com/article/97971906796/)
2. [CSDN] [4种分布式session解决方案](https://blog.csdn.net/qq_35620501/article/details/95047642)
3. Spring Session 源代码
