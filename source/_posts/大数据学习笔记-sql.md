---
title: 大数据学习笔记-sql
date: 2018-10-19 09:13:11
tags: 
categories: 大数据
thumbnail: http://cdn1.itpro.co.uk/sites/itpro/files/2017/02/bigstock-big_data-159884873.jpg
---



# 表1： websites
```
mysql> select * from websites;
+----+--------------+---------------------------+-------+---------+
| id | name         | url                       | alexa | country |
+----+--------------+---------------------------+-------+---------+
|  1 | Google       | https://www.google.cm/    |     1 | USA     |
|  2 | 淘宝         | https://www.taobao.com/   |    13 | CN      |
|  3 | 菜鸟教程     | http://www.runoob.com/    |  5000 | USA     |
|  4 | 微博         | http://weibo.com/         |    20 | CN      |
|  5 | Facebook     | https://www.facebook.com/ |     3 | USA     |
|  6 | JonyChiao    | jiaoqiyuan.cn             |   200 | CN      |
+----+--------------+---------------------------+-------+---------+
6 rows in set (0.00 sec)
```
# 表2：access_log
```
mysql> select * from access_log;
+-----+---------+-------+------------+
| aid | site_id | count | date       |
+-----+---------+-------+------------+
|   1 |       1 |    45 | 2016-05-10 |
|   2 |       3 |   100 | 2016-05-13 |
|   3 |       1 |   230 | 2016-05-14 |
|   4 |       2 |    10 | 2016-05-14 |
|   5 |       5 |   205 | 2016-05-14 |
|   6 |       4 |    13 | 2016-05-15 |
|   7 |       3 |   220 | 2016-05-15 |
|   8 |       5 |   545 | 2016-05-16 |
|   9 |       3 |   201 | 2016-05-17 |
+-----+---------+-------+------------+
9 rows in set (0.01 sec)
```

# 表3：apps
```
mysql> select * from apps;
+----+------------+-------------------------+---------+
| id | app_name   | url                     | country |
+----+------------+-------------------------+---------+
|  1 | QQ APP     | http://im.qq.com/       | CN      |
|  2 | 微博 APP   | http://weibo.com/       | CN      |
|  3 | 淘宝 APP   | https://www.taobao.com/ | CN      |
+----+------------+-------------------------+---------+
3 rows in set (0.00 sec)
```

## 查看表结构

```
mysql> show columns from websites;
+---------+--------------+------+-----+---------+----------------+
| Field   | Type         | Null | Key | Default | Extra          |
+---------+--------------+------+-----+---------+----------------+
| id      | int(11)      | NO   | PRI | NULL    | auto_increment |
| name    | char(20)     | NO   |     |         |                |
| url     | varchar(255) | NO   |     |         |                |
| alexa   | int(11)      | NO   |     | 0       |                |
| country | char(10)     | NO   |     |         |                |
+---------+--------------+------+-----+---------+----------------+
5 rows in set (0.02 sec)
```

## LIMIT语句

```bash
mysql> select * from websites limit 2;
+----+--------+-------------------------+-------+---------+
| id | name   | url                     | alexa | country |
+----+--------+-------------------------+-------+---------+
|  1 | Google | https://www.google.cm/  |     1 | USA     |
|  2 | 淘宝   | https://www.taobao.com/ |    13 | CN      |
+----+--------+-------------------------+-------+---------+
2 rows in set (0.00 sec)
```

## LIKE语句：LIKE 操作符用于在 WHERE 子句中搜索列中的指定模式。

```bash
mysql> select * from websites where name like 'G%';
+----+--------+------------------------+-------+---------+
| id | name   | url                    | alexa | country |
+----+--------+------------------------+-------+---------+
|  1 | Google | https://www.google.cm/ |     1 | USA     |
+----+--------+------------------------+-------+---------+
1 row in set (0.00 sec)
```
**这里面%是通配符**

```bash
mysql> select * from websites where name like 'J%';
+----+-----------+---------------+-------+---------+
| id | name      | url           | alexa | country |
+----+-----------+---------------+-------+---------+
|  6 | JonyChiao | jiaoqiyuan.cn |   200 | CN      |
+----+-----------+---------------+-------+---------+
1 row in set (0.00 sec)
```

```bash
mysql> select * from websites where name not like 'J%';
+----+--------------+---------------------------+-------+---------+
| id | name         | url                       | alexa | country |
+----+--------------+---------------------------+-------+---------+
|  1 | Google       | https://www.google.cm/    |     1 | USA     |
|  2 | 淘宝         | https://www.taobao.com/   |    13 | CN      |
|  3 | 菜鸟教程     | http://www.runoob.com/    |  5000 | USA     |
|  4 | 微博         | http://weibo.com/         |    20 | CN      |
|  5 | Facebook     | https://www.facebook.com/ |     3 | USA     |
+----+--------------+---------------------------+-------+---------+
5 rows in set (0.00 sec)
```
## SQL通配符

|   通配符 |    描述 |
|:---:|:--------------:|
|  %  | 替代0个或多个字符 |
| _   | 替代一个字符 |
| [charlist] | 字符列表中的任何单一字符 |
| [^charlist] 或 [!charlist] | 不在字符列表中的任何一个单一字符 |

