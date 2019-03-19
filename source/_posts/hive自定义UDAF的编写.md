---
title: hive自定义UDAF的编写
date: 2019-03-18 12:35:25
tags:
categories: 大数据
photos: https://github.com/jiaoqiyuan/pics/raw/master/hive_udaf/hive_udaf_title.png
---

> 最近学习中遇到一个编写HIVE UDAF函数的问题，最后没弄明白怎么写的，后来看到别人写的UDAF函数后深受启发，所以在这里记录一下UDAF的编写过程。

<!--more-->

## UDAF简介

UDAF是Hive中用户自定义的聚合函数，Hive内置的UDAF函数有sum()和count()，不过这在实际开发中根本不够用，用户有时候希望能够自定义一些聚合函数来注册使用，Hive也考虑到了这个问题，因此提供了GenericUDAF供用户继承和改写，主要有两个抽象类：

```
org.apache.hadoop.hive.ql.udf.generic.AbstractGenericUDAFResolver
org.apache.hadoop.hive.ql.udf.generic.GenericUDAFEvaluator
```

## 抽象类简介

Hive最终也是将SQL转化成MapReduce程序执行的，只不过Hive隐藏了转化过程，只提供SQL接口给用户，在编写自定义聚合函数时就需要用户熟悉MapReduce的具体过程，并控制不同阶段程序的执行逻辑。UDAF函数先读取数据，也就是mapper过程，汇聚后(combine)，最后将数据聚合(Reduce)阶段处理，然后将最终结果输出。

### AbstractGenericUDAFResolver

这个抽象类的作用主要是指定sql传入的参数要交给那个Evaluator进行处理，需要覆盖实现`public GenericUDAFEvaluator getEvaluator(TypeInfo[] info)`这个方法。

### GenericUDAFEvaluator

UDAF的主要逻辑在`GenericUDAFEvaluator`这里，其中关键是理解Evaluator是怎么处理数据的，这就牵扯到了objectInspector和Model类。

objectInspector是用于解耦数据使用与数据格式的，使得数据流在输入输入端能够切换成不同的格式，不同的处理阶段使用不同的数据格式。

Model类是GenericUDAFEvaluator中的一个内部类，定义了不同的处理阶段：

```java
public static enum Mode {
/**
    * PARTIAL1: from original data to partial aggregation data: iterate() and
    * terminatePartial() will be called.
    * PARTIAL1阶段:这是MapReduce的map阶段，从原始数据到部分聚合数据，这个阶段中iterate()
    * 方法和terminatePartial()方法将会调用。
    */
PARTIAL1,
    /**
    * PARTIAL2: from partial aggregation data to partial aggregation data:
    * merge() and terminatePartial() will be called.
    * PARTIAL2阶段:这是MapReduce的combin阶段，从部分聚合数据到部分聚合数据，也就是数据的合并。
    * 将会调用merge()方法和terminatePartial()方法。
    */
PARTIAL2,
    /**
    * FINAL: from partial aggregation to full aggregation: merge() and
    * terminate() will be called.
    * FINAL阶段：这是MapRduce的reduce阶段，从部分聚合数据到完全聚合
    * merge()方法和terminate()方法将会被调用
    */
FINAL,
    /**
    * COMPLETE: from original data directly to full aggregation: iterate() and
    * terminate() will be called.
    * COMPLETE阶段：这各阶段表示没有reduce，map端的数据处理后聚合，然后直接输出，map端输入什么类型的数据就输出什么类型的数据。
    * 将会调用iterate()方法和terminate()方法。
    */
COMPLETE
};
```

上面的四个过程其实跟MapReduce的数据处理过程息息相关，从数据到达map阶段开始，就是进入了PARTIAL1阶段，这个时候会调用自己实现的接口`iterate()`方法和`terminatePartial()`方法，`iterate()`方法是对每条数据进行相应处理，terminatePartial()方法是返回map和combine处理完成后的中间结果，我们在PARTIAL2阶段也会调用`terminatePartial()`方法。这个阶段用户可以根子需要将输入数据转换成想要的类型输出。

每个map处理完成后根据需要可能会进入combine阶段，也就是PATTIAL2阶段，这个阶段中主要是进行数据到合并，也就是将部分聚合数据合并，组成一个更大的部分聚合数据，然后交给reduce阶段。这个阶段调用的用户自己实现的函数是`merge()`和`terminatePartial()`。这个阶段输入什么类型的数据合并之后就输出什么类型的数据。

