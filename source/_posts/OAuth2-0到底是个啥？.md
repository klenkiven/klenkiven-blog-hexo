---
title: OAuth2.0到底是个啥？
sidebar:
  - toc
date: 2022-08-18 10:53:55
tags: [认证权限]
categories: [认证权限]
---

经过一个月的实习，因为业务相关学习到了很多关于授权鉴权的知识，这里我做一个知识上的总结，扩展一下自己的知识体系。

<!--more-->

## OAuth2.0 是什么？

OAuth的英文全称是**Open Authorization**，它是一种开放授权协议，允许第三方应用程序使用资源所有者的凭据获得对资源有限访问权限。很显然 OAuth2.0 适用于第三方应用的授权。

那么什么是授权访问凭证？

**授权访问凭证**一般来说就是一个表示特定范围、生存周期和其访问权限的一个由字符串组成的**访问令牌(Access Token)**。

在这种模式下OAuth2.0协议中通过引入一个授权层来将第三方应用程序与资源拥有者进行分离，而这个授权层也就是我们常说的“auth认证服务/sso单点登录服务器”。

## 为什么需要 OAuth2.0？

在第三方应用程序在获取类似微信、微博这样的社交账号信息的时候，就会很麻烦。

- 微信和微博账号有很多敏感信息，这些信息是可以直接给第三方提供的吗？
- 如果因为未授权问题，导致大量隐私数据泄露该怎么办？
- 如果需要授权，那么第三方程序可以获取用户的账号密码也可以，但是谁来保证第三方应用程序的安全性呢？

综合以上问题，就急需一种可以保证用户信息安全的情况下，第三方应用程序还可以获取受保护资源的协议或者机制，那么这种机制就是OAuth2.0。

## OAuth2.0 主要的授权方式有哪些？它们如何能够保护你的资源安全？

OAuth2.0 主要授权方式有四种：**授权码（Authorization Code）**、**简化（Implicit）**、**密码模式（Resource Owner Password Credentials）**和**客户凭证（Client Credentials）**。

### Client Credentials && Refresh Token

在这里我首先要介绍的不是目前最严谨的授权码机制，而是最符合业务需求的授权方式-- "client_credentials" 方式，将第三方当作一个 “服务提供商” 来进行授权。

第三方服务提供商首先在开放平台中创建一个应用，然后利用 appkey，appsecret 获取到授权信息。

``` text 来自RFC6749
     +---------+                                  +---------------+
     |         |                                  |               |
     |         |>--(A)- Client Authentication --->| Authorization |
     | Client  |                                  |     Server    |
     |         |<--(B)---- Access Token ---------<|               |
     |         |                                  |               |
     +---------+                                  +---------------+

                     Figure 6: Client Credentials Flow
```
如上可获取到Access Token

但是，访问令牌有2小时或者更长时间的过期时间，如果这中间访问令牌被发现泄露，或者用户想要快速使这个令牌失效。又该怎么办呢？

这个时候就需要引入另外的一个令牌 —— 刷新令牌。**那么刷新令牌用来做什么的呢？**
``` text 刷新令牌
1.5.  Refresh Token

   Refresh tokens are credentials used to obtain access tokens.  Refresh
   tokens are issued to the client by the authorization server and are
   used to obtain a new access token when the current access token
   becomes invalid or expires, or to obtain additional access tokens
   with identical or narrower scope (access tokens may have a shorter
   lifetime and fewer permissions than authorized by the resource
   owner).  Issuing a refresh token is optional at the discretion of the
   authorization server.  If the authorization server issues a refresh
   token, it is included when issuing an access token.

   A refresh token is a string representing the authorization granted to
   the client by the resource owner.  The string is usually opaque to
   the client.  The token denotes an identifier used to retrieve the
   authorization information.  Unlike access tokens, refresh tokens are
   intended for use only with authorization servers and are never sent
   to resource servers.
```

从RFC中可以得知，刷新令牌就是用来获取访问令牌的一个凭据。访问令牌是被鉴权服务器发布到客户端的，被用来获取一个新的访问令牌，并且使得当前的令牌失效或者过期，再或者让访问令牌的权限范围降低。

与此同时需要注意，刷新令牌对于客户是不透明的，这个刷新令牌也仅仅是用来撤回授权信息，不像访问令牌可以用于访问资源，对于刷新令牌而言。因此，刷新令牌为了保证他的安全性，刷新令牌仅仅用于第三方服务商和鉴权服务器，需要被妥善保存。

**官方的文档是如此解释，那么实际上是如何实现呢？**

因为公司是to B业务，所以有资格在开放平台创建应用的客户，也往往都是订阅了公司服务的客户。客户调用公司产品的 OpenAPI 目的也是维护客户自己的应用程序，或者同步数据。

综合上面的需求，那么在设计授权服务器的时候，就可以预想到对于安全性而言，访问令牌和刷新令牌对是可以由客户层面保证的，OpenAPI的用户（客户公司的员工）也不会看到访问令牌。

为了安全性的保证，客户需要时常刷新令牌，除了访问令牌自身的过期时间保证安全，开放平台也提供了刷新令牌的机制。刷新令牌时直接将当前令牌失效，没有选择过期，或者缩小权限范围的方式。

### 授权码（Authorization Code）

授权码模式是功能最完整、流程最严密的授权模式。它的特点是通过客户端的后台服务器，与服务提供商的认证服务进行互动（如微信开放平台）。

首先用户访问豆瓣网，豆瓣网会返回给用户代理一个授权地址，用户代理跳转页面到微信授权页面，用户扫描登陆，完成验证，开放平台便会携带一个临时凭证（授权码）返回到用户代理，用户代理再次重定向请求到豆瓣网，豆瓣网便会获得一个临时凭证。这个时候利用授权码来交换访问令牌，完成整个授权过程。

{% image https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/OAuth20-授权码.png 授权码流程实例 %}

### 简化（Implicit）

简化模式不同于授权码模式，这个模式是一个简化了的授权码模式，为客户端简化流程，在浏览器中使用脚本语言来实现，例如使用JavaScript实现。

{% image https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/OAuth20-简化模式.png 简化模式流程实例 %}

简化模式的流程简化了第三方应用通过授权码来兑换访问令牌的流程，在上述流程中，开放平台返回的不是一个授权码，而是直接返回一个访问令牌，这样的简化，可以使得整个流程可以在浏览器中完成。

### 密码模式（Resource Owner Password Credentials）

密码模式就是直接将用户的账号密码交给第三方服务商提供，然后第三方利用这个账号密码去直接兑换Token信息。这种方式安全性比较低，一般不会采用。

## 参考资料
 
- [The OAuth 2.0 Authorization Framework](https://www.rfc-editor.org/rfc/rfc6749.html)
- [OAuth授权协议](https://baijiahao.baidu.com/s?id=1705608658819242537)