---
title: MySQL数据库事务是个啥？
sidebar:
  - toc
date: 2022-02-18 15:51:22
tags: [MySQL]
categories: [MySQL]
---

MySQL数据库在5.7版本以后支持事务，事务的ACID的属性，要求数据库保证原子性（Atomic），一致性（Consistency），隔离性（Isolation）和持久性（Duration）。

MySQL数据库中，什么是事务，MySQL数据库的事务基本的ACID属性

<!--more-->

## 什么是事务？

所谓事务，就是处理数据的一组逻辑单元，使得数据从一个状态转换到另一个状态。

事务的处理原则：这一组逻辑单元要么执行成功并提交（`commit`）持久化到磁盘中；要么失败，放弃所有更改，并回退/事务回滚（`rollback`）到之前的状态，这一组逻辑单元不可分割。

## 事务的酸不拉几属性（ACID）

事务酸不拉几的属性主要包括：原子性、一致性、隔离性、持久性。这些性质都有各自保证的措施隔离性主要由`锁机制`和`MVCC`来保证，持久性主要由数据库的`日志系统`来保证

### 原子性（Atomic）

原子性意义就是，要么事务操作全部提交，要不全部回滚，不存在部分提交也不存在部分回滚，这样的事务特性称之为`原子性`。

### 一致性（Consistency）

一致性是指事务执行前后，数据从一个`合法性状态` 变换到另外一个 `合法性状态` 。也可以理解，执行事务前后，数据保持一致。例如转账业务中，无论事务是否成功，转账者和收款人的总额应该是不变的。

### 隔离性（Isolation）

并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的。

### 持久性（Duration）

 一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。

## 事务的状态

- 活动（Active）：事务中的SQL正在执行，该事务就是活动状态
- 部分提交（Partially Committed）：事务执行结束，但是还没有存储到磁盘中。
- 失败（Failed）：在活动或者部分提交状态，事务发生了错误，则事务进入失败状态。
- 终止（Aborted）：事务回滚完成后，事务到达终止状态。
- 已提交（Commited）：已经将事务提交，数据持久化到磁盘中。

状态转换图：

{% image https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/img/MySQL-事务状态转换.png 事务各种状态转换 %}

## 事务的分类（事务怎么用？）

说了这么多，事务特性也讲到了，事务具体如何使用，是一个很现实的问题。事务具体有两种用法，一种是显式事务，一种是隐式事务。

### 显式事务

所谓显式事务，也就是使用`BEGIN`或者`START TRANSACTION`开始，使用`COMMIT`或者`ROLLBACK`结束事务的一组SQL。

- `START TRANSACTION`
  
  后面可以跟随几个修饰符：
  - `READ ONLY`：表示只读事务
  - `READ WRITE`：表示读写事务
  - `WITH CONSISTENT SNAPSHOT`：启动一致性读

### 隐式事务

其实在操作MySQL数据库的时候（尤其是使用DDL语句），很明显是需要一些事务场景的，但是上面显式事务提到的语句，似乎很少使用。这究竟是谁做到的？

{% image https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/img/MySQL-autocommit.png 数据库自动事务提交 %}

MySQL存在自动提交功能，开启这个变量，MySQL就会自动提交。当然如果想要关闭这种功能可以使用下面的SQL。

```SQL 关闭自动提交
SET autocommit = OFF;
-- 或
SET autocommit = 0;
```

## 案例分析

**情况1：**事务失败，回滚到的位置
   
```SQL BEGIN事务提交
CREATE TABLE user(name varchar(20), PRIMARY KEY (name)) ENGINE=InnoDB;
   
BEGIN; 
INSERT INTO user SELECT '张三'; 
COMMIT; 

BEGIN; 
INSERT INTO user SELECT '李四'; 
INSERT INTO user SELECT '李四'; 
ROLLBACK; 

SELECT * FROM user;
```
上面的结果为：只有张三一条记录。上面的插入操作一共有两个事务。
  
因为插入张三成功，事务完成；开启第二个事务，插入李四成功，第二条失败，根据上面的事务状态转换，此事务进入失败状态，随后出现了`ROLLBACK`事务将会回滚到，上一次事务成功的状态。

**情况2：**两条李四语句不在`BEGIN`之后

```SQL 情况2
CREATE TABLE user (name varchar(20), PRIMARY KEY (name)) ENGINE=InnoDB; 

BEGIN; 
INSERT INTO user SELECT '张三'; 
COMMIT; 

INSERT INTO user SELECT '李四'; 
INSERT INTO user SELECT '李四'; 
ROLLBACK;

SELECT * FROM user;
```
上面的结果为：有张三，李四两条记录。上面的插入操作一共有三个事务。

第一个事务成功，第二个事务成功，三个事务失败，`ROLLBACK`将会回滚到上次成功提交的事务的状态。所以，会有张三和李四两条记录。
