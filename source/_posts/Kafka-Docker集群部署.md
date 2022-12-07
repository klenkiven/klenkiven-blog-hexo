---
title: Kafka-Docker集群部署简单版本
sidebar:
  - toc
date: 2022-12-07 10:43:49
tags: [Docker,Kafka]
categories: [Docker]
---

Kafka 通过 Docker 和 Docker-Compose 集群部署。编写 Dockerfile 构建镜像，编写 docker-compose.yml 编排部署。

<!--more-->

## Kafka 镜像构建

使用 Docker 生成 kafka 镜像，需要编写 Dockerfile 文件，生成镜像。

1. 获取 Kafka 最新的可执行文件

```bash
curl https://dlcdn.apache.org/kafka/3.3.1/kafka_2.13-3.3.1.tgz --output kafka_2.13-3.3.1.tgz
```

2. 编写启动脚本

```shell
#!/bin/bash
bin/kafka-server-start.sh config/server.properties & 

# 启动命令行bash 防止因为任务结束使得容器结束运行
/bin/bash
```

3. 编写 Dockerfile 文件

```Dockerfile
FROM openjdk:11

################################
#        Config Debain         #
################################
# 安装必要依赖
RUN sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list && \
    apt update && apt install -y openssh-server openssh-client sudo iproute2 vim

# 生成 Kafka 用户
ARG KAFKA_PASSWD=$y$j9T$1J89vjVg17jGFjLEAZLyS/$kTVAaMlWfATsYnhhaBvAgL5XscUiSbc7MB0EwPeLUE9
RUN useradd -m -N -s /bin/bash -u 1000 -p '${KAFKA_PASSWD}' kafka && \
    usermod -aG sudo kafka; \
    /etc/init.d/ssh start

################################
#        Install Kafka         #
################################

ENV KAFKA_BROKER_ID 0
ENV KAFKA_ZOOKEEPER_CONNECT localhost:2181

WORKDIR /opt/kafka
VOLUME [ "/opt/kafka/data" ]
# ADD 在添加压缩包的时候，自动解压
ADD 'kafka_2.13-3.3.1.tgz' '/opt/kafka/'
ADD 'start.sh' '/opt/kafka/start.sh'
RUN chmod +x /opt/kafka/start.sh; \
    mkdir -p /opt/kafka/datas && mv kafka_2.13-3.3.1/* /opt/kafka/; \
    sed -i 's/broker.id=0/broker.id='"${KAFKA_BROKER_ID}"'/1' config/server.properties; \
    sed -i 's#log.dirs=/tmp/kafka-logs#log.dirs=/opt/kafka/data#1' config/server.properties; \
    sed -i 's/zookeeper.connect=localhost:2181/zookeeper.connect='"${KAFKA_ZOOKEEPER_CONNECT}"'/1' config/server.properties; \
    mkdir logs; touch logs/zookeeper.log

CMD /opt/kafka/start.sh
```

4. 执行指令 `docker build`

```bash
sudo docker build -t klenkiven/kafka:3.3.1 .
```

## 配置集群需要的网络环境

| Hostname |    网络IP    |   端口    |
| :------: | :----------: | :-------: |
|   zk01   | 172.20.10.1  | 2183:2181 |
|   zk02   | 172.20.10.2  | 2184:2181 |
|   zk03   | 172.20.10.3  | 2185:2181 |
| kafka01  | 172.20.10.11 | 9092:9092 |
| kafka02  | 172.20.10.12 | 9092:9092 |
| kafka03  | 172.20.10.13 | 9092:9092 |

再次之前需要创建网络 `kafka-net`。

```bash
sudo docker network create kafka-net --subnet 172.20.10.0/16
```

1. 编写 `docker-compose.yml`

