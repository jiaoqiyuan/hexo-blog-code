---
title: Java语言如何实现线程间通信
date: 2019-02-18 14:12:06
tags: [翻译, java]
categories: 翻译
thumbnail: https://fossbytes.com/wp-content/uploads/2018/09/java-11.png
photos: https://fossbytes.com/wp-content/uploads/2018/09/java-11.png
---

> 信息来源：[阮一峰每周分享](http://www.ruanyifeng.com/blog/2019/02/weekly-issue-43.html)

> 文章来源：[How To Implement Inter-thread Communication In Java](https://www.tutorialdocs.com/article/java-inter-thread-communication.html)

---------

Though normally each child thread just need to complete its own task, sometimes we may want multiple threads to work together to fulfil a task, which involves the inter-thread communication.

尽管通常情况下子线程只需要完成自己的本职工作就可以了，但是有时我们可能希望多个线程一起工作来完成一个任务，这里面就牵扯到了线程间通信。


The methods and classes covered in this article are: `thread.join()`, `object.wait()`, `object.notify()`, `CountdownLatch`, `CyclicBarrier`, `FutureTask`, `Callable`, etc.

这篇文章中涉及到的方法和类有：`thread.join()`, `object.wait()`, `object.notify()`, `CountdownLatch`, `CyclicBarrier`, `FutureTask`, `Callable`,等

[Here](https://github.com/wingjay/HelloJava/blob/master/multi-thread/src/ForArticle.java) is the code covered in this article

这是本片文章[代码连接](https://github.com/wingjay/HelloJava/blob/master/multi-thread/src/ForArticle.java)。

I'll use several examples to explain how to implement inter-thread communication in Java.

我将使用一些例子来解释Java中的线程间通信是怎么实现的。

>- How to make two threads execute in sequence?
>- How to make two threads intersect orderly in a specified manner?
>- There are four threads: A, B, C, and D (D won't be executed until A, B and C all have finished executing, and A, B, and C are to be executed synchronously.).
>- Three athletes prepare themselves apart, and then they start to run at the same time after each of them is ready.
>- After the child thread completes a task, it returns the result to the main thread.

>- 怎样让两个线程按顺序执行？
>- 怎样让两个线程以指定的方式有序交替执行？
>- 有四个线程：A, B, C, D（D线程在A，B，C线程之前完之前不会执行，并且A、B、C线程会被同步执行）。
>- 三名运动员分开站立，然后在所有人都准备好后同时开始跑。
>- 子线程完成一个任务后，返回执行结果给主线程。

## How To Make Two Threads Execute In Sequence? 怎样让两个线程顺序执行？


Suppose there are two threads: thread A and thread B. Both of the two threads can print three numbers (1-3) in sequence. Let's take a look at the code:

假设有两个线程：thread A 和 therad B 。两个线程都可以顺序打印三个1-3的数据。我们看一下代码是什么样的：

```java
private static void demo1() {
    Thread A = new Thread(new Runnable() {
        @Override
        public void run() {
            printNumber("A");
        }
    });
    Thread B = new Thread(new Runnable() {
        @Override
        public void run() {
            printNumber("B");
        }
    });
    A.start();
    B.start();
}
```

The implementation for `printNumber(String)` is as follows, which is used to print the three numbers of 1, 2 and 3 in sequence:

`printNumber(String)`的实现如下所示，该函数用于顺序打印1,2,3这个三个数字：

```java
private static void printNumber(String threadName) {
    int i=0;
    while (i++ < 3) {
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(threadName + "print:" + i);
    }
}
```

And the result we get is:

```java
B print: 1
A print: 1
B print: 2
A print: 2
B print: 3
A print: 3
```

You can see that A and B print numbers at the same time.

你可以看到 A 和 B 会同时打印数字。

So, what if we want B to start printing after A has printed over? We can use the `thread.join()` method, and the code is as follows:

那么，如果我们想要 B 在 A 打印结束后再开始打印的话呢？我们可以使用`thread.join()`这个方法，代码实现如下：

```java
private static void demo2() {
    Thread A = new Thread(new Runnable() {
        @Override
        public void run() {
            printNumber("A");
        }
    });
    Thread B = new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("B starts waiting for A");
            try {
                A.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            printNumber("B");
        }
    });
    B.start();
    A.start();
}
```

Now the result obtained is:

现在得到的结果是：

```
B starts waiting for A
A print: 1
A print: 2
A print: 3
 
B print: 1
B print: 2
B print: 3
```

So we can see that the `A.join()` method will make B wait until A finishes printing.

由此我们知道 `A.join()` 方法会使 B 等待，直到 A 打印完毕。

## How To Make Two Threads Intersect Orderly In a Specified Manner? 怎样让两个线程以指定的方式有序交替执行？

So what if now we want B to start printing 1, 2, 3 just after A has printed 1, and then A continues printing 2, 3? Obviously, we need more fine-grained locks to take the control of the order of execution.

那么要是我们想要 B 在 A 刚完成打印1的时候开始打印1,2,3，然后 A 再继续打印2,3的话该怎么办呢？显然，我们需要更细粒度的锁来控制执行的顺序。

Here, we can take the advantage of the `object.wait()` and `object.notify()` methods. The code is as below:

这里，我们可以利用 `object.wait()` 和 `object.notify()` 这两个方法。代码如下：

```java
/**
 * A 1, B 1, B 2, B 3, A 2, A 3
 */
private static void demo3() {
    Object lock = new Object();
    Thread A = new Thread(new Runnable() {
        @Override
        public void run() {
            synchronized (lock) {
                System.out.println("A 1");
                try {
                    lock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("A 2");
                System.out.println("A 3");
            }
        }
    });
    Thread B = new Thread(new Runnable() {
        @Override
        public void run() {
            synchronized (lock) {
                System.out.println("B 1");
                System.out.println("B 2");
                System.out.println("B 3");
                lock.notify();
            }
        }
    });
    A.start();
    B.start();
}
```

The results are as follows:

结果如下：

```
A 1
A waiting…
 
B 1
B 2
B 3
A 2
A 3
```

That's what we want.

这正是我们想要的

> **What happens? 发生了什么？**

>1. First we create an object lock shared by A and B: `lock = new Object()`;
>2. When A gets the lock, it prints 1 first, and then it calls the `lock.wait()` method which will make it go into the wait state, and hands over control of the lock then;
>3. B won't be executed until A calls the `lock.wait()` method to release control and B gets the lock;
>4. B prints 1, 2, 3 after getting the lock, and then calls the `lock.notify()` method to wake up A which is waiting;
>5. A continues printing the remaining 2, 3 after it wakes up.

>1. 首先我们创建了一个被 A 和 B 共享的对象锁（object lock）：`lock = new Object()`；
>2. 当 A 获得锁时，会先打印出1，然后会调用到 `lock.wait()` 方法，该方法使 A 线程进入等待状态，然后交出锁的控制权；
>3. B 在 A 调用到 `lock.wait()` 方法交出锁的控制权之前不会执行，在 A 交出锁的控制权之后 B 会获得这个锁；
>4. B 获取到锁后打印1,2,3，然后调用到 `lock.notify()` 方法来唤醒处于等待状态的 A；
>5. A 被唤醒后继续打印剩余的2,3两个数字。


I add the log to the above code to make it easier to understand.

我给上面的代码增加了日志信息以帮助容易理解一些。

```java
private static void demo3() {
    Object lock = new Object();
    Thread A = new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("INFO: A is waiting for the lock");
            synchronized (lock) {
                System.out.println("INFO: A got the lock");
                System.out.println("A 1");
                try {
                    System.out.println("INFO: A is ready to enter the wait state, giving up control of the lock");
                    lock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("INFO: B wakes up A, and A regains the lock");
                System.out.println("A 2");
                System.out.println("A 3");
            }
        }
    });
    Thread B = new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("INFO: B is waiting for the lock");
            synchronized (lock) {
                System.out.println("INFO: B got the lock");
                System.out.println("B 1");
                System.out.println("B 2");
                System.out.println("B 3");
                System.out.println("INFO: B ends printing, and calling the notify method");
                lock.notify();
            }
        }
    });
    A.start();
    B.start();
```

The result is as follows:

```
INFO: A is waiting for the lock
INFO: A got the lock
A 1
INFO: A is ready to enter the wait state, giving up control of the lock
INFO: B is waiting for the lock
INFO: B got the lock
B 1
B 2
B 3
INFO: B ends printing, and calling the notify method
INFO: B wakes up A, and A regains the lock
A 2
A 3
```

## D Is Executed After A, B and C All Have Finished Executing Synchronously D在A B C 都同步执行完后运行

The method `thread.join()` introduced earlier allows one thread to continue executing after waiting for another thread to finish running. But if we join A, B, and C orderly into the D thread, it will make A, B, and C execute in turn, while we want them three to run synchronously.

前面介绍的 `thread.join()` 方法允许一个线程在等待另一个线程执行完之后再继续执行。但是如果我们依次将 A B C join 到线程 D，这就会试 A B C 三个线程轮流执行，但是我们想要的是让他们同步执行。

The goal we want to achieve is: The three threads A, B, and C can start to run at the same time, and each will notify D after finishing running independently; D won't start to run until A, B, and C all finish running. So we use `CountdownLatch` to implement this type of communication. Its basic usage is:

我们想要达到的目的是： A B C 这三个线程可以同时开始执行，而且每个线程执行完后都会唤醒 D； D 只有在 A B C 都执行完毕后才会开始执行。因此我们可以使用 `CountdownLatch` 来实现这种类型的通信，它的基本用法是：

1. Create a counter, and set an initial value, `CountdownLatch countDownLatch = new CountDownLatch(3)`;

    创建一个计数器，赋上初值，`CountdownLatch countDownLatch = new CountDownLatch(3)`;

2. Call the `countDownLatch.await()` method in the waiting thread and go into the wait state until the count value becomes 0;

    在等待线程中调用 `countDownLatch.await()` 方法，在值变为0后进入等待状态；

3. Call the `countDownLatch.countDown()` method in other threads, and the method will reduce the count value by one;

    在其他线程中调用 `countDownLatch.countDown()` 方法，这个方法会将 `countDownLatch` 的值减一。

4. When the `countDown()` method in other threads turns the count value to 0, the `countDownLatch.await()` method in the waiting thread will exit immediately and continue to execute the following code.

    当其他线程中的 `countDown()` 方法将 `countDownLatch` 的值最终减为了0，等待线程中的 `countDownLatch.await()` 方法就会立即退出，然后该线程继续执行接下来的代码。

The implementation code is as follows:

代码实现如下：

```java
private static void runDAfterABC() {
    int worker = 3;
    CountDownLatch countDownLatch = new CountDownLatch(worker);
    new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("D is waiting for other three threads");
            try {
                countDownLatch.await();
                System.out.println("All done, D starts working");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }).start();
    for (char threadName='A'; threadName <= 'C'; threadName++) {
        final String tN = String.valueOf(threadName);
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(tN + "is working");
                try {
                    Thread.sleep(100);
                } catch (Exception e) {
                    e.printStackTrace();
                }
                System.out.println(tN + "finished");
                countDownLatch.countDown();
            }
        }).start();
    }
}
```

The result is as follows:

结果如下：

```
D is waiting for other three threads
A is working
B is working
C is working
 
A finished
C finished
B finished
All done, D starts working
```

In fact, `CountDownLatch` itself is a countdown counter, and we set the initial count value to 3. When D runs, it first call the `countDownLatch.await()` method to check whether the counter value is 0, and it will stay in wait state if the value is not 0. A, B, and C each will use the `countDownLatch.countDown()` method to decrement the countdown counter by 1 after they finish running separately. And when all of them three finish running, the counter will be reduced to 0; then, the `await()` method of D will be triggered to end A, B, and C, and D will start to go on executing.

事实上， `CountDownLatch` 它自身是一个倒数计数器，而且我们将它的初始值设置为了3。当线程 D 开始执行时，会首先调用到 `countDownLatch.await()` 方法去检查计数器的值是否为0，如果计数器值不是0当前线程就进入等待状态。 A B C 中的每个线程分别在执行完后使用 `countDownLatch.countDown()` 方法来将倒数计数器的值减1。当它们三个都执行完毕时，计数器的值就被减为0；然后线程 D 的 `await()` 方法将被触发来结束 A B C，D会继续执行下面的代码。


Therefore, `CountDownLatch` is suitable for the situation where one thread needs to wait for multiple threads.

因此， `CountDownLatch` 非常适合一个线程需要等待多个线程的场景。

## 3 Runners Preparing To Run 三个跑步者准备起跑

Three runners prepare themselves apart, and then they start to run at the same time after each of them is ready.

三个跑步者分开站立，都准备好后同时开始起跑。

This time, each of the three threads A, B, and C need to prepare separately, and then they start to run simultaneously after all of them three are ready. How should we achieve that?

这次， A B C 这三个线程每一个都需要独立准备，然后它们都准备好后同时开始执行。我们该如何实现这个目标呢？

The `CountDownLatch` above can be used to count down, but when the count is completed, only one of the threads' `await()` method will get a response, so multiple threads cannot be triggered at the same time.

上面提到的 `CountDownLatch` 可用于倒计时，但是当计数器结束时，只有一个线程的 `await()` 方法会得到响应，所以这种情况下多线程不能同时被触发执行。

In order to achieve the effect of threads' waiting for each other, we can use the `CyclicBarrier` data structure, and its basic usage is:

为了达到线程相互等待的结果，我们可以使用 `CyclicBarrier` 这个数据结构，它的基本用法是：

1. First create a public object `CyclicBarrier`, and set the number of threads waiting at the same time, `CyclicBarrier cyclicBarrier = new CyclicBarrier(3)`;

    首先创建一个公有对象 `CyclicBarrier`， 同时设置等待线程的数量： `CyclicBarrier cyclicBarrier = new CyclicBarrier(3)`；

2. These threads start to prepare simultaneously. After they are ready, they need to wait for others to finish preparing, so call the `cyclicBarrier.await()` method to wait for others;

    这些线程同时开始准备，每个线程准备好后，它们需要等待其他人完成准备，通过调用 `cyclicBarrier.await()` 方法来等待其他线程；

3. When the specified threads that need to wait at the same time all call the `cyclicBarrier.await()` method, which means that these threads are ready, then these threads will start to continue executing simultaneously.

    当需要等待的指定线程都调用了 `cyclicBarrier.await()` 方法时，意味着这些线程都应完成了准备工作，然后这些线程就可以同时开始执行了。

The implementation code is as follows. Imagine that there are three runners who need to start to run simultaneously, so they need to wait for others until all of them are ready.

代码实现如下。想象一下有三个运动员需要同时起步，因此它们需要等待其他人都准备好后再起步。

```java
private static void runABCWhenAllReady() {
    int runner = 3;
    CyclicBarrier cyclicBarrier = new CyclicBarrier(runner);
    final Random random = new Random();
    for (char runnerName='A'; runnerName <= 'C'; runnerName++) {
        final String rN = String.valueOf(runnerName);
        new Thread(new Runnable() {
            @Override
            public void run() {
                long prepareTime = random.nextInt(10000) + 100;
                System.out.println(rN + "is preparing for time:" + prepareTime);
                try {
                    Thread.sleep(prepareTime);
                } catch (Exception e) {
                    e.printStackTrace();
                }
                try {
                    System.out.println(rN + "is prepared, waiting for others");
                    cyclicBarrier.await(); // The current runner is ready, waiting for others to be ready
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
                System.out.println(rN + "starts running"); // All the runners are ready to start running together
            }
        }).start();
    }
}
```

The result is as follows:

运行结果如下：

```
A is preparing for time: 4131
B is preparing for time: 6349
C is preparing for time: 8206
 
A is prepared, waiting for others
 
B is prepared, waiting for others
 
C is prepared, waiting for others
 
C starts running
A starts running
B starts running
```

## Child Thread Returns The Result To The Main Thread 子进程返回结果给主进程

In actual development, often we need to create child threads to do some time-consuming tasks, and then pass the execution results back to the main thread. So how to implement this in Java?

在实际开发中，我们经常需要创建子线程来完成一些耗时的操作，然后将执行结果传递会主线程。那么在Java中怎么实现呢？

So generally, when creating the thread, we'll pass the Runnable object to Thread for execution. The definition for Runnable is as follows:

一般情况下，在创建线程时，我们会将 Runnable 对象传递给 Thread 来执行。 Runnable 的定义如下：

```java
public interface Runnable {
    public abstract void run();
}
```

You can see that `run()` method does not return any results after execution. Then what if you want to return the results? Here you can use another similar interface class `Callable`:

可以看到 `run()` 方法在执行完毕后没有返回任何结果。那么要是你想返回执行结果的话该怎么做呢？你可以使用另一个叫做 `Callable` 的类似接口。

```java
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

It can be seen that the biggest difference for `Callable` is that it returns the generics.

可以看到 `Callable` 最大的不同之处是它能够返回泛型。

So the next question is, how to pass the results of the child thread back? Java has a class, `FutureTask`, which can work with `Callable`, but note that the `get` method which is used to get the result will block the main thread.

所以下一个问题是，怎样将子线程的结果传递回来？Java 有一个可以和 `Callable` 一起使用的类叫 `FutureTask`，但是注意用于获取结果的 `get` 方法会阻塞主线程。

For example, we want the child thread to calculate the sum from 1 to 100 and return the result to the main thread.

举个栗子，我们想要子线程计算1到100的和然后将结果返回给主线程。

```java
private static void doTaskWithResultInWorker() {
    Callable<Integer> callable = new Callable<Integer>() {
        @Override
        public Integer call() throws Exception {
            System.out.println("Task starts");
            Thread.sleep(1000);
            int result = 0;
            for (int i=0; i<=100; i++) {
                result += i;
            }
            System.out.println("Task finished and return result");
            return result;
        }
    };
    FutureTask<Integer> futureTask = new FutureTask<>(callable);
    new Thread(futureTask).start();
    try {
        System.out.println("Before futureTask.get()");
        System.out.println("Result:" + futureTask.get());
        System.out.println("After futureTask.get()");
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
}
```

The result is as follows:

运行结果如下：

```
Before futureTask.get()
 
Task starts
Task finished and return result
 
Result: 5050
After futureTask.get()
```

It can be seen that it blocks the main thread when the main thread calls the `futureTask.get()` method; then the `Callable` starts to execute internally and returns the result of the operation; and then the `futureTask.get()` gets the result and the main thread resumes running.

可以看到主线程调用 `futureTask.get()` 时被阻塞了；然后 `Callable` 开始在内部执行并返回操作结果；然后 `futureTask.get()` 获得返回的结果，主线程继续执行。

Here we can learn that the `FutureTask` and `Callable` can get the result of the child thread directly in the main thread, but they will block the main thread. Of course, if you don't want to block the main thread, you can consider using `ExecutorService` to put the `FutureTask` into the thread pool to manage execution.

这里我们可以了解到 `FutureTask` 和 `Callable` 可以直接在主线程中获得子线程的结果，但是它们会阻塞主线程。当然，如果你不想要阻塞主线程，你可以使用 `ExecutorService` 来将 `FutureTask` 放入线程池来管理执行。

## Summary 总结

Multithreading is a common feature for modern languages, and inter-thread communication, thread synchronization, and thread safety are very important topics.

多线程是当今编程语言的常见功能，线程间通信、线程间同步以及线程安全也是很重要的主题。