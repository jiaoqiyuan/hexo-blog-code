---
title: MapReduce源码分析3-ReduceTask
date: 2019-01-26 11:52:15
tags: [大数据, MapReduce]
categories: 大数据
thumbnail: https://blog.eduonix.com/wp-content/uploads/2016/04/Introduction-to-Map-Reduce-Programming-model-reviewed-740X296.jpg
photos: https://blog.eduonix.com/wp-content/uploads/2016/04/Introduction-to-Map-Reduce-Programming-model-reviewed-740X296.jpg
---

> 之前介绍了 [MapReduce 的原理][1]、 [MapReduce 任务的提交与切片][2]以及[Map 任务的执行者MapTask][3]相关的源码，知道了 MapReduce 的原理并了解了部分 Map 任务源码的实现，这里分析一下具体执行 Reduce 任务的 ReduceTask 的源码，看一下 Reduce 任务的具体源码实现。

> 这是 MapReduce 源码分析系列的第三篇文章，有关 MapReduce 相关的所有内容可以到[这里][4]查看。

----

之前看到了 MapTask 任务的具体执行流程，Map 任务执行的输出的一个或多个结果最终会交给 Reduce 端，由 Reduce 汇聚处理之后输出。现在就看一下 ReduceTask 的具体源码，看一下 Reduce 端都做了些什么事。这里有关 Shuffle 的代码先不分析，主要看 Reduce 端的实现。

<!--more-->

## 自定义 Reduce 处理逻辑 - Reducer

在编写 Reduce 端处理逻辑时一般需要继承 `org.apache.hadoop.mapreduce.Reducer` 类，然后重写 `reduce` 方法，就可以用自定义的 `reduce` 方法来处理数据了。

```java
public static class LogReducer extends Reducer<TextLongWritable, LogGenericWritable, Text, Text> {
    ......
    public void reduce(TextLongWritable key, Iterable<LogGenericWritable> values, Context context) throws IOException, InterruptedException {
            ......
        }
    }
}
```

Reducer类中有一个 `run` 方法，reduce函数就是在那里被调用的：

```java
/**
  * Advanced application writers can use the 
  * {@link #run(org.apache.hadoop.mapreduce.Reducer.Context)} method to
  * control how the reduce task works.
  */
public void run(Context context) throws IOException, InterruptedException {
  setup(context);
  try {
    while (context.nextKey()) {
      reduce(context.getCurrentKey(), context.getValues(), context);
      // If a back up store is used, reset it
      Iterator<VALUEIN> iter = context.getValues().iterator();
      if(iter instanceof ReduceContext.ValueIterator) {
        ((ReduceContext.ValueIterator<VALUEIN>)iter).resetBackupStore();        
      }
    }
  } finally {
    cleanup(context);
  }
}
```

这里有关 `context` 上下文的介绍可以参考 [MapTask源码][3] 那篇文章，这里需要注意的一点是 reduce 端最终汇聚到一个 Reduce 数据量可能会远远超出系统内存，Reduce 是怎么处理这种情况的？

下面开始看一下 ReduceTask 这个类。

## Reduce 任务的实际执行者 - ReduceTask

跟 MapTask 一样，ReduceTask 会从run方法开始执行：

