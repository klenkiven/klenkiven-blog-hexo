---
title: DQL -- 一些常见的函数
sidebar:
  - toc
date: 2020-04-17 17:31:01
tags: [MySQL]
categories: [MySQL]
---

## 常见函数

### 概述

> 函数类似于Java中的方法，将实现某种功能的一组逻辑语句封装在方法体中，对外暴露方法名

+ 优点

  1. 隐藏了实现细节
  2. 提高代码的重用性

+ 通用

  ```sql
  SELECT 函数名() [FROM 表];
  ```

+ 特点

  1. 函数名
  2. 函数功能

+ 分类

  1. 单行函数

     如：concat, length, ifnull等等

  2. 分组函数[统计函数，聚合函数，组函数]

     功能：做统计使用

### 单行函数

#### 一、字符函数

1. length()：获取参数值的字节数

   ```sql
   SELECT LENGTH('john');
   ## 这里使用的字符集是utf8mb4，一个中文字符占3个字节
   ## 如果是GBK字符集，那个一个中文字符占2个字节
   SELECT LENGTH('张三丰hahaha');
   #### 查看各种字符集
   show variables like '%char%';
   ```

2. concat()：拼接字符串

   ```sql
   SELECT 
       CONCAT(last_name, '_', first_name) 姓名
   FROM
       employees;
   ```

3. upper(), lower()

   ```sql
   SELECT UPPER('Jhon');
   SELECT LOWER('Jhon');
   ```

   + 案例：将姓变大写，名变小写

     ```sql
     SELECT 
         UPPER(last_name), LOWER(first_name)
     FROM
         employees;
     ```

4. substr(), substring()

   ```sql
   ## Note: SQL中索引都是从1开始
   ## 指定索引往后到结束的子串
   SELECT SUBSTR('KlenKiven', 5);
   ## 指定索引往后n个字符的字串
   SELECT SUBSTR('KlenKiven', 1, 4);
   ```

   + 案例：姓名首字母大写，其他字母小写用下划线拼接，显示出来

     ```sql
     SELECT 
         CONCAT(UPPER(SUBSTR(last_name, 1, 1)),
                 '_',
                 LOWER(SUBSTR(last_name, 2))) out_put
     FROM
         employees;
     ```

5. instr()：返回字串第一次在字符串中的索引位置，如果找不到返回0

   ```sql
   SELECT INSTR('Abc', 'bc');
   ```

6. trim()：去掉前后的空格

   ```sql
   SELECT TRIM('      ABC							');
   SELECT TRIM('a' FROM 'aaaaaaaaaaaaaaaaaKlenaaaaaaaaaaa');
   SELECT TRIM('ab' FROM 'ababababababababababKlenababababab');
   ```

7. lpad()：指定字符左填充指定长度（像是那个格式化输出那个）

   ```sql
   SELECT LPAD('ABC', 10, '*');
   ```

8. rpad()：指定字符右填充指定长度（像是那个格式化输出那个）

   ```sql
   SELECT RPAD('ABC', 10, '*');
   ```

9. replace()：替换

   ```sql
   SELECT REPLACE('ABCDEFBCDGH', 'BCD', 'XYZ');
   ```

#### 二、数学函数

1. round()：四舍五入

   ```sql
   SELECT ROUND(1.65);
   SELECT ROUND(1.45);
   ```

2. ceil()：向上取整

   ```sql
   SELECT CEIL(1.52);
   SELECT CEIL(-1.02);
   ```

3. floor()：向下取整

   ```sql
   SELECT FLOOR(1.99);
   SELECT FLOOR(-1.99);
   ```

4. truncate()：截断

   ```sql
   SELECT TRUNCATE(1.6999, 1);
   ```

5. mod()：取余

   ```sql
   SELECT MOD(10, 3);
   SELECT MOD(- 10, - 3);
   SELECT 10 % 3;
   ```

#### 三、日期函数

1. now()：返回当前系统日期时间

2. curdate()：返回当前的日期不包含时间

