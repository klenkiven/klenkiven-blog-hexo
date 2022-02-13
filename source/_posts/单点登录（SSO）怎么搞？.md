---
title: 单点登录（SSO）怎么搞？
sidebar:
  - toc
date: 2022-02-11 20:43:39
tags:
categories:
---

使用Cookie/Session技术可以记录一个用户登陆过此系统。如果一个企业有多个系统（一级域名相同），那么就可以使用前面一篇post说过的分布式Session来解决这个问题。但是，如果这多个系统之间一级域名都不相同呢？分布式Session没办法解决这种问题，一个个登录对于用户来说又非常繁琐，用户体验差。所以，需要一种**一处登录，多个系统**就能同时登陆的解决方案，这样的解决方案就是 —— 单点登录（SingleSignOn，SSO）。

<!-- more -->

## 单点登录的主要流程

{% image https://apereo.github.io/cas/6.4.x/images/cas_flow_diagram.png download:true %}

## 参考文献

1. [Apereo CAS] [CAS Protocol](https://apereo.github.io/cas/6.4.x/protocol/CAS-Protocol.html)
