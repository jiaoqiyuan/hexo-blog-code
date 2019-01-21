---
title: MapReduce源码分析2-MapTask
date: 2019-01-20 11:02:13
tags: [大数据, MapReduce]
categories: 大数据
thumbnail: https://blog.eduonix.com/wp-content/uploads/2016/04/Introduction-to-Map-Reduce-Programming-model-reviewed-740X296.jpg
photos: https://blog.eduonix.com/wp-content/uploads/2016/04/Introduction-to-Map-Reduce-Programming-model-reviewed-740X296.jpg
---

> 之前介绍了 [MapReduce 的原理][1]和 [MapReduce 任务的提交与切片][2]相关的源码，知道了MapReduce的原理，这里分析一下具体执行Map任务的MapTask的源码，看一下Map任务的具体源码实现。

> 这是 MapReduce 源码分析系列的第二篇文章，有关 MapReduce 相关的所有内容可以到[这里][3]查看。

----

在[MapReduce 任务的提交与切片][2]这篇文章中，从源码角度看了一遍 MapReduce 任务从创建到提交，然后分片的整个流程，了解了分片机制的具体实现。有关资源调度的内容非常复杂，这里先不展开，回头有时间再好好研究一下，下面介绍一下节点 Container 中执行 Map 任务的 MapTask 的源码实现。

<!--more-->

## MapTask 介绍

MapTask 是具体执行 MapReduce 作业的地方，它会将输入的数据输出成Key-Value的形式存放到集群或者本地磁盘，然后由 ReduceTask 读取、聚合、输出到目标文件。

在 MapTask 中 `map` 函数可以由用户重写，进而实现自己想要的业务功能；另外在 Map 阶段还会进行数据混洗排序(shuffle)；由于 Map 阶段输出结果是要保存到磁盘的，而磁盘进行IO读写是非常耗费资源的事情，大数据下进行频繁磁盘读写更加会降低效率，所以在 Map 阶段框架内会做一些优化问题，比如使用环形缓冲区降低数据溢写到磁盘的频率，合并时的合并顺序的优化等等。

## Mapper 类的使用

先看一下怎么使用框架提供的 Mapper 类的，当用户需要自定义(大部分情况下都是需要的) Map 阶段数据处理所作的事情时，就需要创建一个自定义类继承自 `org.apache.hadoop.mapreduce.Mapper`，并自定义一个 `map` 函数，如下所示，完整代码[点击这里][4]：

```java
public static class LogMapper extends Mapper<LongWritable, Text, NullWritable, Text> {
    public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        try {
            //parseLog是我自定义个一个函数用于处理日志内容
            Text parsedLog = parseLog(value.toString());
            context.write(null, parsedLog);
        } catch (ParseException e) {
            e.printStackTrace();
        }
    }
}
```

设置好自定义 Map 类后，在创建 Job 时将 Map 类配置进去即可，类似于这样：

```java
public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
    //创建job
    Configuration config = new Configuration();
    Job job = Job.getInstance(config);
    //通过job设置一些参数
    ...
    job.setMapperClass(LogMapper.class);
    ...
}
```

## Mapper类做了什么

Mapper 中就是在一个循环中调用 `map` 方法：

```java
/**
  * Expert users can override this method for more complete control over the
  * execution of the Mapper.
  * @param context
  * @throws IOException
  */
public void run(Context context) throws IOException, InterruptedException {
  setup(context);
  try {
    while (context.nextKeyValue()) {
      map(context.getCurrentKey(), context.getCurrentValue(), context);
    }
  } finally {
    cleanup(context);
  }
}
```

这里有一个 `setup` 和 `cleanup` 方法，这两个方法都是只在创建Mapper对象的时候执行一次，适合做一些初始化和清理资源的工作。

这里的run方法是由框架调用的，在调用到map函数时，由框架利用反射机制进行Mapper的实例化然后调用的run。

我们在之前的YARN原理中说给，节点最终的执行任务都是由YARN在节点的container中先启动一个YarnChild，然后又YarnChild开启MapTask或ReduceTask的。这是Yarn的执行流程：

![Yarn执行流程][5]

程序具体执行时会启动一个叫 YarnChild 的类，在里面调用run防止执行具体的Task任务，相关具体代码如下。

