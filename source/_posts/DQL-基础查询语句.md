---
title: DQL -- 基础查询语句
sidebar:
  - toc
date: 2020-04-17 17:16:09
tags: [MySQL]
categories: [MySQL]
---

## 基础查询

### 语法及其特点

+ 语法

  ```sql
  select 查询列表
  from 表名;
  ```

   类似于，System.out.println();
  
+ 特点：

  1. 查询列表可以是：表中的字段，常量值，表达式，函数。
  2. 查询的结果是一个虚拟的表格

### 用法

0. 使用某个数据库的时候，需要使用USE

   ```sql
   USE myemployees;
   ```

1. 查询表中的**单个字段**

   ```sql
   SELECT first_name FROM employees;
   ```

2. 查询表中的**多个字段** 

   ```sql
   SELECT last_name, salary, email FROM employees;
   ```

3. 查询表中的**所有字段**

   1. 方法一：列举出所有的列，缺点就是比较繁琐

       ```sql
       SELECT
        employee_id,
           first_name,
           last_name,
           email,
           phone_number,
           job_id,
           salary,
           commission_pct,
           manager_id,
           department_id,
           hiredate
       FROM
        employees;
       ```

   2. 方法二：使用通配符，缺点是不够灵活

       ```sql
       SELECT * FROM employees;
       ```

4. 查询**常量值**

   ```sql
   SELECT 100;
   SELECT 'join';
   ```

5. 查询**函数**

   ```sql
   SELECT VERSION();  ## 查看但前MySQL的版本
   ```

6. **起别名**

   1. 方法一：AS
   
      ```sql
      SELECT 100%98 AS result;
      SELECT 
      	last_name AS 姓, 
          first_name AS 名 
      FROM employees;
      ```
   
   2. 方法二：使用空格
   
      ```sql
      SELECT 
      	last_name 姓, 
          first_name 名 
      FROM employees;
      ```
   
   3. 案例：查询salary，显示为 "out put"
   
      > 别名有特殊字符，在MySQL中建议使用双引号，把别名引起来
   
      ```sql
      SELECT salary AS "out put" FROM employees;
      ```
   
7. **去重**：关键字 DISTINCT

   + 案例：查询员工表中涉及到的所有的部门编号

     ```sql
     SELECT department_id FROM employees; ## 不去重版本
     SELECT DISTINCT department_id FROM employees;
     ```

8. 在MySQL中**加号**的作用

   + java中的加号：

     1. **运算符**，两个操作数都为数值型
     2. **连接符**，只要油一个操作数为字符串

   + MySQL中的加号只作为运算符：

     1. 两者都是数值型，则直接相加 
     2. 其中一个是字符型的，那么试图酱字符型数值转换成数值型
        + 如果转换成功，则继续做加法运算;
        + 如果转换失败，则将字符型转换成0，在做加法运算;
     3. 如果有一方是NULL，则结果肯定为NULL

   + 案例：查询员工名和姓名连接成一个字段，并显示为 姓名

     ```sql
     SELECT 
     	CONCAT(last_name, ' ', first_name) AS 姓名
     FROM employees;
     ```

     > CONCAT()函数是用于凭借字符串的函数

### 相关测试

1. 示表departments的结构，并查询其中的全部数据 

   ```sql
   DESC departments;
   SELECT * FROM departments;
   ```

2. 显示表employees中全部的job_id（不能重复）

   ```sql
   SELECT DISTINCT job_id FROM employees;
   ```

3. 显示出表employees的部分列，各个列之间用逗号连接，列头显示成OUT_PUT

   ```sql
   SELECT 
   	CONCAT(employee_id, ', ', 
   			first_name, ', ', 
               last_name, '') AS OUT_PUT
   FROM employees;
   ```
