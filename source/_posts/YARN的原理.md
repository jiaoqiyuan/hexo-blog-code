---
title: YARN的原理
date: 2018-12-27 07:32:12
tags:
categories: 大数据
thumbnail: https://2s7gjr373w3x22jf92z99mgm5w-wpengine.netdna-ssl.com/wp-content/uploads/2017/05/hadoop-yarn.png
photos: https://2s7gjr373w3x22jf92z99mgm5w-wpengine.netdna-ssl.com/wp-content/uploads/2017/05/hadoop-yarn.png
---


YARN是Hadoop生态中重要的资源调度和管理框架，理解好YARN的原理和执行流程对集群的资源分配和管理有很大帮助。

<!--more-->

## YARN是什么

>YARN是Hadoop的集群通用管理系统，在Hadoop2.x的时候被引入，最初是为了改善MapReduce的实现，但它具有足够的通用性，可以支持MapReduce之外的其他分布式计算模式。 ---《Hadoop权威指南》

YARN提供了使用系统资源的API，但是这些API很少给用户直接使用，一般都是将这些API封装好后，交给更上层的应用开发的用户使用，封装后的API隐藏了YARN资源管理的细节，用户只需要知道怎么使用即可，毕竟越靠底层越复杂，有时候将底层复杂操作封装一个通用框架给用户使用可以避免重复一些困难而且复杂的工作。