```java
public static void main(String[] args) throws Throwable {
  ...
  // Create a final reference to the task for the doAs block
  final Task taskFinal = task;
  childUGI.doAs(new PrivilegedExceptionAction<Object>() {
    @Override
    public Object run() throws Exception {
      // use job-specified working directory
      setEncryptedSpillKeyIfRequired(taskFinal);
      FileSystem.get(job).setWorkingDirectory(job.getWorkingDirectory());
      //最终执行MapTask或ReduceTask的具体的地方
      taskFinal.run(job, umbilical); // run the task
      return null;
    }
  });
  ...
}
```

## MapTask 是什么

MapTask 是具体执行map任务的代码实现。在 MapTask 中，`run` 函数的代码如下：

```java
@Override
public void run(final JobConf job, final TaskUmbilicalProtocol umbilical)
  throws IOException, ClassNotFoundException, InterruptedException {
  this.umbilical = umbilical;

  if (isMapTask()) {
    // If there are no reducers then there won't be any sort. Hence the map 
    // phase will govern the entire attempt's progress.
    if (conf.getNumReduceTasks() == 0) {
      mapPhase = getProgress().addPhase("map", 1.0f);
    } else {
      // If there are reducers then the entire attempt's progress will be 
      // split between the map phase (67%) and the sort phase (33%).
      mapPhase = getProgress().addPhase("map", 0.667f);
      sortPhase  = getProgress().addPhase("sort", 0.333f);
    }
  }
  TaskReporter reporter = startReporter(umbilical);

  boolean useNewApi = job.getUseNewMapper();
  initialize(job, getJobID(), reporter, useNewApi);

  // check if it is a cleanupJobTask
  if (jobCleanup) {
    runJobCleanupTask(umbilical, reporter);
    return;
  }
  if (jobSetup) {
    runJobSetupTask(umbilical, reporter);
    return;
  }
  if (taskCleanup) {
    runTaskCleanupTask(umbilical, reporter);
    return;
  }

  if (useNewApi) {
    runNewMapper(job, splitMetaInfo, umbilical, reporter);
  } else {
    runOldMapper(job, splitMetaInfo, umbilical, reporter);
  }
  done(umbilical, reporter);
}
```

使用 `getNumReduceTasks` 获取 reduce 的数量，如果有reduce就增加排序，如果没有就不排序。**Reduce 的数量可以自己设置，在job中通过`setNumReduceTasks` 可以手动设置reduce的数量。**

接下来是一些框架调度的内容，先跳过，来到 `runNewMapper` 使用新的API启动Map，这是重点，解析来分析一下 `runNewMapper` 这个方法。

## MapTask任务的具体执行者 - runNewMapper

`runNewMapper` 的代码如下：

```java
@SuppressWarnings("unchecked")
private <INKEY,INVALUE,OUTKEY,OUTVALUE>
void runNewMapper(final JobConf job,
                  final TaskSplitIndex splitIndex,
                  final TaskUmbilicalProtocol umbilical,
                  TaskReporter reporter
                  ) throws IOException, ClassNotFoundException,
                            InterruptedException {
  // make a task context so we can get the classes
  org.apache.hadoop.mapreduce.TaskAttemptContext taskContext =
    new org.apache.hadoop.mapreduce.task.TaskAttemptContextImpl(job, 
                                                                getTaskID(),
                                                                reporter);
  // make a mapper
  org.apache.hadoop.mapreduce.Mapper<INKEY,INVALUE,OUTKEY,OUTVALUE> mapper =
    (org.apache.hadoop.mapreduce.Mapper<INKEY,INVALUE,OUTKEY,OUTVALUE>)
      ReflectionUtils.newInstance(taskContext.getMapperClass(), job);
  // make the input format
  org.apache.hadoop.mapreduce.InputFormat<INKEY,INVALUE> inputFormat =
    (org.apache.hadoop.mapreduce.InputFormat<INKEY,INVALUE>)
      ReflectionUtils.newInstance(taskContext.getInputFormatClass(), job);
  // rebuild the input split
  org.apache.hadoop.mapreduce.InputSplit split = null;
  split = getSplitDetails(new Path(splitIndex.getSplitLocation()),
      splitIndex.getStartOffset());
  LOG.info("Processing split: " + split);

  org.apache.hadoop.mapreduce.RecordReader<INKEY,INVALUE> input =
    new NewTrackingRecordReader<INKEY,INVALUE>
      (split, inputFormat, reporter, taskContext);
  
  job.setBoolean(JobContext.SKIP_RECORDS, isSkipping());
  org.apache.hadoop.mapreduce.RecordWriter output = null;
  
  // get an output object
  if (job.getNumReduceTasks() == 0) {
    output = 
      new NewDirectOutputCollector(taskContext, job, umbilical, reporter);
  } else {
    output = new NewOutputCollector(taskContext, job, umbilical, reporter);
  }

  org.apache.hadoop.mapreduce.MapContext<INKEY, INVALUE, OUTKEY, OUTVALUE> 
  mapContext = 
    new MapContextImpl<INKEY, INVALUE, OUTKEY, OUTVALUE>(job, getTaskID(), 
        input, output, 
        committer, 
        reporter, split);

  org.apache.hadoop.mapreduce.Mapper<INKEY,INVALUE,OUTKEY,OUTVALUE>.Context 
      mapperContext = 
        new WrappedMapper<INKEY, INVALUE, OUTKEY, OUTVALUE>().getMapContext(
            mapContext);

  try {
    input.initialize(split, mapperContext);
    mapper.run(mapperContext);
    mapPhase.complete();
    setPhase(TaskStatus.Phase.SORT);
    statusUpdate(umbilical);
    input.close();
    input = null;
    output.close(mapperContext);
    output = null;
  } finally {
    closeQuietly(input);
    closeQuietly(output, mapperContext);
  }
}
```

