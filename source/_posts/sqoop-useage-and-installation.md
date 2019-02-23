---
title: Sqoop的安装配置及原理
date: 2019-01-30 09:43:10
tags: [大数据, Sqoop]
categories: 大数据
thumbnail: https://hadoopsters.files.wordpress.com/2015/09/sqoopflow.png
photos: https://hadoopsters.files.wordpress.com/2015/09/sqoopflow.png
---

> Sqoop是为解决传统数据仓库在应对大数据时的弊端而创建的数据导入导出工具，可以将数据从传统数据仓库中导入到分布式文件系统或者大数据数据仓库（如Hive）。

<!--more-->

## Sqoop出现的历史背景

### 业务数据系统

> 随着互联网的发展，各行各业都在产生数据巨大的数据，各式应用渗透到生活的各个方面，进而产生了各种类型的数据。

大量数据的出现为商业公司进行市场分析和决策提供了重要依据，也帮助产生更好的产品。数据也是运营的关键，数据通常存储在数据库中，为后续的数据分析提供了方便。

**电商系统的数据**

- 用户信息

  存储一些比如姓名、年龄、性别、地域、手机、注册时间等信息。

- 商品信息

  比如商品的名称、类别、品牌、价格、型号、尺寸等等。

- 订单信息

  包括订单号、商品、金额、商家、客户、时间等。

- 仓储信息

  比如仓储地点给、货物名称、库存数量、进货时间、货物位置、仓库名称等。

知道了这些信息就可以创建用户画像，进行有针对性的信息推送，比如云音乐。

### 数据同步与传统数据仓库

**数据同步**

分析业务系统的数据，一种简单的操作方式是直接访问业务系统的数据库，如下图所示：

![数据同步][1]

业务系统和数据分析应用依赖同一个数据库，数据库既要处理OLTP事务型操作（如线上的用户请求），又要处理OLAP的查询（如离线数据分析）。

两种业务使用同一个数据库的优点是可以保证数据的实时性，业务系统产生的数据变化可以立即反馈到OLAT的查询中，没有任何数据延迟。

这种框架的缺点也十分明显，数据库同时要处理两种业务需求，会产生额外的压力，进而影响线上业务的服务质量，就是因为又这个缺点，这个框架在生产环境中很少使用。

**主从复制框架**

为克服多种业务直接访问同一个数据库引起的问题，避免线上业务和数据分析业务在数据库层面的互相影响，一般会使用下面这种数据同步框架：

![主从复制][2]

在线应用服务访问数据库的Master节点，数据分析服务访问数据库的Slave节点，Master和Slave之间使用主从同步的方式保证数据的一致性，这种系统可以保证在不影响线上系统的情况下进行数据的实时分析业务。

该框架的核心在于主从同步，那么主从同步的原理是什么？

以MySQL为例，每条数据库操作请求都会记录在binlog中，操作请求的时间、操作的表、修改的字段值等信息都会被记录下来，从节点Slave只要拿到了Master节点的binlog，就可以按Master节点的操作对Slave节点重复进行一次数据库的相同操作，等于将Master节点的数据库复制了一份到从节点。

这个框架解决了在线和离线不同业务系统相互影响的问题，但是所查询的数据还是仅限于某个应用系统中，未能实现不同服务的数据打通，生成统一的数据视图，所以该框架还不能完全满足数据分析师的需求，也就是数据孤岛问题。

**传统数据仓库**

Hadoop出现之前，为解决数据孤岛问题，需要使用传统的数据仓库工具和聚合业务工具来解决。

![传统数据仓库][3]

MySql、Oracle数据库的数据需要先通过ETL清洗、聚合，导入到数据仓库服务中，然后数据仓库作为统一数据源，直接对接报表系统、数据挖掘、数据分析等服务。

传统数据仓库服务包括 [Teradata][4] 、 [ODS][5] 等，他们都提供了SQL语法支持，学习门槛较低。另外还支持JDBC、ODBC连接，接入也比较方便。这使得当时数据仓库变得十分流行。

