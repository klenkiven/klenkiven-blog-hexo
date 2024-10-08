---
title: '[HOMELAB] Docker 镜像仓库安装部署指南'
sidebar:
  - toc
date: 2024-09-28 16:50:34
tags: [Nexus, Repository, HomeLab]
categories: [Nexus私服]
---

## Harbor

### 下载Harbor

### 安装 Harbor

## Nexus 3

### 拉取 Nexus 3 镜像

直接拉取镜像即可，至于镜像源可以自己找
```bash
docker pull sonatype/nexus3:3.72.0-ubi
```

### 部署 Nexus 3

首先需要自己创建一个目录用于存储Nexus数据
```bash
# 创建 Nexus 数据目录
mkdir -p ~/docker/nexus-data && chown -R 200 ~/docker/nexus-data

# 创建 Docker 容器，并执行
docker run -d -p 8081:8081 --name nexus \
    -v /home/klenkiven/docker/nexus-data:/nexus-data \
    sonatype/nexus3:3.72.0-ubi
```

观察 Nexus 3 是否启动成功了

```bash
# 查看 Nexus 日志
docker logs -f nexus
```

如果观察到下面的日志就可以判断为启动成功了
```
2024-09-28 15:10:18,562+0000 INFO  [jetty-main-1] *SYSTEM org.sonatype.nexus.siesta.SiestaServlet - Initialized
2024-09-28 15:10:18,564+0000 INFO  [jetty-main-1] *SYSTEM org.sonatype.nexus.repository.httpbridge.internal.ViewServlet - Initialized
2024-09-28 15:10:18,576+0000 INFO  [jetty-main-1] *SYSTEM org.eclipse.jetty.server.handler.ContextHandler - Started o.e.j.w.WebAppContext@444974c3{Sonatype Nexus,/,null,AVAILABLE}
2024-09-28 15:10:18,594+0000 INFO  [jetty-main-1] *SYSTEM org.eclipse.jetty.server.AbstractConnector - Started ServerConnector@6bf9b290{HTTP/1.1, (http/1.1)}{0.0.0.0:8081}
2024-09-28 15:10:18,595+0000 INFO  [jetty-main-1] *SYSTEM org.eclipse.jetty.server.Server - Started @18179ms
2024-09-28 15:10:18,595+0000 INFO  [jetty-main-1] *SYSTEM org.sonatype.nexus.bootstrap.jetty.JettyServer - 
-------------------------------------------------

Started Sonatype Nexus OSS 3.72.0-04

-------------------------------------------------
```

### 创建一个 Docker 镜像仓库

可以参考[教程](https://wiki.eryajf.net/pages/1816.html)创建