### 读取Map输入的类 - NewTrackingRecordReader

在 `runNewMapper` 方法中，有这样一段代码：

```java
// make the input format
org.apache.hadoop.mapreduce.InputFormat<INKEY,INVALUE> inputFormat =
  (org.apache.hadoop.mapreduce.InputFormat<INKEY,INVALUE>)
    ReflectionUtils.newInstance(taskContext.getInputFormatClass(), job);
...
org.apache.hadoop.mapreduce.RecordReader<INKEY,INVALUE> input =
  new NewTrackingRecordReader<INKEY,INVALUE>
    (split, inputFormat, reporter, taskContext);
```

这段代码的作用是创建Map输入读取类，用于后续帮助Map从分片中读取数据，打开 NewTrackingRecordReader 继续跟踪进去：

```java
NewTrackingRecordReader(org.apache.hadoop.mapreduce.InputSplit split,
        org.apache.hadoop.mapreduce.InputFormat<K, V> inputFormat,
        TaskReporter reporter,
        org.apache.hadoop.mapreduce.TaskAttemptContext taskContext)
        throws InterruptedException, IOException {
  ...
  this.real = inputFormat.createRecordReader(split, taskContext);
  ...
}
```

`NewTrackingRecordReader` 的构造函数创建了一个 `private final org.apache.hadoop.mapreduce.RecordReader<K,V>` 的 RecordReader 类，默认情况下 Map 的 inputFormat 是 TextInputFormat 类型，我们跟踪进 TextInputFormat 中看一下它的 `createRecordReader` 方法：

```java
@Override
public RecordReader<LongWritable, Text> 
  createRecordReader(InputSplit split,
                      TaskAttemptContext context) {
  String delimiter = context.getConfiguration().get(
      "textinputformat.record.delimiter");
  byte[] recordDelimiterBytes = null;
  if (null != delimiter)
    recordDelimiterBytes = delimiter.getBytes(Charsets.UTF_8);
  return new LineRecordReader(recordDelimiterBytes);
}
```

可以看到它返回的是一个 LineRecordReader 类型的 RecordReader，这里 LineRecordReader 是一个线性读取文件的工具类，也就是说我们在使用这个类读取Map的输入数据时是一行一行线性读取的，LineRecordReader 里面包含一些我们常用的方法，比如 `getCurrentKey`、`getCurrentValue`、`nextKeyValue`等方法。
 

### 分区及合并的实现 - NewOutputCollector

在 `runNewMapper` 方法中，有这样一段代码：

```java
// get an output object
if (job.getNumReduceTasks() == 0) {
  output = 
    new NewDirectOutputCollector(taskContext, job, umbilical, reporter);
} else {
  output = new NewOutputCollector(taskContext, job, umbilical, reporter);
}
```

如果没有Reduce Task，直接输出到磁盘目录，如果没有，就创建一个 `NewOutputCollector` 对象，在里面进行分区和排序后输出到磁盘。

