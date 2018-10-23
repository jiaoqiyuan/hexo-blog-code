---
title: 大数据学习笔记-Hive
date: 2018-10-14 12:30:11
tags: 
categories: 大数据
thumbnail: http://cdn1.itpro.co.uk/sites/itpro/files/2017/02/bigstock-big_data-159884873.jpg
---


# 大数据仓库Hive

## Hive产生背景

- MapReduce编程不便利
- HDFS上的文件缺少Schema（一些文件属性信息），无法使用sql查询。使用不方便

## hive是什么

- 由Facebook开源，用于解决海量结构化的日志数据统计问题

- 构建在Hadoop之上的数据仓库

- Hive定义了一种类SQL查询语言：HQL

- 通常用于进行离线数据处理

- 底层支持多种不同执行引擎（支持Spark、Tez）

- 支持多种不同的压缩格式、存储格式以及自定义函数
    - 压缩：GZIP、LZO、Snappy、BZIP2
    - 存储：TextFile、SequenceFile、RCFile、ORC、Parquet
    - UDF：自定义函数

## 为什么要使用Hive

- 简单、容易上手（使用类似SQL查询语言HQL）

- 为超大数据集设计的计算/存储扩展能力（MR计算、HDFS存储）

- 统一的元数据管理（可与Presto/Impala（基于内存，数据较快）/SparkSQL等共享数据）


## Hive体系架构
略

## Hive配置

### 配置系统环境变量
```
export HIVE_HOME=/home/jony/applications/hive-1.1.0-cdh5.7.0
export PATH=$SCALA_HOME/bin:$PATH
```

### 安装Mysql

```
sudo apt-get install mysql-server
sudo apt install mysql-client
sudo apt install libmysqlclient-dev
```

检查是否安装成功：
```
sudo netstat -tap | grep mysql
```

设置mysql密码：
去s/etc/mysql/debian.cnf这个文件查看默认用户名和密码：
```
sudo vim /etc/mysql/debian.cnf

user     = debian-sys-maint
password = 54ApxY9C9NncqKn8
```

使用提示的用户名和密码登录：
```
mysql -u debian-sys-maint -p
```
重新设置root用户密码为123456：
```
update mysql.user set authentication_string=password('123456') where user='root'and Host = 'localhost'; st';

```

设置mysql远程可以访问：
```
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```
注销这句：
```
# bind-address          = 127.0.0.1
```

mysql删除方法：
```
sudo apt-get autoremove --purge mysql-server-5.x
sudo apt-get remove mysql-server
sudo apt-get autoremove mysql-server
sudo apt-get remove mysql-common

可能会有残留，那就：
 dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P
```

修改mysql中root用户密码：

- 打开/etc/mysql/debian.cnf：
```
sudo vim /etc/mysql/debian.cnf
```

- 使用默认用户名和密码登录：
```
user     = debian-sys-maint
password = 54ApxY9C9NncqKn8
```

执行：
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
```
退出保存即可。

### 配置mysql信息到Hive

- 创建hive-set.xml，内容如下：
因为mysql连接时需要指定是否使用SSL，所以要加上&amp;useSSL=true。
```
<configuration>
        <property>
                <name>javax.job.option.ConnectionURL</name>
                <value>jdbc:mysql://localhost:3306/sparksql?createDatabaseIfNotExist=true&amp;useSSL=true</value>
        </property>
        <property>
                <name>javax.job.option.ConnectionDriverName</name>
                <value>com.mysql.jdbc.Driver</value>
        </property>
        <property>
                <name>javax.job.option.ConnectionUserName</name>
                <value>root</value>
        </property>
        <property>
                <name>javax.job.option.ConnectionPassword</name>
                <value>123456</value>
        </property>

</configuration>
```

- 拷贝mysql驱动包到hive中的lib中去。

- 配置/home/jony/applications/hive-1.1.0-cdh5.7.0/conf/hive-env.sh
```
HADOOP_HOME=/home/jony/applications/hadoop-2.6.0-cdh5.7.0
export HIVE_CONF_DIR=/home/jony/applications/hive-1.1.0-cdh5.7.0/confexport HIVE_AUX_JARS_PATH=/home/jony/applications/hive-1.1.0-cdh5.7.0/lib
```

### 启动Hive

- 直接运行hive命令： 
```
hive
```


### 使用hive

- 创建表
```
create table hive_wordcount(context string);
```

- 在数据库中查询创建的表信息：
```
mysql -uroot -p123456

use sparksql;
show tables;
select * from TBLS;

+--------+-------------+-------+------------------+-------+-----------+-------+----------------+---------------+--------------------+--------------------+----------------+
| TBL_ID | CREATE_TIME | DB_ID | LAST_ACCESS_TIME | OWNER | RETENTION | SD_ID | TBL_NAME       | TBL_TYPE      | VIEW_EXPANDED_TEXT | VIEW_ORIGINAL_TEXT | LINK_TARGET_ID |
+--------+-------------+-------+------------------+-------+-----------+-------+----------------+---------------+--------------------+--------------------+----------------+
|      1 |  1539488456 |     1 |                0 | jony  |         0 |     1 | hive_wordcount | MANAGED_TABLE | NULL               | NULL               |           NULL |
+--------+-------------+-------+------------------+-------+-----------+-------+----------------+---------------+--------------------+--------------------+----------------+
1 row in set (0.01 sec)


select * from COLUMNS_V2;
+-------+---------+-------------+-----------+-------------+
| CD_ID | COMMENT | COLUMN_NAME | TYPE_NAME | INTEGER_IDX |
+-------+---------+-------------+-----------+-------------+
|     1 | NULL    | context     | string    |           0 |
+-------+---------+-------------+-----------+-------------+
1 row in set (0.00 sec)

``` 

- 加载本地数据到hive表
```
LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)]

load data local inpath '/home/jony/tmp/imooc_data/data/hello.txt' into table hive_wordcount;

select * from hive_wordcount;
```

- 统计词频
```
select word, count(1) from hive_wordcount lateral view explode(split(context, '\t')) wc as word group by word;

lateral view explode():把每行记录按照指定分隔符进行拆解
```
hive sql提交执行后会生成MR作业，在yarn上运行。

- 创建员工表

```
create table emp(
empno int,
ename string,
job string,
mgr int,
hiredate string,
sal double,
comm double,
deptno int
) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';

create table dept(
deptno int,
dname string,
localtion string
) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';
```

加载本地数据：
```
load data local inpath '/home/jony/tmp/imooc_data/data/emp.txt' into table emp;
load data local inpath '/home/jony/tmp/imooc_data/data/dept.txt' into table dept;
```

求每个部门的人数
```
select deptno, count(1) from emp group by deptno;
```
