---
title: DQL -- 连接查询（Join Query）
sidebar:
  - toc
date: 2020-04-17 17:43:55
tags: [MySQL]
categories: [MySQL]
---

## 概述

+ 含义：称为多表查询，当查询的字段来自于多个表时，就会用到连接查询
+ 分类：
  + 按照年代分类
    + sql92标准：仅仅支持内连接
    + sql99标准[**推荐**]：支持内连接，外连接（左，右外连接），交叉连接
  + 按照功能分类
    + 内连接
      + 等值连接
      + 非等值连接
      + 自连接
    + 外连接
      + 左外连接
      + 右外连接
      + 全外连接
    + 交叉连接

## 引入：两张表之间同时查询的匹配问题

```sql
## 非连接查询
SELECT 
    name, boyName
FROM
    beauty,
    boys;
## 连接查询
SELECT 
    name, boyName
FROM
    boys,
    beauty
WHERE
    beauty.boyfriend_id = boys.id;
```

+ 问题：相当于，第一张表的一行和另一张表的每一行匹配

  > 笛卡尔集的错误情况：
  > `select count(*) from table_A;`有12行
  > `select count(*) from table_B;`有4行
  > 那么，
  > `select A, B from table_A, table_B; `有12×4=48行

  + 发生原因：没有有效的连接条件
  + 如何避免：添加有效的连接条件

## 一、sql92标准 -- 内连接

### 等值连接

> 1. 多表等值连接的结果外多表的交集部分
> 2. n表连接，至少需要n-1个连接条件
> 3. 一般要为表起别名
> 4. 可以搭配所有的子句使用，比如，排序，分组，筛选

+ 案例1：查询女神名和对应的男神名

  ```sql
  USE girls;  ## 这里的数据库在MYSQL概述的最后面
  SELECT 
      name, boyName
  FROM
      boys,
      beauty
  WHERE
      beauty.boyfriend_id = boys.id;
  ```

+ 案例2：查询员工名和对应的部门名

  ```sql
  USE myemployees;
  SELECT 
      last_name, department_name
  FROM
      employees,
      departments
  WHERE
      employees.department_id = departments.department_id;
  ```

#### 为表起别名

> 起别名：
> 1. 提高语句的简洁度
> 2. 别名用于区分多个重名字段
>
> 注意！
> 如果为表起了别名，则产寻的字段不能用原来的表名去限定

+ 案例：查询员工名，工种号，工种名

  ```sql
  SELECT 
      last_name, e.job_id, job_title
  FROM
  	## 两个表的顺序可以调换
      employees AS e,
      jobs j
  WHERE
      e.job_id = j.job_id;
  ```

#### 添加筛选

+ 案例1：查询有奖金的员工名和部门名

    ```sql
    SELECT 
        last_name, department_name, commission_pct
    FROM
        employees e,
        departments d
    WHERE
        e.department_id = d.department_id
            AND e.commission_pct IS NOT NULL;
    ```

+ 案例2：查询城市中第二个字符为'o'的部门名和城市名

  ```sql
  SELECT 
      department_name, city
  FROM
      departments d,
      locations l
  WHERE
      d.location_id = l.location_id
          AND city LIKE '_o%';
  ```

#### 添加分组

+ 案例1：查询每个城市的部门个数

  ```sql
  SELECT 
      city, COUNT(*)
  FROM
      locations l,
      departments d
  WHERE
      d.location_id = l.location_id
  GROUP BY city;
  ```

+ 案例2：查询有奖金的每个部门的部门名和部门的领导编号和该部门的最低工资 

  ```sql
  SELECT 
      department_name, d.manager_id, MIN(salary)
  FROM
      employees e,
      departments d
  WHERE
      d.department_id = e.department_id
          AND commission_pct IS NOT NULL
  GROUP BY department_name, manager_id;
  ```

#### 添加排序

+ 案例：查询每个工种的工种名和员工的个数，并且按员工个数降序排序

  ```sql
  SELECT 
      job_title, COUNT(*) STUF_COUNT
  FROM
      employees e,
      jobs j
  WHERE
      e.job_id = j.job_id
  GROUP BY job_title
  ORDER BY STUF_COUNT DESC;
  ```