NewOutputCollector 中实现了具体的分区操作：

```java
@SuppressWarnings("unchecked")
NewOutputCollector(org.apache.hadoop.mapreduce.JobContext jobContext,
                    JobConf job,
                    TaskUmbilicalProtocol umbilical,
                    TaskReporter reporter
                    ) throws IOException, ClassNotFoundException {
  collector = createSortingCollector(job, reporter);
  partitions = jobContext.getNumReduceTasks();
  if (partitions > 1) {
    partitioner = (org.apache.hadoop.mapreduce.Partitioner<K,V>)
      ReflectionUtils.newInstance(jobContext.getPartitionerClass(), job);
  } else {
    partitioner = new org.apache.hadoop.mapreduce.Partitioner<K,V>() {
      @Override
      public int getPartition(K key, V value, int numPartitions) {
        return partitions - 1;
      }
    };
  }
}
```

Partitioner 类就是在这里被实例化的，如果分区数量大于1，就通过反射创建一个Partitioner 实例，如果分区数量小于等于1，就直接返回分区数量为-1，最终就是只创建一个分区。

### map中的write的具体实现

我们知道自定义的map函数最终要调用 `context.write` 将结果写出，这里的 context 就是在 MapTask 中我们创建的 NewOutputCollector 对象中实现的：

先创建一个mapContext，再利用mapContext创建一个mapperContext。

```java
org.apache.hadoop.mapreduce.MapContext<INKEY, INVALUE, OUTKEY, OUTVALUE> 
mapContext = 
  new MapContextImpl<INKEY, INVALUE, OUTKEY, OUTVALUE>(job, getTaskID(), 
      input, output, 
      committer, 
      reporter, split);

org.apache.hadoop.mapreduce.Mapper<INKEY,INVALUE,OUTKEY,OUTVALUE>.Context 
    mapperContext = 
      new WrappedMapper<INKEY, INVALUE, OUTKEY, OUTVALUE>().getMapContext(
          mapContext);
```

接下来调用 `mapper.run(mapperContext)` ，最终执行的就是mapperContext中的write方法。

```java
try {
  input.initialize(split, mapperContext);
  mapper.run(mapperContext);
  mapPhase.complete();
  setPhase(TaskStatus.Phase.SORT);
  statusUpdate(umbilical);
  input.close();
  input = null;
  output.close(mapperContext);
  output = null;
} finally {
  closeQuietly(input);
  closeQuietly(output, mapperContext);
}
```

我们看一下 NewOutputCollector 是怎么实现write方法的：

```java
  @Override
  public void write(K key, V value) throws IOException, InterruptedException {
    collector.collect(key, value,
                      partitioner.getPartition(key, value, partitions));
  }
```

再深入，进入 collector 的实现类 MapOutputBuffer，这个类也在MapTask中，是个内部类，它的 `collect` 实现如下：

```java
if (bufferRemaining <= 0) {
  // start spill if the thread is not running and the soft limit has been
  // reached
  spillLock.lock();
  try {
    do {
      if (!spillInProgress) {
        final int kvbidx = 4 * kvindex;
        final int kvbend = 4 * kvend;
        // serialized, unspilled bytes always lie between kvindex and
        // bufindex, crossing the equator. Note that any void space
        // created by a reset must be included in "used" bytes
        final int bUsed = distanceTo(kvbidx, bufindex);
        final boolean bufsoftlimit = bUsed >= softLimit;
        if ((kvbend + METASIZE) % kvbuffer.length !=
            equator - (equator % METASIZE)) {
          // spill finished, reclaim space
          resetSpill();
          bufferRemaining = Math.min(
              distanceTo(bufindex, kvbidx) - 2 * METASIZE,
              softLimit - bUsed) - METASIZE;
          continue;
        } else if (bufsoftlimit && kvindex != kvend) {
          // spill records, if any collected; check latter, as it may
          // be possible for metadata alignment to hit spill pcnt
          startSpill();
          final int avgRec = (int)
            (mapOutputByteCounter.getCounter() /
            mapOutputRecordCounter.getCounter());
          // leave at least half the split buffer for serialization data
          // ensure that kvindex >= bufindex
          final int distkvi = distanceTo(bufindex, kvbidx);
          final int newPos = (bufindex +
            Math.max(2 * METASIZE - 1,
                    Math.min(distkvi / 2,
                              distkvi / (METASIZE + avgRec) * METASIZE)))
            % kvbuffer.length;
          setEquator(newPos);
          bufmark = bufindex = newPos;
          final int serBound = 4 * kvend;
          // bytes remaining before the lock must be held and limits
          // checked is the minimum of three arcs: the metadata space, the
          // serialization space, and the soft limit
          bufferRemaining = Math.min(
              // metadata max
              distanceTo(bufend, newPos),
              Math.min(
                // serialization max
                distanceTo(newPos, serBound),
                // soft limit
                softLimit)) - 2 * METASIZE;
        }
      }
    } while (false);
  } finally {
    spillLock.unlock();
  }
```