```java
@Override
@SuppressWarnings("unchecked")
public void run(JobConf job, final TaskUmbilicalProtocol umbilical)
  throws IOException, InterruptedException, ClassNotFoundException {
  job.setBoolean(JobContext.SKIP_RECORDS, isSkipping());

  if (isMapOrReduce()) {
    copyPhase = getProgress().addPhase("copy");
    sortPhase  = getProgress().addPhase("sort");
    reducePhase = getProgress().addPhase("reduce");
  }
  // start thread that will handle communication with parent
  TaskReporter reporter = startReporter(umbilical);
  
  boolean useNewApi = job.getUseNewReducer();
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
  
  // Initialize the codec
  codec = initCodec();
  RawKeyValueIterator rIter = null;
  ShuffleConsumerPlugin shuffleConsumerPlugin = null;
  
  Class combinerClass = conf.getCombinerClass();
  CombineOutputCollector combineCollector = 
    (null != combinerClass) ? 
    new CombineOutputCollector(reduceCombineOutputCounter, reporter, conf) : null;

  Class<? extends ShuffleConsumerPlugin> clazz =
        job.getClass(MRConfig.SHUFFLE_CONSUMER_PLUGIN, Shuffle.class, ShuffleConsumerPlugin.class);
        
  shuffleConsumerPlugin = ReflectionUtils.newInstance(clazz, job);
  LOG.info("Using ShuffleConsumerPlugin: " + shuffleConsumerPlugin);

  ShuffleConsumerPlugin.Context shuffleContext = 
    new ShuffleConsumerPlugin.Context(getTaskID(), job, FileSystem.getLocal(job), umbilical, 
                super.lDirAlloc, reporter, codec, 
                combinerClass, combineCollector, 
                spilledRecordsCounter, reduceCombineInputCounter,
                shuffledMapsCounter,
                reduceShuffleBytes, failedShuffleCounter,
                mergedMapOutputsCounter,
                taskStatus, copyPhase, sortPhase, this,
                mapOutputFile, localMapFiles);
  shuffleConsumerPlugin.init(shuffleContext);

  rIter = shuffleConsumerPlugin.run();

  // free up the data structures
  mapOutputFilesOnDisk.clear();
  
  sortPhase.complete();                         // sort is complete
  setPhase(TaskStatus.Phase.REDUCE); 
  statusUpdate(umbilical);
  Class keyClass = job.getMapOutputKeyClass();
  Class valueClass = job.getMapOutputValueClass();
  RawComparator comparator = job.getOutputValueGroupingComparator();

  if (useNewApi) {
    runNewReducer(job, umbilical, reporter, rIter, comparator, 
                  keyClass, valueClass);
  } else {
    runOldReducer(job, umbilical, reporter, rIter, comparator, 
                  keyClass, valueClass);
  }

  shuffleConsumerPlugin.close();
  done(umbilical, reporter);
}
```

### Reduce 端的输入迭代器

这里面有一个 `RawKeyValueIterator` 类型的迭代器，用于迭代 Reduce 端的输入数据：

```java
public void run(JobConf job, final TaskUmbilicalProtocol umbilical)
    throws IOException, InterruptedException, ClassNotFoundException {
    ...
    RawKeyValueIterator rIter = null;
    rIter = shuffleConsumerPlugin.run();
    ...
    }
```

这里的 `shuffleConsumerPlugin.run()` 运行 Shuffle 从多个 Map 中拉取数据到 Reduce 端，供 Reduce 消费。

### Reduce 端的 `RawComparator` 分组比较器

MapTask 中的 `run` 方法下还有一个设置分组比较器的代码：

```java
public void run(JobConf job, final TaskUmbilicalProtocol umbilical)
    throws IOException, InterruptedException, ClassNotFoundException {
    ...
    RawComparator comparator = job.getOutputValueGroupingComparator();
    ...
    }
```

设置分组比较器的作用是为 reduce 进行分组， `getOutputValueGroupingComparator` 的实现如下：

```java
/** 
  * Get the user defined {@link WritableComparable} comparator for 
  * grouping keys of inputs to the reduce.
  * 
  * @return comparator set by the user for grouping values.
  * @see #setOutputValueGroupingComparator(Class) for details.
  */
public RawComparator getOutputValueGroupingComparator() {
  Class<? extends RawComparator> theClass = getClass(
    JobContext.GROUP_COMPARATOR_CLASS, null, RawComparator.class);
  if (theClass == null) {
    return getOutputKeyComparator();
  }
  
  return ReflectionUtils.newInstance(theClass, this);
}
```

先通过 `GROUP_COMPARATOR_CLASS` 看一下用户有没有配置分组比较器，如果有就使用用户自定义分组比较器，如果没有就调用 `getOutputKeyComparator()` 返回一个默认的分组比较器。

看一下默认的分组比较器是什么，进入 `getOutputKeyComparator`，代码如下：

```java
/**
  * Get the {@link RawComparator} comparator used to compare keys.
  * 
  * @return the {@link RawComparator} comparator used to compare keys.
  */
public RawComparator getOutputKeyComparator() {
  Class<? extends RawComparator> theClass = getClass(
    JobContext.KEY_COMPARATOR, null, RawComparator.class);
  if (theClass != null)
    return ReflectionUtils.newInstance(theClass, this);
  return WritableComparator.get(getMapOutputKeyClass().asSubclass(WritableComparable.class), this);
}
```