```bash
mysql> select * from websites where url like 'https%';
+----+----------+---------------------------+-------+---------+
| id | name     | url                       | alexa | country |
+----+----------+---------------------------+-------+---------+
|  1 | Google   | https://www.google.cm/    |     1 | USA     |
|  2 | 淘宝     | https://www.taobao.com/   |    13 | CN      |
|  5 | Facebook | https://www.facebook.com/ |     3 | USA     |
+----+----------+---------------------------+-------+---------+
3 rows in set (0.00 sec)


mysql> select * from websites where name like '_onyChiao';
+----+-----------+---------------+-------+---------+
| id | name      | url           | alexa | country |
+----+-----------+---------------+-------+---------+
|  6 | JonyChiao | jiaoqiyuan.cn |   200 | CN      |
+----+-----------+---------------+-------+---------+
1 row in set (0.00 sec)

选取以J开头的name的字段：
mysql> select * from websites where name regexp '^[J]';
+----+-----------+---------------+-------+---------+
| id | name      | url           | alexa | country |
+----+-----------+---------------+-------+---------+
|  6 | JonyChiao | jiaoqiyuan.cn |   200 | CN      |
+----+-----------+---------------+-------+---------+
1 row in set (0.00 sec)

选取以非J开头的name字段：
mysql> select * from websites where name regexp '^[^J]';
+----+--------------+---------------------------+-------+---------+
| id | name         | url                       | alexa | country |
+----+--------------+---------------------------+-------+---------+
|  1 | Google       | https://www.google.cm/    |     1 | USA     |
|  2 | 淘宝         | https://www.taobao.com/   |    13 | CN      |
|  3 | 菜鸟教程     | http://www.runoob.com/    |  5000 | USA     |
|  4 | 微博         | http://weibo.com/         |    20 | CN      |
|  5 | Facebook     | https://www.facebook.com/ |     3 | USA     |
+----+--------------+---------------------------+-------+---------+
5 rows in set (0.00 sec)
```
**注意：Mysql不支持[charsite]、[!charsite]通配符**

## IN操作符：N 操作符允许在 WHERE 子句中规定多个值

```bash
mysql> select * from websites where name in ('JonyChiao', 'Google');
+----+-----------+------------------------+-------+---------+
| id | name      | url                    | alexa | country |
+----+-----------+------------------------+-------+---------+
|  1 | Google    | https://www.google.cm/ |     1 | USA     |
|  6 | JonyChiao | jiaoqiyuan.cn          |   200 | CN      |
+----+-----------+------------------------+-------+---------+
2 rows in set (0.01 sec)
```

## BETWEEN操作符
- BETWEEN 操作符用于选取介于两个值之间的数据范围内的值。这些值可以是数值、文本或者日期。

    ```bash
    mysql> select * from websites where alexa between 1 and 10;
    +----+----------+---------------------------+-------+---------+
    | id | name     | url                       | alexa | country |
    +----+----------+---------------------------+-------+---------+
    |  1 | Google   | https://www.google.cm/    |     1 | USA     |
    |  5 | Facebook | https://www.facebook.com/ |     3 | USA     |
    +----+----------+---------------------------+-------+---------+
    2 rows in set (0.00 sec)

    mysql> select * from websites where alexa not between 1 and 10;
    +----+--------------+-------------------------+-------+---------+
    | id | name         | url                     | alexa | country |
    +----+--------------+-------------------------+-------+---------+
    |  2 | 淘宝         | https://www.taobao.com/ |    13 | CN      |
    |  3 | 菜鸟教程     | http://www.runoob.com/  |  5000 | USA     |
    |  4 | 微博         | http://weibo.com/       |    20 | CN      |
    |  6 | JonyChiao    | jiaoqiyuan.cn           |   200 | CN      |
    +----+--------------+-------------------------+-------+---------+
    4 rows in set (0.00 sec)
    ```

- 选取alexa介于1到20但是country不是USA和IND的所有网站：

    ```bash
    mysql> select * from websites where (alexa between 1 and 20) and country not in ('USA', 'IND');
    +----+--------+-------------------------+-------+---------+
    | id | name   | url                     | alexa | country |
    +----+--------+-------------------------+-------+---------+
    |  2 | 淘宝   | https://www.taobao.com/ |    13 | CN      |
    |  4 | 微博   | http://weibo.com/       |    20 | CN      |
    +----+--------+-------------------------+-------+---------+
    2 rows in set (0.00 sec)
    ```

- 选取 name 以介于 'A' 和 'H' 之间字母开始的所有网站：

    ```bash
    mysql> select * from websites where name between 'A' and 'H';
    +----+----------+---------------------------+-------+---------+
    | id | name     | url                       | alexa | country |
    +----+----------+---------------------------+-------+---------+
    |  1 | Google   | https://www.google.cm/    |     1 | USA     |
    |  5 | Facebook | https://www.facebook.com/ |     3 | USA     |
    +----+----------+---------------------------+-------+---------+
    2 rows in set (0.00 sec)
    ```
- 选取日期区间
    ```
    mysql> select * from access_log where date between '2016-5-10' and '2016-05-14';
    +-----+---------+-------+------------+
    | aid | site_id | count | date       |
    +-----+---------+-------+------------+
    |   1 |       1 |    45 | 2016-05-10 |
    |   2 |       3 |   100 | 2016-05-13 |
    |   3 |       1 |   230 | 2016-05-14 |
    |   4 |       2 |    10 | 2016-05-14 |
    |   5 |       5 |   205 | 2016-05-14 |
    +-----+---------+-------+------------+
    5 rows in set (0.00 sec)
    ```
    请注意，在不同的数据库中，BETWEEN 操作符会产生不同的结果！
    在某些数据库中，BETWEEN 选取介于两个值之间但不包括两个测试值的字段。
    在某些数据库中，BETWEEN 选取介于两个值之间且包括两个测试值的字段。
    在某些数据库中，BETWEEN 选取介于两个值之间且包括第一个测试值但不包括最后一个测试值的字段。

    因此，请检查您的数据库是如何处理 BETWEEN 操作符！

## 别名

- 通过使用 SQL，可以为表名称或列名称指定别名。基本上，创建别名是为了让列名称的可读性更强。
- 列的别名：

    ```bash
    mysql> select name as n, country as c from websites;
    +--------------+-----+
    | n            | c   |
    +--------------+-----+
    | Google       | USA |
    | 淘宝         | CN  |
    | 菜鸟教程     | CN  |
    | 微博         | CN  |
    | Facebook     | USA |
    +--------------+-----+
    5 rows in set (0.00 sec)
    ```