3. curtime()：返回当前的时间不包含日期

4. year(), month(), day(), hour(), minute(), second()：返回特定的值

5. str_to_date()：将日期格式的字符串转换成指定格式的日期

   ```sql
   SELECT STR_TO_DATE('9-3-1999', '%m-%d-%Y');
   ```

   + 查询入职日期为1992-4-3的员工信息

     ```sql
     SELECT 
         *
     FROM
         employees
     WHERE
         hiredate = '1992-4-3';
     ```

     + 使用str_to_date()版本

     ```sql
     SELECT 
         *
     FROM
         employees
     WHERE
         hiredate = STR_TO_DATE('1992-4-3', '%Y-%m-%d');
     ```

6. date_format()：将日期转换成字符

   ```sql
   SELECT DATE_FORMAT('2018/6/6', '%Y年%m月%d日');
   ```

   + 查询有奖金的员工名和入职时间（XX月/XX日 XX年）

     ```sql
     SELECT 
         last_name, 
         DATE_FORMAT(hiredate, '%m月/%d日 %Y年') 入职日期
     FROM
         employees;
     ```

#### 四、其他函数

1. version()：MySQL的版本
2. databases()：数据库
3. user()：用户

#### 五、流程控制函数

1. if()：实现if-else的效果

   ```sql
   SELECT IF(10 > 5, '大', '小');
   
   SELECT 
   	last_name,
       IF(commission_pct IS NOT NULL,
           '有奖',
           '没有奖') 有无
   FROM employees;
   ```

2. case-when：switch-case使用效果

   >case 要判断的字段或者表达式
   >	when 常量1 then 要显示的值1或语句1;
   >	when 常量2 then 要显示的值2或语句2;
   >	else 要显示的值或语句;
   >end

   + 案例：查询员工的工资，要求

     > 如果部门号=30,显示的工资为1.1倍
     > 如果部门号=40,显示的工资为1.2倍
     > 如果部门号=50,显示的工资为1.3倍
     > 其他部门,显示的工资为保底工资

     ```sql
     SELECT 
         salary 原始工资,
         department_id,
         CASE department_id
             WHEN 30 THEN salary * 1.1
             WHEN 40 THEN salary * 1.2
             WHEN 50 THEN salary * 1.3
             ELSE salary
         END 新工资
     FROM
         employees;
     ```

3. case：多重if

   > case
   > 	when 判断条件1 then 要显示的值1或语句1;
   > 	when 判断条件2 then 要显示的值2或语句2;
   > 	when 判断条件3 then 要显示的值3或语句3;
   > 	else 要显示的值或语句;
   > end

   + 案例：查询员工的新工资，要求

     > 如果工资>20000，显示A级别
     > 如果工资>15000，显示B级别
     > 如果工资>10000，显示C级别
     > 否则，显示D级别

     ```sql
     SELECT 
         salary,
         CASE
             WHEN salary > 20000 THEN 'A级别'
             WHEN salary > 15000 THEN 'B级别'
             WHEN salary > 10000 THEN 'C级别'
             ELSE 'D级别'
         END 级别
     FROM
         employees;
     ```

### 多行函数

#### 概述

+ 功能

  用作统计使用，又称为聚合函数或统计函数或组函数

+ 分类

  + 数值计算：sum 求和、avg 平均值
  + 数据分析：max 最大值 、min 最小值 、count 计算个数

+ 特点

  1. sum、avg一般用于处理数值型

  2. max、min、count可以处理任何类型

  3. 以上分组函数都忽略null值

  4. 可以和distinct搭配实现去重的运算

  5. count函数：

     一般使用`count(*)`统计行数

  6. 和分组函数一同查询的字段要求是，`GROUP BY`后的字段

#### 1. 简单使用

