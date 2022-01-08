---
title: DML -- 数据操作语言
sidebar:
  - toc
date: 2020-04-22 18:21:29
tags: [MySQL]
categories: [MySQL]
---

## DML：数据操作语言

### 概述

+ **数据操作语言**
  + 插入：insert
  + 修改：update
  + 删除：delete

### 一、插入语句

#### 插入方式一：经典的插入方式

+ 语法

  ```sql
  insert into 表名(列1, ...)
  values(值1, ...);
  ```


1. 插入的值的类型要与列的类型一致或者兼容

   ```sql
   INSERT INTO beauty(id, name, sex, borndate, phone, photo, boyfriend_id)
   VALUES(13, 'AAA', '女', '1990-4-23', '18888888888',null ,2);
   ```

2. 可以为NULL的列插入值

   + 方式一：直接在对应列写NULL

     ```sql
     INSERT INTO beauty(id, name, sex, borndate, phone, photo, boyfriend_id)
     VALUES(14, 'BBB', '女', '1990-4-24', '18888888888',null ,2);
     ```

   + 方式二：直接不要有对应列

     ```sql
     INSERT INTO beauty(id, name, sex, borndate, phone)
     VALUES(15, 'CCC', '女', '1990-4-25', '18888888888');
     ```

3. 列的顺序可以调换，但是必须一一对应

   ```sql
   INSERT INTO beauty(name, sex, id, phone)
   VALUES('DDD', '女', 16, '100100');
   ```

4. 列数和值的个数必须一致

   ```sql
   -- ! 会发生错误，因为列数和值的个数必须一致
   INSERT INTO beauty(name, sex, id, phone, boyfriend_id)
   VALUES('DDD', '女', 17, '101');
   ```

5. 可以省略列名，默认为所有列，而且列的顺序和表中列的顺序时一致的

   ```sql
   INSERT INTO beauty
   VALUES(17, 'EEE', '女', NULL, '1001', NULL, '4');
   ```

#### 插入方式二：键值对方式插入

+ 语法

  ```sql
  INSERT INTO 表名
  SET 列1=值1, 列2=值2, ...;
  ```

  实例：

  ```sql
  INSERT INTO beauty
  SET id=18, name='FFF', sex='女', phone='10010';
  ```

#### 两种插入方式对比

1. 方式一支持一次性插入多行

   ```sql
   INSERT INTO beauty
   VALUES(19, 'EEA', '女', NULL, '1001', NULL, '4'),
   (20, 'EEB', '女', NULL, '1001', NULL, '4'),
   (21, 'EEC', '女', NULL, '1001', NULL, '4');
   ```

2. 方式一支持子查询，方式二不支持

   ```sql
   INSERT INTO beauty(id, name, phone)
   SELECT 24, 'GGG', '188100';
   
   INSERT INTO beauty(id, name, phone)
   SELECT id+24, boyname, '123456'
   from boys WHERE id<3;
   ```

### 二、修改语句

#### 语法

1. 修改单表中的记录(**重要**)

   ```sql
   UPDATE 表
   SET 列1=值1, 列2=值2, ...
   WHERE 筛选条件
   LIMIT 限定行数; ## sql_safe_updates选项开启时就需要注意这个问题;
   ```

2. 修改多表中的记录（补充）

   ```sql
   -- SQL92语法：
   UPDATE 表1 别名, 表2 别名, ...
   SET 列1 = 值1, 列2 = 值2, ...
   WHERE 连接条件 AND 筛选条件
   LIMIT 限制条件;
   
   -- SQL99语法：
   UPDATE 表1 别名
   INNER | LEFT/RIGHT OUTER | FULL JOIN 表2 别名
   ON 连接条件
   WHERE 筛选条件
   LIMIT 限制条件;
   ```

#### 修改单表中的记录

+ 案例1：修改bueaty表中，第一个字母是E的电话都改为13899999999

  ```sql
  UPDATE beauty
  SET phone = '13899999998'
  WHERE name like 'E%' ## 由于这里不是主键列就需要LIMIT限制
  LIMIT 10;
  ```

+ 案例2：修改boys表中id为2的名称为张飞，魅力值为10

  ```sql
  UPDATE boys
  SET boyname = '张飞', usercp = 10
  WHERE id = 2; ## 由于这里是主键列就不用LIMIT了
  ```

#### 修改多表的记录

+ 案例1：修改张无忌女朋友的电话号为114

  ```sql
  UPDATE boys bo
  INNER JOIN beauty b ON b.boyfriend_id = bo.id
  SET b.phone = '112'
  WHERE bo.boyname = '张无忌';
  ```

+ 案例2：修改没有男朋友的女生的男朋友编号都为张飞

  ```sql
  UPDATE beauty b
  LEFT OUTER JOIN boys bo ON b.boyfriend_id = bo.id
  SET b.boyfriend_id = 2
  WHERE b.id IS NULL;
  ```

### 三、删除语句

#### 语法

+ 方式一：DELETE

  > 删除的时候整条记录都被删除

  ```sql
  -- 1. 单表的删除
  DELETE FROM 表 WHERE 筛选条件;
  
  -- 2. 多表的删除
  DELETE 表1别名, 表2别名
  FROM 表1 别名
  INNER | LEFT/RIGHT OUTER | FULL JOIN 表2 别名
  ON 连接条件
  WHERE 筛选条件
  LIMIT 限制条件;
  ```

+ 方式二：TRUNCATE

  > 删除整个表的数据

  ```sql
  TRUNCATE TABLE 表;
  ```

#### 方式一：DELETE

