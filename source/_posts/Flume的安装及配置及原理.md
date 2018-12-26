---
title: Flume的安装及配置及原理
date: 2018-12-26 05:33:15
tags:
categories: 大数据
thumbnail: https://stayrelevant.globant.com/wp-content/uploads/2012/06/apache-flume-1-728.jpg
---

    Flume是一个高可用、高可靠、分布式的海量数据采集、聚合、传输的系统。

Flume作为一个日志收集工具，非常轻量级，基于一个个Flume Agent，能够构建一个复杂而强大的日志手机系统。运行在服务器上的进程将需要关注的日志文件采集到指定的存储服务，这个进程就叫Flume Agent。

## 本地环境

>Hadoop版本:2.7.6

>操作系统 : manjaro-64bit

>内存 : 8GB

>CPU : i7-4770S

## 前期准备

>Flume安装包：[下载链接](http://mirrors.tuna.tsinghua.edu.cn/apache/flume/1.8.0/apache-flume-1.8.0-bin.tar.gz) 。


## Flume安装和配置

1. 解压Flume：

    ```
    tar xzvf apache-flume-1.8.0-bin.tar.gz ~/apps
    ```

2. 配置环境变量，打开~/.bashrc(使用zsh的话打开~/.zshrc)，在文件末尾添加如下内容（FLUME_HOME的具体路径根据你自己解压的flume文件所在路径进行配置）：

    ```
    # FLUME
    export FLUME_HOME=/home/jony/apps/apache-flume-1.8.0-bin
    export PATH=$FLUME_HOME/bin:$PATH
    ```

    使环境变量生效

    ```
    source ~/.bashrc
    or
    source ~/.zshrc
    ```

3. 修改Flume配置文件，将apache-flume-1.8.0-bin/conf/flume-env.sh.template重命名为flume-env.sh，打开flume-env.sh配置如下内容：

    ```
    # 这里如果你在安装Hadoop那里已经配置过JAVA_HOME，这里就不需要配置JAVA_HOME啦
    # export JAVA_HOME=/usr/lib/jvm/java-8-oracle
    export HADOOP_HOME=/home/hadoop/hadoop-current
    FLUME_CLASSPATH="$HADOOP_HOME/bin:$PATH"
    ```

4. 修改apache-flume-1.8.0-bin/conf/flume-conf.properties.template为flume-conf.properties，测试Flume配置是否正确，测试命令如下：

    ```
    ./bin/flume-ng agent --conf conf --conf-file conf/flume-conf.properties --name agent
    ```
    打开log文件查看运行结果：
    ```
    26 十二月 2018 14:08:56,172 INFO  [SinkRunner-PollingRunner-DefaultSinkProcessor] (org.apache.flume.sink.LoggerSink.process:95)  - Event: { headers:{} body: 37 34 31 36 38                                  74168 }
    26 十二月 2018 14:08:56,172 INFO  [SinkRunner-PollingRunner-DefaultSinkProcessor] (org.apache.flume.sink.LoggerSink.process:95)  - Event: { headers:{} body: 37 34 31 36 39                                  74169 }
    26 十二月 2018 14:08:56,172 INFO  [SinkRunner-PollingRunner-DefaultSinkProcessor] (org.apache.flume.sink.LoggerSink.process:95)  - Event: { headers:{} body: 37 34 31 37 30                                  74170 }
    26 十二月 2018 14:08:56,172 INFO  [SinkRunner-PollingRunner-DefaultSinkProcessor] (org.apache.flume.sink.LoggerSink.process:95)  - Event: { headers:{} body: 37 34 31 37 31                                  74171 }
    26 十二月 2018 14:08:56,172 INFO  [SinkRunner-PollingRunner-DefaultSinkProcessor] (org.apache.flume.sink.LoggerSink.process:95)  - Event: { headers:{} body: 37 34 31 37 32                                  74172 }
    ```

## Flume使用示例

### 1. 监听端口，输出到控制台

1. 在flume的conf目录下编写配置文件flume-conf-netsrc.properties，文件配置内容如下：

    ```conf
    agent.sources = netSrc
    agent.channels = memoryChannel
    agent.sinks = loggerSink

    # For each one of the sources, the type is defined
    agent.sources.netSrc.type = netcat
    agent.sources.netSrc.bind = 0.0.0.0
    agent.sources.netSrc.port = 12345

    # The channel can be defined as follows.
    agent.sources.netSrc.channels = memoryChannel

    # Each sink's type must be defined
    agent.sinks.loggerSink.type = logger

    #Specify the channel the sink should use
    agent.sinks.loggerSink.channel = memoryChannel

    # Each channel's type is defined.
    agent.channels.memoryChannel.type = memory

    # Other config values specific to each type of channel(sink or source)
    # can be defined as well
    # In this case, it specifies the capacity of the memory channel
    agent.channels.memoryChannel.capacity = 100
    ```

    sources类型配置为netcat，用于监听网络端口，这里bind为监听的ip地址，0.0.0.0表示监听本机。port为监听的端口。sink的类型设置为logger表示直接将监听到的信息打印到终端输出。channel选用memor类型，表示使用内存作为Channel。

2. 启动flume：

    ```
    ./bin/flume-ng agent --conf conf --conf-file conf/flume-conf-netsrc.properties --name agent -Dflume.root.logger=INFO,console
    ```

    启动命令最后添加的-Dflume.root.logger=INFO,console是为了将日志输出到控制台，所以直接修改logger，使用控制台作为输出，并且日志输出级别设置为INFO。

3. 开启telnet，向flume发送数据：

    ```
    telnet localhost 12345
    ```

    输入一些内容：

    ```
    hello
    OK
    workd
    OK
    workd
    OK
    world
    OK
    ```

    在flume启动的终端下面查看输出：

    ```
    2018-12-17 15:12:02,438 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 68 65 6C 6C 6F 0D                               hello. }
    2018-12-17 15:12:03,616 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 77 6F 72 6B 64 0D                               workd. }
    2018-12-17 15:12:05,047 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 77 6F 72 6B 64 0D                               workd. }
    2018-12-17 15:12:06,454 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 77 6F 72 6C 64 0D                               world. }
    ```


### 2. 监听端口，输出到服务器文件

1. 在flume的conf目录下编写配置文件flume-conf-netsrc2localfile.properties，文件配置内容如下：

    ```conf
    agent.sources = netSrc
    agent.channels = memoryChannel
    agent.sinks = fileSink

    # For each one of the sources, the type is defined
    agent.sources.netSrc.type = netcat
    agent.sources.netSrc.bind = 0.0.0.0
    agent.sources.netSrc.port = 12345

    # The channel can be defined as follows.
    agent.sources.netSrc.channels = memoryChannel

    # Each sink's type must be defined
    agent.sinks.fileSink.type = file_roll
    agent.sinks.fileSink.sink.directory=/home/jony/tmp/flume_workspace/file_roll/data
    agent.sinks.fileSink.sink.rollInterval = 60

    #Specify the channel the sink should use
    agent.sinks.fileSink.channel = memoryChannel

    # Each channel's type is defined.
    agent.channels.memoryChannel.type = memory

    # Other config values specific to each type of channel(sink or source)
    # can be defined as well
    # In this case, it specifies the capacity of the memory channel
    agent.channels.memoryChannel.capacity = 100
    ```

    这里sink的类型使用的是file_roll，表示要将最终结果输出到本地文件中，directory配置的是输出的文件路径，需要注意的是这个目录需要被提前创建，否则执行会出错。

2. 启动flume

    ```
    ./bin/flume-ng agent --conf conf --conf-file conf/flume-conf-netsrc2localfile.properties --name agent -Dflume.root.logger=INFO,console
    ```

3. 使用telnet往对应端口发送数据：

    ```
    telnet localhost 12345
    ```

    发送数据：

    ```
    hello
    OK
    world
    OK
    flume
    OK
    ```

4. 在指定目录下（这里是/home/jony/tmp/flume_workspace/file_roll/data）查看相关文件：

      ```
      $ cat 1545031636278-1 
      hello
      world
      flume
      ```

    60秒后，该目录下又生成了一个文件1545031636278-2.


### 3. 监听服务器文件，输出到HDFS（memoryChannel）

1. 修改flume的conf目录下的flume-conf-taildir2hdfs.properties，文件配置内容如下：

    ```conf
    agent.sources = fileSrc
    agent.channels = memoryChannel
    agent.sinks = hdfsSink

    # For each one of the sources, the type is defined
    agent.sources.fileSrc.type = taildir
    agent.sources.fileSrc.positionFile = /home/jony/tmp/flume_workspace/mem_ch/positionFile
    agent.sources.fileSrc.filegroups = f1
    agent.sources.fileSrc.filegroups.f1 = /home/jony/tmp/log/.*.log

    # The channel can be defined as follows.
    agent.sources.fileSrc.channels = memoryChannel

    # Each sink's type must be defined
    agent.sinks.hdfsSink.type = hdfs
    agent.sinks.hdfsSink.hdfs.path = /user/jony/flume_data/mem_ch/%Y-%m-%d
    agent.sinks.hdfsSink.hdfs.useLocalTimeStamp = true
    agent.sinks.hdfsSink.hdfs.fileType = DataStream
    agent.sinks.hdfsSink.hdfs.rollSize = 0
    agent.sinks.hdfsSink.hdfs.rollCount = 0
    agent.sinks.hdfsSink.hdfs.rollInterval = 60

    #Specify the channel the sink should use
    agent.sinks.hdfsSink.channel = memoryChannel

    # Each channel's type is defined.
    agent.channels.memoryChannel.type = memory

    # Other config values specific to each type of channel(sink or source)
    # can be defined as well
    # In this case, it specifies the capacity of the memory channel
    agent.channels.memoryChannel.capacity = 100
    ```

2. 修改conf/flume-env.sh，添加Hadoop的路径，Hadoop的配置可以参考我的另一篇文章：[配置本地Hadoop环境](https://jiaoqiyuan.cn/2018/12/26/%E9%85%8D%E7%BD%AE%E6%9C%AC%E5%9C%B0Hadoop%E7%8E%AF%E5%A2%83/) 。如果你已经配置了Hadoop，这里是不用在flume-env.sh中配置任何东西的:

    ```
    export HADOOP_HOME=/home/jony/apps/hadoop-2.7.6
    FLUME_CLASSPATH="$HADOOP_HOME/bin:$PATH"
    ```

3. 启动flume：

    ```
    ./bin/flume-ng agent --conf conf --conf-file conf/flume-conf-taildir2hdfs.properties --name agent
    ```
4. 查看HDFS上是否已经写入数据：

    ```
    $ hadoop fs -ls /user/jony/flume_data/mem_ch
    Found 1 items
    drwxr-xr-x   - jony supergroup          0 2018-12-26 15:53 /user/jony/flume_data/mem_ch/2018-12-26
    ```

### 4. 监听服务器文件，输出到HDFS（fileChannel）

1. 修改flume的conf目录下的flume-conf-taildir2hdfs.properties，文件配置内容如下：

    ```conf
    agent.sources = fileSrc
    agent.channels = fileChannel
    agent.sinks = hdfsSink

    # For each one of the sources, the type is defined
    agent.sources.fileSrc.type = taildir
    agent.sources.fileSrc.positionFile = /home/jony/tmp/flume_workspace/hdfs_sink/file_ch/positionFile
    agent.sources.fileSrc.filegroups = f1
    agent.sources.fileSrc.filegroups.f1 = /home/jony/tmp/log/.*.log

    # The channel can be defined as follows.
    agent.sources.fileSrc.channels = fileChannel

    # Each sink's type must be defined
    agent.sinks.hdfsSink.type = hdfs
    agent.sinks.hdfsSink.hdfs.path = /user/jony/flume_data/file_ch/%Y-%m-%d
    agent.sinks.hdfsSink.hdfs.useLocalTimeStamp = true
    agent.sinks.hdfsSink.hdfs.fileType = DataStream
    agent.sinks.hdfsSink.hdfs.rollSize = 0
    agent.sinks.hdfsSink.hdfs.rollCount = 0
    agent.sinks.hdfsSink.hdfs.rollInterval = 60

    #Specify the channel the sink should use
    agent.sinks.hdfsSink.channel = fileChannel

    # Each channel's type is defined.
    agent.channels.fileChannel.type = file
    agent.channels.fileChannel.checkpointDir = /home/jony/tmp/flume_workspace/hdfs_sink/file_ch/checkpoint
    agent.channels.fileChannel.checkpointInterval = 60000
    agent.channels.fileChannel.dataDirs = /home/jony/tmp/flume_workspace/hdfs_sink/file_ch/data

    ```

2. 启动flume：

    ```
    ./bin/flume-ng agent --conf conf --conf-file conf/flume-conf-taildir2hdfs-filech.properties --name agent
    ```

3. 查看本地checkpoint和positionFile文件:

    ```
    hdfs_sink
    └── file_ch
        ├── checkpoint
        │   ├── checkpoint
        │   ├── checkpoint.meta
        │   ├── inflightputs
        │   ├── inflighttakes
        │   └── queueset
        ├── data
        │   ├── log-2
        │   ├── log-2.meta
        │   ├── log-3
        │   ├── log-3.meta
        │   ├── log-4
        │   ├── log-4.meta.tmp
        │   ├── log-5
        │   └── log-5.meta
        └── positionFile

    ```
4. 查看HDFS上的日志文件

    ```
    hadoop fs -ls /user/jony/flume_data/file_ch/2018-12-26
    ```

## 总结

在之前配置的Hadoop的基础上，我们安装了Flume日志收集工具，并通过四个例子，简单了解了flume的基本用法，其实在实际应用中flume的配置是比较复杂的，一般都会多级连接，并及进行负载均衡配置，将多个采集节点上的数据安全可靠地收集起来，汇聚到一个节点，然后对数据进行处理！