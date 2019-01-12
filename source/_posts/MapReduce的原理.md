---
title: MapReduce的原理
date: 2018-12-24 06:25:10
tags:
categories: 大数据
thumbnail: https://blog.eduonix.com/wp-content/uploads/2016/04/Introduction-to-Map-Reduce-Programming-model-reviewed-740X296.jpg
photos: https://blog.eduonix.com/wp-content/uploads/2016/04/Introduction-to-Map-Reduce-Programming-model-reviewed-740X296.jpg
---

    MapReduce是hadoop中的一个重要的计算框架，善于进行大量数据的离线处理，这里总结一下我对MapReduce的理解。

## MapReduce组成部分

MapReduce分Map和Reduce两个阶段，中间穿插有Shuffle过程。

这里Map阶段一般是对规模较大的数据进行分片、解析、整理，最后输出Key-Value的键值对;

Reduce阶段，主要是接收Map阶段发来的Key—Value键值对，不过这里接收到的是经过Shuffle混洗过的数据，一般是Key值相同的所有Value值的集合，而且Value值的集合放在了一个迭代器中，可以很方便的进行遍历处理。

Shuffle阶段发生在Map和Reduce阶段，Map中的Shuffle主要负责数据的溢写（Spill）、数据的分区（Partitino）、数据的排序、数据的合并（Merge）。Reduce中的Shuffle主要负责将Map阶段的数据拷贝到Reduce的缓冲区、数据的溢写（Spill），数据的合并（Merger）。在后面讲解Shuffle过程是时再详细讲解。

## MapReduce的原理

### Map

Map阶段主要负责从数据源（常用的是HDFS）中读取数据，然后进行第一阶段的处理。这里如果数据源太大，比如大于HDFS的默认Block块大小128M，Map就会对数据进行分片（Split），分片后有多少个分片MR（MapReduce的简称）框架就会分配多少个Map来处理对应的分片数据，这里可以得到一个信息就是，数据分片的个数就是Map的个数。分片后，每个Spilt分片由Mapper类的map函数进行数据处理，根据业务需要，将数据抽取、整理、组合成想要的Key-Value格式的数据发送给reduce端。

![map][1]

这里的Key值需要是MR中满足Hadoop序列化要求的类型，或者是自定义满足Hadoop序列化要求的类型。Hadoop使用自己的序列化格式Writable，它紧凑而且速度快，这是Hadoop不使用Java自带序列化方法的主要原因。Writable接口的定义如下：

```java
public interface Writable {
  /** 
   * Serialize the fields of this object to <code>out</code>.
   * 
   * @param out <code>DataOuput</code> to serialize this object into.
   * @throws IOException
   */
  void write(DataOutput out) throws IOException;

  /** 
   * Deserialize the fields of this object from <code>in</code>.  
   * 
   * <p>For efficiency, implementations should attempt to re-use storage in the 
   * existing object where possible.</p>
   * 
   * @param in <code>DataInput</code> to deseriablize this object from.
   * @throws IOException
   */
  void readFields(DataInput in) throws IOException;
}
```

Hadoop本身提供了对Java基本类型的Writable封装：

| Java基本类型 | Writable | 序列化后长度 |
|:-----------:|:--------:|:-----------:|
| 布尔型 boolean | BooleanWritable | 1 |
| 字节型 byte | ByteWritable | 1 |
| 整形 int | IntWritable or VIntWritable | 4 or 1~5 |
| 浮点型 float | FloatWritable | 4 |
| 长整型 | LongWritable or VLongWritable | 8 or 1~9 |
| 双精度浮点类型 double | DoubleWritable | 8 |

对String类型封装为Text类，也提供了对Array和Map集合类型的Writable封装。集合类型的封装一般来说，首先要在序列化的时候把序列的大小写入进去，然后再将集合中的元素依次序列化。

在Hadoop框架中不仅可以使用Hadoop已经提供的序列化类型，还可使用自定义的序列化类型。这里不再展开，有兴趣的话可以搜索Hadoop序列化。

在Map将数据处理完后，会组织成Key-Value格式的数据交给Reduce进行处理，在将数据交给reduce之前，这中间MR框架还会做很多事情，而且Shuffle过程也是发生在这段时间的。现在你简单地认为Map将数据处理完后，经过MR框架的整理，然后交给了Reduce进行处理就行啦。具体MR框架进行了那些整理操作，以及Reduce端进行了哪些处理，后面我们再讲。