`getOutputKeyComparator` 中如果用户设置了 `KEY_COMPARATOR`，就用用户自定义的 Key 比较器，如果用户没有定义Key比较器，就使用 map 端的key比较器进行分组。

### `runNewReducer` 进入reduce任务

```java
@SuppressWarnings("unchecked")
private <INKEY,INVALUE,OUTKEY,OUTVALUE>
void runNewReducer(JobConf job,
                  final TaskUmbilicalProtocol umbilical,
                  final TaskReporter reporter,
                  RawKeyValueIterator rIter,
                  RawComparator<INKEY> comparator,
                  Class<INKEY> keyClass,
                  Class<INVALUE> valueClass
                  ) throws IOException,InterruptedException, 
                          ClassNotFoundException {
  ...  
  // make a task context so we can get the classes
  org.apache.hadoop.mapreduce.TaskAttemptContext taskContext =
    new org.apache.hadoop.mapreduce.task.TaskAttemptContextImpl(job,
        getTaskID(), reporter);
  // make a reducer
  org.apache.hadoop.mapreduce.Reducer<INKEY,INVALUE,OUTKEY,OUTVALUE> reducer =
    (org.apache.hadoop.mapreduce.Reducer<INKEY,INVALUE,OUTKEY,OUTVALUE>)
      ReflectionUtils.newInstance(taskContext.getReducerClass(), job);
  org.apache.hadoop.mapreduce.RecordWriter<OUTKEY,OUTVALUE> trackedRW = 
    new NewTrackingRecordWriter<OUTKEY, OUTVALUE>(this, taskContext);
  job.setBoolean("mapred.skip.on", isSkipping());
  job.setBoolean(JobContext.SKIP_RECORDS, isSkipping());
  org.apache.hadoop.mapreduce.Reducer.Context 
        reducerContext = createReduceContext(reducer, job, getTaskID(),
                                              rIter, reduceInputKeyCounter, 
                                              reduceInputValueCounter, 
                                              trackedRW,
                                              committer,
                                              reporter, comparator, keyClass,
                                              valueClass);
    ...                                      
}
```

在 `runNewReducer` 中，先获取上下文、反射获取 Reducer 对象，最后创建 `reducerContext` 上下文，这是关键，下面看一下创建 `reducerContext` 的 `createReduceContext()` 是怎么实现的：

```java
@SuppressWarnings("unchecked")
protected static <INKEY,INVALUE,OUTKEY,OUTVALUE> 
org.apache.hadoop.mapreduce.Reducer<INKEY,INVALUE,OUTKEY,OUTVALUE>.Context
createReduceContext(org.apache.hadoop.mapreduce.Reducer
                      <INKEY,INVALUE,OUTKEY,OUTVALUE> reducer,
                    Configuration job,
                    org.apache.hadoop.mapreduce.TaskAttemptID taskId, 
                    RawKeyValueIterator rIter,
                    org.apache.hadoop.mapreduce.Counter inputKeyCounter,
                    org.apache.hadoop.mapreduce.Counter inputValueCounter,
                    org.apache.hadoop.mapreduce.RecordWriter<OUTKEY,OUTVALUE> output, 
                    org.apache.hadoop.mapreduce.OutputCommitter committer,
                    org.apache.hadoop.mapreduce.StatusReporter reporter,
                    RawComparator<INKEY> comparator,
                    Class<INKEY> keyClass, Class<INVALUE> valueClass
) throws IOException, InterruptedException {
  org.apache.hadoop.mapreduce.ReduceContext<INKEY, INVALUE, OUTKEY, OUTVALUE> 
  reduceContext = 
    new ReduceContextImpl<INKEY, INVALUE, OUTKEY, OUTVALUE>(job, taskId, 
                                                            rIter, 
                                                            inputKeyCounter, 
                                                            inputValueCounter, 
                                                            output, 
                                                            committer, 
                                                            reporter, 
                                                            comparator, 
                                                            keyClass, 
                                                            valueClass);

  org.apache.hadoop.mapreduce.Reducer<INKEY,INVALUE,OUTKEY,OUTVALUE>.Context 
      reducerContext = 
        new WrappedReducer<INKEY, INVALUE, OUTKEY, OUTVALUE>().getReducerContext(
            reduceContext);

  return reducerContext;
}
```

这里创建了 ReduceContextImpl 类的对象 `reduceContext`，然后由 `reduceContext` 创建出 `reducerContext` 返回。ReduceContextImpl 具体实现如下：