传统数据仓库相对于现在使用场景来说也有自己的缺点：

是闭源系统，代码不公开，定位和解决问题比较困难，只能依赖文档，有很大的局限性。有BUG只能等待官方修复。

成本昂贵，部署和服务支持的费用都很高。

扩容复杂，集群上限较低。

性能有瓶颈，比如社交推荐可能认识好友的场景，随着用户数量的增长，由于扩容复杂，集群计算能力越来越难以满足要求。

Hadoop不存在这样的问题，所以打通传统数据仓库与Hadoop之间的数据流动就变得非常有必要。

## Sqoop功能与架构

### Sqoop介绍

为解决传统数据仓库的痛点，比如系统闭源、成本昂贵、扩容复杂、性能有明显瓶颈，Apache社区开源了Sqoop这个数据创数框架。

- Sqoop由Cloudera开发并贡献给Apache社区。

- Sqoop是Apache顶级项目。

- 结构化数据与大数据系统间的数据同步。

- 有两个版本：1.4.x和1.99.x，其中1.99.x是基于sqoop2的框架，但功能还没完全覆盖1.4.x的sqoop，所以建议还是先使用1.4.x。

### Sqoop功能

- 将关系型数据库数据导入HDFS，解决HDFS的数据来源问题，使数据能够方便地进入HDFS。

- 支持HDFS数据导出到关系型数据库，Hadoop计算出的数据可以方便地写回到数据库中。

- 支持关系型数据库直接将数据导入到Hive。

- Sqoop是批处理类型的任务，不是常驻服务，不需要像WEB服务一样常驻运行，在需要的时候提交任务就可以完成数据的导入导出。

- Sqoop是使用命令行进行任务的提交，提供类似于Shell的脚本的Sqopp命令，提交方式很简便。

- Sqoop支持各种存储类型，包括行存、列存以及各种数压缩算法。

### Sqoop架构

如果自己设计一个数据库数据导入导出工具的话，会怎么实现呢？首先比较简单而且也容易想到的方式是使用JDBC，将数据从数据库中拉取出来然后写入HDFS，这种方法简单易行，但是缺点也很明显，数据量较大时，效率不高。

Sqoop是怎么做的呢，它其实是依赖MapReduce的计算框架，将数据导入并行化，采用分而治之的思想，每个Map只处理一部分数据，然后由Reduce将Map的中间结果聚合起来。

![Sqoop架构][6]

其实并不需要Reduce，只是用Map就可以完成数据的并行导入导出工作了，每个Map使用JDBC将数据从数据库抽取出来，写入到HDFS，就可以完成数据的导入任务。

由于使用了MapReduce并发计算的特性，Sqoop可以显著提高数据导入导出的效率。在实际使用中，Sqoop一般不会称为性能的瓶颈，在磁盘读写和宽带都不是瓶颈的前提下，数据的导入导出效率往往取决于DB的性能。

上面的框架中Sqoop和数据库之间使用的是JDBC，所以逻辑上讲，所有支持JDBC操作的数据库都支持使用Sqoop将数据导入到HDFS中，当然各个数据库之间会存在差异，目前在不改造Sqoop的前提下，Sqoop支持的数据库有：MySQL，Oracle，SqlServer， postgreSQL，DB2等，基本涵盖了所有主流的数据库。

### Sqoop任务执行流程

Sqoop执行流程如下图所示：

![Sqoop任务执行流程][7]

#### 数据对象生成

在实际开发过程中，在使用JDBC访问数据库里数据的时候，通常会使用一个Java Bean存储数据库数据，JavaBean中每个成员变量对应数据库中的一个Column（列），并且JavaBean中会有大量的get、set函数来获取和修改数据，程序运行过程中，JavaBean可以用来保留数据，这样，对数据的操作就转换成了对JavaBean对象的操作。