这里有一个 `spillLock` 锁， 这个锁跟 MapTask 中的另一个进行排序、合并、写入磁盘操作的内部类共享：

```java
protected class SpillThread extends Thread {

  @Override
  public void run() {
    spillLock.lock();
    spillThreadRunning = true;
    try {
      while (true) {
        spillDone.signal();
        while (!spillInProgress) {
          spillReady.await();
        }
        try {
          spillLock.unlock();
          sortAndSpill();
        } catch (Throwable t) {
          sortSpillException = t;
        } finally {
          spillLock.lock();
          if (bufend < bufstart) {
            bufvoid = kvbuffer.length;
          }
          kvstart = kvend;
          bufstart = bufend;
          spillInProgress = false;
        }
      }
    } catch (InterruptedException e) {
      Thread.currentThread().interrupt();
    } finally {
      spillLock.unlock();
      spillThreadRunning = false;
    }
  }
}
```

这里的 `sortAndSpill` 方法就是进行排序和溢写的方法，具体实现接着往下看：

```java
  private void sortAndSpill() throws IOException, ClassNotFoundException,
                                      InterruptedException {
    //approximate the length of the output file to be the length of the
    //buffer + header lengths for the partitions
    final long size = distanceTo(bufstart, bufend, bufvoid) +
                partitions * APPROX_HEADER_LENGTH;
    FSDataOutputStream out = null;
    try {
      // create spill file
      final SpillRecord spillRec = new SpillRecord(partitions);
      final Path filename =
          mapOutputFile.getSpillFileForWrite(numSpills, size);
      out = rfs.create(filename);

      final int mstart = kvend / NMETA;
      final int mend = 1 + // kvend is a valid record
        (kvstart >= kvend
        ? kvstart
        : kvmeta.capacity() + kvstart) / NMETA;
      //在溢写数据前先排序
      sorter.sort(MapOutputBuffer.this, mstart, mend, reporter);
      int spindex = mstart;
      final IndexRecord rec = new IndexRecord();
      final InMemValBytes value = new InMemValBytes();
      for (int i = 0; i < partitions; ++i) {
        IFile.Writer<K, V> writer = null;
        try {
          long segmentStart = out.getPos();
          FSDataOutputStream partitionOut = CryptoUtils.wrapIfNecessary(job, out);
          writer = new Writer<K, V>(job, partitionOut, keyClass, valClass, codec,
                                    spilledRecordsCounter);
          //如果设置不进行合并操作
          if (combinerRunner == null) {
            // spill directly
            DataInputBuffer key = new DataInputBuffer();
            while (spindex < mend &&
                kvmeta.get(offsetFor(spindex % maxRec) + PARTITION) == i) {
              final int kvoff = offsetFor(spindex % maxRec);
              int keystart = kvmeta.get(kvoff + KEYSTART);
              int valstart = kvmeta.get(kvoff + VALSTART);
              key.reset(kvbuffer, keystart, valstart - keystart);
              getVBytesForOffset(kvoff, value);
              //写入磁盘
              writer.append(key, value);
              ++spindex;
            }
          } else {  //设置进行合并操作
            int spstart = spindex;
            while (spindex < mend &&
                kvmeta.get(offsetFor(spindex % maxRec)
                          + PARTITION) == i) {
              ++spindex;
            }
            // Note: we would like to avoid the combiner if we've fewer
            // than some threshold of records for a partition
            if (spstart != spindex) {
              combineCollector.setWriter(writer);
              RawKeyValueIterator kvIter =
                new MRResultIterator(spstart, spindex);
              // 执行合并操作
              combinerRunner.combine(kvIter, combineCollector);
            }
          }

          // close the writer
          writer.close();

          // record offsets
          rec.startOffset = segmentStart;
          rec.rawLength = writer.getRawLength() + CryptoUtils.cryptoPadding(job);
          rec.partLength = writer.getCompressedLength() + CryptoUtils.cryptoPadding(job);
          spillRec.putIndex(rec, i);

          writer = null;
        } finally {
          if (null != writer) writer.close();
        }
      }

      if (totalIndexCacheMemory >= indexCacheMemoryLimit) {
        // create spill index file
        Path indexFilename =
            mapOutputFile.getSpillIndexFileForWrite(numSpills, partitions
                * MAP_OUTPUT_INDEX_RECORD_LENGTH);
        spillRec.writeToFile(indexFilename, job);
      } else {
        indexCacheList.add(spillRec);
        totalIndexCacheMemory +=
          spillRec.size() * MAP_OUTPUT_INDEX_RECORD_LENGTH;
      }
      LOG.info("Finished spill " + numSpills);
      ++numSpills;
    } finally {
      if (out != null) out.close();
    }
  }
```