```java
public ReduceContextImpl(Configuration conf, TaskAttemptID taskid,
                          RawKeyValueIterator input, 
                          Counter inputKeyCounter,
                          Counter inputValueCounter,
                          RecordWriter<KEYOUT,VALUEOUT> output,
                          OutputCommitter committer,
                          StatusReporter reporter,
                          RawComparator<KEYIN> comparator,
                          Class<KEYIN> keyClass,
                          Class<VALUEIN> valueClass
                        ) throws InterruptedException, IOException{
  super(conf, taskid, output, committer, reporter);
  this.input = input;
  this.inputKeyCounter = inputKeyCounter;
  this.inputValueCounter = inputValueCounter;
  this.comparator = comparator;
  this.serializationFactory = new SerializationFactory(conf);
  this.keyDeserializer = serializationFactory.getDeserializer(keyClass);
  this.keyDeserializer.open(buffer);
  this.valueDeserializer = serializationFactory.getDeserializer(valueClass);
  this.valueDeserializer.open(buffer);
  hasMore = input.next();
  this.keyClass = keyClass;
  this.valueClass = valueClass;
  this.conf = conf;
  this.taskid = taskid;
}
```

这里就是根据参数调用构造函数创建出一个 ReduceContextImpl 对象，需要特别注意的是这里的 input，其实就是 Reducer 中调用的 `context.nextKey()` 时的输入迭代器。

## Reduce中的 `nextKey getCurrentKey getValues`

ReduceContextImpl 中还有 `nextKey` 、 `getCurrentKey`、`getValues` 方法的具体实现。

### nextKey

以nextKey为例：

```java
/** Start processing next unique key. */
public boolean nextKey() throws IOException,InterruptedException {
  while (hasMore && nextKeyIsSame) {
    nextKeyValue();
  }
  if (hasMore) {
    if (inputKeyCounter != null) {
      inputKeyCounter.increment(1);
    }
    return nextKeyValue();
  } else {
    return false;
  }
}
```

先判断是否还有输入数据，如果有数据而且下一个数据的key跟当前的key相同，就直接调用 `nextKeyValue` 寻找下一个数据，如果有数据但是跟当前的key不相同，就将记录key数量的计数器加1，然后再寻找下一个数据。

看一下 `nextKeyValue` 的具体实现：

```java
/**
  * Advance to the next key/value pair.
  */
@Override
public boolean nextKeyValue() throws IOException, InterruptedException {
  if (!hasMore) {
    key = null;
    value = null;
    return false;
  }
  firstValue = !nextKeyIsSame;
  DataInputBuffer nextKey = input.getKey();
  currentRawKey.set(nextKey.getData(), nextKey.getPosition(), 
                    nextKey.getLength() - nextKey.getPosition());
  buffer.reset(currentRawKey.getBytes(), 0, currentRawKey.getLength());
  key = keyDeserializer.deserialize(key);
  DataInputBuffer nextVal = input.getValue();
  buffer.reset(nextVal.getData(), nextVal.getPosition(), nextVal.getLength()
      - nextVal.getPosition());
  value = valueDeserializer.deserialize(value);

  currentKeyLength = nextKey.getLength() - nextKey.getPosition();
  currentValueLength = nextVal.getLength() - nextVal.getPosition();

  if (isMarked) {
    backupStore.write(nextKey, nextVal);
  }
  //判断是否还有数据
  hasMore = input.next();
  if (hasMore) {
    nextKey = input.getKey();
    nextKeyIsSame = comparator.compare(currentRawKey.getBytes(), 0, 
                                    currentRawKey.getLength(),
                                    nextKey.getData(),
                                    nextKey.getPosition(),
                                    nextKey.getLength() - nextKey.getPosition()
                                        ) == 0;
  } else {
    nextKeyIsSame = false;
  }
  inputValueCounter.increment(1);
  return true;
}
```

其实 `nextKeyValue` 的主要功能就是寻找下一个数据

nextKeyValue 做的事情有：

1. 如果hasMore为假表示没有数据了，就清空 key 和 value 的值，直接返回false；如果hasMore为真就继续往下走。

2. 读取数据，先进性反序列化，对key和value赋值。