Map阶段要理解好数据的分片机制，默认情况下如果数据文件比HDFS块大就要进行分片，另外在多个输入文件的情况下，即使文件比HDFS块小也会当作一个Split分片进行处理。比如我有两个输入文件，一个5M大小，一个150M，那么这个5M大小的文件会作为一个分片进行处理，另一个150M大小的文件因为大于HDFS默认分片大小128M，所以会被分成两个Split，也就是总共有三个Split，对应需要3个Map进行处理。

### Reduce

Reduce负责接收Map发来的Key-Value格式的数据，不过这里的Key-Value跟Map输出的Key-Value并不一样，我们上面说过Map在将数据交给Reduce之前，数据会经过MR框架的整理和Shuffle混洗，这就是Map输出的数据跟Reduce实际处理的数据不同的原因。

Reduce接收到的数据的Key值跟Map输出的Key值相同，但是Reduce接收到的Value值是一个集合，集合内存放的是Map输出的所有与当前Key值相同的Value值的集合。这里可以这样理解，Map输出的是一个Key值对应一个Value值，整个Map过程最终会输出很多个Key-Value键值对，这里面会有一些Key值相等的键值对，MR框架和Shuffle就是将这些键值对按Key值排序，并且按Key进行合并，将相同Key值的所有Value值放到一个集合中，然后将这个集合与Key值再组成个键值对交给Reduce进行处理。

![reduce][2]

Reduce个数默认由Partition分区个数决定，有多少个Partition就会有多少个Reduce，但是也可以手动指定Reduce的个数。每个Reduce会对应一个输出文件，文件名默认以Part-开头，后面是分区编号。

Partition是在Shuffle过程中产生的，后面讲Shuffle的时候再详细展开。

### Shuffle

系统执行排序、将Map输出作为输入传给Reduce的过程成为Shuffle，shuffle是MR的心脏，了解Shuffle有助于我们理解MR的工作机制，进而方便我们优化MR的运行。

Map将数据输出时并不是简单地将数据写出到磁盘上，而是要经过shuffle过程。

![shuffle][3]

#### Map中的Shuffle

首先，map函数处理完数据后，会将数据放入到一个环形缓冲区内，默认情况下，缓冲区的大小是100M，缓冲区有一个阈值默认是80%，就是说如果缓冲区内空间被占满了80%，就会有一个后台进程开始把缓冲区的内容溢写（Spill）到磁盘。

在溢写过程中，map输出继续向缓冲区写数据，如果在此期间缓冲区被填满，map就会被阻塞，直到写磁盘过程完成。

每次内存缓冲区达到溢出阈值，就会新建一个溢出文件（spill file），因此map任务写完最后一个溢出文件后，可能会有多个溢出文件。后台进程在将数据从缓冲区溢写到溢出文件前，会对数据进行分区，并在每个分区中按Key值进行内存中排序，每条数据经过处理后都会有一个对应的partition号，从而被copy到指定的Reduce中。

Partition分区获取reducer号：

```java
collector.collect(key,value, partitioner.getPartition(key, value, partitions))
```

其中partitions是这么得到的：

```java
partitions = jobContext.getNumReduceTasks()
```

getNumReduceTasks()默认值是1，：

```java
getInt(JobContext.NUM_REDUCES, 1);
public int getNumReduceTasks() { 
    return getInt(JobContext.NUM_REDUCES, 1);
}
```

partitions的值有两种情况：

- partitions > 1时，通过反射得到partitioner：

```java
partitioner =(org.apache.hadoop.mapreduce.Partitioner<K,V>); ReflectionUtils.newInstance(jobContext.getPartitionerClass(),job);
```

getPartitionerClass()返回值是conf.getClass(PARTITIONER_CLASS_ATTR, HashPartitioner.class)默认是HashPartitioner实现，getPartition()的实现如下：

```java
public int getPartition(K key, V value, int numReduceTasks) {
    //按位与&操作是为了消除符号位
    //注意：Java中的基本数据类型都是有符号的
    return (key.hashCode() & Integer.MAX_VALUE) % numReduceTasks;
}
```

- patitions <= 1时，直接返回0