- 表的别名：

    ```bash
    mysql> select w.name, w.url, a.count, a.date from websites as w, access_log as a where a.site_id=w.id;
    +--------------+---------------------------+-------+------------+
    | name         | url                       | count | date       |
    +--------------+---------------------------+-------+------------+
    | Google       | https://www.google.cm/    |    45 | 2016-05-10 |
    | 菜鸟教程     | http://www.runoob.com/    |   100 | 2016-05-13 |
    | Google       | https://www.google.cm/    |   230 | 2016-05-14 |
    | 淘宝         | https://www.taobao.com/   |    10 | 2016-05-14 |
    | Facebook     | https://www.facebook.com/ |   205 | 2016-05-14 |
    | 微博         | http://weibo.com/         |    13 | 2016-05-15 |
    | 菜鸟教程     | http://www.runoob.com/    |   220 | 2016-05-15 |
    | Facebook     | https://www.facebook.com/ |   545 | 2016-05-16 |
    | 菜鸟教程     | http://www.runoob.com/    |   201 | 2016-05-17 |
    +--------------+---------------------------+-------+------------+
    9 rows in set (0.00 sec)
    这里面的as可以省略

    select w.name, w.url, a.count, a.date from websites w, access_log a where a.site_id=w.id;
    ```

- 别名一般使用在以下情况：
    - 查询中涉及超过一个表
    - 在查询中使用了函数
    - 类名称很长或者可读性很差
    - 需要把两个列或者多个列结合在一起

## JOIN：SQL JOIN 子句用于把来自两个或多个表的行结合起来，基于这些表之间的共同字段。

```bash
mysql> select w.id, w.name, a.count, a.date from websites w join access_log a on w.id=a.site_id;
+----+--------------+-------+------------+
| id | name         | count | date       |
+----+--------------+-------+------------+
|  1 | Google       |    45 | 2016-05-10 |
|  3 | 菜鸟教程     |   100 | 2016-05-13 |
|  1 | Google       |   230 | 2016-05-14 |
|  2 | 淘宝         |    10 | 2016-05-14 |
|  5 | Facebook     |   205 | 2016-05-14 |
|  4 | 微博         |    13 | 2016-05-15 |
|  3 | 菜鸟教程     |   220 | 2016-05-15 |
|  5 | Facebook     |   545 | 2016-05-16 |
|  3 | 菜鸟教程     |   201 | 2016-05-17 |
+----+--------------+-------+------------+
9 rows in set (0.00 sec)
```

- INNER JOIN:关键字在表中存在至少一个匹配时返回行。
    - inner join 与join相同

    ![Inner Join][1]

    ```bash
    mysql> select * from websites w inner join access_log a on w.id=a.site_id order by a.count;
    +----+--------------+---------------------------+-------+---------+-----+---------+-------+------------+
    | id | name         | url                       | alexa | country | aid | site_id | count | date       |
    +----+--------------+---------------------------+-------+---------+-----+---------+-------+------------+
    |  2 | 淘宝         | https://www.taobao.com/   |    13 | CN      |   4 |       2 |    10 | 2016-05-14 |
    |  4 | 微博         | http://weibo.com/         |    20 | CN      |   6 |       4 |    13 | 2016-05-15 |
    |  1 | Google       | https://www.google.cm/    |     1 | USA     |   1 |       1 |    45 | 2016-05-10 |
    |  3 | 菜鸟教程     | http://www.runoob.com/    |  4689 | CN      |   2 |       3 |   100 | 2016-05-13 |
    |  3 | 菜鸟教程     | http://www.runoob.com/    |  4689 | CN      |   9 |       3 |   201 | 2016-05-17 |
    |  5 | Facebook     | https://www.facebook.com/ |     3 | USA     |   5 |       5 |   205 | 2016-05-14 |
    |  3 | 菜鸟教程     | http://www.runoob.com/    |  4689 | CN      |   7 |       3 |   220 | 2016-05-15 |
    |  1 | Google       | https://www.google.cm/    |     1 | USA     |   3 |       1 |   230 | 2016-05-14 |
    |  5 | Facebook     | https://www.facebook.com/ |     3 | USA     |   8 |       5 |   545 | 2016-05-16 |
    +----+--------------+---------------------------+-------+---------+-----+---------+-------+------------+
    9 rows in set (0.00 sec)
    ```

- LEFT JOIN:关键字从左表（table1）返回所有的行，即使右表（table2）中没有匹配。如果右表中没有匹配，则结果为 NULL。

    ```bash
    mysql> insert into websites value(6, 'stackoverflow', 'http://stackoverflow.com/', 0, 'IND');

    mysql> select w.name, a.count, a.date from websites w left join access_log a on w.id=a.site_id order by a.count desc;
    +---------------+-------+------------+
    | name          | count | date       |
    +---------------+-------+------------+
    | Facebook      |   545 | 2016-05-16 |
    | Google        |   230 | 2016-05-14 |
    | 菜鸟教程      |   220 | 2016-05-15 |
    | Facebook      |   205 | 2016-05-14 |
    | 菜鸟教程      |   201 | 2016-05-17 |
    | 菜鸟教程      |   100 | 2016-05-13 |
    | Google        |    45 | 2016-05-10 |
    | 微博          |    13 | 2016-05-15 |
    | 淘宝          |    10 | 2016-05-14 |
    | stackoverflow |  NULL | NULL       |
    +---------------+-------+------------+
    10 rows in set (0.00 sec)
    ```

- RIGHT JOIN:关键字从右表（table2）返回所有的行，即使左表（table1）中没有匹配。如果左表中没有匹配，则结果为 NULL。

