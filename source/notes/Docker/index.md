---
robots: noindex,nofollow
sitemap: false
menu_id: notes
layout: wiki
wiki: 便笺
seo_title: 🐋Docker常用命令
order: 301
---

{% noteblock %}
Docker在使用方面非常方便，一些常用的配置记录下来，防止后面忘了
{% endnoteblock %}

## Nacos注册中心的Docker配置

### Nacos安装

```bash 拉取镜像 并 创建对应目录
sudo docker pull nacos/nacos-server
mkdir -p ~/docker/nacos/logs/#新建logs目录
mkdir -p ~/docker/nacos/init.d/  
```

### 配置容器

```bash
sudo docker  run \
--name nacos -d \
-p 8848:8848 \
--privileged=true \
--restart=always \
-e JVM_XMS=256m \
-e JVM_XMX=256m \
-e MODE=standalone \
-e PREFER_HOST_MODE=hostname \
-v ~/docker/nacos/logs:/data/nacos/logs \
nacos/nacos-server
```

## Ngnix的常用配置

+ 随便启动一个nginx实例，只是为了复制出配置

  ```bash
  sudo docker run -p 80:80 --name nginx -d --privileged=true nginx
  ```
+ 将容器内的配置文件拷贝到 `~/docker/nginx/conf/`  下

  ```bash
  mkdir -p ~/docker/nginx/html
  mkdir -p ~/docker/nginx/logs
  mkdir -p ~/docker/nginx/conf
  mkdir -p ~/docker/ngnix/conf/nginx
  chown root ~/docker/ngnix/conf/ngnix
  ## 由于所有者不一样，即便是ROOT也没有权限创建文件目录，因此在此之前需要转让权限
  docker container cp nginx:/etc/nginx/*  ~/docker/nginx/conf/ 
  #由于拷贝完成后会在config中存在一个nginx文件夹，所以需要将它的内容移动到conf中
  mv ~/docker/nginx/conf/nginx/* ~/docker/nginx/conf/
  rm -rf ~/docker/nginx/conf/nginx
  ```
+ 删除原有容器，创建新容器

  ```bash
  sudo docker run -p 80:80 --name nginx \
   -v ~/docker/nginx/html:/usr/share/nginx/html \
   -v ~/docker/nginx/logs:/var/log/nginx \
   -v ~/docker/nginx/conf/:/etc/nginx \
   --restart=always \
   -d nginx
  ```

## MySQL的Docker配置

### 简单部署版本

```bash
sudo docker run --name mysql \
-p 3306:3306 \
--restart=always \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql
```

### 绑定数据部署版本

```bash
sudo docker run --name mysql \
-p 3306:3306 \
--restart=always \
-v ~/docker/mysql/log:/var/log/mysql \
-v ~/docker/mysql/data:/var/ib/mysql \
-v ~/docker/mysql/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql
```

## ElasticSearch的Docker配置

### 下载镜像文件

```bash
sudo docker pull elasticsearch:7.14.2
```

### 配置

```bash
mkdir -p ~/docker/elasticsearch/config
mkdir -p ~/docker/elasticsearch/data
mkdir -p ~/docker/elasticsearch/plugins
echo "http.host: 0.0.0.0" > ~/docker/elasticsearch/config/elasticsearch.yml
```

### 创建实例

```bash
sudo docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
-e  "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms64m -Xmx512m" \
-v ~/docker/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v ~/docker/elasticsearch/data:/usr/share/elasticsearch/data \
-v ~/docker/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
--restart=always \
-d elasticsearch:7.14.2
```

### 安装kibana

1. 下载镜像

   ```bash
   sudo docker pull kibana:7.14.2
   ```
2. 创建实例

   ```bash
   ## 这里的IP地址使用Docker的网卡
   sudo docker run --name kibana \
   -e ELASTICSEARCH_HOSTS=http://172.17.0.1:9200 \
   -p 5601:5601 \
   --restart=always \
   -d kibana:7.14.2
   ```

## Redis的Docker容器配置

### Docker 安装命令

```bash
mkdir -p ~/docker/redis/conf
touch ~/docker/redis/conf/redis.conf
vim ~/docker/redis/conf/redis.conf
添加：appendonly yes

sudo docker run -p 6379:6379 --name redis \
--restart=always \
-v ~/docker/redis/data:/data \
-v ~/docker/redis/conf/redis.conf:/etc/redis/redis.conf \
-d redis redis-server /etc/redis/redis.conf
```

## RabbitMQ容器安装配置

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

访问：http://localhost:15672/