1. 单表的删除

   + 案例：删除手机号以9结尾的女神信息

     ```sql
     DELETE FROM beauty WHERE phone LIKE '%9';
     
     DELETE FROM beauty 
     WHERE id = ANY (
     	SELECT id
         from beauty
         WHERE phone LIKE '%0'
     )
     LIMIT 10;  ## 安全模式
     ```

2. 多表的删除

   + 案例：删除张无忌女朋友的信息

     ```sql
     DELETE FROM beauty
     WHERE id = ANY (
     	SELECT b.id
         FROM beauty b
         INNER JOIN boys bo
     	ON b.boyfriend_id = bo.id
         WHERE bo.boyName = '张无忌'
     );
     ```

   + 案例：删除黄晓明的信息及其女朋友的信息

     ```sql
     -- sql_safe_updates=OFF
     DELETE b, bo
     FROM beauty b
     INNER JOIN boys bo ON b.boyfriend_id = bo.id
     WHERE bo.boyName = '黄晓明';
     -- sql_safe_updates=ON
     DELETE bo, b
     FROM boys bo
     INNER JOIN beauty b ON b.boyfriend_id = bo.id
     INNER JOIN (
     	SELECT id
         FROM boys
         WHERE boyName = '黄晓明'
     ) IDS ON true
     WHERE bo.id = IDS.id;
     
     ```

#### 方式二：TRUNCATE

+ 案例：将魅力值>100的男神信息删除(**不可能实现**，TRUNCATE只能删除整个表里面的数据)

  ```sql
  TRUNCATE TABLE boys;
  ```

#### DELETE 和 TRUNCATE 的对比[面试题]

1. DELETE 可以加 WHERE 条件; TRUNCATE 不能加

2. TRUNCATE 执行效率更高

3. 加入要删除的表中，有自增长(Auto Increase)列，如果用DELETE删除后，再插入数据，自增长的值从断电开始，而TRUNCATE删除后，再插入数据，自增长列的值从1开始

   ```sql
   -- 执行下面的表以后，可以发现，最后的值不是从1开始的
   DELETE FROM boys;
   INSERT INTO boys(boyName, usercp)
   VALUES('张飞', 100),
   ('刘备', 200),
   ('关羽', 200);
   
   -- 执行下面的表以后，最后的值从1开始
   TRUNCATE TABLE boys;
   INSERT INTO boys(boyName, usercp)
   VALUES('张飞', 100),
   ('刘备', 200),
   ('关羽', 200);
   ```

4. TRUNCATE 删除以后无返回值，DELETE 删除以后有返回值

5. TRUNCATE 删除不能回滚，DELETE删除可以回滚

### 相关测试

1. 运行以下脚本创建表my_employees

   ```sql
   USE myemployees;
   CREATE TABLE my_employees(
   	Id INT(10),
   	First_name VARCHAR(10),
   	Last_name VARCHAR(10),
   	Userid VARCHAR(10),
   	Salary DOUBLE(10,2)
   );
   CREATE TABLE users(
   	id INT,
   	userid VARCHAR(10),
   	department_id INT
   
   );
   ```

2. 显示表my_employees的结构

   ```sql
   DESC my_employees;
   ```

3. 向my_employees表中插入下列数据

   > ID	FIRST_NAME	LAST_NAME	USERID	SALARY
   > 1	patel		Ralph		Rpatel	895
   > 2	Dancs		Betty		Bdancs	860
   > 3	Biri		Ben		Bbiri	1100
   > 4	Newman		Chad		Cnewman	750
   > 5	Ropeburn	Audrey		Aropebur	1550

   ```sql
   -- 方式一：VALUES
   INSERT INTO my_employees
   VALUES(1, 'patel', 'Ralph', 'Rpatel', 895),
   (2,'Dancs','Betty','Bdancs',860),
   (3,'Biri','Ben','Bbiri',1100),
   (4,'Newman','Chad','Cnewman',750),
   (5,'Ropeburn','Audrey','Aropebur',1550);
   DELETE FROM my_employees;
   
   -- 方式二：SELECT
   INSERT INTO my_employees
   SELECT 1, 'patel', 'Ralph', 'Rpatel', 895 UNION
   SELECT 2,'Dancs','Betty','Bdancs',860 UNION
   SELECT 3,'Biri','Ben','Bbiri',1100 UNION
   SELECT 4,'Newman','Chad','Cnewman',750 UNION
   SELECT 5,'Ropeburn','Audrey','Aropebur',1550;
   ```

4. 向users表中插入数据

   > 1	Rpatel	10
   > 2	Bdancs	10
   > 3	Bbiri	20
   > 4	Cnewman	30
   > 5	Aropebur	40

   ```sql
   INSERT INTO users
   VALUES(1, 'Rpatel', 10),
   (2,'Bdancs',10),
   (3,'Bbiri',20),
   (4,'Cnewman',30),
   (5, 'Aropebur', 40);
   ```

5.  将3号员工的last_name修改为“drelxer”

   ```sql
   UPDATE my_employees
   SET Last_name = 'drelxer'
   WHERE Id = 3;
   ```

6. 将所有工资少于900的员工的工资修改为1000

   ```sql
   UPDATE my_employees
   SET Salary = 1000
   WHERE Salary < 900;
   ```

7. 将userid 为Bbiri的user表和my_employees表的记录全部删除

   ```sql
   DELETE u, m
   FROM users u
   INNER JOIN my_employees m ON u.userid = m.Userid
   WHERE u.userid = 'Bbiri';
   ```

8. 删除所有数据

   ```sql
   DELETE FROM my_employees;
   DELETE FROM users;
   ```

9. 检查所作的修正

   ```sql
   SELECT * FROM my_employees;
   SELECT * FROM users;
   ```

10. 清空表my_employees

    ```sql
    TRUNCATE TABLE my_employees;
    ```