![RIGHTJOIN][2]

```bash
mysql> select w.name, a.count, a.date from websites w right join access_log a on w.id=a.site_id order by a.count desc;
+--------------+-------+------------+
| name         | count | date       |
+--------------+-------+------------+
| Facebook     |   545 | 2016-05-16 |
| Google       |   230 | 2016-05-14 |
| 菜鸟教程     |   220 | 2016-05-15 |
| Facebook     |   205 | 2016-05-14 |
| 菜鸟教程     |   201 | 2016-05-17 |
| 菜鸟教程     |   100 | 2016-05-13 |
| Google       |    45 | 2016-05-10 |
| 微博         |    13 | 2016-05-15 |
| 淘宝         |    10 | 2016-05-14 |
+--------------+-------+------------+
9 rows in set (0.00 sec)
注意这里没有了StackOverflow。
```

- FULL JOIN:

    ![FULL OUTER JOIN][3]
    
    - mysql不支持full join

- JOINS
   
    ![JOINS][5]

扩展阅读 : [图解SQL的JOIN][4]

## UNION操作符：合并两个或多个 SELECT 语句的结果。
- 默认地，UNION 操作符选取不同的值。如果允许重复的值，请使用 UNION ALL。

    ```bash
    mysql> select country from websites union select country from apps;
    +---------+
    | country |
    +---------+
    | USA     |
    | CN      |
    +---------+
    2 rows in set (0.01 sec)

    mysql> select country from websites union all select country from apps;
    +---------+
    | country |
    +---------+
    | USA     |
    | CN      |
    | USA     |
    | CN      |
    | USA     |
    | CN      |
    | CN      |
    | CN      |
    | CN      |
    +---------+
    9 rows in set (0.00 sec)

    mysql> select country, name from websites where country='CN' union all select country, app_name from apps where country='CN' order by country;
    +---------+------------+
    | country | name       |
    +---------+------------+
    | CN      | JonyChiao  |
    | CN      | 淘宝 APP   |
    | CN      | 淘宝       |
    | CN      | QQ APP     |
    | CN      | 微博       |
    | CN      | 微博 APP   |
    +---------+------------+
    6 rows in set (0.00 sec)
    ```

## SQL SELECT INTO 语句:从一个表复制数据，然后把数据插入到另一个新表中。

- MySQL 数据库不支持 SELECT ... INTO 语句，但支持 INSERT INTO ... SELECT 。

## SQL INSERT INTO SELECT 语句:从一个表复制数据，然后把数据插入到一个已存在的表中。目标表中任何已存在的行都不会受影响。

- 复制 "apps" 中的数据插入到 "Websites" 中：

    ```bash
    mysql> insert into websites(name, country) select app_name, country from apps;
    Query OK, 3 rows affected (0.04 sec)
    Records: 3  Duplicates: 0  Warnings: 0

    mysql> select * from websites;
    +----+--------------+---------------------------+-------+---------+
    | id | name         | url                       | alexa | country |
    +----+--------------+---------------------------+-------+---------+
    |  1 | Google       | https://www.google.cm/    |     1 | USA     |
    |  2 | 淘宝         | https://www.taobao.com/   |    13 | CN      |
    |  3 | 菜鸟教程     | http://www.runoob.com/    |  5000 | USA     |
    |  4 | 微博         | http://weibo.com/         |    20 | CN      |
    |  5 | Facebook     | https://www.facebook.com/ |     3 | USA     |
    |  6 | JonyChiao    | jiaoqiyuan.cn             |   200 | CN      |
    |  7 | QQ APP       |                           |     0 | CN      |
    |  8 | 微博 APP     |                           |     0 | CN      |
    |  9 | 淘宝 APP     |                           |     0 | CN      |
    +----+--------------+---------------------------+-------+---------+
    9 rows in set (0.00 sec)
    ```
## 创建数据库：create database my_db

## 创建表

```bash
create table people( PersonID int, LastName varchar(255), FirstName varchar(255), Address varchar(255), City varchar(255) );
```

## SQL约束
- SQL 约束用于规定表中的数据规则。
- 约束可以在创建表时规定（通过 CREATE TABLE 语句），或者在表创建之后规定（通过 ALTER TABLE 语句）。
- 在 SQL 中，我们有如下约束：
    - NOT NULL - 指示某列不能存储 NULL 值。
    - UNIQUE - 保证某列的每行必须有唯一的值。
    - PRIMARY KEY - NOT NULL 和 UNIQUE 的结合。确保某列（或两个列多个列的结合）有唯一标识，有助于更容易更快速地找到表中的一个特定的记录。
    - FOREIGN KEY - 保证一个表中的数据匹配另一个表中的值的参照完整性。
    - CHECK - 保证列中的值符合指定的条件。、
    - DEFAULT - 规定没有给列赋值时的默认值。

### NOT NULL约束
- 约束强制字段始终包含值。这意味着，如果不向字段添加值，就无法插入新记录或者更新记录。

    ```bash
    CREATE TABLE Persons
    (
    P_Id int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Address varchar(255),
    City varchar(255)
    )
    ```

### NUIQUE约束
- UNIQUE 约束唯一标识数据库表中的每条记录。
- UNIQUE 和 PRIMARY KEY 约束均为列或列集合提供了唯一性的保证。
- PRIMARY KEY 约束拥有自动定义的 UNIQUE 约束。
- 每个表可以有多个 UNIQUE 约束，但是每个表只能有一个 PRIMARY KEY 约束。

    ```bash
    在 "Persons" 表创建时在 "P_Id" 列上创建 UNIQUE 约束：
    CREATE TABLE Persons
    (
    P_Id int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Address varchar(255),
    City varchar(255),
    UNIQUE (P_Id)
    )

    如需命名（CONSTRAINT） UNIQUE 约束，并定义多个列的 UNIQUE 约束，请使用下面的 SQL 语法：
    CREATE TABLE Persons
    (
    P_Id int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Address varchar(255),
    City varchar(255),
    CONSTRAINT uc_PersonID UNIQUE (P_Id,LastName)
    )
    ```
