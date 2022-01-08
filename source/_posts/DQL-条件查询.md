---
title: DQL -- 条件查询
sidebar:
  - toc
date: 2020-04-17 17:29:48
tags: [MySQL]
categories: [MySQL]
---

## 条件查询

### 语法及其特点

+ 语法：

  ```sql
  SELECT 
  	查询列表
  FROM
  	表名
  WHERE
  	筛选条件;
  ```

+ 特点：

  1. 按照条件表达式筛选 
  2. 按照逻辑表达式筛选
  3. 模糊查询
  4. 安全等于 <=>

### 用法

1. 按照条件表达式筛选 

   > 条件运算符：< > = <= >= <>

   + 案例1：查询工资大于1200的信息

     ```sql
     SELECT * FROM employees WHERE salary>12000;
     ```

   + 案例2：查询部门编号不等于90的员工名和部门编号

     ```sql
     SELECT 
     	last_name,
         department_id
     FROM
     	employees
     WHERE
     	department_id <> 90;
     ```

2. 按照逻辑表达式筛选

   > 逻辑运算符：&& || !
   > 			AND OR NOT

   + 作用：用于连接条件表达式

   + 案例1：查询工资在10000到22000之间的员工名，工资以及奖金

     ```sql
     SELECT
     	last_name,
         salary,
         commission_pct
     FROM 
     	employees
     WHERE
     	10000 <= salary 
     		AND salary <= 22000;
     ```

   + 案例2：查询部门编号不是在90到110之间，或者工资高于15000的员工信息

     ```sql
     SELECT *
     FROM employees
     WHERE
     	NOT(department_id>=90 
             AND department_id<=110) 
             OR salary>15000;
     ```

3. 模糊查询

   > LIKE
   > BETWEEN AND
   > IN
   > IS NULL

   + 特点：和通配符搭配使用

     > 通配符： 
     > 		% 任意多个字符，包含0个字符
     >         _ 任意单个字符

   1. **LIKE**

      + 案例1：查询员工名中包含字符a的员工信息

        ```sql
        SELECT 
        	*
        FROM 
        	employees
        WHERE 
        	last_name LIKE '%a%';
        ```

      + 案例2：查询员工名中第三个字符为e，第五个字符为e的员工名和工资

        ```sql
        SELECT
        	last_name,
            salary
        FROM
        	employees
        WHERE
        	last_name LIKE '__e_e%';
        ```

      + 案例3：查询员工名中第二个字符为_的员工名

        ```sql
        SELECT
        	last_name,
            salary
        FROM
        	employees
        WHERE
        	last_name LIKE '_\_%';
        ```

   2. **BETWEEN AND**

      + 注意：

        1. 查询结果包含临界值
        2. AND 左边的必须小于右边的

      + 案例：查询员工编号在100到120之间的员工信息

        ```sql
        SELECT 
            *
        FROM
            employees
        WHERE
            employee_id BETWEEN 100 AND 120;
        ```

   3. **IN()**

      + 含义：判断字段的信息是否属于in列表中的某一项

      + Note：

        1. 提高语句的简洁度
        2. IN里面的值必须是同一个类型，或者是兼容
        3. 不可以使用通配符

      + 案例：查询员工的公众编号是 IT_PROG, AD_PRES中的员工名和工种编号

        ```sql
        SELECT 
            last_name, job_id
        FROM
            employees
        WHERE
            job_id IN ('IT_PROG' , 'AD_VP');
        ```

   4. **IS NULL**

      + Note：

        1. = 或者 <> 不能用于判断NULL
        2. IS 只能和 NULL 配合使用 

      + 案例：查询没有奖金的员工名和奖金率

        ```sql
        SELECT 
            last_name, commission_pct
        FROM
            employees
        WHERE
            commission_pct IS NOT NULL;
        ```

4. 安全等于 <=>

   > IS NULL：仅仅可以判断NULL值，可读性较高
   >
   > <=>：既可以判断NULL值，也可以判断普通数值，可读性较低

   + 案例1：查询没有奖金的员工名和奖金率

     ```sql
     SELECT 
         last_name, commission_pct
     FROM
         employees
     WHERE
         commission_pct <=> NULL;
     ```

   + 案例2：查询工资为1200的员工名和工资

     ```sql
     SELECT 
         last_name, salary
     FROM
         employees
     WHERE
         salary <=> 12000;
     ```

### 相关测试

1. 查询员工号为176的员工的姓名和部门号和年薪

   ```sql
   SELECT 
       CONCAT(last_name, ' ', first_name) AS 姓名,
       department_id AS 部门ID,
       salary * 12 * (1 + commission_pct) AS 年薪
   FROM
       employees
   WHERE
       employee_id IN (175);
   ```

2. 查询没有奖金，且工资小于18000的salary, last_name

   ```sql
   SELECT 
       last_name, salary
   FROM
       employees
   WHERE
       commission_pct IS NULL 
       AND salary < 18000;
   ```

3. 查询employees表中，job_id不为'IT'或者工资为12000的员工信息 

   ```sql
   SELECT 
       *
   FROM
       employees
   WHERE
       NOT (job_id LIKE 'IT%')
           OR salary = 12000;
   ```

4. 查看部门departments表的结构

   ```sql
   DESC departments
   ```

5. 查询部门部门departments表中涉及到了那些位置编号

   ```sql
   SELECT DISTINCT
       location_id
   FROM
       departments;
   ```

### 面试题

1. A：`select * from employees; `和`select * from employees where last_name like '%%' and commission_pct like '%%'`是否一样？

   Q：不一样，因为‘%%’通配符不能匹配NULL。