map和combine都完成后，就进入到了reduce阶段，也就是FINAL阶段，这个阶段主要是对部分聚合数据进行全聚合处理，然后根据业务需要，把想要的数据提取出来，以要想的格式返回出来就可以了，这个阶段调用的函数是`merge()`和`terminate()`方法。这个阶段的输入数据就是PARTIAL2阶段的输出数据，用户可以根据业务需要指定输出类型。

还有一种情况是不需要reduce的操作，比如最终想要的数据类型跟输入的数据类型一致，这时只需要进行map操作就可以了，这时就只调用`iterate()`和`terminate()`就可以了。

### GenericUDAFEvaluator方法

上面说到的`iterate()`, `terminatePartial()`, `merge()`, `terminate()`这些方法都是在GenericUDAFEvaluator中定义的抽象方法，另外还有两个抽象方法也要实现：`getNewAggregationBuffer`和`reset`，分别用于创建保存数据聚合结果的实例和重置聚合结果。

```java
// 确定各个阶段输入输出参数的数据格式ObjectInspectors
public  ObjectInspector init(Mode m, ObjectInspector[] parameters) throws HiveException;
 
// 保存数据聚集结果的类
abstract AggregationBuffer getNewAggregationBuffer() throws HiveException;
 
// 重置聚集结果
public void reset(AggregationBuffer agg) throws HiveException;
 
// map阶段，迭代处理输入sql传过来的列数据
public void iterate(AggregationBuffer agg, Object[] parameters) throws HiveException;
 
// map与combiner结束返回结果，得到部分数据聚集结果
public Object terminatePartial(AggregationBuffer agg) throws HiveException;
 
// combiner合并map返回的结果，还有reducer合并mapper或combiner返回的结果。
public void merge(AggregationBuffer agg, Object partial) throws HiveException;
 
// reducer阶段，输出最终结果
public Object terminate(AggregationBuffer agg) throws HiveException;
```

引用一下其他人的图，下面是Model中各个阶段调用函数的情况：

![1][1]

这是各个阶段中参数的类型及传递情况，理解好数据是怎么流动的很关键，基本上知道怎么流动的就知道怎么处理了：

![2][2]


## UDAF需求

这里的需求是编写一个Hive自定义聚合函数，用来计算用户数据中当前时刻(time_tag)前30分钟内用户的行为(active_name)。

### 数据库中数据格式

Hive中的数据存放在一个叫做bigdata.weblog的表中，表格式如下：

```
hive> desc weblog;
OK
time_tag                bigint                  时间
active_name             string                  事件名称
device_id               string                  设备id
session_id              string                  会话id
user_id                 string                  用户id
ip                      string                  ip地址
address                 map<string,string>      地址
req_url                 string                  http请求地址
action_path             array<string>           访问路径
product_id              string                  商品id
order_id                string                  订单id
day                     string                  日期

# Partition Information
# col_name              data_type               comment

day                     string                  日期
Time taken: 0.589 seconds, Fetched: 17 row(s)
```

里面的数据内容大概是这样的：

```
hive> select * from weblog limit 5;
OK
1527604188966   pageview        879367ed5ea2473d9121779bc48f4765        0000f7714f3c48f4838513a65ad7383b        4921528165744221        111.37.0.130    {"city":"济南","province":"山东","country":"中国"}    http://www.bigdataclass.com/category     ["http://www.bigdataclass.com/category"]        NULL    NULL    2018-05-29
1527604410990   pageview        879367ed5ea2473d9121779bc48f4765        0000f7714f3c48f4838513a65ad7383b        4921528165744221        111.37.0.130    {"city":"济南","province":"山东","country":"中国"}    http://www.bigdataclass.com      ["http://www.bigdataclass.com/category","http://www.bigdataclass.com"]  NULL    NULL    2018-05-29
1527604638195   pageview        879367ed5ea2473d9121779bc48f4765        0000f7714f3c48f4838513a65ad7383b        4921528165744221        111.37.0.130    {"city":"济南","province":"山东","country":"中国"}    http://www.bigdataclass.com      ["http://www.bigdataclass.com/category","http://www.bigdataclass.com","http://www.bigdataclass.com"]    NULL    NULL    2018-05-29
1527604714462   pageview        879367ed5ea2473d9121779bc48f4765        0000f7714f3c48f4838513a65ad7383b        4921528165744221        111.37.0.130    {"city":"济南","province":"山东","country":"中国"}    http://www.bigdataclass.com/my/4921528165744221  ["http://www.bigdataclass.com/category","http://www.bigdataclass.com","http://www.bigdataclass.com","http://www.bigdataclass.com/my/4921528165744221"]  NULL  NULL     2018-05-29
1527604879475   pageview        879367ed5ea2473d9121779bc48f4765        0000f7714f3c48f4838513a65ad7383b        4921528165744221        111.37.0.130    {"city":"济南","province":"山东","country":"中国"}    http://www.bigdataclass.com/category     ["http://www.bigdataclass.com/category","http://www.bigdataclass.com","http://www.bigdataclass.com","http://www.bigdataclass.com/my/4921528165744221","http://www.bigdataclass.com/category"]  NULL    NULL    2018-05-29
Time taken: 0.636 seconds, Fetched: 5 row(s)
```