Sqoop也使用类似的机制，在导入导出前会现根据数据库的数据结构，生成对应的JavaBean对象，加入有一个名为Order的数据表，存放的是订单信息，其中id是主键，在导入过程中就会生成一个Order的JavaBean，JavaBean里的属性分别对应于数据库中的几个Column（列）。

![数据对象生成][8]

生成JavaBean的一个关键问题是怎样做类型映射，也就是把数据库中的列数据类型和JavaBean中的成员变量的类型映射起来，Sqoop中定义的映射关系如下：

| SQL TYPE | JAVA TYPE |
|:--------:|:---------:|
| INTEGER | java.lang.Integer |
| VARCHAR | java.lang.String |
| LONGVARCHAR | java.lang.String |
| NUMERIC | java.math.BigDecimal |
| DECIMAL | java.math.BigDecimal |
| BOOLEAN | java.lang.Boolean |
| DATE | java.sql.Date |
| ... | ... |

#### 数据导入Hive中

![数据导入hive][9]

如果命令指定要新建HIVE Table，Sqoop会先生成HIVE table的定义语句，Hive table的column和数据库的column建立一一映射的关系，然后Sqoop会生成一个rawData语句，rawdata语句会在Hive MateStore中注册元数据，并进行数迁移。

上面的Hive的建表语句和rawdata语句都会被写入到一个script脚本中，最后会sqoop启动.hive命令执行刚刚生成的script脚本，提交Hive任务，完成Hive导出。

## Sqoop原理

> 这里将会讲解Sqoop是怎么尽量在MapReduce中均衡的切分数据的

下面说的导入和导出都是针对HDFS来说，也即导入是指向HDFS到处数据，导出是指从HDFS导出数据。

### 数据导入

现有一个Order表，需要将其数据导入到HDFS上，主键是id。如下图所示：

![数据导入][10]

Sqoop在解析完命令参数后会查找参数指定的分片字段的数据范围，假设用户指定的分片字段为id，通过jdbc接口可以找到id的最大值和最小值，分别是5000和1。

数据范围除以map数量可以得到每个Map传输的数据id范围，假设map数是5个，那么每个map的范围就是5000/5=1000，如上图所示。每个map使用SQL语句进行筛选，筛选方法就是在SQL中添加where条件，比如第一个map的where条件就可以写为id >= 1 and id <= 1000。

每个map都会在HDFS上生成一个对应的文件，存储向HDFS导入的数据。

### 数据导出

数据导出跟导入正好相反，是把HDFS上的文件导出到数据库中，过程如下图所示：

![数据导出][11]

导出任务的数据划分完全依赖于HDFS的 FileInputFormat 的 split 划分，默认情况下会使用 CombineInputFormat ，尽量把同一个节点的数据使用同一个map来处理，这样可以众多小文件导致的创建map任务的开销，提高任务的执行效率。

但是如果导出的数据文件是压缩格式，且这种压缩格式不能再分片，那么文件就会当以单个文件进行分片，每一个文件会生成一个map进行导出。

导出任务的分片逻辑比较复杂，用户指定的并发参数往往不会生效。

**Sqoop是基于MapReduce框架的数据同步工具，启用多个map进行数据的并发导入和导出。对于导入任务，数据的分片是基于数据中数据的范围和用户指定的并发数；对于导出任务，数据分片是依赖HDFS的 FileInputFormat 的分片策略。**


## Sqoop安装与配置

### 准备工作

Sqoop依赖于Hadoop，在安装Sqoop之前需要安装Hadoop和JAVA环境。Hadoop的安装可以参考之前的文章，[配置本地Hadoop环境][12], [Hive的使用][13]。

### Sqoop包结构

sqoop的安装包结构如下：

