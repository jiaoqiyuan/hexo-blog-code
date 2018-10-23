---
title: MySQL基础语法小结
date: 2018-03-11 17:16:43
tags:
categories: 数据库
thumbnail: https://zhidao91.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2015/11/databaseDevelopment.jpg
---

## Mysql基本语法

显示当前服务器版本：
```
SELECT VERSION();
```

显示当前日期：
```
SELECT NOW();
```
显示当前用户：
```
SELECT USER();
```

```
创建数据库：
CREAT {DATABASE | SCHEMA} [IF NOT EXISTS] db_name [DEFAULT] CHARACTER SET [=] charset_name;
查看数据库：SHOW {DATABASE | SCHEMAS} [LIKE ‘pattern’ WHERE expr]
查看数据库数据格式：SHOW CREATE DATABASE db_name;
修改数据库：ALTER DATABASE t2 CHARACTER SET=gbk;
删除数据库：DROP DATABASE IF EXISTS t2
```

```
创建数据表：CREATE TABLE IF NOT EXISTS table_name(column_name data_type, ……)
查看数据表：SHOW TABLES [FROM db_name] [LIKE ‘pattern’ | WHERE expr]
查看数据表结构：SHOW COLUMNS FROM tbl_name
插入记录：INSERT INTO tbl_name [(col_name, ……)] VALUES(val,……)
查找数据内容：SELECT expr,…… FROM tbl_name
```

自动编号：AUTO_INCREMENT(必须和PRIMARY KEY一起使用)
非空：NOT NULL
主键：PRIMARY KEY
唯一约束：UNIQUE KEY

** 设置数据表默认存储引擎 **
Mysql配置文件：
```
/etc/mysql/mysql.conf.d/mysqld.cnf 中mysqld下增加一行：
default-storage-engine=INNODB
```
创建外键约束的数据表：
```
CREATE TABLE provinces( id SMALLINT UNSIGNED PRIMARY KEY AUTO_INCREMENT, pname VARCHAR(20) NOT NULL );
CREATE TABLE users( id SMALLINT UNSIGNED PRIMARY KEY AUTO_INCREMENT, username VARCHAR(20) NOT NULL, pid SMALLINT UNSIGNED, FOREIGN KEY (pid) REFERENCES provinces (id) );
```
** 注意： **
1.	父表（provinces）和子表（users）必须使用相同的存储引擎
2.	数据表的存储引擎只能为INNODB
3.	外键列（pid）和参照列（id）必须具备相似的数据类型（数字的长度有无符号为敏感，字符长度可以不同）
4.	外键列（pid）和参照列（id）必须创建索引，如果外键列不存在索引的话mysql会自动创建索引，参照列也会自动创建索引。

** 外键约束的参照操作 **
** CASCADE: **在父表中删除或更新且自动删除或更新子表中匹配的行
** SET NULL: **从附表中删除或者更新行，并设置子表中的外键列为NULL。如果使用该项，必须保证子表列没有指定NOT NULL。
** RESTRICT： **拒绝父表的删除或者更新操作。
** NO ACTION： **标准sql关键字，与MySQL中的RESTRICT相同。
物理外键约束和逻辑外键约束：物理指的是mysql语句定义的约束规则，逻辑指的是代码定义的逻辑规则。

列级约束（较多用到）：对一个数据列建立的约束，可以在列定义时声明也可以在列定义后声明
表级约束（较少用到）：对多个数据列建立的约束，只能在列定义后声明。

增加/删除列：
添加单列：
```
ALTER TABLE tbl_name ADD [COLUMN] col_name column_definition [FIRST | AFTER col_name]
Eg : ALTER TABLE users1 ADD age TINYINT UNSIGNED NOT NULL DEFAULT 10;
ALTER TABLE users1 ADD truename VARCHAR(20) NOT NULL FIRST;
```
添加多列：
```
ALTER TABLE tbl_name ADD [COLUMN] (col_name column_definition ……)

```
删除列：
```
ALTER TABLE tbl_name DROP [COLUMN] col_name
Eg: ALTER TABLE users1 DROP truename;
```
	
删除多列：
```
ALTER TABLE users1 DROP passwd, DROP age;
```

添加主键约束：
```
ALTER TABLE tbl_name ADD [CONSTRAINT [symbol]] PRIMARY KEY [index_type](index_col_ name,……)
ALTER TABLE users2 ADD CONSTRAINT PK_users2_id PRIMARY KEY (id);
```
添加唯一约束：
```
ALTER TABLE users2 ADD UNIQUE (username);
```
添加外键约束：
```
ALTER TABLE users2 ADD FOREIGN KEY (pid) REFERENCES provinces (id);
```
添加或者删除默认约束：
```
ALTER TABLE users2 ALTER age SET DEFAULT 15;
ALTER TABLE users2 ALTER age DROP DEFAULT;
```
修改数据表
修改列定义：
```	
ALTER TABLE users2 MODIFY id SMALLINT NOT NULL FIRST;//将id字段放到所有字段的最前面。
```
修改数据类型：
```	
ALTER TABLE users2 MODIFY id TINYINT NOT NULL FIRST;
```
修改列名称：
```	
ALTER TABLE users2 CHANGE pid p_id TINYINT UNSIGNED NOT NULL;
```
修改数据表名字（建议不要随意更改）：
```	
ALTER TABLE users2 RENAME users3;
RENAME TABLE users3 TO users2;
```
插入记录
```
CREATE TABLE users( id SMALLINT UNSIGNED PRIMARY KEY AUTO_INCREMENT, username VARCHAR(20) NOT NULL, passwd VARCHAR(20) NOT NULL, age TINYINT UNSIGNED NOT NULL DEFAULT 10, sex BOOLEAN );
INSERT INTO users VALUES(NULL, 'Jhon', '456', 26, 0);
INSERT INTO users VALUES(DEFAULT, 'Jhon', '456', 3*7-5, 0);
INSERT INTO users VALUES(DEFAULT, 'Jhon', '456', 3*7-5, 0),((DEFAULT, 'Jony', '456', 3*7-5, 0);
```
第二种插入方法：
```
INSERT users SET username='Ben', passwd='789';
```

更新数据表：
```
UPDATE users set age = age+5;
UPDATE users SET age=age-id, sex=0;
UPDATE users SET age=age+10 WHERE id%2=0;
```
** 删除记录 **
 
单表删除：
```
DELETE FROM users WHERE id=6;
```

查找记录
```
SELECT sex FROM users GROUP BY sex;
SELECT sex FROM users GROUP BY sex HAVING count(id) >=2;
SELECT * FROM users ORDER BY id DESC;
```
限制记录返回的数量
```
SELECT * FROM users LIMIT 2;
SELECT * FROM users LIMIT 2,2;	
```
以特定编码格式显示数据类型
```
SET NAMES gbk
```

多表更新:
```
UPDATE tdb_goods(要更新的表) INNER JOIN tdb_goods_cates(要更新的列) ON goods_cate = cate_name（更新条件）SET goods_cate = cate_id（更新的值）
```

复合命令，创建数据表的同时将查询结果写入数据表：
```
CREATE TABLE IF NOT EXISTS tbl_name [create_definition,……] Select_statement
```
