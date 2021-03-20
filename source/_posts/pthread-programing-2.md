---
title: pthread-programing-2
date: 2021-03-08 17:16:55
tags:
categories: linux
thumbnail: https://gitee.com/jiaoqiyuan/pics/raw/master/blog/pthread-programing/pthread-programing-2.png
photos: https://gitee.com/jiaoqiyuan/pics/raw/master/blog/pthread-programing/pthread-programing-2.png
---

接着上次的 [pthread-programing-1](https://jiaoqiyuan.github.io/2021/01/16/pthread-programing-1/#more)，把 pthread 的其他内容补充完整。

<!--more-->

## 线程管理

### 线程堆栈管理

接口程序有：

```C
pthread_attr_getstacksize(attr, stacksize)
pthread_attr_setstacksize(attr, stacksize)
pthread_attr_getstackaddr(attr, stackaddr)
pthread_attr_setstackaddr(attr, stackaddr)
```

- POSIX 标准不规定线程堆栈的大小，具体大小取决于不同的实现方式。

- 很容易做到超过默认堆栈限制，通常的结果是：程序终止和/或数据损坏

- 安全和可移植的程序不依赖于默认的堆栈限制，而是使用 pthread_attr_setstacksize 接口为每个线程显式分配足够的堆栈。

- 在必须将线程的堆栈放置在内存的某些特定区域的环境中时，应用程序可以使用 pthread_attr_getstackaddr 和 pthread_attr_setstackaddr 接口实现。

### 一些实际的例子

- 默认线程堆栈大小变化很大。可获得的最大值也相差很大，并且可能取决于每个节点的线程数。

- 过去和现在的硬件架构都显示了默认线程堆栈大小的波动很大。

|Node Architecture|	#CPUs| Memory (GB)| Default Size (bytes)|
|:--:|:--:|:--:|:--:|
|Intel Xeon E5-2670	| 16	| 32	| 2,097,152
|Intel Xeon 5660	| 12	| 24	| 2,097,152
|AMD Opteron	    | 8	    | 16	| 2,097,152
|Intel IA64	        | 4     | 8     |33,554,432
|Intel IA32	        | 2	    | 4	    | 2,097,152
|IBM Power5	        | 8	    | 32	| 196,608
|IBM Power4	        | 8	    | 16	| 196,608
|IBM Power3	        | 16    | 16	| 98,304


本示例演示如何查询和设置线程的堆栈大小。 

```c
 #include <pthread.h>
 #include <stdio.h>
 #define NTHREADS 4
 #define N 1000
 #define MEGEXTRA 1000000
 
 pthread_attr_t attr;
 
 void *dowork(void *threadid)
 {
    double A[N][N];
    int i,j;
    long tid;
    size_t mystacksize;

    tid = (long)threadid;
    pthread_attr_getstacksize (&attr, &mystacksize);
    printf("Thread %ld: stack size = %li bytes \n", tid, mystacksize);
    for (i=0; i<N; i++)
      for (j=0; j<N; j++)
       A[i][j] = ((i*j)/3.452) + (N-i);
    pthread_exit(NULL);
 }
 
 int main(int argc, char *argv[])
 {
    pthread_t threads[NTHREADS];
    size_t stacksize;
    int rc;
    long t;
 
    pthread_attr_init(&attr);
    pthread_attr_getstacksize (&attr, &stacksize);
    printf("Default stack size = %li\n", stacksize);
    stacksize = sizeof(double)*N*N+MEGEXTRA;
    printf("Amount of stack needed per thread = %li\n",stacksize);
    pthread_attr_setstacksize (&attr, stacksize);
    printf("Creating threads with stack size = %li bytes\n",stacksize);
    for(t=0; t<NTHREADS; t++){
       rc = pthread_create(&threads[t], &attr, dowork, (void *)t);
       if (rc){
          printf("ERROR; return code from pthread_create() is %d\n", rc);
          exit(-1);
       }
    }
    printf("Created %ld threads.\n", t);
    pthread_exit(NULL);
 }
```

### 其他各种接口

```c
pthread_self()
pthread_equal(thread1,thread2)
```

- pthread_self 返回线程自身ID。

- pthread_equal 比较两个线程 ID， 如果两个线程 ID 不同返回0，否则返回其他非零值。请注意，对于这两个例程，线程标识符对象都是不透明的，因此不容易检查。因为线程ID是不透明的对象，所以不应使用C语言等效运算符==来将两个线程ID相互比较，或将一个线程ID与另一个值进行比较

```c
pthread_once (once_control, init_routine)
```

- pthread_once在一个进程中仅执行一次init_routine。进程中的任何线程对该接口的第一次调用都会执行给定的 init_routine，而不带参数。任何后续调用均无效。

- init_routine 例程通常是一个初始化例程。

- Once_control参数是一个同步控制结构，需要在调用pthread_once之前进行初始化。例如：

    ```c
    pthread_once_t once_control = PTHREAD_ONCE_INIT;
    ```

## Mutex Variables 互斥变量

### 简介

- "Mutex" 是 "mutual exclusion" 的缩写，互斥变量是实现线程同步和在发生多次写操作时保护共享数据的主要方法之一。

- 互斥变量就像“锁”一样，保护对共享数据资源的访问。 Pthreads中使用的互斥锁的基本概念是，在任何给定时间，只有一个线程可以锁定（或拥有）互斥锁变量。因此，即使多个线程尝试锁定互斥锁，也只有一个线程会成功。在拥有线程解锁该互斥锁之前，没有其他线程可以拥有该互斥锁。线程必须“轮流”访问受保护的数据。

- 互斥体可用于防止“竞争”情况，涉及银行交易的竞争条件示例如下所示：

| Thread 1 | Thread2 | Balance |
|:-:|:-:|:-:|
| Read balance: $1000 |             | $1000 |
|  | Read balance: $1000            | $1000 |
|  | 	Deposit $200                | $1000 |
| 	Deposit $200 |                  | $1000 |
| 	Update balance $1000+$200 |     | $1200 |
| | 	Update balance $1000+$200   | $1200 |

- 在上面的示例中，当线程正在使用此共享数据资源时，应该使用互斥锁来锁定“平衡”。

- 通常，拥有互斥量的线程执行的操作是更新全局变量。这是一种安全的方法，可确保当多个线程更新同一变量时，最终值与只有一个线程执行更新时的最终值相同。被更新的变量属于“关键部分”。

使用互斥锁的典型顺序如下:

1. 创建并初始化互斥量变量

2. 多个线程试图锁定互斥锁

3. 只有一个线程成功锁定拥有互斥变量操作权

4. 拥有锁的线程执行一些操作

5. 拥有锁的线程解锁

6. 另一个线程获取互斥锁并重复该过程

7. 最终互斥体被销毁

- 当多个线程竞争一个互斥锁时，竞争失败者将在该调用时阻塞 - 通过 "trylock" 而不是 "lock" 调用可以进行无阻塞调用。

- 保护共享数据时，程序员有责任确保每个需要使用互斥锁的线程都这样做。例如，如果4个线程正在更新相同的数据，但是只有一个线程使用互斥锁，则数据仍可能被破坏。

### 创建和销毁互斥对象

```c

pthread_mutex_init(mutex,attr)
pthread_mutex_destroy(mutex)
pthread_mutexattr_init(attr)
pthread_mutexattr_destroy(attr)
```

- 用法：

- 互斥变量必须使用pthread_mutex_t类型声明，并且必须先初始化才能使用。有两种初始化互斥量变量的方法：

    1. 静态声明，例如：

        ```c
        pthread_mutex_t mymutex = PTHREAD_MUTEX_INITIALIZER;
        ```

    2. 使用 pthread_mutex_init() 动态生成，此方法允许设置互斥对象属性attr。

    互斥锁初始状态处于解锁状态。

- attr对象用于建立互斥对象的属性，并且必须使用pthread_mutexattr_t类型（如果使用）（可以将其指定为NULL以接受默认值）。Pthreads标准定义了三个可选的互斥属性：

    1. Procotol: 指定用于防止互斥量优先级倒置的协议。

    2. Prioceiling: 指定互斥锁的优先级上限。

    3. Process-shared: 指定互斥量的进程共享。

    请注意，并非所有实现都可以提供三个可选的互斥属性。

- pthread_mutexattr_init() 和 pthread_mutexattr_destroy() 例程分别用于创建和销毁互斥体属性对象。

- 应该使用 pthread_mutex_destroy() 释放不再需要的互斥对象。

### 锁定和解锁互斥锁

```c
pthread_mutex_lock (mutex)
pthread_mutex_trylock (mutex)
pthread_mutex_unlock (mutex)
```

- 线程使用 pthread_mutex_lock() 获取对指定互斥锁变量的锁定。如果互斥锁已经被另一个线程锁定，则此调用将阻塞调用线程，直到互斥锁被解锁为止。

- pthread_mutex_trylock() 将尝试锁定互斥锁。如果互斥锁已被锁定，则例程将立即返回“忙”错误代码。如在优先级倒置的情况下，此例程可能对防止死锁情况很有用。

- 如果由拥有线程调用，则 pthread_mutex_unlock() 将解锁互斥锁。如果其他线程要获取互斥量以使用受保护的数据，则在线程完成其对受保护数据的使用之后需要调用此例程。如果出现以下情况，将返回错误：

    1. 如果互斥锁已被解锁

    2. 如果互斥锁由另一个线程拥有

- 互斥锁没有什么 "magic" ……实际上，它们类似于参与线程之间的 "gentlemen's agreement"。由代码编写者来确保所有必要的线程都能正确地进行互斥锁和解锁调用。下面的情况演示了一个逻辑错误：

    ```
    Thread 1     Thread 2     Thread 3
    Lock         Lock         
    A = 2        A = A+1      A = A*B
    Unlock       Unlock  
    ```

### 例子程序

该示例程序说明了在执行点积的线程程序中互斥变量的使用。通过全局可访问的结构，主数据可用于所有线程。每个线程在数据的不同部分上工作。主线程等待所有线程完成计算，然后输出结果总和.

## 条件变量

- 条件变量为线程提供了另一种同步方式。互斥锁通过控制线程对数据的访问来实现同步时，条件变量允许线程根据数据的实际值进行同步。

- 没有条件变量，程序员将需要使线程连续轮询（可能在关键部分），以检查是否满足条件。 这可能会非常消耗资源，因为线程将在此活动中持续忙碌。条件变量是一种无需轮询即可实现相同目标的方法。

- 条件变量始终与互斥锁一起使用。

- 下面显示一种使用条件变量的典型序列：

    主线程：

    - 声明并初始化需要同步的全局数据/变量（例如“ count”）

    - 声明并初始化条件变量对象

    - 声明并初始化关联的互斥锁

    - 创建线程A和B进行工作

    线程 A：

    - 执行程序到一定的条件（例如“ count”必须达到指定值）后终止

    - 锁定关联的互斥锁并检查全局变量的值

    - 调用pthread_cond_wait() 来执行阻塞等待，以等待来自线程B的信号。请注意，对pthread_cond_wait() 的调用会自动原子地解锁相关的互斥变量，以便Thread-B可以使用它。

    - 收到信号后，被唤醒。互斥锁是自动和原子锁定的。

    - 显式解锁互斥锁

    - 继续执行程序

    线程 B ：

    - 执行线程程序

    - 锁定关联的互斥锁

    - 更改线程A正在等待的全局变量的值。

    - 检查全局Thread-A等待变量的值。如果满足所需条件，则向Thread-A发出信号。

    - 接触互斥锁

    - 继续执行程序

### 创建和销毁条件变量

```c
pthread_cond_init(condition,attr)
pthread_cond_destroy(condition)
pthread_condattr_init(attr)
pthread_condattr_destroy(attr)
```

- 用法：

    - 条件变量必须使用 pthread_cond_t 类型声明，并且必须先进行初始化，然后才能使用它们。有两种方法可以初始化条件变量：

        1. 静态声明：

            ```c
            pthread_cond_t myconvar = PTHREAD_COND_INITIALIZER;
            ```

        2. 使用 pthread_cond_init() 动态创建，通过条件参数将创建的条件变量的ID返回给调用线程。此方法允许设置条件变量对象属性attr。

    - 可选的 attr 对象用于设置条件变量属性。仅为条件变量定义了一个属性：进程共享，它允许条件变量被其他进程中的线程查看。属性对象（如果使用）必须为 pthread_condattr_t 类型（可以指定为NULL以接受默认值）。

        注意，并非所有实现都可以提供进程共享属性。

    - pthread_condattr_init() 和 pthread_condattr_destroy() 例程用于创建和销毁条件变量属性对象。

    - 应该使用pthread_cond_destroy() 释放不再需要的条件变量。

### 等待和发信号通知条件变量

```c
pthread_cond_wait(condition,mutex)
pthread_cond_signal(condition)
pthread_cond_broadcast(condition)
```

- pthread_cond_wait() 阻塞调用线程，等待指定条件的信号到来。该例程应在互斥锁被锁定时调用，并且它将在等待时自动释放互斥锁。接收到信号并唤醒线程后，互斥体将自动锁定以供线程使用。线程完成后，程序员负责解锁互斥锁。

    **建议：** 使用WHILE循环而不是IF语句（请参见下面的示例中的watch_count例程）来检查等待的条件，可以帮助解决一些潜在的问题，例如：

    1. 如果多个线程正在等待相同的唤醒信号，则它们将轮流获取互斥对象，然后它们中的任何一个都可以修改它们都等待的条件。

    1. 如果线程由于程序错误而收到错误的信号

    1. 允许Pthreads库在不违反标准的情况下向等待的线程发出虚假唤醒。

- pthread_cond_signal() 例程用于发信号（或唤醒）另一个正在等待条件变量的线程。它应该在互斥锁锁定后调用，并且必须解锁互斥锁才能完成pthread_cond_wait() 例程。

- 如果有多个线程处于阻塞等待状态，则应使用pthread_cond_broadcast() 例程而不是pthread_cond_signal() 。

- 在调用pthread_cond_wait() 之前先调用pthread_cond_signal() 是逻辑错误。

使用这些例程时，正确锁定和解锁相关的互斥变量至关重要。例如：

1. 在调用pthread_cond_wait() 之前未锁定互斥锁可能导致其未阻塞。

1. 调用pthread_cond_signal() 后未能解锁互斥锁可能无法完成匹配的pthread_cond_wait() 例程（它将保持阻塞状态）。

例子：

这个简单的示例代码演示了几个Pthread条件变量例程的用法。主例程创建三个线程。其中两个线程执行工作并更新“ count”变量。第三个线程等待，直到count变量达到指定值。

```c
 #include <pthread.h>
 #include <stdio.h>
 #include <stdlib.h>

 #define NUM_THREADS  3
 #define TCOUNT 10
 #define COUNT_LIMIT 12

 int     count = 0;
 int     thread_ids[3] = {0,1,2};
 pthread_mutex_t count_mutex;
 pthread_cond_t count_threshold_cv;

 void *inc_count(void *t) 
 {
   int i;
   long my_id = (long)t;

   for (i=0; i<TCOUNT; i++) {
     pthread_mutex_lock(&count_mutex);
     count++;

     /* 
     Check the value of count and signal waiting thread when condition is
     reached.  Note that this occurs while mutex is locked. 
     */
     if (count == COUNT_LIMIT) {
       pthread_cond_signal(&count_threshold_cv);
       printf("inc_count(): thread %ld, count = %d  Threshold reached.\n", 
              my_id, count);
       }
     printf("inc_count(): thread %ld, count = %d, unlocking mutex\n", 
	    my_id, count);
     pthread_mutex_unlock(&count_mutex);

     /* Do some "work" so threads can alternate on mutex lock */
     sleep(1);
     }
   pthread_exit(NULL);
 }

 void *watch_count(void *t) 
 {
   long my_id = (long)t;

   printf("Starting watch_count(): thread %ld\n", my_id);

   /*
   Lock mutex and wait for signal.  Note that the pthread_cond_wait 
   routine will automatically and atomically unlock mutex while it waits. 
   Also, note that if COUNT_LIMIT is reached before this routine is run by
   the waiting thread, the loop will be skipped to prevent pthread_cond_wait
   from never returning. 
   */
   pthread_mutex_lock(&count_mutex);
   while (count<COUNT_LIMIT) {
     pthread_cond_wait(&count_threshold_cv, &count_mutex);
     printf("watch_count(): thread %ld Condition signal received.\n", my_id);
     }
     count += 125;
     printf("watch_count(): thread %ld count now = %d.\n", my_id, count);
   pthread_mutex_unlock(&count_mutex);
   pthread_exit(NULL);
 }

 int main (int argc, char *argv[])
 {
   int i, rc;
   long t1=1, t2=2, t3=3;
   pthread_t threads[3];
   pthread_attr_t attr;

   /* Initialize mutex and condition variable objects */
   pthread_mutex_init(&count_mutex, NULL);
   pthread_cond_init (&count_threshold_cv, NULL);

   /* For portability, explicitly create threads in a joinable state */
   pthread_attr_init(&attr);
   pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE);
   pthread_create(&threads[0], &attr, watch_count, (void *)t1);
   pthread_create(&threads[1], &attr, inc_count, (void *)t2);
   pthread_create(&threads[2], &attr, inc_count, (void *)t3);

   /* Wait for all threads to complete */
   for (i=0; i<NUM_THREADS; i++) {
     pthread_join(threads[i], NULL);
   }
   printf ("Main(): Waited on %d  threads. Done.\n", NUM_THREADS);

   /* Clean up and exit */
   pthread_attr_destroy(&attr);
   pthread_mutex_destroy(&count_mutex);
   pthread_cond_destroy(&count_threshold_cv);
   pthread_exit(NULL);
 } 
```

参考链接：

[POSIX Threads Programming](https://computing.llnl.gov/tutorials/pthreads/)