3. 判断是否还有数据，如果有就比较当前 key 和下一条数据中的 key 是否相等，如果相等，nextKeyIsSame为true，如果不相等，nextKeyIsSame为false。

4. 最后没调用一个nextKeyValue， `inputValueCounter` 计数器就加1.

### getCurrentKey

getCurrentKey 直接返回key就行了

```java
public KEYIN getCurrentKey() {
  return key;
}
```


### getValues

```java
private ValueIterable iterable = new ValueIterable();
...
public 
Iterable<VALUEIN> getValues() throws IOException, InterruptedException {
  return iterable;
}
```

这里直接返回了一个 `iterable` 内部类 ValueIterable.

```java
protected class ValueIterable implements Iterable<VALUEIN> {
  private ValueIterator iterator = new ValueIterator();
  @Override
  public Iterator<VALUEIN> iterator() {
    return iterator;
  } 
}
```

而 ValueIterable 中有创建了一个 ValueIterator 的内部类， ValueIterator实现如下：

```java
protected class ValueIterator implements ReduceContext.ValueIterator<VALUEIN> {

  private boolean inReset = false;
  private boolean clearMarkFlag = false;

  @Override
  public boolean hasNext() {
    try {
      if (inReset && backupStore.hasNext()) {
        return true;
      } 
    } catch (Exception e) {
      e.printStackTrace();
      throw new RuntimeException("hasNext failed", e);
    }
    return firstValue || nextKeyIsSame;
  }

  @Override
  public VALUEIN next() {
    if (inReset) {
      try {
        if (backupStore.hasNext()) {
          backupStore.next();
          DataInputBuffer next = backupStore.nextValue();
          buffer.reset(next.getData(), next.getPosition(), next.getLength()
              - next.getPosition());
          value = valueDeserializer.deserialize(value);
          return value;
        } else {
          inReset = false;
          backupStore.exitResetMode();
          if (clearMarkFlag) {
            clearMarkFlag = false;
            isMarked = false;
          }
        }
      } catch (IOException e) {
        e.printStackTrace();
        throw new RuntimeException("next value iterator failed", e);
      }
    } 

    // if this is the first record, we don't need to advance
    if (firstValue) {
      firstValue = false;
      return value;
    }
    // if this isn't the first record and the next key is different, they
    // can't advance it here.
    if (!nextKeyIsSame) {
      throw new NoSuchElementException("iterate past last value");
    }
    // otherwise, go to the next key/value pair
    try {
      nextKeyValue();
      return value;
    } catch (IOException ie) {
      throw new RuntimeException("next value iterator failed", ie);
    } catch (InterruptedException ie) {
      // this is bad, but we can't modify the exception list of java.util
      throw new RuntimeException("next value iterator interrupted", ie);        
    }
  }

  @Override
  public void remove() {
    throw new UnsupportedOperationException("remove not implemented");
  }
  ...
}

```

这里面比较重要的时 `next` 和 `hasNext` 方法， `hasNext` 方法通过 `firstValue || nextKeyIsSame` 知道接下来还有没有数据可以使用，这里使用 `nextKeyIsSame` 作为条件是因为Reduce处理数据迭代器是针对相同 key 的数据放入同一个迭代器中，所以在实现时就采用 `nextKeyIsSame` 来判断下一个是否还有相同 key 值的数据可供读取。

ValueIterator 中的 `next` 方法最终调用的还是 ReduceContextImpl 中的 `nextKeyValue` 方法，因为最终读取数据还是要使用 ReduceContextImpl 中的 input，只不过用 `nextKeyIsSame` 来判断要不要放到同一个迭代器中给用户使用而已。

## 总结
 
Reduce端通过迭代器一次读取两条数据，第一条用于读取数据，第二条用于判断第二条数据是否交由同一个Reduce进行处理。所以即使数据量在Reduce端再大，也不会导致内存不足的情况发生，因为我们一次只需要能够存放两条数据的剩余内存即可，通过反复迭代，达到处理所有数据的效果。

[1]: https://jiaoqiyuan.cn/2018/12/24/MapReduce%E7%9A%84%E5%8E%9F%E7%90%86/
[2]: https://jiaoqiyuan.cn/2019/01/19/MapReduceSourceCode-JobAndSplite/
[3]: https://jiaoqiyuan.cn/2019/01/20/MapReduceSourceCode-MapTask/
[4]: https://jiaoqiyuan.cn/tags/MapReduce/