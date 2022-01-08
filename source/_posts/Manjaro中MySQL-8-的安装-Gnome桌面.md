---
title: Manjaro中MySQL 8 的安装 -- Gnome桌面
sidebar:
  - toc
date: 2020-04-01 15:44:02
tags: [Linux, MySQL]
categories: [Linux]
---

> 本人的系统是Manjaro，查询了一下Archwiki发现，很多linux默认的SQL数据库是MariaDB了。同时说，MySQL已经被移到了AUR库里面。
>
> 按照Oracle的德行，MySQL的创始人创建了了一个MySQL的分支，MariaDB，如果MySQL闭源，MariaDB可以立刻代替MySQL使用。因此，不难知道MariaDB是完全兼容MySQL的。
>
> 不过的话，MySQL很多项目都使用这个，最后决定安装MySQL。

## 安装MySQL

安装MySQL

```bash
sudo pacman -S mysql
```

## 安装后的一系列配置

### 查看安装日志

```
[2020-04-01T13:29:05+0800] [ALPM-SCRIPTLET] :: You need to initialize the MySQL data directory prior to starting
[2020-04-01T13:29:05+0800] [ALPM-SCRIPTLET]    the service. This can be done with mysqld --initialize command, e.g.:
[2020-04-01T13:29:05+0800] [ALPM-SCRIPTLET]    mysqld --initialize --user=mysql --basedir=/usr --datadir=/var/lib/mysql
[2020-04-01T13:29:05+0800] [ALPM-SCRIPTLET] :: Additionally you should secure your MySQL installation using
[2020-04-01T13:29:05+0800] [ALPM-SCRIPTLET]    mysql_secure_installation command after starting the mysqld service

```

看到需要初始化 MySQL

执行命令：

```bash
mysqld --initialize --user=mysql --basedir=/usr --datadir=/var/lib/mysql
```

### 初始化MySQL

```bash
mysqld --initialize --user=mysql --basedir=/usr --datadir=/var/lib/mysql
```

执行以后马上就报错了。（烦人）

```
2020-04-01T05:45:51.941236Z 0 [Warning] [MY-010915] [Server] 'NO_ZERO_DATE', 'NO_ZERO_IN_DATE' and 'ERROR_FOR_DIVISION_BY_ZERO' sql modes should be used with strict mode. They will be merged with strict mode in a future release.
2020-04-01T05:45:51.941340Z 0 [System] [MY-013169] [Server] /usr/bin/mysqld (mysqld 8.0.19) initializing of server in progress as process 75054
2020-04-01T05:45:51.943568Z 0 [ERROR] [MY-010457] [Server] --initialize specified but the data directory has files in it. Aborting.
2020-04-01T05:45:51.943574Z 0 [ERROR] [MY-013236] [Server] The designated data directory /var/lib/mysql/ is unusable. You can remove all files that the server added to it.
2020-04-01T05:45:51.943645Z 0 [ERROR] [MY-010119] [Server] Aborting
2020-04-01T05:45:51.943786Z 0 [System] [MY-010910] [Server] /usr/bin/mysqld: Shutdown complete (mysqld 8.0.19)  Source distribution.

```

查看 [ERROR] 信息以后发现

> The designated data directory /var/lib/mysql/ is unusable. You can remove all files that the server added to it.

好吧，那也只好这样了。`sudo su` 切换到root账户下面。~~`rm -rf /var/lib/mysql/*` 删除里面的文件。~~打开文件夹`cd /后面这样安全写var/lib` ，然后删除mysql里面的文件`rm -rf mysql/*`。后面这样做就算出错，也不至于把系统删了。

### 再次初始化

```bash
mysqld --initialize --user=mysql --basedir=/usr --datadir=/var/lib/mysql
```

出现日志

```
2020-04-01T05:52:22.017420Z 0 [Warning] [MY-010915] [Server] 'NO_ZERO_DATE', 'NO_ZERO_IN_DATE' and 'ERROR_FOR_DIVISION_BY_ZERO' sql modes should be used with strict mode. They will be merged with strict mode in a future release.
2020-04-01T05:52:22.017498Z 0 [System] [MY-013169] [Server] /usr/bin/mysqld (mysqld 8.0.19) initializing of server in progress as process 75837
2020-04-01T05:52:39.779289Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: s?2P***skGeq(这是临时密码)
```

发现，没有[ERROR]开心的很。

### 启动服务

```bash
systemctl start mysqld
```

没有报错以后就是安装成功了。

## 初次登录MySQL

### 重置root密码

1. 登录

   ```bash
   mysql -u root -p
   ```

   Enter password: 

   密码就是上面记录的临时密码：s?2P***skGeq

2. 修改密码

   进入MySQL的shell里面以后，执行命令

   ```mysql
   ALTER USER 'root'@'localhost' IDENTIFIED BY 'root-password';
   ```

## 安装mysql可视化界面

mysql有一个叫做workbench的可视化界面，感觉还不错。

```bash
sudo pacman -S mysql-workbench
```



**参考源：**

https://dev.mysql.com/doc/refman/8.0/en/installing.html