我们只需要根据time_tag，将取出的active_name根据最后一条time_tag的大小往前推30分钟，将30分钟内这个区间段内的active_name放到一个列表中返回即可。


### 代码编写

```java
package com.bigdata.etl.udaf;

import com.google.common.collect.Lists;
import com.google.common.collect.Maps;
import org.apache.hadoop.hive.ql.exec.UDFArgumentTypeException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.parse.SemanticException;
import org.apache.hadoop.hive.ql.udf.generic.AbstractGenericUDAFResolver;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDAFEvaluator;
import org.apache.hadoop.hive.serde2.objectinspector.*;
import org.apache.hadoop.hive.serde2.typeinfo.TypeInfo;
import org.apache.hadoop.hive.serde2.typeinfo.TypeInfoUtils;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;

import java.util.*;

public class UDAFCollectAction extends AbstractGenericUDAFResolver {

    @Override
    public GenericUDAFEvaluator getEvaluator(TypeInfo[] parameters) throws SemanticException {
        //判断参数个数
        if (parameters.length != 2) {
            throw new UDFArgumentTypeException(parameters.length - 1, "Two argument is excepted.");
        }

        ObjectInspector oiKey = TypeInfoUtils.getStandardJavaObjectInspectorFromTypeInfo(parameters[0]);
        ObjectInspector oiValue = TypeInfoUtils.getStandardJavaObjectInspectorFromTypeInfo(parameters[1]);
        if (oiKey.getCategory() != ObjectInspector.Category.PRIMITIVE || oiValue.getCategory() != ObjectInspector.Category.PRIMITIVE) {
            throw new UDFArgumentTypeException(0, "Argument must be PRIMITIVE");
        }

        //判断参数类型
        PrimitiveObjectInspector inputKeyOI = (PrimitiveObjectInspector) oiKey;
        PrimitiveObjectInspector inputValueOI = (PrimitiveObjectInspector) oiValue;
        if (inputKeyOI.getPrimitiveCategory() != PrimitiveObjectInspector.PrimitiveCategory.LONG) {
            throw new UDFArgumentTypeException(0, "Argument must be String, but"
                    + inputKeyOI.getPrimitiveCategory().name()
                    + " was passed.");
        }
        if (inputValueOI.getPrimitiveCategory() != PrimitiveObjectInspector.PrimitiveCategory.STRING) {
            throw new UDFArgumentTypeException(0, "Argument must be Timstamp, but "
                    + inputValueOI.getPrimitiveCategory().name()
                    + " was passed.");
        }

        return new AllActionsOfThisPeople30MinBefore();
    }

    public static class AllActionsOfThisPeople30MinBefore extends GenericUDAFEvaluator {
        private PrimitiveObjectInspector inputKeyOI;
        private PrimitiveObjectInspector inputValueOI;
        private StandardMapObjectInspector internalMergeOI;

        public ObjectInspector init(Mode m, ObjectInspector[] parameters) throws HiveException{
            super.init(m, parameters);
            if (m == Mode.PARTIAL1) {
                inputKeyOI = (PrimitiveObjectInspector) parameters[0];
                inputValueOI = (PrimitiveObjectInspector) parameters[1];
                return ObjectInspectorFactory.getStandardMapObjectInspector(
                        ObjectInspectorUtils.getStandardObjectInspector(inputKeyOI),
                        ObjectInspectorUtils.getStandardObjectInspector(inputValueOI));
            } else if (m == Mode.PARTIAL2) {
                internalMergeOI = (StandardMapObjectInspector) parameters[0];
                return ObjectInspectorUtils.getStandardObjectInspector(internalMergeOI);

            } else if (m == Mode.FINAL){
                internalMergeOI = (StandardMapObjectInspector) parameters[0];
                inputKeyOI = (PrimitiveObjectInspector) internalMergeOI.getMapKeyObjectInspector();
                inputValueOI = (PrimitiveObjectInspector) internalMergeOI.getMapValueObjectInspector();
                return ObjectInspectorFactory.getStandardListObjectInspector(inputValueOI);
            } else {
                inputKeyOI = (PrimitiveObjectInspector) parameters[0];
                inputValueOI = (PrimitiveObjectInspector) parameters[1];
                return ObjectInspectorFactory.getStandardListObjectInspector(inputValueOI);
            }
        }

        static class MKArrayAggregationBuffer implements AggregationBuffer {
            Map<Object, Object> container = Maps.newHashMap();
        }

        public AggregationBuffer getNewAggregationBuffer() throws HiveException {
            MKArrayAggregationBuffer ret = new MKArrayAggregationBuffer();
            return ret;
        }

        public void reset(AggregationBuffer agg) throws HiveException {
            ((MKArrayAggregationBuffer) agg).container.clear();

        }

        public void iterate(AggregationBuffer agg, Object[] parameters) throws HiveException {
            if (parameters == null || parameters.length != 2) {
                return;
            }
            if (parameters[0] != null && parameters[1] != null) {
                MKArrayAggregationBuffer myagg = (MKArrayAggregationBuffer) agg;
                Object key = ObjectInspectorUtils.copyToStandardObject(parameters[0], inputKeyOI);
                Object value = ObjectInspectorUtils.copyToStandardObject(parameters[1], inputValueOI);
                myagg.container.put(key, value);
            }
        }

        public Object terminatePartial(AggregationBuffer agg) throws HiveException {
            MKArrayAggregationBuffer myagg = (MKArrayAggregationBuffer) agg;
            Map<Object, Object> ret = Maps.newHashMap(myagg.container);
            return ret;
        }

        public void merge(AggregationBuffer agg, Object partial) throws HiveException {
            assert (partial != null);
            MKArrayAggregationBuffer my_agg = (MKArrayAggregationBuffer) agg;
            Map<Object, Object> partialResult = (Map<Object, Object>) internalMergeOI.getMap(partial);
            for (Map.Entry<Object, Object> entry : partialResult.entrySet()) {
                Object kCopy = ObjectInspectorUtils.copyToStandardObject(entry.getKey(), this.inputKeyOI);
                Object vCopy = ObjectInspectorUtils.copyToStandardObject(entry.getValue(), this.inputValueOI);
                my_agg.container.put(kCopy, vCopy);
            }
        }

        public Object terminate(AggregationBuffer agg) throws HiveException {
            MKArrayAggregationBuffer my_agg = (MKArrayAggregationBuffer) agg;
            Map map = new HashMap(my_agg.container.size());
            map.putAll(my_agg.container);

            List<Map.Entry<LongWritable, Text>> listData = Lists.newArrayList(map.entrySet());
            Collections.sort(listData, new Comparator<Map.Entry<LongWritable, Text>>() {
                public int compare(Map.Entry<LongWritable, Text> o1, Map.Entry<LongWritable, Text> o2) {
                    return (o1.getKey().compareTo(o2.getKey()));
                }
            });

            List<Text> result = Lists.newArrayList();
            LongWritable currTime = listData.get(listData.size() - 1).getKey();
            for (Map.Entry<LongWritable, Text> entry : listData) {
                Long timeInterval = (currTime.get() - entry.getKey().get()) / 60000;
                if (timeInterval <= 30) {
                    result.add(entry.getValue());
                }
            }

            return result;
        }
    }
}
```

