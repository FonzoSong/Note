# SQL高级查询01

# 1 学习目标

1. **重点掌握**基础查询语法
2. **重点掌握**条件查询语法
3. **重点掌握**distinct关键字的使用

# 2 基础查询

## 2.1 语法

```mysql
SELECT 字段1,字段2,... [FROM 表名];
```
- **备注**

  类似于Java中 :System.out.println(要打印的东西);
  
  特点：
  
  ① 通过select查询完的结果 ，是一个虚拟的表格，不是真实存在
  
  ② 要查询的东西可以是常量值、可以是表达式、可以是字段、可以是函数

## 2.2 例子

①切入到tedu库

```mysql
USE tedu;
```

②查询student表中的所有字段内容，省略字段

```mysql
SELECT * FROM student;
```

③查询student表中的所有字段内容，不省略字段

```mysql
SELECT id,
       name,
       age,
       gender,
       job,
       birth,
       location_id,
       team_leader,
       class_id
FROM student;
```

④查询student表中的部分字段，但是字段显示顺序为name、age、gender

```mysql
SELECT name,
       age,
       gender
FROM student;
```

⑤查询student表中的部分字段，但是字段显示顺序为name、gender、age

```mysql
SELECT name,
       gender,
       age
FROM student;
```

# 3 条件查询

## 3.1 含义

- 条件查询：根据条件过滤原始表的数据，查询到想要的数据

## 3.2 语法

```mysql
SELECT 要查询的字段|表达式|常量值|函数 FROM 表 WHERE 条件;
```

## 3.3 条件表达式

### 3.3.1 条件运算符

|    条件运算符     | 含义                 |
| :---------------: | -------------------- |
|         >         | 大于                 |
|         <         | 小于                 |
|        \>=        | 大于等于             |
|        <=         | 小于等于             |
|         =         | 等于                 |
|        !=         | 不等于               |
|        <>         | 不等于               |
| BETWEEN 小 AND 大 | 在指定范围之间       |
|        IN         | 在一组指定的值中取值 |
|      IS NULL      | 字段为NULL           |

### 3.3.2 例子

①查看student表结构

```mysql
DESC student;
```

②查询**`学员id是110`**的学员部分信息(学员编号,学员姓名,学员年龄,学员性别)

```mysql
SELECT id,
       name,
       age,
       gender
FROM student
WHERE id = 110;
```

③查询**`性别为男`**的学生

```mysql
SELECT id,
       name,
       age,
       gender
FROM student
WHERE gender = '男';
```

④查询**`性别不是男性`**的学员（排除男性学员）

```mysql
SELECT id,
       name,
       age,
       gender
FROM student
WHERE gender <> '男';
```

或者

```mysql
SELECT id,
       name,
       age,
       gender
FROM student
WHERE gender != '男';
```

⑤查询**`年龄小于 9岁`**的学员

```mysql
SELECT id,
       name,
       age,
       gender
FROM student
WHERE age < 9;
```

⑥查询**`学员id在 [300, 400]范围`**的学员

```mysql
SELECT id,
       name,
       age,
       gender
FROM student
WHERE id BETWEEN 300 AND 400;
```

⑦查询**`学员id是100、120、122`**的学员

```mysql
SELECT id,
       name,
       age,
       gender
FROM student
WHERE id IN (100,120,122);
```

⑧没有职位的学生(job 是null值)

```mysql
SELECT id,
       name,
       age,
       gender,
       job
FROM student
WHERE job IS NULL;
```

## 3.4 逻辑表达式

### 3.4.1 逻辑运算符

| 逻辑运算符 | 含义                                            |
| :--------: | ----------------------------------------------- |
|     &&     | 两个条件如果同时成立，结果为true，否则为false   |
|    AND     | 两个条件如果同时成立，结果为true，否则为false   |
|    \|\|    | 两个条件只要有一个成立，结果为true，否则为false |
|     OR     | 两个条件只要有一个成立，结果为true，否则为false |
|    NOT     | 如果条件成立，则not后为false，否则为true        |

### 3.4.2 例子

①查询**`工资是 [8000, 10000]范围`**的老师

```mysql
SELECT id,
       name,
       title,
       salary
FROM teacher
WHERE salary >= 8000
  AND salary <= 10000;
```

或者

```mysql
SELECT id,
       name,
       title,
       salary
FROM teacher
WHERE salary >= 8000
  && salary <= 10000;
```

②查询**`薪资等于8000或者薪资等于10000`**的员工

```mysql
SELECT id,
       name,
       title,
       salary
FROM teacher
WHERE salary = 8000
  OR salary = 10000;
```

或者

```mysql
SELECT id,
       name,
       title,
       salary
FROM teacher
WHERE salary = 8000
  || salary = 10000;
```

③查询**`工资小于3000，或者工资大于10000`**

```mysql
SELECT id,
       name,
       title,
       salary
FROM teacher
WHERE salary < 3000
  OR salary > 10000;
```

或者

```mysql
SELECT id,
       name,
       title,
       salary
FROM teacher
WHERE salary NOT BETWEEN 3000 AND 10000;
```

④查询所有老师信息,**`排除职称为一级讲师、二级讲师和三级讲师`**

```mysql
SELECT id,
       name,
       title,
       salary
FROM teacher
WHERE title NOT IN ('一级讲师', '二级讲师', '三级讲师');
```

⑤有提成的员工(comm不是null)

```sql
SELECT id,
       name,
       title,
       salary,
       comm
FROM teacher
WHERE comm IS NOT NULL AND comm != 0;
```

## 3.5 模糊查询

### 3.5.1 概述

- 使用LIKE关键字可以进行字符串的模糊查询，但是需要使用通配符
- 通配符
  - _   单个字符(必须有1个任意字符)
  - %   多个字符(大于等于0个的任意字符)

- 格式示例

```SQL
LIKE '%X%' 	表示字符串中包含字符X
LIKE '_X%' 	表示字符串中第二个字符是X
LIKE 'X%'  	表示字符串以X开始
LIKE '%X'  	表示字符串以X结束
LIKE '%X_Y'	表示字符串倒数第三个字符数X并且最后一个字符是Y
```

### 3.5.2 例子

①查询**`姓王`**的学员信息

```mysql
SELECT id,
       name,
       gender
FROM student
WHERE name LIKE '王%';
```

②查询**`姓名中包含王字`**的学员信息

```sql
SELECT id,
       name,
       gender
FROM student
WHERE name LIKE '%王%';
```

③查询**`姓名第三字为林字`**的学员信息

```mysql
SELECT id,
       name,
       gender
FROM student
WHERE name LIKE '__林%';
```

④查询学员信息,**`排除姓名中包含王字`**的学员

```mysql
SELECT id,
       name,
       gender
FROM student
WHERE name NOT LIKE '%王%';
```

# 4 DISTINCT关键字

## 4.1 含义

- 去除重复数据

## 4.2 例子

①查询学员**`所有的职位`**(要求部分id不重复显示)，并且不显示null

```mysql
SELECT DISTINCT job
FROM student
WHERE job IS NOT NULL;
```

②所有班级中，有哪些职位(要求不显示班级为null和职位为null的记录)

```mysql
SELECT DISTINCT class_id, job
FROM student
WHERE job IS NOT NULL AND class_id IS NOT NULL;
```
