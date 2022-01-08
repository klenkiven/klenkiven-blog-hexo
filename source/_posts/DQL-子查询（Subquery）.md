---
title: DQL -- 子查询（Subquery）
sidebar:
  - toc
date: 2020-04-18 02:16:54
tags: [MySQL]
categories: [MySQL]
---

## 子查询

+ 含义：出现在**其他语句**中的select语句，称为子查询或者内查询
  	外部的查询语句，叫做主查询或外查询

+ 分类

  + 按子查询出现的位置：
  	+ select语句
  		只支持标量子查询
      + from语句
  		支持表子查询
      + where或者having后面（重点）
  		标量子查询（单行）
          列子查询（多行）
          行子查询（使用较少）
      + exists语句后面（相关子查询）
  		表子查询
  + 按结果集的行列数不同
  		+ 标量子查询（结果只有一行一列）
  		  	+ 列子查询（只有一列多行）
  		  	+ 行子查询（结果可以有一行一列）
  		  	+ 表子查询（结果集一般为多行多列）

### 一、where或者having后面的子查询

+ 列子查询多行比较运算符

  1. IN/NOT IN：等于列表中的任意一个
  2. ANY/SOME：和子查询列表中的某一个比较（类似于每一个值比较都是或运算）
  3. ALL：和子查询列表中的每一个值比较（类似于每一个值比较都是与运算）

+ 特点

  1. 子查询放在小括号里面

  2. 子查询一般放在条件的最右侧

  3. 标量子查询，一般搭配单行操作符
  	
  	><, > , <=, >=, =, !=
  	
  4. 列子查询，一般搭配多行操作符使用
  	
  	> in, any/some, all
  	
  5. 子查询的执行优先与主查询执行

1. 标量子查询（单行子查询）

   + 案例1：谁的工资比Abel高

     ```sql
     SELECT 
         last_name, salary
     FROM
         employees
     WHERE
         salary > (
     		SELECT 
                 salary
             FROM
                 employees
             WHERE
                 last_name = 'Abel');
     ```

   + 案例2：返回job_id与141号员工相同，salary比141号员工多的员工姓名，job_id和工资

     > 执行单子句查询

     ```sql
     SELECT 
         last_name, job_id, salary
     FROM
         employees
     WHERE
         job_id = (SELECT 
                 job_id
             FROM
                 employees
             WHERE
                 employee_id = 141)
             AND salary > (SELECT 
                 salary
             FROM
                 employees
             WHERE
                 employee_id = 143);
     ```

   + 案例3：返回公司工资最少的员工的last_name，job_id和salary

     > 子查询中使用组函数

     ```sql
     SELECT 
         last_name, job_id, salary
     FROM
         employees
     WHERE
         salary = (SELECT 
                 MIN(salary)
             FROM
                 employees);
     ```

   + 案例4：查询最低工资大于50号部门最低工资的部门id和其最低工资

     > 子句中的HAVING语句

     ```sql
     SELECT 
         department_id, MIN(salary)
     FROM
         employees
     GROUP BY department_id
     HAVING MIN(salary) > (SELECT 
             MIN(salary)
         FROM
             employees
         WHERE
             department_id = 50);
     ```

2. 列子查询（多行子查询）

   + 案例1：返回location_id是1400或1700的部门中的所有员工姓名

     ```sql
     ## 连接查询
     SELECT 
         d.location_id, e.last_name
     FROM
         departments d
             INNER JOIN
         employees e ON e.department_id = d.department_id
     WHERE
         d.location_id IN (1400 , 1700);
     ## 列子查询
     SELECT 
         last_name
     FROM
         employees
     WHERE
         department_id IN (SELECT 
                 department_id
             FROM
                 departments
             WHERE
                 location_id IN (1400 , 1700));
     ```

   + 案例2：返回其他部门中比job_id为'IT_PROG'部门**任一**工资低的员工的工号，姓名，job_id，以及salary

     > 在多行子查询中使用ANY操作符

     ```sql
     SELECT 
         employee_id, last_name, job_id, salary
     FROM
         employees
     WHERE
         salary < ANY (SELECT DISTINCT
                 salary
             FROM
                 employees
             WHERE
                 job_id = 'IT_PROG')
             AND job_id <> 'IT_PROG';
     ```

   + 案例3：回其他部门中比job_id为'IT_PROG'部门**所有**工资低的员工的工号，姓名，job_id，以及salary

     > 在多行子查询中使用ALL操作符

     ```sql
     SELECT 
         employee_id, last_name, job_id, salary
     FROM
         employees
     WHERE
         salary < ALL (SELECT 
                 salary
             FROM
                 employees
             WHERE
                 job_id = 'IT_PROG')
             AND job_id <> 'IT_PROG';
     ```