## 初始化及运行

上面配置好Map的输入读取工具类以及分区和溢写方法后，在 `MapTask` 中，接下来就是初始化和运行的代码：

```java
try {
  input.initialize(split, mapperContext);
  mapper.run(mapperContext);
  mapPhase.complete();
  setPhase(TaskStatus.Phase.SORT);
  statusUpdate(umbilical);
  input.close();
  input = null;
  output.close(mapperContext);
  output = null;
} finally {
  closeQuietly(input);
  closeQuietly(output, mapperContext);
}
```

我们知道 input 的类型是 NewTrackingRecordReader ，而 NewTrackingRecordReader 最终是创建了一个 LineRecordReader ，所以这里的 `initialize` 最终调用的是 LineRecordReader 中的 `initialize` 方法，代码如下：

```java
public void initialize(InputSplit genericSplit,
                        TaskAttemptContext context) throws IOException {
  FileSplit split = (FileSplit) genericSplit;
  Configuration job = context.getConfiguration();
  this.maxLineLength = job.getInt(MAX_LINE_LENGTH, Integer.MAX_VALUE);
  //从切片中获得文件位置、起始偏移量start、结束偏移量end。
  start = split.getStart();
  end = start + split.getLength();
  final Path file = split.getPath();

  // open the file and seek to the start of the split
  //通过文件位置信息获取到分布式文件的 FileSystem ，然后打开文件。
  final FileSystem fs = file.getFileSystem(job);
  fileIn = fs.open(file);
  
  //判断是否是压缩文件
  CompressionCodec codec = new CompressionCodecFactory(job).getCodec(file);
  if (null!=codec) {
    isCompressedInput = true;	
    decompressor = CodecPool.getDecompressor(codec);
    if (codec instanceof SplittableCompressionCodec) {
      final SplitCompressionInputStream cIn =
        ((SplittableCompressionCodec)codec).createInputStream(
          fileIn, decompressor, start, end,
          SplittableCompressionCodec.READ_MODE.BYBLOCK);
      in = new CompressedSplitLineReader(cIn, job,
          this.recordDelimiterBytes);
      start = cIn.getAdjustedStart();
      end = cIn.getAdjustedEnd();
      filePosition = cIn;
    } else {
      in = new SplitLineReader(codec.createInputStream(fileIn,
          decompressor), job, this.recordDelimiterBytes);
      filePosition = fileIn;
    }
  } else {
    fileIn.seek(start);
    in = new UncompressedSplitLineReader(
        fileIn, job, this.recordDelimiterBytes, split.getLength());
    filePosition = fileIn;
  }
  // If this is not the first split, we always throw away first record
  // because we always (except the last split) read one extra line in
  // next() method.
  //判断是否为第一个切片，如果不是第一个切片就更新文件起始偏移量的值start
  if (start != 0) {
    start += in.readLine(new Text(), 0, maxBytesToConsume(start));
  }
  this.pos = start;
}
```

这里初始化所作的工作就是：

- 从切片中获得文件位置、起始偏移量start、结束偏移量end。

- 通过文件位置信息获取到分布式文件的 FileSystem ，然后打开文件。

