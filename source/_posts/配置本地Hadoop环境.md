---
title: 配置本地Hadoop环境
date: 2018-12-26 01:27:43
tags:
categories: 大数据
thumbnail: https://blog.rackspace.com/wp-content/uploads/2014/10/Hadoop.png
---

    进行大数据开发一般都离不开Hadoop环境，Hadoop慢慢也变成了一个生态环境的代名词，这里记录一下Hadoop本地伪分布式环境的搭建。

## 本地环境

>操作系统 : manjaro-64bit

>内存 : 8GB

>CPU : i7-4770S

## 前期准备

>Hadoop2.7.6,[下载链接](https://archive.apache.org/dist/hadoop/common/hadoop-2.7.6/hadoop-2.7.6.tar.gz)。

>JDK1.8，[下载链接](https://download.oracle.com/otn-pub/java/jdk/8u191-b12/2787e4a523244c269598db4e85c51e0c/jdk-8u191-linux-x64.tar.gz)。

## 配置工作

### 解压和配置环境变量

1. 将hadoop-2.7.6.tar.gz和jdk-8u191-linux-x64.tar.gz拷贝到本地～/apps目录下（这里我习惯将软件安装到自己创建的~/apps文件夹下），然后解压：

```
tar xzvf hadoop-2.7.6.tar.gz
tar xzvf jdk-8u191-linux-x64.tar.gz
```

2. 配置HADOOP_HOME和JAVA_HOME环境变量，修改当前用户～/.bashrc（如果使用的是zsh就是修改~/.zshrc），在文件尾添加如下内容：

```
# JDK
export JAVA_HOME=/home/jony/apps/jdk1.8.0_191
export PATH=$JAVA_HOME/bin:$PATH

# HADOOP_HOME
export HADOOP_HOME=/home/jony/apps/hadoop-2.7.6
export PATH=$HADOOP_HOME/bin:$PATH
```

3. 使环境变量生效：

如果是bash环境下：

```
source ~/.bashrc
```

如果是zsh环境：

```
source ~/.zshrc
```

## 配置HDFS

1. 配置hadoop-2.7.6/etc/hadoop/core-site.xml，添加默认文件路径，配置内容如下：

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:50000</value>
    </property>
</configuration>
```

这里如果自己配置了hostname，可以使用自己配置的hostname替换localhost，默认使用localhost，端口信息也可以自己指定为未使用的端口。

2. 配置hadoop-2.7.6/etc/hadoop/hdfs-site.xml，配置namenode和datanode相关属性：

```
<configuration>
    <property>
        <name>dfs.namenode.rpc-address</name>
        <value>localhost:50000</value>
    </property>
    <property>
        <name>dfs.namenode.http-address</name>
        <value>localhost:50070</value>
    </property>
    <property>
        <name>dfs.datanode.address</name>
        <value>localhost:50010</value>
    </property>
    <property>
        <name>dfs.datanode.http.address</name>
        <value>localhost:50075</value>
    </property>
    <property>
        <name>dfs.datanode.ipc.address</name>
        <value>localhost:50020</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/home/jony/apps/hadoop-2.7.6/tmp/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/home/jony/apps/hadoop-2.7.6/tmp/data/0,/home/jony/apps/hadoop-2.7.6/tmp/data/1,/home/jony/apps/hadoop-2.7.6/tmp/data/2</value>
    </property>
</configuration>
```

    这里dfs.namenode.name.dir和dfs.datanode.data.dir可以根据自己的喜好进行配置，dfs.namenode.name.dir用于确定将HDFS文件系统的元信息保存在什么目录下。如果这个参数设置为多个目录，那么这些目录下都保存着元信息的多个备份。dfs.datanode.data.dir用于确定将HDFS文件系统的数据保存在什么目录下。

3. 初次运行HDFS前需要先格式化HDFS，使用如下命令即可:

```
hdfs namenode -format
```

4. 前台启动namenode和datanode

```
hdfs namenode
hdfs datanode
```

5. 使用浏览器查看HDFS，在浏览器输入localhost:50070，注意50070端口是hdfs-site.xml中配置的dfs.namenode.http-address的值。如果能够看到相关页面信息，就代表HDFS配置成功了。

6. 使用后台启动HDFS的namenode和datanode，因为前台启动一般是做调试用，正常生产环境下都是后台启动的，后台启动和停止namenode和datanode的方式如下：

```
sbin/hadoop-daemon.sh start namenode
sbin/hadoop-daemon.sh start datanode
```

```
sbin/hadoop-daemon.sh stop namenode
sbin/hadoop-daemon.sh stop datanode
```

7. 使用HDFS的一些shell命令操作HDFS：

- 查看HDFS目录：

  ```
  hadoop fs -ls /
  ```
- 创建文件夹：

  ```
  hadoop fs -mkdir /user/jony
  ```

- 将本地文件上传到HDFS：

  ```
  将本地~/test.txt文件上传到HDFS的/user/jony目录下:
  hadoop fs -put ~/test.txt /user/jony
  or
  hadoop fs -copyFromLocal ~/test.txt /user/jony
  
  查看是否上传成功:
  hadoop fs -ls /user/jony/test.txt
  ```
- 查看HDFS上文件的内容：

  ```
  hadoop fs -text /user/jony/test.txt
  ```
  或者
  ```
  hadoop fs -cat /user/jony/test.txt
  ```

- 从HDFS上将文件拷贝到本地：

  ```
  hadoop fs -get /user/jony/test.txt ~/test_hdfs.txt
  or
  hadoop fs -copyToLocal /user/jony/test.txt ~/test_hdfs.txt
  ```

### 配置YARN

1. 配置hadoop-2.7.6/etc/hadoop/yarn-site.xml，添加默认文件路径，配置内容如下：

```xml
<configuration>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>localhost:8088</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>localhost</value>
    </property>
    <property>
        <name>yarn.nodemanager.log-dirs</name>
        <value>/home/jony/apps/hadoop-2.7.6/tmp/yarn/0/logs,/home/jony/apps/hadoop-2.7.6/tmp/yarn/1/logs,/home/jony/apps/hadoop-2.7.6/tmp/yarn/2/logs</value>
    </property>
    <property>
        <name>yarn.nodemanager.local-dirs</name>
        <value>/home/jony/apps/hadoop-2.7.6/tmp/yarn/0/local,/home/jony/apps/hadoop-2.7.6/tmp/yarn/1/local,/home/jony/apps/hadoop-2.7.6/tmp/yarn/2/local</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

yarn.nodemanager.log-dirs表示yarn日志存放地址（可配置多个目录）。yarn.nodemanager.local-dirs表示中间结果存放位置，这个参数通常会配置多个目录，以分摊磁盘IO负载。yarn.nodemanager.aux-services一般要配置为mapreduce_shuffle才能运行MapReduce程序。

2. 配置hadoop-2.7.6/etc/hadoop/mapred-site.xml，添加默认文件路径，配置内容如下：

```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>localhost:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>localhost:19888</value>
    </property>
</configuration>
```

3. 配置本地ssh免密登录，启动YARN时需要：

-  创建rsa密钥：

    ``` 
    ssh-keygen -t rsa
    ```
- 加入authorized_keys：

    ```
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    ```
4. 启动resourcemanager和nodemanager:

```
sbin/yarn-daemon.sh start resourcemanager
sbin/yarn-daemon.sh start nodemanager
```

5. 在浏览器查看YARN，输入http://localhost:8088

6. 运行计算Pi的MapReduce程序测试YARN：

```
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.6.jar pi 5 10
```

6. 关闭resourcemanager和nodemanager：

```
sbin/yarn-daemon.sh stop resourcemanager
sbin/yarn-daemon.sh stop nodemanager
```

## 总结

到此Hadoop基本环境就配置完成了，主要是配置了HDFS和YARN，并运行了简单的MapReduce程序来测试YARN环境的搭建是否正确。

HDFS是分布式文件系统，是大数据基础的存储管理系统，用于管理分布式文件的存储，实现分布式文件的高可用、线性扩展。

YARN是资源调度管理系统，负责在任务运行时调度集群资源给任务使用。

Hadoop生态还有很多内容，这里先不展开，后续用到其他的工具时再单独记录。

2018.12.26 晴 温度零下