默认情况下patitions=1,也就是getPartition只有一个返回值就是0,shuffle溢写出的文件只有一个partition分区，最终只有一个reduce任务进行数据的处理。当然你可以手动指定reduce的数量和编写自定义partition分区函数，来控制partition分区数和reduce任务数，不过需要注意的是输出文件的个数是由reduce的数量决定的，而数据具体写到哪个分区是由partition函数决定的，比如reduce数量设置为3,但是自定义partition函数返回5个partition，那么最终输出文件是3个，但是3个输出文件中的数据是不全的，因为有两个partition被抛弃掉了。具体参考链接:[mapreduce中Partitioner数量与reducetask数量对结果影响][4]。

在溢写数据划分partition并排序后，如果设置了combiner函数，那么combiner函数就在排序后的输出上运行，如果在溢写过程中至少存在3个溢出文件（通过mapreduce.map.combine.minspills属性设置），则combiner就会在输出文件写到磁盘之前运行，将溢写出的小文件合并成一个大文件，如果只有1或2个溢出文件，那么由于map输出规模较少，因而不值得调用combiner带来的开销，因此不会为该map输出运行combiner。

#### reduce的shuttfle

map将输出数据写到本地磁盘，在每个map任务都完成后，reduce任务根据map的分区情况将同一分区的数据拷贝到reduce端进行处理，这就是reduce的复制阶段。reduce任务默认有5个复制线程，因此能够并行取得map输出，这个默认值可以通过mapreduce.reduce.shuffle.parallelcopies设置。

> map任务完成后，会通过心跳机制通知它们的application master，因此对于指定作业，application master知道map输出和主机位置之间的映射关系。reduce中的一个线程定期询问master以便获取map输出主机的位置，知道获得所有输出位置。

如果map输出相当小，会被复制到reduce任务JVM的内存，否则map输出被复制到磁盘。一旦内存缓冲区达到阈值大小或者map出处的阈值，则合并后溢写到磁盘中。如果指定combiner，则在合并期间运行combiner以降低写入磁盘的数据量。

随着磁盘上溢写出的Split文件增多，后台线程会将他们合并成更大的、排好序的文件，这会为后面的合并节省一些时间。

相同的partition都被复制到同一个reduce后，reduce进入排序阶段（应该说是合并阶段，因为map阶段已经排过序了），这里合并是循环进行的，比如有50个map输出，而合并因子是10（10是默认设置，可以自行设置），合并将进行5次，每次将10个文件和秉承一个文件，因此最后产生5个中间文件。

实际中，对于50个map输出，并不是每次进行5次合并，而是在最后一次reduce时直接把数据写入reduce函数，从而省略一次磁盘往返行程。也就是第一次合并5个文件，然后接下来的四次合并完整的10个文件，最后一次合并中5个已合并的文件和剩下5个未合并的文件共计10个文件。


## 总结

MR的原理重点是理解好map和reduce的输入输出文件的格式，shuffle过程中溢写的时间，排序的依据。

从数据输入到map，经过map处理，将数据放入缓冲区，再分区排序后溢写到split文件，然后多个split文件合并成一个大文件;reduce后台线程在所有map输出都完成后将同一分区的数据拷贝到reduce的内存缓冲区中，缓冲区满后将数据溢写到reduce的spilit文件，然后多个split文件合并成一个文件，这些由split合并而成的文件再合并成一个大文件，交给reduce程序处理。reduce处理完成后输出到HDFS上。

[1]: https://github.com/jiaoqiyuan/163-bigdate-note/raw/master/%E6%97%A5%E5%BF%97%E8%A7%A3%E6%9E%90%E5%8F%8A%E8%AE%A1%E7%AE%97%EF%BC%9AMR/img/Map%E8%BF%90%E8%A1%8C%E8%BF%87%E7%A8%8B.png
[2]: https://github.com/jiaoqiyuan/163-bigdate-note/raw/master/%E6%97%A5%E5%BF%97%E8%A7%A3%E6%9E%90%E5%8F%8A%E8%AE%A1%E7%AE%97%EF%BC%9AMR/img/Reduce%E5%A4%84%E7%90%86%E8%BF%87%E7%A8%8B.png
[3]: https://github.com/jiaoqiyuan/163-bigdate-note/raw/master/%E6%97%A5%E5%BF%97%E8%A7%A3%E6%9E%90%E5%8F%8A%E8%AE%A1%E7%AE%97%EF%BC%9AMR/img/shuffle%E8%BF%87%E7%A8%8B.png
[4]: https://blog.csdn.net/qq_35699475/article/details/75582072