- 判断是否是压缩文件，如果不使用压缩，就直接从分配给自己的起始偏移量start的位置开始读取文件。如果使用了压缩就需要读取所有的文件内容后再解压缩，然后再进行处理。根据输入数据是否进行了压缩，设置不同的 SplitLineReader 进行接下来的数据读取操作。

- 判断是否为第一个切片，如果不是第一个切片就更新文件起始偏移量的值start。这里采用的是直接忽略掉块的第一行的做法，因为数据可能分布在不同的块中，进而导致数据的不完整性。


## nextKeyValue 、 getCurrentKey 、 getCurrentValue

在最开始提到的Mapper中，map是夹在while循环中的，代码如下：

```java
while (context.nextKeyValue()) {
  map(context.getCurrentKey(), context.getCurrentValue(), context);
}
```

这里的 `nextKeyValue、getCurrentKey、getCurrentValue` 三个方法通过我们分析，可以知道就是 LineRecordReader 中的三个方法。

### nextKeyValue

LineRecordReader 中 `nextKeyValue` 的代码实现如下：

```java
public boolean nextKeyValue() throws IOException {
  if (key == null) {
    key = new LongWritable();
  }
  key.set(pos);
  if (value == null) {
    value = new Text();
  }
  int newSize = 0;
  // We always read one extra line, which lies outside the upper
  // split limit i.e. (end - 1)
  while (getFilePosition() <= end || in.needAdditionalRecordAfterSplit()) {
    if (pos == 0) {
      newSize = skipUtfByteOrderMark();
    } else {
      newSize = in.readLine(value, maxLineLength, maxBytesToConsume(pos));
      pos += newSize;
    }

    if ((newSize == 0) || (newSize < maxLineLength)) {
      break;
    }

    // line too long. try again
    LOG.info("Skipped line of size " + newSize + " at pos " + 
              (pos - newSize));
  }
  if (newSize == 0) {
    key = null;
    value = null;
    return false;
  } else {
    return true;
  }
}
```

所做的事情就是：

- 设置默认key类型为LongWritable，value默认类型为Text类型

- 将行偏移量赋值给key，也就是说Map的输入Key存放的就是行的偏移量。

- 从 UncompressedSplitLineReader 中读取文件数据到value中。

- 读完将偏移量信息加上读取的数据长度。

- 如果最终没有数据可读就退出。

##  getCurrentKey 、 getCurrentValue

这两个在上面的 `nextKeyValue` 中已经赋值过，直接返回就行，很简单，代码如下：

```java
@Override
public LongWritable getCurrentKey() {
  return key;
}

@Override
public Text getCurrentValue() {
  return value;
}
```

## 总结

有点乱，我是一遍翻看源码，一遍查资料，反复看了很多遍，总结下来就是MapTask是由每个节点的YarnChild启动的，在真正开始运行map之前，会先根据job配置确定好输入输出类型、是否压缩、输入文件的路径信息、分片信息，加载好相应的读取工具类，然后进行初始化操作，确认好文件偏移量信息。在最终执行读取操作的时候利用已经配置好的工具类，逐行将文件内的数据读取到内存供 `map` 函数使用。当然这中间还会加入许许多多错误判断机制和默认设置，但是Map的大致流程就是上面说的那样了。源码分析起来好累！

---- 

## 参考文章

> [Hadoop-MapReduce源码分析][6]

[1]: https://jiaoqiyuan.cn/2018/12/24/MapReduce%E7%9A%84%E5%8E%9F%E7%90%86/
[2]: https://jiaoqiyuan.cn/2019/01/19/MapReduceSourceCode-JobAndSplite/
[3]: https://jiaoqiyuan.cn/tags/MapReduce/
[4]: https://github.com/jiaoqiyuan/163-bigdate-note/blob/master/%E6%97%A5%E5%BF%97%E8%A7%A3%E6%9E%90%E5%8F%8A%E8%AE%A1%E7%AE%97%EF%BC%9AMR/%E7%BC%96%E5%86%99%E4%B8%80%E4%B8%AAMR%E7%A8%8B%E5%BA%8F/etl/src/main/com/bigdata/etl/job/ParseLogJob.java
[5]: https://dev.tencent.com/u/jiaoqiyuan/p/hexo-blog-code/git/raw/master/source/img/YARN%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B.png
[6]: https://blog.xiaoxiaomo.com/2016/07/08/Hadoop-MapReduce%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/