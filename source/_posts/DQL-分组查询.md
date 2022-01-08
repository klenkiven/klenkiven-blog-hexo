---
title: DQL -- 分组查询
sidebar:
  - toc
date: 2020-04-17 17:35:50
tags: [MySQL]
categories: [MySQL]
---

## 分组查询

### 概述

+ 语法

  ```sql
  SELECT column, group_function(column)
  FROM table
  [WHERE condition]
  GROUP BY group_by_expression
  [ORDER BY column];
  ```

+ 注意

  查询列表必须特殊，要求时分组函数和group by后出现的字段

+ 特点

  1. 分组筛选中的筛选条件分为两类：

     |  分组状态  |     数据源     |        位置        | 关键字 |
     | :--------: | :------------: | :----------------: | :----: |
     | 分组前筛选 |     原始表     | group by子句的前面 | where  |
     | 分组后筛选 | 分组厚的结果集 | group by子句的后面 | having |

     + 分组函数最条件肯定是放在having子句中
     + 能用分组前少选的，就优先考虑使用分组前筛选 

  2. group by语句支持单个字段分组，也可以支持多个 字段分组（多个字段之间用逗号隔开），表达式或函数（用的较少）

  3. 排序也可以被添加，是放在最后的

### 引入：查询每个部门的平均工资 

```sql
SELECT DISTINCT
    department_id, AVG(salary)
FROM
    employees;
```

+ 案例1：查询每个工种的最高工资，并且降序排序

  ```sql
  SELECT 
      job_id, MAX(salary) MAX_SAL
  FROM
      employees
  GROUP BY job_id
  ORDER BY MAX_SAL DESC;
  ```

+ 案例2：查询每个位置上的部门的个数

  ```sql
  SELECT 
      COUNT(*), location_id
  FROM
      departments
  GROUP BY location_id;
  ```

### 添加筛选条件

+ 案例1：查询邮箱中包含a字符的，每个部门的平均工资

  ```sql
  SELECT 
      AVG(salary), department_id
  FROM
      employees
  WHERE
      email LIKE '%a%'
  GROUP BY department_id;
  ```

+ 案例2：查询有奖金的每个领导手下员工的最高工资

  ```sql
  SELECT 
      MAX(salary), manager_id
  FROM
      employees
  WHERE
      commission_pct IS NOT NULL
  GROUP BY manager_id;
  ```

### 添加复杂的筛选条件

+ 案例1：查询那个部门员工的个数大于2

  ```sql
  ## 1. 查询每个部门的员工数
  SELECT 
      department_id, COUNT(*) DEP_COUNT
  FROM
      employees
  GROUP BY department_id;
  ## 2. 根据1的结果进行筛选(使用关键字 HAVING)
  SELECT 
      department_id, COUNT(*) DEP_COUNT
  FROM
      employees
  GROUP BY department_id
  HAVING DEP_COUNT > 2;
  ```

+ 案例2：查询每个工种有奖金的员工的最高工资大于12000的工种编号和最高工资 

  ```sql
  SELECT 
      job_id, MAX(salary) MAX_SAL
  FROM
      employees
  GROUP BY job_id
  HAVING MAX_SAL > 12000;
  ```

+ 案例3：查询领导编号大于120的每个领导手下的最低工资大于5000的林道编号是那个，及其最低工资

  ```sql
  SELECT 
      manager_id, MIN(salary) MIN_SAL
  FROM
      employees
  WHERE
      manager_id > 120
  GROUP BY manager_id
  HAVING MIN_SAL > 5000;
  ```

### 按表达式或函数分组

+ 案例：按员工行民的长度分组，查询每一组的员工个数，筛选员工个数大于5的有那些

  ```sql
  SELECT 
      LENGTH(last_name) NAME_LEN, COUNT(*)
  FROM
      employees
  GROUP BY NAME_LEN
  HAVING COUNT(*) > 5;
  ```

### 按多个字段分组

+ 案例：查询每个部门每个工种的员工的平均工资 

```sql
SELECT 
    department_id, job_id, ROUND(AVG(salary), 2)
FROM
    employees
GROUP BY department_id , job_id;
```

### 相关测试

1. 查询job_id的员工工资的最大值，最小值，平均值，总和，并按照job_id升序 

   ```sql
   SELECT 
       job_id,
       MAX(salary),
       MIN(salary),
       ROUND(AVG(salary), 2),
       SUM(salary)
   FROM
       employees
   GROUP BY job_id
   ORDER BY job_id ASC;
   ```

2. 查询员工最高工资和最低工资的差距（DIFFERENCE）

   ```sql
   SELECT 
       MAX(salary) - MIN(salary) DIFFERENCE
   FROM
       employees;
   ```

3. 查询各个管理者手下员工的最低工资，其中最低工资不能低于6000,没有管理者的员工不计算在内 

   ```sql
   SELECT 
       manager_id, MIN(salary) MIN_SAL
   FROM
       employees
   WHERE
       manager_id IS NOT NULL
   GROUP BY manager_id
   HAVING MIN_SAL >= 6000;
   ```

4. 查询所有部门的编号，员工数量和工资平均值，并按照平均工资降序

   ```sql
   SELECT 
       department_id, COUNT(*) STUFF_Q, AVG(salary) AVG_SAL
   FROM
       employees
   GROUP BY department_id
   ORDER BY AVG_SAL DESC;
   ```