- ALTER TABLE 时的 SQL UNIQUE 约束
    alter与update的区别：有一句话很形象，alter相当于修改房间，而update相当于更新家具。alter操作的是表内数据的类型和约束，update用于操作数据。

    ```bash
    当表已被创建时，如需在 "P_Id" 列创建 UNIQUE 约束，请使用下面的 SQL：
    ALTER TABLE Persons ADD UNIQUE (P_Id)

    如需命名 UNIQUE 约束，并定义多个列的 UNIQUE 约束，请使用下面的 SQL 语法：
    ALTER TABLE Persons ADD CONSTRAINT uc_PersonID UNIQUE (P_Id,LastName)
    ```

- 撤销 UNIQUE 约束

    ```bash
    ALTER TABLE Persons DROP INDEX uc_PersonID
    ```

### SQL PRIMARY KEY 约束
- 约束唯一标识数据库表中的每条记录。
- 主键必须包含唯一的值。
- 主键列不能包含 NULL 值。
- 每个表都应该有一个主键，并且每个表只能有一个主键。
- CREATE TABLE 时的 SQL PRIMARY KEY 约束
    ```bash
    在 "Persons" 表创建时在 "P_Id" 列上创建 PRIMARY KEY 约束：
    CREATE TABLE Persons
    (
    P_Id int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Address varchar(255),
    City varchar(255),
    PRIMARY KEY (P_Id)
    )

    如需命名 PRIMARY KEY 约束，并定义多个列的 PRIMARY KEY 约束，请使用下面的 SQL 语法：
    CREATE TABLE Persons
    (
    P_Id int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Address varchar(255),
    City varchar(255),
    CONSTRAINT pk_PersonID PRIMARY KEY (P_Id,LastName)
    )
    ```
- ALTER TABLE 时的 SQL PRIMARY KEY 约束
    ```bash
    当表已被创建时，如需在 "P_Id" 列创建 PRIMARY KEY 约束，请使用下面的 SQL：
    ALTER TABLE Persons ADD PRIMARY KEY (P_Id)

    如需命名 PRIMARY KEY 约束，并定义多个列的 PRIMARY KEY 约束，请使用下面的 SQL 语法：
    ALTER TABLE Persons ADD CONSTRAINT pk_PersonID PRIMARY KEY (P_Id,LastName)
    ```

- 撤销 PRIMARY KEY 约束
    ```bash
    ALTER TABLE Persons DROP PRIMARY KEY
    ```

## SQL FOREIGN KEY 约束:一个表中的 FOREIGN KEY 指向另一个表中的 UNIQUE KEY(唯一约束的键)。
- 例子:
- Persons表：

| P_Id | LastName | FirstName | Address | City |
|:----:|:--------:|:---------:|:-------:|:----:|
| 1    | Hansen   | Ola       | Timoteivn 10 | Sandnes|
| 2    | Svendson | Tove      | Borgvn 23 | Sandnes |
| 3    | Pettersen | Kari     | Srotgt 20 | Stavanger |

- Orders表

| O_Id | OrderNo | P_Id |
|:----:|:-------:|:----:|
| 1    | 77999   | 3    |
| 2    | 88888   | 3    |
| 3    | 23545   | 2    |
| 4    | 34354   | 1    |

- Orders表中的P_Id列指向Persons表中的P_Id列。
- Persons表中的P_Id列是Persons表中的PRIMARY KEY。
- Orders表中的P_Id列是Orders表中的FOREIGN KEY。
- FOREIGN KEY约束用于预防破坏表之间连接的行为。
- FOREIGN KEY约束也能预防非法数据插入外键列，因为它必须是它指向的那个表中的值之一。

- CREATE TABLE 时的 SQL FOREIGN KEY 约束
    - 在 "Orders" 表创建时在 "P_Id" 列上创建 FOREIGN KEY 约束：
    ```bash
    CREATE TABLE Orders
    (
    O_Id int NOT NULL,
    OrderNo int NOT NULL,
    P_Id int,
    PRIMARY KEY (O_Id),
    FOREIGN KEY (P_Id) REFERENCES Persons(P_Id)
    )
    ```

    - 如果需要命名FOREIGN KEY约束，并定义多列的FOREIGN KEY约束，使用这样的SQL语法：

    ```bash
    CREATE TABLE Orders
    (
    O_Id int NOT NULL,
    OrderNo int NOT NULL,
    P_Id int,
    PRIMARY KEY (O_Id),
    CONSTRAINT fk_PerOrders FOREIGN KEY (P_Id) REFERENCES Persons(P_Id)
    )
    ```

- ALTER TABLE 时的 SQL FOREIGN KEY 约束:

- 当 "Orders" 表已被创建时，如需在 "P_Id" 列创建 FOREIGN KEY 约束，请使用下面的 SQL：

    ```bash
    ALTER TABLE Orders
    ADD FOREIGN KEY (P_Id)
    REFERENCES Persons(P_Id)
    ```
    
- 如需命名 FOREIGN KEY 约束，并定义多个列的 FOREIGN KEY 约束，请使用下面的 SQL 语法：

    ```bash
    ALTER TABLE Orders
    ADD CONSTRAINT fk_PerOrders
    FOREIGN KEY (P_Id)
    REFERENCES Persons(P_Id)
    ```
- 撤销 FOREIGN KEY 约束
    ```
    ALTER TABLE Orders
    DROP FOREIGN KEY fk_PerOrders
    ```