```
[1015146591@bigdata4 sqoop-1.4.7.bin__hadoop-2.6.0]$ tree -L 1
.
├── bin                     - sqoop命令文件目录
├── build.xml       
├── CHANGELOG.txt
├── COMPILING.txt
├── conf                    - 配置文件路径
├── docs
├── ivy
├── ivy.xml
├── lib                     - jar包依赖目录
├── LICENSE.txt
├── NOTICE.txt
├── pom-old.xml
├── README.txt
├── sqoop-1.4.7.jar         - sqoop核心jar包
├── sqoop-patch-review.py   - sqoop核心jar包
├── sqoop-test-1.4.7.jar    - sqoop核心jar包
├── src
└── testdata
```

### 验证 Java 和 Hadoop 已经安装

1. 终端输入java命令查看返回值：

```bash
1015146591@bigdata4.novalocal:/mnt/home/1015146591 $ java -version
openjdk version "1.8.0_171"
OpenJDK Runtime Environment (build 1.8.0_171-b10)
OpenJDK 64-Bit Server VM (build 25.171-b10, mixed mode)
1015146591@bigdata4.novalocal:/mnt/home/1015146591 $ 
```

2. 启动Hadoop自带MapReduce示例程序：

```
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.6.jar pi 2 2
```

OK，都可以运行就好。

### 下载Sqoop安装包