```sql
SELECT SUM(salary) FROM employees;
SELECT AVG(salary) FROM employees;
SELECT MIN(salary) FROM employees;
SELECT MAX(salary) FROM employees;
SELECT COUNT(salary) FROM employees;


SELECT SUM(salary) 和,AVG(salary) 平均,MAX(salary) 最高,MIN(salary) 最低,COUNT(salary) 个数
FROM employees;

SELECT SUM(salary) 和,ROUND(AVG(salary),2) 平均,MAX(salary) 最高,MIN(salary) 最低,COUNT(salary) 个数
FROM employees;
```

#### 2. 是否忽略null

```sql
SELECT SUM(commission_pct) ,AVG(commission_pct),SUM(commission_pct)/35,SUM(commission_pct)/107 FROM employees;

SELECT MAX(commission_pct) ,MIN(commission_pct) FROM employees;

SELECT COUNT(commission_pct) FROM employees;
SELECT commission_pct FROM employees;

```

#### 3. 和distinct搭配

```sql
SELECT 
	SUM(DISTINCT salary),
	SUM(salary) 
FROM 
	employees;

SELECT 
	COUNT(DISTINCT salary),
	COUNT(salary) 
FROM 
	employees;
```

#### 4. count函数的详细介绍

```sql
SELECT 
    COUNT(salary)
FROM
    employees;
## 统计行数
SELECT 
    COUNT(*)
FROM
    employees;
## 统计1的个数，相当于加了一列的1的个数
SELECT 
    COUNT(1)
FROM
    employees;
```

> MYISAM存储引擎下  ，COUNT(*)的效率高
>
> INNODB存储引擎下，COUNT(*)和COUNT(1)的效率差不多，比COUNT(字段)要高一些

#### 5. 和分组函数一同查询的字段有限制

```sql
SELECT 
    AVG(salary), employee_id
FROM
    employees;
```

### 相关测试

1. 显示系统时间

   ```sql
   SELECT NOW();
   ```

2. 询员工号，姓名，工资，以及工资提高20%以后的结果（new salary）

   ```sql
   SELECT 
       employee_id,
       last_name,
       salary,
       salary * (1 + 0.2) 'new salary'
   FROM
       employees;
   ```

3. 将员工的姓名按照，并写出姓名的长度

   ```sql
   SELECT 
       last_name, LENGTH(last_name) length
   FROM
       employees
   ORDER BY SUBSTR(last_name, 1, 1);
   ```

4. 做一个查询，产生下面的结果

   > <last_name> earns <salary> monthly but wants <salary*3> Dream Salary

   ```sql
   SELECT 
       CONCAT(last_name,
               ' earns ',
               salary,
               'monthly but wants ',
               salary * 3,
               'Dream Salary') Sentence
   FROM
       employees;
   ```

5. 使用case-when，按照下面的条件

   > job_id		grade
   > AD_PRES		A
   > ST_MAN		 B
   > IT_PROG		C

   ```sql
   SELECT 
       last_name,
       job_id Job,
       CASE job_id
           WHEN 'AD_PRES' THEN 'A'
           WHEN 'ST_MAN' THEN 'B'
           WHEN 'IT_PROG' THEN 'C'
       END Grade
   FROM
       employees
   WHERE
       job_id IN ('AD_PRES' , 'ST_MAN', 'IT_PROG');
   ```

6. 查询员工粽子最大值，最小值，平均值，总和。

   ```sql
   SELECT 
       MAX(salary) MAX_SAL,
       MIN(salary) MIN_SAL,
       AVG(salary) AVG_SAL,
       SUM(salary) SUM_SAL
   FROM
       employees;
   ```

7. 查询员工表中的最大入职时间和最小入职时间相差的天数

   ```sql
   FROM
       employees;
       
   SELECT 
       MAX(hiredate) MAX_DATE,
       MIN(hiredate) MIN_DATE,
       DATEDIFF(MAX(hiredate), MIN(hiredate)) DIFFERENCE
   FROM
       employees;
   ```

8. 查询部门编号为90的员工个数

   ```sql
   SELECT 
       COUNT(*)
   FROM
       employees
   WHERE
       department_id = 90;
   ```