## SQL CHECK 约束
- CHECK 约束用于限制列中的值的范围。
    -  在 "Persons" 表创建时在 "P_Id" 列上创建 CHECK 约束。CHECK 约束规定 "P_Id" 列必须只包含大于 0 的整数。
    ```
    CREATE TABLE Persons
    (
    P_Id int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Address varchar(255),
    City varchar(255),
    CHECK (P_Id>0)
    )
    ```

    - 如需命名 CHECK 约束，并定义多个列的 CHECK 约束，请使用下面的 SQL 语法：
    ```
    CREATE TABLE Persons
    (
    P_Id int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Address varchar(255),
    City varchar(255),
    CONSTRAINT chk_Person CHECK (P_Id>0 AND City='Sandnes')
    )
    ```
- 已经创建数据表，要修改数据表check约束时
    ```
    ALTER TABLE Persons ADD CHECK (P_Id>0)
    ```

- 撤销CHeck约束
    ```
    ALTER TABLE Persons DROP CHECK chk_Person
    ```

## DEFAULT约束
- DEFAULT 约束用于向列中插入默认值。如果没有规定其他的值，那么会将默认值添加到所有的新记录。
    - 在 "Persons" 表创建时在 "City" 列上创建 DEFAULT 约束：
    ```
    CREATE TABLE Persons
    (  
    P_Id int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Address varchar(255),
    City varchar(255) DEFAULT 'Sandnes'
    )
    ```
    - 通过使用类似 GETDATE() 这样的函数，DEFAULT 约束也可以用于插入系统值：
    ```
    CREATE TABLE Orders
    (
    O_Id int NOT NULL,
    OrderNo int NOT NULL,
    P_Id int,
    OrderDate date DEFAULT GETDATE()
    )
    ```
- 数据表创建后添加default约束时
    ```
    ALTER TABLE Persons ALTER City SET DEFAULT 'SANDNES'
    ```

- 撤销DEFAULT约束
    ```
    ALTER TABLE Persons ALTER City DROP DEFAULT
    ```

## CREATE INDEX语句
- CREATE INDEX 语句用于在表中创建索引。在不读取整个表的情况下，索引使数据库应用程序可以更快地查找数据。
- 用户无法看到索引，索引只能用来加速搜索或查询
- 但是更新一个包含索引的表比更新一个没有索引的表要话费更多时间，因为索引本身也要被更新。
- 因此，理想的做法是仅仅在经常被搜索的列或表上创建索引。
- 语法
    - 创建一个简单索引：
    ```
    CREATE TABLE index_name ON table_name (column_name)
    ```
    - CREATE UNIQUE INDEX语法：在边上创建一个唯一的索引，不允许使用重复的值，唯一的索引意味着两个行不能拥有相同的索引值
    ```
    CREATE UNIQUE INDEX index_name ON table_name (column_name)
    ```
    - 实例：在Persons表的LastName列上创建一个名为Pindex的索引
    ```
    CREATE INDEX Pindex on Persons (LastName)
    CREATE INDEX Pindex ON Persons (LastName, FirstName)
    ```

## 撤销索引、撤销表以及撤销数据库
- DROP INDEX：删除表中的索引
    ```
    ALTER TABLE table_name DROP INDEX index_name
    ```
- DROP TABLE:用于删除表
    ```
    DROP TABLE table_name
    ```
- DROP DATABASW:用于删除数据库
    ```
    DROP DATABASE database_name
    ```
- TRUNCATE TABLE:仅仅删除表内数据，不删除表本身
    ```
    TRUNCATE TABLE table_name
    ```
- DELETE DROP TRUNCATE的区别
    - DROP删除表（数据库/索引），并释放空间，将表删除得一干二净
    - TRUNCATE删除表内容，但不释放空间，表结构还在
    - DELETE仅删除表内容，但保留表定义，不释放空间。
    - 速度：DROP > TRUNCATE > DELETE
    - 安全性：小心使用DROP和truncate，尤其没有备份的时候。
    - 使用上删除部分数据用delete，加上where语句，想删除表用drop，想保留表而删除所有数据用TRUNCATE。

## ALTER TABLE语句:在已有的表中添加、删除或修改列
- 在已有表中添加列
    ```
    ALTER TABLE table_name ADD column_name datatype
    ```
- 删除已有表中的列（有些数据库不允许这种在数据库表中删除列的方式）
    ```
    ALTER TABLE table_name DROP COLUMN column_name
    ```
- 改变列的数据类型
    ```
    ALTER TABLE table_name MODIFY COLUMN column_name datatype
    ```
- 实例
- Persons表

| P_Id | LastName | FirstName | Address | City |
|:----:|:--------:|:---------:|:-------:|:----:|
| 1    | Hansen   | Ola       | Timoteivn 10 | Sandnes|
| 2    | Svendson | Tove      | Borgvn 23 | Sandnes |
| 3    | Pettersen | Kari     | Srotgt 20 | Stavanger |

- 在 "Persons" 表中添加一个名为 "DateOfBirth" 的列：
```
ALTER TABLE Persons ADD DateOfBirth data
这里DataOfBirth的类型是data，可以存放日期。
```
- 现在Persons表内容如下：
| P_Id | LastName | FirstName | Address | City | DataOfBirth |
|:----:|:--------:|:---------:|:-------:|:----:| |
| 1    | Hansen   | Ola       | Timoteivn 10 | Sandnes| |
| 2    | Svendson | Tove      | Borgvn 23 | Sandnes | |
| 3    | Pettersen | Kari     | Srotgt 20 | Stavanger | |

- 改变数据类型，将DataOfBirth改为存放年份的列：
```
ALTER TABLE Persons ALTER COLUMN DataOfBirth year
```

- DROP COULUMN实例：
```
ALTER TABLE Persons DROP COLUMN DataOfBirth
```
- Persons数据表现在的内容是：