代码编写过程中遇到小坑，是自己没有认真思考造成的，我在最初传入参数时是先传入active_name后传入time_tag的，而且是以active_name作为`terminate()`中Map的key来处理的数据，结果就是最终返回的active_name只有三种类型：`pageview`, `order`, `pay`。这是必然的，因为在`merge()`中往myagg.container中put数据时，active_name重复的话会覆盖之前相同的key下的value，所以不管怎样最终只有三个数据输出啦，(⊙﹏⊙)b。


接下来启动hive，将jar包添加进去，创建临时函数，使用临时函数获取数据：

```sql
add jar /mnt/home/1015146591/jars/etl-1.0-jar-with-dependencies.jar;

create temporary function collect_list as 'com.bigdata.etl.udaf.UDAFCollectAction';


select collect_list(time_tag, active_name) from bigdata.weblog where user_id='8511528166018276';
```

最终结果：

```
hive> create temporary function collect_list as 'com.bigdata.etl.udaf.UDAFCollectAction';
OK
Time taken: 1.056 seconds
hive> select collect_list(time_tag, active_name) from bigdata.weblog where user_id='8511528166018276';
Query ID = 1015146591_20190319101703_d282f2d1-e412-4c9c-8ff4-5a392cbcc8f5
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Starting Job = job_1550731843098_4984, Tracking URL = http://bigdata0.novalocal:8088/proxy/application_1550731843098_4984/
Kill Command = /home/hadoop/hadoop-current/bin/hadoop job  -kill job_1550731843098_4984
Hadoop job information for Stage-1: number of mappers: 6; number of reducers: 1
2019-03-19 10:17:12,655 Stage-1 map = 0%,  reduce = 0%
2019-03-19 10:17:20,012 Stage-1 map = 17%,  reduce = 0%, Cumulative CPU 5.7 sec
2019-03-19 10:17:23,118 Stage-1 map = 19%,  reduce = 0%, Cumulative CPU 46.44 sec
2019-03-19 10:17:24,159 Stage-1 map = 20%,  reduce = 0%, Cumulative CPU 55.98 sec
2019-03-19 10:17:26,234 Stage-1 map = 41%,  reduce = 0%, Cumulative CPU 69.93 sec
2019-03-19 10:17:27,278 Stage-1 map = 46%,  reduce = 0%, Cumulative CPU 73.23 sec
2019-03-19 10:17:29,349 Stage-1 map = 47%,  reduce = 0%, Cumulative CPU 86.07 sec
2019-03-19 10:17:30,395 Stage-1 map = 59%,  reduce = 6%, Cumulative CPU 90.79 sec
2019-03-19 10:17:31,428 Stage-1 map = 69%,  reduce = 6%, Cumulative CPU 93.28 sec
2019-03-19 10:17:32,461 Stage-1 map = 100%,  reduce = 6%, Cumulative CPU 103.23 sec
2019-03-19 10:17:33,493 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 105.66 sec
MapReduce Total cumulative CPU time: 1 minutes 45 seconds 660 msec
Ended Job = job_1550731843098_4984
MapReduce Jobs Launched: 
Stage-Stage-1: Map: 6  Reduce: 1   Cumulative CPU: 105.66 sec   HDFS Read: 1571426690 HDFS Write: 136 SUCCESS
Total MapReduce CPU Time Spent: 1 minutes 45 seconds 660 msec
OK
["pageview","pageview","pageview","pageview","pageview","pageview","pageview","pageview","pageview","pageview","pageview","pageview","pageview","pageview","order","pay"]
Time taken: 32.413 seconds, Fetched: 1 row(s)
```

## 总结

这是UDAF的典型使用方法，了解UDAF开发的关键是要理解MapReduce的数据处理过程以及hive的`GenericUDAFEvaluator`这个抽象类是怎么定义数据处理流程的，其中数据的流动是怎么在具体的方法中传递的是最关键的点，了解了Model中各个阶段与MapReduce的对应关系以及函数调用关系，就可以根据不同的也无需要编写任何你想要的UDAF函数了。

## 参考链接

1. [Hive UDAF开发详解][3]
2. [再谈GenericUDAF（以collect_set源码分析）][4]

[1]: https://github.com/jiaoqiyuan/pics/raw/master/hive_udaf/model_evaluator.png
[2]: https://github.com/jiaoqiyuan/pics/raw/master/hive_udaf/model_evalueator1.png
[3]: https://blog.csdn.net/kent7306/article/details/50110067
[4]: https://yugouai.iteye.com/blog/1876140