#### 实现三表连接

+ 案例：查询员工名，部门名，所在的城市

  ```sql
  SELECT 
      last_name, department_name, city
  FROM
      departments d,
      employees e,
      locations l
  WHERE
      e.department_id = d.department_id
          AND d.location_id = l.location_id;
  ```

### 非等值连接

+ 案例：查询员工的工资和工资级别

  ```sql
  SELECT 
      grade_level, salary
  FROM
      employees e,
      job_grades j
  WHERE
      lowest_sal <= salary
          AND salary >= highest_sal
  ORDER BY grade_level;
  ```

### 自连接

+ 案例：查询员工名和上级的名称

  ```sql
  SELECT 
      e.employee_id, e.last_name, m.employee_id, m.last_name
  FROM
  	## 利用别名，把一张表当作两张表来使用
      employees e,
      employees m
  WHERE
      e.manager_id = m.employee_id;
  ```

## 二、sql99标准 -- 外连接，交叉连接

+ 语法

  ```sql
  SELECT 查询列表
  FROM 表1 别名 [连接类型]
  ## join和where分离 实现了
  JOIN 表2 别名
  ON 连接条件
  [WHERE 筛选条件]
  [GROUP BY 分组]
  [HAVING 筛选条件]
  [ORDER BY 排序列表]
  ```

+ 分类

  内连接：INNER
  外连接：
  	左外：LEFT
      右外：RIGHT
      全外：FULL
  交叉连接：CORSS

### 一）内连接

+ 语法

  ```sql
  SELECT 查询列表
  FROM 表1 别名
  INNER JOIN 表2 别名
  ON 连接条件;
  ```

+ 特点
  1. 添加排序，分组，筛选
  2. inner可以省略
  3. 筛选条件放在where后面，连接条件，放在on后米那，提高分离性，方便阅读
  4. inner join连接和sql92语法中的等值连接效果是一样的，都是产寻多表的交集

#### 等值连接

+ 案例1：查询员工名，部门名

  ```sql
  SELECT 
      last_name, department_name
  FROM
      employees e
          INNER JOIN
      departments d ON e.department_id = d.department_id;
  ## 两个表交换位置
  SELECT 
      last_name, department_name
  FROM
      departments d
          INNER JOIN 
      employees e ON e.department_id = d.department_id;
  ```

+ 案例2： 查询名字中包含e的员工名和工种名（添加筛选）

  ```sql
  SELECT 
      last_name， job_title
  FROM
      jobs j
          INNER JOIN
      employees e ON j.job_id = e.job_id
  WHERE
      last_name LIKE '%e%';
  ```

+ 案例3：查询部门个数为大于3的城市名和部门个数（添加分组和筛选）

  ```sql
  SELECT 
      city, COUNT(*) COUNT
  FROM
      departments d
          INNER JOIN
      locations l ON d.location_id = l.location_id
  GROUP BY city
  HAVING COUNT > 3;
  ```

+ 案例4：查询那个部门的部门员工个数>3的部门名个员工个数，并按个数进行降序（添加排序）

  ```sql
  SELECT 
      department_name, COUNT(*) COUNT
  FROM
      employees e
          INNER JOIN
      departments d ON e.department_id = d.department_id
  GROUP BY department_name
  HAVING COUNT > 3
  ORDER BY COUNT DESC;
  ```

+ 案例5：查询员工名，部门名，工种名（三表连接）

  ```sql
  SELECT 
      last_name, department_name, job_title
  FROM
      employees e
          INNER JOIN
      departments d ON e.department_id = d.department_id
          INNER JOIN
      jobs j ON e.job_id = j.job_id
  ORDER BY department_name DESC;
  ```

#### 非等值连接

+ 案例1：查询员工的工资级别

  ```sql
  SELECT 
      salary, grade_level
  FROM
      employees e
          INNER JOIN
      job_grades j ON e.salary BETWEEN j.lowest_sal AND j.highest_sal;
  ```