3. 行子查询（一行多列，多行多列）

   + 案例：查询员工编号最小的并且是工资最高的员工工号。姓名，工资，job_id

     ```sql
     SELECT 
         employee_id, last_name, salary, job_id
     FROM
         employees
     WHERE
         employee_id = (SELECT 
                 MIN(employee_id)
             FROM
                 employees)
             AND salary = (SELECT 
                 MAX(salary)
             FROM
                 employees);
     ```

### 二、select语句后面的子查询

> select内部仅仅支持标量子查询

+ 案例1：查询每个部门的员工个数

  ```sql
  ## 子查询
  SELECT 
      d.department_id,
      (SELECT 
              COUNT(*)
          FROM
              employees e
          WHERE
              e.department_id = d.department_id) COUNT
  FROM
      departments d;
  ## 分组查询
  SELECT 
      department_id, COUNT(last_name) COUNT
  FROM
      employees
  GROUP BY department_id;
  ```

+ 案例2：查询员工号等于102的部门名

  ```sql
  ## 子查询版本
  SELECT 
      department_name
  FROM
      departments
  WHERE
      department_id = (SELECT 
              department_id
          FROM
              employees
          WHERE
              employee_id = 102);
  ## 连接查询
  SELECT 
      d.department_name
  FROM
      departments d
          INNER JOIN
      employees e ON e.department_id = d.department_id
  WHERE
      e.employee_id = 102;
  ```

### 三、from后面的子查询

+ 案例1：查询每个部门的平均工资的工资等级

  ```sql
  SELECT 
      A.department_id, round(A.AVG_SAL, 2), j.grade_level
  FROM
      (SELECT 
          AVG(salary) AVG_SAL, department_id
      FROM
          employees
      GROUP BY department_id) A
          INNER JOIN
      job_grades j ON A.AVG_SAL BETWEEN lowest_sal AND highest_sal;
  ```

### 四、exists后面的子查询（相关子查询）

```sql
## 判断子查询里面有没有值
SELECT 
    EXISTS( SELECT 
            employee_id
        FROM
            employees); ## 1 有
SELECT 
    EXISTS( SELECT 
            salary
        FROM
            employees
        WHERE
            salary = 30000); ## 0 没有
```

+ 案例：查询员工的部门名

  ```sql
  ## exists版本
  SELECT 
      department_name
  FROM
      departments d
  WHERE
      EXISTS( SELECT 
              *
          FROM
              employees e
          WHERE
              d.department_id = e.department_id);
  ## in版本
  SELECT 
      department_name
  FROM
      departments d
  WHERE
      d.department_id IN (SELECT 
              department_id
          FROM
              employees e
          WHERE
              e.department_id = d.department_id);
  ```

### 相关测试

1. 查询和Zlotkey相同部门的员工姓名和工资

   ```sql
   SELECT 
       last_name, salary
   FROM
       employees
   WHERE
       department_id = (SELECT 
               department_id
           FROM
               employees
           WHERE
               last_name = 'Zlotkey');
   ```

2. 查询工资比公司平均工资高得员工的员工号，姓名和工资

   ```sql 
   SELECT 
       employee_id, last_name, salary
   FROM
       employees
   WHERE
       salary > (SELECT 
               AVG(salary)
           FROM
               employees);
   ```

3. 查询各部门中工资比本部门平均工资高的员工的员工号和姓名

   ```sql 
   SELECT 
       employee_id, last_name
   FROM
       employees e
           INNER JOIN
       (SELECT 
           AVG(salary) av, department_id
       FROM
           employees
       GROUP BY department_id) A ON e.department_id = A.department_id
   WHERE
       e.salary > A.av;
   ```

4. 查询姓名中包含字母u的员工在相同部门的员工的员工号和姓名

   ```sql 
   SELECT 
       employee_id, last_name
   FROM
       employees e
   WHERE
       e.department_id IN (SELECT DISTINCT
               department_id
           FROM
               employees
           WHERE
               last_name LIKE '%u%');
   ```

5. 查询在部门的location_id为1700的部门工作的员工的员工号

   ```sql
   SELECT 
       employee_id
   FROM
       employees
   WHERE
       department_id IN (SELECT 
               department_id
           FROM
               departments
           WHERE
               location_id = 1700);
   ```

6. 查询管理者是K_ing的员工姓名和工资

   ```sql
   SELECT 
       last_name, salary
   FROM
       employees
   WHERE
       manager_id IN (SELECT 
               employee_id
           FROM
               employees
           WHERE
               last_name = 'K_ing');
   ```

7. 查询工资最高的员工姓名，要求first_name和last_name为一列

   ```sql
   SELECT 
       CONCAT(first_name, ' ', last_name)
   FROM
       employees
   WHERE
       salary = (SELECT 
               MAX(salary)
           FROM
               employees);