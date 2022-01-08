---
title: DQL - 分页查询（Paging Query）、联合查询（Union Query）
sidebar:
  - toc
date: 2020-04-21 10:37:22
tags: [MySQL]
categories: [MySQL]
---

## 分页查询

### 应用场景

> 当要显示的数据一页显示不全，需要分页提交sql提交

### 语法

```sql
select 查询列表
from 表
[join type] join 表2
on 连接条件
where 筛选条件
group by 分组字段
having 分组后的筛选
order by 排序的字段
## offset 要显示条目的其实索引（起始从0开始）
## size 要显示的条目数量
limit offset, size;
```

### 特点

1. limit语句放在查询语句的最后
2. **公式**
	要显示的页数是page，每页的条目数时size
    select 查询列表
    from 表
    limit (page-1)*size size;

### 相关案例

+ 案例1：查询前五条的员工信息

  ```sql
  SELECT 
      *
  FROM
      employees
  LIMIT 0 , 5;
  ## offset[Optional]
  SELECT 
      *
  FROM
      employees
  LIMIT 5;
  ```

+ 案例2：查询第11条到第25条

  ```sql
  SELECT 
      *
  FROM
      employees
  LIMIT 11, 15;
  ```

+ 案例3：有奖金的员工姓名，工资，奖金率，并且工资较高的前10名

  ```sql
  SELECT 
      last_name, salary, commission_pct
  FROM
      employees
  WHERE
      commission_pct IS NOT NULL
  ORDER BY salary DESC
  LIMIT 10;
  ```

## union联合查询

### 概念

> union 联合 合并：将多条查询的结果合并成一个结果

### 语法

```sql
查询语句1
union
查询语句2
union
...
```

### 应用场景

要查询的结果来自于多个表，且多个表没有直接的连接关系，但**查询的信息一致**

### 特点

1. 要求多条查询语句的查询列数是一致的
2. 要求多条查询语句的查询的每一列的类型和顺序最好是一致的
3. union关键字默认去重，如果要使Union关键字不去重则需要使用关键字all

### 相关案例

+ 案例：查询部门编号大于90,或者邮箱包含a的员工信息

  ```sql
  ## 普通的筛选
  SELECT 
      *
  FROM
      employees
  WHERE
      employee_id > 90 OR email LIKE '%a%';
  ## 联合查询
  SELECT 
      *
  FROM
      employees
  WHERE
      employee_id > 90 
  UNION SELECT 
      *
  FROM
      employees
  WHERE
      email LIKE '%a%';
  ```