+ 案例2：查询每个工资级别的个数大于20的，并且按工资级别降序排序

  ```sql
  SELECT 
      grade_level, COUNT(*) COUNT
  FROM
      employees e
          INNER JOIN
      job_grades j ON e.salary BETWEEN j.lowest_sal AND j.highest_sal
  GROUP BY grade_level
  HAVING COUNT > 20
  ORDER BY grade_level DESC;
  ```

#### 自连接

+ 案例：查询包含字符k的员工的名字和上级的名字

  ```sql
  SELECT 
      e.last_name, m.last_name
  FROM
      employees e
          INNER JOIN
      employees m ON m.employee_id = e.manager_id
  WHERE
      e.last_name LIKE '%k%';
  ```

### 二）外连接

+ 应用场景：用于查询一个表中有，另一个表中没有的记录
+ 特点
  1. 外连接的查询结果为主表中的所有记录
  	如果从表中有和它匹配的，就显示匹配的值
      如果从表中没有和它匹配的，则显示null
      外连接查询结果=内连接结果+主表中有而从表没有的记录
  2. 左外连接：left左边的是主表
  3. 右外连接：right右边的是主表
  4. 左外和右外交换两个表的，可以实现同样的效果
  4. 全外连接 = 内链接结果 + 表1中有但是表2没有的 + 表2中有但表1中没有的

#### 引入：查询没有男朋友不在男生表里面的女神名

```sql
## 左外连接
SELECT 
    b.name, bo.*
FROM
    beauty b
        LEFT OUTER JOIN
    boys bo ON b.boyfriend_id = bo.id
WHERE
    bo.id IS NULL;
## 右外连接
SELECT 
    b.*, bo.*
FROM
    boys bo
        RIGHT OUTER JOIN
    beauty b ON b.boyfriend_id = bo.id
WHERE
    bo.id IS NULL;
```

#### 左右外连接

+ 案例1：查询那个部门没有员工

  ```sql
  ## 左外连接
  SELECT 
      d.*, e.employee_id
  FROM
      departments d
          LEFT OUTER JOIN
      employees e ON d.department_id = e.department_id
  WHERE
      e.employee_id IS NULL;
  ## 右外连接
  SELECT 
      d.*, e.employee_id
  FROM
      employees e
          RIGHT OUTER JOIN
      departments d ON e.department_id = d.department_id
  WHERE
      e.employee_id IS NULL;
  ```

#### 全外连接

> MySQL和MariaDB并不支持全连接，但仍然可以同过左外连接+ union+右外连接实现

### 三）交叉连接

> 其实本质上就是笛卡尔乘积

```sql
SELECT 
    b.*, bo.*
FROM
    beauty b
        CROSS JOIN
    boys bo;

```

## 连接查询总结

### 1. 各种连接的图示

![SQL-DQL-2](https://gitee.com/klenkiven/Blogs/raw/master/img/SQL-DDL-2.png)

![SQL-DQL-1](https://gitee.com/klenkiven/Blogs/raw/master/img/SQL-DDL-1.png)

### 2. sql92和sql99对比

+ 功能：sql99支持的功能较多
+ 可读性：sql99实现连接调价娜和筛选条件的分离，可读性较高

## 相关测试

1. 查询编号>3的女神的男朋友信息，如果有则列出详细，如果没有用null填充

   ```sql
   SELECT 
       b.id, b.name, bo.*
   FROM
       beauty b
           LEFT OUTER JOIN
       boys bo ON b.boyfriend_id = bo.id
   WHERE
       b.id > 3;
   ```

2. 查询那个城市没有部门

   ```sql
   use myemployees;
   SELECT 
       l.city, COUNT(d.department_id) COUNT_D
   FROM
       locations l
           LEFT OUTER JOIN
       departments d ON d.location_id = l.location_id
   WHERE
       d.location_id IS NULL
   GROUP BY l.city;
   ```

3. 查询部门名为SAL或的员工信息

   ```sql
   SELECT 
       e.*
   FROM
       departments d
           INNER JOIN
       employees e ON e.department_id = d.department_id
   WHERE
       d.department_name IN ('SAL' , 'IT');
   ```