![YARN应用](https://dev.tencent.com/u/jiaoqiyuan/p/hexo-blog-code/git/raw/master/source/img/YARN%E5%BA%94%E7%94%A8.png)

## 为什么会有YARN

YARN的出现是了为解决Hadoop1.x中MapReduce1.x版本资源管理和调度机制效率低下和存在管理任务数量的瓶颈问题。而且YARN在设计时就考虑了通用性，也就是YARN可以兼容MapReduce之外的其他分布式计算模式。

### MapReduce1架构

在MapReduce1.x的版本中，MapReduce的架构是下面这样的：

![MapReduce1.x](https://dev.tencent.com/u/jiaoqiyuan/p/hexo-blog-code/git/raw/master/source/img/MapReduce1.x.png)

在这里面，JobTracker负责资源管理，一个客户端向Hadoop集群发出一个MapReduce任务执行请求时，这个请求就是由JobTracker管理的，JobTracker和namenode联合将任务交给离要处理的数据尽可能近的位置。

JobTracker会将Map和Reduce任务放到一个或多个TaskTracker上执行，然后TaskTracker再开启具体执行Map或Reduce任务的Task，Task是实际执行计算任务的基本单元。

当Task的任务执行完成后，会通知TaskTracker，然后TaskTracker会告知JobTracker，JobTracker在确认所有TaskTracker都执行完毕后告知用户作业已执行完成。

### MapReduce1存在的一些问题

1. JobTracker是MapReduce的管理核心，存在单点故障的问题！

2. 集群中节点数量比较多时（一般认为超过4000个后），由于JobTracker要完成的任务很多，当提交的MR任务太多的时候，会造成极大的内存开销。

3. 给TaskTracker分配任务时以Task数目作为标准，没有考虑cpu/内存占用情况，如果两个大内存的Task被分到了一个TaskTracker，容易出现内存不够用（OOM）的情况。

4. TaskTracker为Map和Reduce分配资源的方式不合理，强制将资源划分为Map和Reduce两部分，如果系统只有Map任务或者Reduce任务，会造成资源浪费。

5. 不具有通用性，当然这是针对YARN来说的，或许也不能算个缺点吧。

### YARN出现

YARN在针对上面MR1.x存在的问题，YARN从设计上解决了这些问题。

## YARN架构

Hadoop官网的YARN结构图如下：

![YARN](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/yarn_architecture.gif)

相比与MapReduce1.x，YARN将JobTracker的功能进行了拆分，将资源管理和任务调度拆分成了两个进程分别负责相应的功能：全局资源管理器（ResourceManager和NodeManager）和作业管理器（ApplicationMaster）。ResourceManager和NodeManager负责资源的管理和任务的调度，ApplicationMaster负责任务的执行。

- ResourceManager：负责全局资源管理和任务调度。

- NodeManager：负责单个节点的资源管理和监控。

- ApplicationMaster：单个作业的资源管理和任务监控。

- Container：资源申请的单位和任务运行的容器。

上图结合HDFS存储后：

![YARN框架](https://dev.tencent.com/u/jiaoqiyuan/p/hexo-blog-code/git/raw/master/source/img/YARN%E6%A1%86%E6%9E%B6.png)

这里将ResourceManager和NodeManager分别简称为RM和NM，后面都以简称代替全称。

YARN分层的本质是RM和NM，RM将各种资源（cpu、内存、带宽等）精心安排给NM，RM还与AppMaster一起分配资源，与NM一起启动和监视他们在Container中启动的应用程序，AppMaster承担了一部分之前TaskTracker的角色，RM承担了JobTracker的角色。

### RM

RM的主要功能有两个：接收Job任务请求(ApplicationManager)和资源调度(Scheduler)。

AppManager接收客户端的Job请求后，会为应用找到一个合适的Container运行AppMaster，然后会一直监控AppMaster的运行状态，当AppMaster返回失败时RM就重新启动一个新的AppMaster重新运行该任务。另外客户端也会定期查询AppMaster的进度和状态，一旦发现其失败，就向RM询问新的AppMaster在哪儿，然后定期向新的AppMaster询问程序运行状态信息。

Scheduler主要负责资源的调度任务，它不参与任何和具体应用程序相关的工作，只负责资源分配，具体可以参考本文下面介绍的[YARN资源调度](#anchor)里面的内容。

RM会使用checkpoint机制，定时将其状态备份到磁盘上，一旦RM出现运行错误，就可从磁盘回复备份的状态重新启动运行。RM一般会结合zookeeper实现状态同步。

### NM

NM用于管理具体某个节点的资源，每个节点上都运行有NM服务，从监控节点的资源和跟踪节点的健康情况到具体该节点每个容器的终生管理，NM都会负责到底。MapReduce1.x通过Slot模式管理Map和Reduce任务的执行，而NM是管理一个抽象的容器，容器代表可供一个特定应用程序使用的节点资源。

NM会定时想RM汇报当前节点的资源信息以及该节点上各个Container的运行状态，另外NM也会接收来自AppMaster的各种Container操作请求操作。

NM会定期想RM发送心跳信息，RM一旦超时没有收到心跳信息，RM就会将改NM移除，此时运行在该NM上的任务和AppMaster都会在其他的NM上进行恢复。如果一个NM失败的次数太多，AppMaster会将其加入黑名单（RM没有黑名单），从此不会再将任务分配到该节点上。

### AppMaster

每个交给YARN执行的任务都会启动一个AppMaster，这个AppMaster就负责来管理当前这个任务的执行。具体来说就是AppMaster会向RM申请资源并接收来自NM的命令启动容器来运行作业，并且监视容器的执行和资源（CPU、内存等资源）的使用情况。

AppMaster会定期向RM发送心跳信息，一旦AppMaster运行失败，RM就会开启新的AppMaster。

### Container

Container是YARN中资源的抽象称呼，它包含有当前节点的多种资源信息，当AppMaster向RM申请资源时，RM为AppMaster返回的资源是以Container表示的。AppMaster会根据应用需要启动一个或多个Container运行任务。

Container对应于MapReduce1.x中的slot概念，不同的是Container是一个动态分配的资源单位，根据应用需求动态生成。

从YARN接受一个客户请求开始，RM启动一个AppMaster，然后AppMaster协商每个节点上可用的容器资源执行应用程序，AppMaster一直监视所有容器执行完毕，然后注销这些容器，向RM汇报执行完毕的状态。

## MapReduce1.x与YARN对比

![MapReduce_YARN](https://dev.tencent.com/u/jiaoqiyuan/p/hexo-blog-code/git/raw/master/source/img/MapReduce_YARN.png)

YARN将资源调度和管理抽离出来，单独管理，将任务执行放到了一个或多个容器中，由AppMaster统一管理，AppMaster负责向资源管理者申请和释放资源。这样就将MapReduce1.x中JobTracker复杂的设计分离成多个服务，更加高效地使用系统资源。

客户端不变，其调用的大部分API接口也是向下兼容的，这也是充分考虑到开发迁移的便利性，但是JobTracker和TaskTracker不见了，取而代之的是RM，AppMaster，NM。相比与MapReduce1.x，Job中Task的启动、停止、重启的操作不见了，这是因为使用AppMaster进行Task的管理啦，这些擦做都有AppMaster自动进行完成。

YARN相对来说的优点总结下来有：

1. 设计减小了JobTracker（现在的RM）的资源消耗，并且监控每一个Job子任务状态的程序的方式更加安全优雅。

2. YARN中AppMaster是可变更的一部分，用户可以针对不同编程模型编写自己的AppMaster,让更多种变成模型能够跑在Hadoop集群中。可参考mapred-site.xml的配置。

3. 资源表示现在以内存为单位，取代了之前的剩余slot数目，这样设计更加合理。

4. MapReduce1.x下JobTracker监控Job下的tasks的运行状况需要消耗很多资源，现在这部分工作交给AppMaster做了，然后由RM中的ApplicationMasters监控AppMaster的运行状况，如果出问题了就在其他机器上重启AppMaster。

5. YARN借鉴了Mesos将Container作为资源隔离的单位，将资源表示成内存，避免了之前使用map slot/reduce slot造成的集群资源闲置的尴尬情况。

## YARN执行流程

YARN执行流程如图所示：

![YARN执行流程](https://dev.tencent.com/u/jiaoqiyuan/p/hexo-blog-code/git/raw/master/source/img/YARN%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B.png)

1. 客户端启动一个MapReduce程序。

2. 该Job程序向RM提交应用并请求一个AppMaster。

3. Job程序将使用到的资源信息上传到Cash云缓存中，方便后续各个任务节点使用。

4. Job程序接发送申请AppMaster的请求后收到RM返回信息，然后提交作业。

5. a RM找到一个可以运行Container的NM; b 并在这个Container中启动AppMaster实例。

6. AppMaster做一些初始化之类的工作。

7. AppMaster从云缓存中拿到自己需要的资源。

8. AppMaster向RM注册和申请资源，注册之后客户端就可以查询RM以获得AppMaster的详细信息了，以后客户端就直接和AppMaster进行交互。

9. 在向RM申请完资源后，根据RM的返回结果AppMaster向特定的NM发送container-lanch-specification信息来启动Container，container-lanch-specification中包含了能够让container和AppMaster交流的所有信息。

10. NM启动Container，并把Container的运行进度、状态等信息通过application-specific协议发送给AppMaster。

11. 应用代码在启动的Container中运行。

12. Task运行期间会将状态信息反馈给AppMaster。

一旦应用程序执行完毕，AppMaster想RM取消注册然后关闭，运行AppMaster和Task任务的Container会将资源重回交给NM供其他应用使用。

## <span id = "anchor">YARN资源调度</span>

集群资源一般是有限的，对于YARN发出的资源请求需要根据实际资源剩余情况和任务需要的资源情况进行资源的调度，这是一个不断调优的过程，没有“最好”调度策略一说。

YARN有三种调度器：FIFO调度器，容量调度器和公平调度器。

FIFO调度器是将应用放在一个队列中，按照先后顺序依次进行任务资源的分配。优点是简单移动，不需要任何配置，缺点是不适合共享集群。因为大的应用会占用集群中的所有资源，所以每个应用都必须等待直到轮到自己执行。

![YARN资源调度](https://dev.tencent.com/u/jiaoqiyuan/p/hexo-blog-code/git/raw/6fd5621eba03e1164fb16f7923b2a20ce8c302ec/source/img/YARN%E8%B5%84%E6%BA%90%E8%B0%83%E5%BA%A6.png)

共享集群更适合使用容量调用器和公平调度器，这两种调度器都允许在大任务运行的同时运行后来的小任务，并且及时返回结果。

使用容量调度器时，一个独立的专门队列会保证小作业一提交就可以被启动，由于需要单独预留一个队列资源，这种策略的整体集群利用率是被降低了的，与FIFO相比，大作业的执行时间要更长。

公平调度器不需要提前预留一定的资源，因为所有运行的作业之间是动态调整资源的。第一个大作业启动时，它独自使用所有资源，第二个小作业启动时，大作业所占用的资源会强制被分配给小作业一半的资源，这样每个作业都能公平共享资源。

调度的具体流程可以参考Hadoop权威指南的4.3节和这个链接（[Yarn 调度器Scheduler详解](https://blog.csdn.net/suifeng3051/article/details/49508261)）。

## 总结

本文简述了YARN的由来，YARN与MapReduce1.x的对比，从设计到具体运行流程，最后简述了一下YARN的资源调度，这里没有展开资源调度的内容，有兴趣的朋友可以参考一下官方文档和Google。

写这篇文章的目的是加深自己对YARN的理解，同时记录下来希望能够方便自己后期查阅以及帮助他人了解YARN，我知道我记录的这些深度远远不够，而且很多是参考别人的文章，我没读过YARN的源码，对于它的设计思想也只是通过书本和博客了解到的，希望在实际工作中遇到相关内容的时候能够对YARN有更深刻的理解！

----

>参考书籍: 《Hadoop权威指南》

>参考链接1: [Hadoop YARN](https://www.w3cschool.cn/hadoop/rh161hda.html)   

>参考链接2: [Hadoop YARN的发展史与详细解析](https://www.csdn.net/article/2013-12-18/2817842-bd-hadoopyarn) 

>参考链接3: [Apache Hadoop YARN](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html)

>参考链接4: [Hadoop Yarn 框架原理及运作机制](https://blog.csdn.net/liuwenbo0920/article/details/43304243)

>参考链接5: [Hadoop Yarn详解](https://blog.csdn.net/suifeng3051/article/details/49486927)

>参考链接6: [Hadoop新MapReduce框架Yarn详解](https://blog.csdn.net/he582754810/article/details/54142363)

>参考链接7: [初步掌握Yarn的架构及原理](http://www.cnblogs.com/codeOfLife/p/5492740.html)