| P_Id | LastName | FirstName | Address | City |
|:----:|:--------:|:---------:|:-------:|:----:|
| 1    | Hansen   | Ola       | Timoteivn 10 | Sandnes|
| 2    | Svendson | Tove      | Borgvn 23 | Sandnes |
| 3    | Pettersen | Kari     | Srotgt 20 | Stavanger |

## AUTO_INCREMENT字段
- 通常在每次插入新纪录时，希望能够自动创建主键的值，可以通过创建一个auto_increment字段
    - 把Persons表中的ID列定义为auto increment主键字段：
    ```
    CREATE TABLE Persons
    (
        ID int NOT NULL AUTO_INCREMENT,
        LastName varchar(255) NOT NULL,
        FirstName varchar(255),
        Address varchar(255),
        City varchar(255),
        PRIMARY KEY (ID)
    )

    auto_increment开始值是1，每条递增1
    要以其他值作为起始值，使用这样的SQL语句：
    ALTER TABLE Persons AUTO_INCREMENT=100

    INSERT INTO Persons(FirstName, LastName) VALUES('Lars', 'Monsen')
    ```

## SQL Date 函数：日期
- MySQL Date函数：

| 函数 | 描述 |
|:----:|:-----:|
| NOW() | 返回当前的日期和时间 |
| CURDATE() | 返回当前的日期 |
| CURTIME() | 返回当前时间 |
| DATE() | 提取日期或日期/时间表达式的日期部分 |
| EXTRACT() | 返回日期/时间的单独部分 |
| DATE_ADD() | 想日期减去指定的时间间隔 |
| DATE_SUB() | 从日期减去指定的时间间隔 |
| DATEDIFF() | 返回连个日期之间的天数 |
| DATE_FORMAT() | 用不同格式显示日期/时间 |

- SQL Date数据类型
    - DATE-格式：YYYY-MM-DD
    - DATETIME-格式：YYYY-MM-DD HH:MM:SS
    - TIMESTAMP-格式：YYYY-MM-DD HH:MM:SS
    - YEAR-格式：YYYY或YY


## SQL通用数据类型
- [参考链接][6]
- [参考连接][7]


## SQL AVG()函数：返回数值列的平均值
- 从access_log表的count列获取平均值
    ```bash
    mysql> select avg(count) as CountAverage from access_log;
    +--------------+
    | CountAverage |
    +--------------+
    |     174.3333 |
    +--------------+
    1 row in set (0.00 sec)

    选择访问量高于平均值的site_id和count
    mysql> select site_id, count from access_log where count > (select avg(count) from access_log);
    +---------+-------+
    | site_id | count |
    +---------+-------+
    |       1 |   230 |
    |       5 |   205 |
    |       3 |   220 |
    |       5 |   545 |
    |       3 |   201 |
    +---------+-------+
    5 rows in set (0.00 sec)
    ```
## COUNT()函数
- 返回匹配指定条件的行数
    - 返回指定列的值的数目（NULL不计入）：
    ```
    SELECT COUNT(column_name) FROM table_name
    ```

    - 返回表中的记录数
    ```
    SELECT COUNT(*) FROM table_name
    ```

    - 返回指定列的不同值得数目
    ```
    SELECT COUNT(DISTINCT column_name) FROM table_name
    ```

    ```bash
    mysql> select count(site_id) as nums from access_log;
    +------+
    | nums |
    +------+
    |    9 |
    +------+
    1 row in set (0.00 sec)

    计算access_log表中不同site_id的记录数：
    mysql> select count(distinct site_id) as nums from access_log;
    +------+
    | nums |
    +------+
    |    5 |
    +------+
    1 row in set (0.00 sec)
    ```


## LIMIT
- 返回指定列中第一个记录的值
    ```bash
    mysql> select name as FirstSite from websites limit 1;
    +-----------+
    | FirstSite |
    +-----------+
    | Google    |
    +-----------+
    1 row in set (0.00 sec)
    ```
- 返回指定列中最后一个值
    ```bash
    mysql> select name as LastName from websites order by id desc limit 1;
    +---------------+
    | LastName      |
    +---------------+
    | stackoverflow |
    +---------------+
    1 row in set (0.00 sec)
    ```
    
## MAX()函数
- 返回指定列的最大值
    ```bash
    mysql> select max(alexa) as max_alexa from websites;
    +-----------+
    | max_alexa |
    +-----------+
    |      4689 |
    +-----------+
    1 row in set (0.00 sec)
    ```
## MIN()函数
- 返回指定列的最小值
    ```bash
    mysql> select min(alexa) as min_alexa from websites;
    +-----------+
    | min_alexa |
    +-----------+
    |         0 |
    +-----------+
    1 row in set (0.00 sec)
    ```

## SUM()函数
- 返回列的总数
    - 查找access_log表中的count字段总数
    ```bash
    mysql> select sum(count) as nums from access_log;
    +------+
    | nums |
    +------+
    | 1569 |
    +------+
    1 row in set (0.00 sec)
    ```

## GROUP BY语句
- 用于结合聚合函数，根据一个或多个列对结果集进行分组。
    ```bash
    mysql> select site_id, sum(access_log.count) as nums from access_log group by site_id;
    +---------+------+
    | site_id | nums |
    +---------+------+
    |       1 |  275 |
    |       2 |   10 |
    |       3 |  521 |
    |       4 |   13 |
    |       5 |  750 |
    +---------+------+
    5 rows in set (0.00 sec)


    mysql> select websites.name, count(access_log.aid) as nums from access_log left join websites on access_log.site_id=websites.id group by websites.name;
    +--------------+------+
    | name         | nums |
    +--------------+------+
    | Facebook     |    2 |
    | Google       |    2 |
    | 微博         |    1 |
    | 淘宝         |    1 |
    | 菜鸟教程     |    3 |
    +--------------+------+
    5 rows in set (0.00 sec)
    ```

## HAVING子句