1. 下载地址： [sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz](https://mirrors.tuna.tsinghua.edu.cn/apache/sqoop/1.4.7/sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz)。

2. 解压到本地目录，我在自己的home目录下创建了一个apps文件，用于存放需要安装的软件：

```bash
tar xzvf sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz -C ~/apps
```

### 配置Sqoop

1. 进入sqoop的配置目录conf，添加sqoop-env.sh，这里直接拷贝模板文件即可:

    ```bash
    cp sqoop-env-template.sh sqoop-env.sh
    ```

2. 修改sqoop-env.sh文件，由于之前已经配置过HADOOP_HOME环境变量，而且加入到了系统环境变量中，所以可以直接使用HADOOP_HOME这个环境变量。

    ```conf
    #Set path to where bin/hadoop is available
    export HADOOP_COMMON_HOME=${HADOOP_HOME}

    #Set path to where hadoop-*-core.jar is available
    export HADOOP_MAPRED_HOME=${HADOOP_HOME}
    export HADOOP_HDFS_HOME=${HADOOP_HOME}
    export HIVE_HOME=/mnt/home/1015146591/apps/hive-1.2.2
    export HIVE_CONF_DIR=$HIVE_HOME/conf
    export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:$HIVE_HOME/lib
    export PATH=$HIVE_HOME/bin:$PATH
    ```

    复制Hive的配置文件hive-site.xml到sqoop的conf目录下，否则可能会报 Hive 数据库没有建立的错误。

3. 连接数据库(这是远端的MySQL数据库，可以自己搭建一个数据库进行测试)：

    ```
    mysql -u root -pGg/ru,.#5 -h 10.173.32.6 -P3306 -Dsqoop
    mysql> 

    ```
    查看数据库表：

    ```
    mysql> select * from sqoop_test;
    +------+-------+
    | id   | name  |
    +------+-------+
    |    1 | allen |
    |    2 | bob   |
    +------+-------+
    2 rows in set (0.00 sec)

    ```

## Sqoop语法分析

### 数据导入

以下面这个sqoop语法为例：

```
./sqoop import --connect "jdbc:mysql://*.*.*.*:protNum/DbName?characterEncoding=UTF-8&useCursorFetch=true" --username *** --password *** --query "SELECT * FROM HIVE_JDBC_TEST2 WHERE id > 2 AND \$CONDITIONS" --split-by id --target-dir "/user/hadoop/HIVE_JDBC_TEST2_QUERY" --delete-target-dir --as-textfile
```

参数介绍：

- --connect：指定JDBC连接数据库所在的地址、端口、数据库名称、字符编码格式等信息

- --username：数据库连接用户名

- --password：数据库连接密码

- --query：获取导入数据的查询语句

- --split-by：指定分片字段，后续会使用改字段对数据进行换分，放到不同的map中执行，一般使用主键或者有索引的字段，可提高执行效率

- --target-dir：指定HDFS上的目标路径，即导入数据的存储路径

- --delete-target-dir：如果目标文件存在就先删除在导入

- --as-textfile：指定导入数据到HDFS后的数据存储格式。

    - 支持有text、sequence、avro、parquet

- --boudary-query：指定数据边界查找语句。sqoop默认使用select min(), max() from 来查找split的总边界。但是这种自动生成的查询语句执行效率并不高。

- -m/--num-mappers指定Map个数。这个参数会显著影响程序的执行效率，太小的话并发数不够，程序执行效率低，太大的话任务的启动开销过大，任务执行也会受影响。

- --table,column,where分别对应于Table，Column，Where限制条件。

- --compress，--compression-codec：使用压缩

- --map-column-java：指定Column在Sqoop对象中的数据类型，一般不需要手动添加。

- 增量导入（incremental）：指定关系数据库的某个Column和last value，下次导入的时候只导入比last value大的数据。增量导入分两种模式：

    - append：把新数据追加到目标路径中

    - last modified：会将新数据和老数据进行合并，合并的话需要新老数据的对应关系，所以还需要添加参数merge key来指定主键，最后执行结果只保留最新数据。该模式下last value的值一定要是timestamp或者data。

### Hive导入

实际使用中的具体命令如下：

```
./sqoop import --connect "jdbc:mysql://*.*.*.*:protNum/DbName?characterEncoding=UTF-8&useCursorFetch=true" --username *** --password *** --table sqooptest --split-by id --target-dir "/user/hadoop/sqooptest" --delete-target-dir --hive-import --hive-table sqooptest map-column-hive id=INT,name=STRING
```

- 支持hive table的创建与覆盖：create-hive-table, hive-overwritess

- 支持hive分区：hive-partition-key,hive-partition-value

- map-column-hive:指定hive column的数据类型。例如key=value。限制：Hive Column顺序需等同与sqoop导入的hdfs文件column顺序，否则会导入失败（可在-column或者-query中指定column顺序）

- 支持指定查询语句，解决多表关联插入。

    ```
    ./sqoop import --connect "jdbc:mysql://10.173.32.6:3306/sqoop?characterEncoding=UTF-8&useCursorFetch=true" --username root --password Gg/ru,.#5 --table sqoop_test --delete-target-dir --target-dir /user/1015146591/sqoop_test --split-by id
    ```

### 数据导出

```
sqoop export --connet jdbc:mysql://10.173.121.103:3306/test?characterEncoding=UTF8 --username mammoth_test --password VwvV8r37ccrR --table mysql_base_export -columns id,name --export-dir /user/hadoop/mysql_base_no_meta
```

- table, colums:指定导出rdms中的table，columns

- 支持sequence，avro，parquet导出。其中sequence，avro，parquet格式导出sqoop可自动识别

- update-mode:分为updateonly(默认)和allowinsert两种类型。allowinsert可进行update insert操作

- update-key:指定根据指定列完成update。

### RDMS工具

在导入导出数据时，常常需要对数据库中的数据进行探测，看看数据库中表的结构是什么。sqoop内部提供了查询工具。

```
sqoop list-databases --connect jdbc:mysql://*.*.*.*:3306/dbName?characterEncoding-UTF-8 --username *** --password ***
```
rdms结构：

- list-databases:列出所有数据库

- list-tables: 列出所有的表

查看数据表内数据：

```
sqoop eval --connect jdbc:mysql://*.*.*.*:3306/dbName?characterEncoding-UTF-8 --username *** --password *** -e "select * from ..."
```
- -e,-query:添加要执行的SQL查询语句

## Sqoop的使用

### 使用Sqoop导入数据到HDFS

1. 下载MySQL的JDBC驱动: [mysql-connector-java-5.1.47.tar.gz](https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.47.tar.gz)，我在官网只找到了5.1.47的版本，不知道能不能使用，也可以直接从老师的sqoop目录下拷贝：

    ```
    cp /home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/lib/mysql-connector-java-5.1.46.jar .
    ```

2. 执行sqoop命令验证安装是否成功：

    ```
    ./sqoop import --connect "jdbc:mysql://10.173.32.6:3306/sqoop?characterEncoding=UTF-8&useCursorFetch=true" --username root --password Gg/ru,.#5 --table sqoop_test --delete-target-dir --target-dir /user/1015146591/sqoop_test --split-by id
    ```

3. 在HDFS上查看导入的数据：

    ```bash
    1015146591@bigdata4.novalocal:/mnt/home/1015146591/apps/sqoop-1.4.7.bin__hadoop-2.6.0/bin $ hadoop fs -ls /user/1015146591/sqoop_test
    Found 3 items
    -rw-r-----   3 1015146591 supergroup          0 2019-01-21 15:57 /user/1015146591/sqoop_test/_SUCCESS
    -rw-r-----   3 1015146591 supergroup          8 2019-01-21 15:57 /user/1015146591/sqoop_test/part-m-00000
    -rw-r-----   3 1015146591 supergroup          6 2019-01-21 15:57 /user/1015146591/sqoop_test/part-m-00001
    ```

    文件内的具体内容如下：

    ```
    1015146591@bigdata4.novalocal:/mnt/home/1015146591/apps/sqoop-1.4.7.bin__hadoop-2.6.0/bin $ hadoop fs -text /user/1015146591/sqoop_test/part-m-00000
    19/01/21 15:59:18 INFO lzo.GPLNativeCodeLoader: Loaded native gpl library from the embedded binaries
    19/01/21 15:59:18 INFO lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 5e360bdefd8923280b1a0234b845448050bf0caa]
    1,allen
    1015146591@bigdata4.novalocal:/mnt/home/1015146591/apps/sqoop-1.4.7.bin__hadoop-2.6.0/bin $ hadoop fs -text /user/1015146591/sqoop_test/part-m-00001
    19/01/21 15:59:26 INFO lzo.GPLNativeCodeLoader: Loaded native gpl library from the embedded binaries
    19/01/21 15:59:26 INFO lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 5e360bdefd8923280b1a0234b845448050bf0caa]
    2,bob
    1015146591@bigdata4.novalocal:/mnt/home/1015146591/apps/sqoop-1.4.7.bin__hadoop-2.6.0/bin $ 
    ```

### 使用Sqoop导入数据到Hive

1. 首选将hive安装目录下lib目录中的hive-common-1.1.0-cdh5.7.0.jar和hive-shims*拷贝到sqoop安装目录下的lib目录。然后执行：

    ```
    ./sqoop import --connect "jdbc:mysql://10.173.32.6:3306/sqoop?characterEncoding=UTF-8&useCursorFetch=true" --username root --password Gg/ru,.#5 --table product --split-by price --target-dir "/user/1015146591/sqooptest" --delete-target-dir --hive-import --hive-table sqoophomework –hive-overwrite
    ```

    执行完成后查看( **在sqoop的bin目录下启动hive** )hive中default.sqoophomework中的数据：

    ```
    [1015146591@bigdata4 bin]$ hive
    hive> select * from default.sqoophomework limit 10;
    OK
    28	1527235438751389	商品3
    21	1527235438751248	商品4
    25	1527235438751118	商品5
    33	1527235438750902	商品6
    25	1527235438750711	商品8
    27	1527235438750331	商品11
    11	1527235438750030	商品17
    28	1527235438749094	商品24
    11	1527235438748767	商品26
    16	1527235438748507	商品28
    Time taken: 1.755 seconds, Fetched: 10 row(s)
    hive> 
    ```
  
2. 查询hive表product表中拥有多少个课程，执行查询语句：

    ```
    hive> select count(product_id) from default.sqoophomework;
    Query ID = 1015146591_20190129111455_3bc982ce-0295-4050-963f-69f081580f9d
    Total jobs = 1
    Launching Job 1 out of 1
    Number of reduce tasks determined at compile time: 1
    In order to change the average load for a reducer (in bytes):
      set hive.exec.reducers.bytes.per.reducer=<number>
    In order to limit the maximum number of reducers:
      set hive.exec.reducers.max=<number>
    In order to set a constant number of reducers:
      set mapreduce.job.reduces=<number>
    Starting Job = job_1547603700477_1639, Tracking URL = http://bigdata0.novalocal:8088/proxy/application_1547603700477_1639/
    Kill Command = /home/hadoop/hadoop-current/bin/hadoop job  -kill job_1547603700477_1639
    Hadoop job information for Stage-1: number of mappers: 2; number of reducers: 1
    2019-01-29 11:15:04,571 Stage-1 map = 0%,  reduce = 0%
    2019-01-29 11:15:10,817 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 3.29 sec
    2019-01-29 11:15:16,086 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 5.51 sec
    MapReduce Total cumulative CPU time: 5 seconds 510 msec
    Ended Job = job_1547603700477_1639
    MapReduce Jobs Launched: 
    Stage-Stage-1: Map: 2  Reduce: 1   Cumulative CPU: 5.51 sec   HDFS Read: 13903 HDFS Write: 4 SUCCESS
    Total MapReduce CPU Time Spent: 5 seconds 510 msec
    OK
    100
    Time taken: 21.809 seconds, Fetched: 1 row(s)
    ```

## 案例

假设有这样一张表：

| order | order_id | item | user | description | create_time |
|:-----:|:--------:|:----:|:----:|:-----------:|:-----------:|
| 1 | 0001 | hive课程 | 李磊 | 1学时 | 15833081600 |
| 2 | 0002 | spark课程 | 韩梅 | 1学时 | 15833081600 |

### 数据导入

1. 快照导入：

    ```
    ./sqoop import --connect "jdbc:mysql://*.*.*.*:protNum/DbName?characterEncoding=UTF-8" --username *** --password *** --table order --split-by order_id --target-dir "/user/hadoop/order_snapshot/2018-08-01" --delete-target-dir --as-parquet
    ```

2. 增量导入，每天只导入增量的数据：

    ```
    ./sqoop import --connect "jdbc:mysql://*.*.*.*:protNum/DbName?characterEncoding=UTF-8" --username *** --password *** --query "SELECT * FROM order WHERE create_time > 1533081600 and \$CONDITIONS" --split-by order_id --target-dir "/user/hadoop/order_snapshot/2018-08-01" --as-parquet
    ```

    $CONDITIONS在这里是起到占位符的作用，在任务执行过程中会被数据划分生成的条件替代，一定要注意添加。

选择全量导入还是增量导入需根据ETL规范视情况而定。

### 数据导出

数据导入HDFS后，业务程序关联各种数据后可能每天会生成一个收入报表：

| order | catalog | income | order_num | description | create_time |
|:-----:|:--------:|:----:|:----:|:-----------:|:-----------:|
| 1 | hive课程 | 100 | 1000 | 1学时 | 15833081600 |
| 2 | spark课程 | 100 | 1000 | 1学时 | 15833081600 |

对于数据量较小的访问，HDFS的效率并不高，所以此时可以将每天的收入报表从HDFS导出到关系型数据库中来进行分析。

导出任务

```
./sqoop export --connect "jdbc:mysql://*.*.*.*:protNum/DbName?characterEncoding=UTF-8" --username *** --password *** --table income_report --columns catalog,income,order_num,description,create_time --export-dir "/user/hadoop/order_report/"
```

--export-dir:表示待导出数据在HDFS上的路径。

## 总结

- 为什么需要sqoop，是由于数据同步的需求。

- sqoop是什么，sqoop是一个基于MapReduce框架的数据同步工具

- Sqoop是怎么工作的，利用MapReduce并行进行数据导入工作。

- sqoop的使用方法，详见上面介绍的导入导出方法。


[1]: https://github.com/jiaoqiyuan/163-bigdate-note/raw/master/%E6%95%B0%E6%8D%AE%E8%8E%B7%E5%8F%96%E5%92%8C%E9%A2%84%E5%A4%84%E7%90%86%EF%BC%9ASqoop/img/%E7%9B%B4%E6%8E%A5%E8%AE%BF%E9%97%AE%E6%95%B0%E6%8D%AE%E5%BA%93.png
[2]: https://github.com/jiaoqiyuan/163-bigdate-note/raw/master/%E6%95%B0%E6%8D%AE%E8%8E%B7%E5%8F%96%E5%92%8C%E9%A2%84%E5%A4%84%E7%90%86%EF%BC%9ASqoop/img/%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6%E6%A8%A1%E5%BC%8F.png
[3]: https://github.com/jiaoqiyuan/163-bigdate-note/raw/master/%E6%95%B0%E6%8D%AE%E8%8E%B7%E5%8F%96%E5%92%8C%E9%A2%84%E5%A4%84%E7%90%86%EF%BC%9ASqoop/img/%E4%BC%A0%E7%BB%9F%E6%95%B0%E6%8D%AE%E4%BB%93%E5%BA%93.png
[4]: https://baike.baidu.com/item/Teradata
[5]: https://zh.wikipedia.org/wiki/ODS
[6]: https://github.com/jiaoqiyuan/163-bigdate-note/raw/master/%E6%95%B0%E6%8D%AE%E8%8E%B7%E5%8F%96%E5%92%8C%E9%A2%84%E5%A4%84%E7%90%86%EF%BC%9ASqoop/img/Sqoop%E6%A1%86%E6%9E%B6.png
[7]: https://github.com/jiaoqiyuan/163-bigdate-note/raw/master/%E6%95%B0%E6%8D%AE%E8%8E%B7%E5%8F%96%E5%92%8C%E9%A2%84%E5%A4%84%E7%90%86%EF%BC%9ASqoop/img/Sqoop%E4%BB%BB%E5%8A%A1%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B.png
[8]: https://github.com/jiaoqiyuan/163-bigdate-note/raw/master/%E6%95%B0%E6%8D%AE%E8%8E%B7%E5%8F%96%E5%92%8C%E9%A2%84%E5%A4%84%E7%90%86%EF%BC%9ASqoop/img/%E6%95%B0%E6%8D%AE%E5%AF%B9%E8%B1%A1%E7%94%9F%E6%88%90.png
[9]: https://github.com/jiaoqiyuan/163-bigdate-note/raw/master/%E6%95%B0%E6%8D%AE%E8%8E%B7%E5%8F%96%E5%92%8C%E9%A2%84%E5%A4%84%E7%90%86%EF%BC%9ASqoop/img/Hive%E5%AF%BC%E5%85%A5.png
[10]: https://github.com/jiaoqiyuan/163-bigdate-note/raw/master/%E6%95%B0%E6%8D%AE%E8%8E%B7%E5%8F%96%E5%92%8C%E9%A2%84%E5%A4%84%E7%90%86%EF%BC%9ASqoop/img/Sqoop%E5%AF%BC%E5%85%A5.png
[11]: https://github.com/jiaoqiyuan/163-bigdate-note/raw/master/%E6%95%B0%E6%8D%AE%E8%8E%B7%E5%8F%96%E5%92%8C%E9%A2%84%E5%A4%84%E7%90%86%EF%BC%9ASqoop/img/Sqoop%E5%AF%BC%E5%87%BA.png
[12]: https://jiaoqiyuan.cn/2018/12/26/%E9%85%8D%E7%BD%AE%E6%9C%AC%E5%9C%B0Hadoop%E7%8E%AF%E5%A2%83/
[13]: https://jiaoqiyuan.cn/2019/01/08/Hive%E7%9A%84%E4%BD%BF%E7%94%A8/