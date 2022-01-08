---
title: DQL -- 排序查询
sidebar:
  - toc
date: 2020-04-17 17:33:09
tags: [MySQL]
categories: [MySQL]
---

## 排序查询

> 按照数据库里面的数据排序
>
> ```sql
> SELECT * FROM employees;
> ```
>
> 但是，想要按照salary工资顺序查看的时候，就需要排序

### 语法及其特点

+ 语法

  ```sql
  SELECT
  	查询列表
  FROM
  	表
  [WHERE 筛选条件]
  ORDER BY 
  	排序列表1 [desc 或 asc], 
  	排序列表2 [desc 或 asc];
  ```

+ 特点：

  1. asc代表升序，desc代表的是降序

     > 如果不写默认就是升序

  2. ORDER BY子句中可以支持**单个字段，多个字段，表达式，函数，别名** 

  3. ORDER BY子句一般是放在查询语句的最后面（LIMIT子句除外）

### 用法

+ 案例：查询员工信息，要求工资按照从低到高和从高到低排序

  ```sql
  ## 升序
  SELECT 
      *
  FROM
      employees
  ORDER BY salary ASC;
  
  ## 降序
  SELECT 
      *
  FROM
      employees
  ORDER BY salary DESC;
  ```

+ 案例2：查询部门编号大于等于90的员工信息，按照入职时间先后排序

  ```sql
  SELECT 
      *
  FROM
      employees
  WHERE
      department_id >= 90
  ORDER BY hiredate;
  ```

+ 案例3：按年薪高低显示员工的信息和年薪[**按表达式排序**]

  ```sql
  SELECT 
      *, salary * 12 * (1 + IFNULL(commission_pct, 0)) 年薪
  FROM
      employees
  ORDER BY salary * 12 * (1 + IFNULL(commission_pct, 0)) DESC;
  ```

+ 案例4：按年薪高低显示员工的信息和年薪[**按别名排序**]

  ```sql
  SELECT 
      *, 
      salary * 12 * (1 + IFNULL(commission_pct, 0)) 年薪
  FROM
      employees
  ORDER BY 年薪 DESC;
  ```

+ 案例5：按照姓名的长度显示员工的姓名和工资[**按函数排序**]

  ```sql
  SELECT 
      LENGTH(last_name) 字节长度, last_name, salary
  FROM
      employees
  ORDER BY 字节长度 DESC;
  ```

+ 案例6：查询员工信息，要求先按照工资排序，再按照员工号排序[**按多个字段排序**]

  ```sql
  SELECT 
      *
  FROM
      employees
  ORDER BY salary ASC, employee_id DESC;
  ```

### 相关测试

1. 查询员工姓名和部门号和年薪，按年薪降序，按姓名升序

   ```sql
   SELECT 
       last_name, 
       salary*12*(1+ifnull(commission_pct, 0)) 年薪
   FROM
       employees
   ORDER BY 年薪 DESC , last_name ASC;
   ```

2. 选择工资不再8000到17000员工的姓名和工资，按工资降序

   ```sql
   SELECT 
       last_name, salary
   FROM
       employees
   WHERE
       salary NOT BETWEEN 8000 AND 17000
   ORDER BY salary DESC;
   ```

3. 查询邮箱中包含e的员工信息，并先按邮箱的字节数降序，再按部门号降序

   ```sql
   SELECT 
       *
   FROM
       employees
   WHERE
       email LIKE '%e%'
   ORDER BY email DESC , department_id ASC;
   ```