```bash
select websites.name, sum(access_log.count) as nums from websites inner join access_log on websites.id=access_log.site_id group by websites.name having sum(access_log.count) > 200;
select websites.name, sum(access_log.count) as nums from websites inner join access_log on access_log.site_id=websites.id group by websites.name having sum(access_log.count) > 200;


mysql> select websites.name, sum(access_log.count) as nums from websites inner join access_log
-> on websites.id=access_log.site_id
-> where websites.alexa < 200
-> group by websites.name
-> having sum(access_log.count) > 200;
+----------+------+
| name     | nums |
+----------+------+
| Facebook |  750 |
| Google   |  275 |
+----------+------+
2 rows in set (0.00 sec)
```

## UCASE()函数
- 把字段值转换成大写
    ```bash
    mysql>     select ucase(name) as site_title, url from websites;
    +---------------+---------------------------+
    | site_title    | url                       |
    +---------------+---------------------------+
    | GOOGLE        | https://www.google.cm/    |
    | 淘宝          | https://www.taobao.com/   |
    | 菜鸟教程      | http://www.runoob.com/    |
    | 微博          | http://weibo.com/         |
    | FACEBOOK      | https://www.facebook.com/ |
    | STACKOVERFLOW | http://stackoverflow.com/ |
    +---------------+---------------------------+
    6 rows in set (0.00 sec)
    ```

## LCASE()函数
- 把字段值转化为小写
    ```bash
    mysql>     select lcase(name) as site_title, url from websites;
    +---------------+---------------------------+
    | site_title    | url                       |
    +---------------+---------------------------+
    | google        | https://www.google.cm/    |
    | 淘宝          | https://www.taobao.com/   |
    | 菜鸟教程      | http://www.runoob.com/    |
    | 微博          | http://weibo.com/         |
    | facebook      | https://www.facebook.com/ |
    | stackoverflow | http://stackoverflow.com/ |
    +---------------+---------------------------+
    6 rows in set (0.00 sec)
    ```

## MID()函数
- 从文本字段中提取字符
    ```bash
    mysql> select mid(name, 1, 3) as ShortTitle from websites;
    +------------+
    | ShortTitle |
    +------------+
    | Goo        |
    | 淘宝       |
    | 菜鸟教     |
    | 微博       |
    | Fac        |
    | sta        |
    +------------+
    6 rows in set (0.00 sec)
    ```

## LEN()函数
- 返回文本字段中值的长度
    ```bash
    mysql> select name, LENGTH(url) as LengthOfUrl from websites;
    +---------------+-------------+
    | name          | LengthOfUrl |
    +---------------+-------------+
    | Google        |          22 |
    | 淘宝          |          23 |
    | 菜鸟教程      |          22 |
    | 微博          |          17 |
    | Facebook      |          25 |
    | stackoverflow |          25 |
    +---------------+-------------+
    6 rows in set (0.00 sec)
    ```

## ROUND()函数
- 用于把数值字段舍入为指定的小数位数
    ```bash
    返回参数X的四舍五入的一个整数。
    mysql> select round(-1.58);
    +--------------+
    | round(-1.58) |
    +--------------+
    |           -2 |
    +--------------+
    1 row in set (0.00 sec)

    ROUND(X,D)：返回参数X的四舍五入的有 D 位小数的一个数字。如果D为0，结果将没有小数点或小数部分。
    mysql> select round(1.2786, 1);
    +------------------+
    | round(1.2786, 1) |
    +------------------+
    |              1.3 |
    +------------------+
    1 row in set (0.00 sec)
    ```

## NOW()函数
- 返回当前系统的日期和时间
    ```bash
    mysql>     select name, url, now() as date from websites;
    +---------------+---------------------------+---------------------+
    | name          | url                       | date                |
    +---------------+---------------------------+---------------------+
    | Google        | https://www.google.cm/    | 2018-10-21 17:16:57 |
    | 淘宝          | https://www.taobao.com/   | 2018-10-21 17:16:57 |
    | 菜鸟教程      | http://www.runoob.com/    | 2018-10-21 17:16:57 |
    | 微博          | http://weibo.com/         | 2018-10-21 17:16:57 |
    | Facebook      | https://www.facebook.com/ | 2018-10-21 17:16:57 |
    | stackoverflow | http://stackoverflow.com/ | 2018-10-21 17:16:57 |
    +---------------+---------------------------+---------------------+
    6 rows in set (0.00 sec)

    ```

## FORMAT()函数
- 用于对字段进行格式化
    ```bash
    mysql> select name, url, date_format(now(), '%Y-%m-%d') as date from websites;
    +---------------+---------------------------+------------+
    | name          | url                       | date       |
    +---------------+---------------------------+------------+
    | Google        | https://www.google.cm/    | 2018-10-21 |
    | 淘宝          | https://www.taobao.com/   | 2018-10-21 |
    | 菜鸟教程      | http://www.runoob.com/    | 2018-10-21 |
    | 微博          | http://weibo.com/         | 2018-10-21 |
    | Facebook      | https://www.facebook.com/ | 2018-10-21 |
    | stackoverflow | http://stackoverflow.com/ | 2018-10-21 |
    +---------------+---------------------------+------------+
    6 rows in set (0.00 sec)
    ```

[1]: https://coolshell.cn/wp-content/uploads/2011/01/Inner_Join.png
[2]: http://www.runoob.com/wp-content/uploads/2013/09/img_rightjoin.gif
[3]: https://coolshell.cn/wp-content/uploads/2011/01/Full_Outer_Join.png
[4]: https://coolshell.cn/articles/3463.html
[5]: https://coolshell.cn/wp-content/uploads/2011/01/SQL-Join.jpg
[6]: http://www.runoob.com/sql/sql-datatypes-general.html
[7]: http://www.runoob.com/sql/sql-datatypes.html