```yaml
version: '3.3'
services:
  # Zookeeper Cluster - zk01,zk02,zk03
  zookeeper01:
    image: zookeeper:latest
    restart: always
    container_name: zookeeper01
    hostname: zk01
    ports:
      - 2183:2181
    volumes:
      - ~/docker/data/zookeeper1/data:/data
      - ~/docker/data/zookeeper1/datalog:/datalog
      - ~/docker/data/zookeeper1/logs:/logs
    environment:
        ZOO_MY_ID: 1  #即是zookeeper的节点值，也是kafka的brokerid值
        ZOO_SERVERS: server.1=zk01:2888:3888;2181 server.2=zk02:2888:3888;2181 server.3=zk03:2888:3888;2181
    networks:
        kafka-net:
            ipv4_address: 172.20.10.1
  
  zookeeper02:
    image: zookeeper:latest
    restart: always
    container_name: zookeeper02
    hostname: zk02
    ports:
      - 2184:2181
    volumes:
      - ~/docker/data/zookeeper2/data:/data
      - ~/docker/data/zookeeper2/datalog:/datalog
      - ~/docker/data/zookeeper2/logs:/logs
    environment:
        ZOO_MY_ID: 2  #即是zookeeper的节点值，也是kafka的brokerid值
        ZOO_SERVERS: server.1=zk01:2888:3888;2181 server.2=zk02:2888:3888;2181 server.3=zk03:2888:3888;2181
    networks:
        kafka-net:
            ipv4_address: 172.20.10.2
  
  zookeeper03:
    image: zookeeper:latest
    restart: always
    container_name: zookeeper03
    hostname: zk03
    ports:
      - 2185:2181
    volumes:
      - ~/docker/data/zookeeper3/data:/data
      - ~/docker/data/zookeeper3/datalog:/datalog
      - ~/docker/data/zookeeper3/logs:/logs
    environment:
        ZOO_MY_ID: 3  #即是zookeeper的节点值，也是kafka的brokerid值
        ZOO_SERVERS: server.1=zk01:2888:3888;2181 server.2=zk02:2888:3888;2181 server.3=zk03:2888:3888;2181
    networks:
        kafka-net:
            ipv4_address: 172.20.10.3

  # Kafka Cluster - kafka01,kafka02,kafka03
  kafka-01:
    image: klenkiven/kafka:3.3.1
    restart: always
    hostname: kafka01
    container_name: kafka01
    ports: 
      - 9092:9092
    environment:
      KAFKA_BROKER_ID: 0
      KAFKA_ZOOKEEPER_CONNECT: zk01:2181,zk02:2181,zk03:2181
    volumes:
      - /home/klenkiven/docker/kafka01/data:/opt/kafka/data
    links:
      - zookeeper01:zk01
      - zookeeper02:zk02
      - zookeeper03:zk03
    networks:
      kafka-net:
        ipv4_address: 172.20.10.11

  kafka-02:
    image: klenkiven/kafka:3.3.1
    restart: always
    hostname: kafka02
    container_name: kafka02
    ports: 
      - 9093:9092
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zk01:2181,zk02:2181,zk03:2181
    volumes:
      - /home/klenkiven/docker/kafka02/data:/opt/kafka/data
    links:
      - zookeeper01:zk01
      - zookeeper02:zk02
      - zookeeper03:zk03
    networks:
      kafka-net:
        ipv4_address: 172.20.10.12
  
  kafka-03:
    image: klenkiven/kafka:3.3.1
    restart: always
    hostname: kafka03
    container_name: kafka03
    ports: 
      - 9094:9092
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: zk01:2181,zk02:2181,zk03:2181
    volumes:
      - /home/klenkiven/docker/kafka03/data:/opt/kafka/data
    links:
      - zookeeper01:zk01
      - zookeeper02:zk02
      - zookeeper03:zk03
    networks:
      kafka-net:
        ipv4_address: 172.20.10.13

networks:
  kafka-net:
    external: true
    driver: kafka-net # 网络驱动 选择已经创建好的 kafka-net
```

2. 执行指令 `docker-compose` 启动集群

```bash
sudo docker-compose -p kafka-zk-cluster -f docker-compose.yml up -d
```

3. 如果需要关闭Docker容器并且删除Docker容器，只需要执行下面的指令就可以了

```bash
sudo docker stop kafka01 kafka02 kafka03 zookeeper01 zookeeper02 zookeeper03
sudo docker rm kafka01 kafka02 kafka03 zookeeper01 zookeeper02 zookeeper03
```