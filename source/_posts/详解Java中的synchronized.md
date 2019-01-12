---
title: 详解Java中的synchronized
date: 2019-01-04 13:18:33
tags:
categories: java
thumbnail: https://fossbytes.com/wp-content/uploads/2018/09/java-11.png
photos: https://fossbytes.com/wp-content/uploads/2018/09/java-11.png
---

## Synchronized

> Synchronized methods enable a simple strategy for preventing thread interference and memory consistency errors: if an object is visible to more than one thread, all reads or writes to that object's variables are done through synchronized methods.  
------ JDK Document - [Synchronized Methods](https://docs.oracle.com/javase/tutorial/essential/concurrency/syncmeth.html#brandProdName)

> Synchronized方法支持使用一种简单的策略来防止线程间的干扰和内存一致性错误：如果一个对象对多个线程可见，则对该对象变量的所有读写都是通过通过Synchronized方法完成。

<!--more-->

## Synchronized的作用

上面的介绍一句话概括就是：

> 同一时刻保证只有一个线程执行该代码以达到保证并发安全的效果。

如果一段代码被Synchronized修饰了，那么这段代码就会以原子方式执行，多个线程在执行这段代码时不会相互干扰，因为他们不会同时执行这段代码，只要不同时执行就不会出现并发问题。那么线程之间怎么知道不同时执行这段代码呢？这里可以想象有一把锁，被Synchronized修饰的这段代码在被第一段代码执行的时候时会先加把锁，知道地一段代码执行结束后或者在一定的条件下再打开锁，在这把锁释放前，其他线程的代码执行到加Synchronized修饰的这段代码时，会检测到有锁的存在，所以就先不执行加锁的代码，而是等待锁的打开。等第一段代码执行完，打开锁后，其他线程的代码才能执行Synchronized修饰的代码段。

Synchronized是Java的**关键字**，被Java语言原生支持。是最基本的互斥同步手段，而且是并发变成的元老级角色，是并发变成必学的内容。


## 不使用并发手段会有什么后果

举个栗子：两个线程同时对变量a++，最后结果比预期的要小。

```java
public class DisappearedRequest1 implements Runnable{

    static DisappearedRequest1 instance = new DisappearedRequest1();

    static int i = 0;

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);

        t1.start();
        t2.start();
        //这里加入join的作用是让t1和t2先执行完再执行main函数接下来的代码
        t1.join();
        t2.join();

        System.out.println(i);
    }

    @Override
    public void run() {
        for (int j = 0; j < 100000; j++) {
            i++;
        }
    }
}
```

最后结果:
```
103066

Process finished with exit code 0
```

为什么会这样呢？原因就是：i++这个操作看上去是一步完成的，实际上不是的，它包含三个步骤：

1. 读取i的值
2. 将i加一
3. 将i的值写入到内存中

在这三步执行过程中t1、t2两个线程的操作可能会有干扰，比如t1刚读取了i的值并将其加1，t2就执行了步骤一的操作读取i的值，此时由于t1将i的值加一后没有写入到内存，所以t2此时读取的还是原来的i值，接下来也会进行加一操作，然后t1、t2都会进行写入内存的操作，这就会造成i少加一的情况。

这就是线程不安全的情况。

## Synchronized两种用法

### 对象锁

对象锁分两种：

- 同步代码块锁：自己指定锁对象。

- 方法锁：默认锁定对象为this当前实例对象。

#### 同步代码块

先上代码：

```java
package demo;

/**
 * 描述：对象锁，同步代码块
 */
public class SynchronizedObjectCodeBlock2 implements Runnable{
    static SynchronizedObjectCodeBlock2 instance = new SynchronizedObjectCodeBlock2();
    @Override
    public void run() {
//        synchronized (this) {
            System.out.println("对象锁的代码块形式：" + Thread.currentThread().getName());
            try {
                //休眠3s，使效果更明显
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "运行结束！");
//        }
    }
    public static void main(String[] args) {
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();
        t2.start();
        //这里不使用join，采用while循环判断的方式，功能跟join类似
        while(t1.isAlive() || t2.isAlive()) {
        }
        System.out.println("Finished!");
    }
}
```

不加synchronized修饰代码块的情况下，运行结果是这样的：

```
对象锁的代码块形式：Thread-0
对象锁的代码块形式：Thread-1
Thread-0运行结束！
Thread-1运行结束！
Finished!

Process finished with exit code 0
```

修改代码，加上synchronized后，代码如下：

```java
package demo;

/**
 * 描述：对象锁，同步代码块
 */
public class SynchronizedObjectCodeBlock2 implements Runnable{
    static SynchronizedObjectCodeBlock2 instance = new SynchronizedObjectCodeBlock2();
    @Override
    public void run() {
       synchronized (this) {
            System.out.println("对象锁的代码块形式：" + Thread.currentThread().getName());
            try {
                //休眠3s，使效果更明显
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "运行结束！");
       }
    }
    public static void main(String[] args) {
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();
        t2.start();
        //这里不使用join，采用while循环判断的方式，功能跟join类似
        while(t1.isAlive() || t2.isAlive()) {
        }
        System.out.println("Finished!");
    }
}
```
加入synchronized修饰代码块后运行结果如下：
```
对象锁的代码块形式：Thread-1
Thread-1运行结束！
对象锁的代码块形式：Thread-0
Thread-0运行结束！
Finished!

Process finished with exit code 0
```
这里可以看出使用synchronized和不使用的差别，使用了synchronized后线程是一个一个执行的，只有在当前线程执行完毕后，其他线程才能执行。线程执行顺序与系统调度有关。

这里锁对象使用的是this，如果一个代码文件中有多个需要同步的线程，而且他们之间不是完全互斥的，就是说一个线程执行的时候也可以有其他线程执行，也就是有多个线程对（或者叫线程组吧），那么此时锁对象就需要自己指定啦。可以自定义一个：

```java
Object lock = new Object();
```

这里的lock对象唯一的作用就是充当锁对象。

使用多把锁的情况：

```java
package demo;

/**
 * 描述：对象锁，同步代码块
 */
public class SynchronizedObjectCodeBlock2 implements Runnable{
    static SynchronizedObjectCodeBlock2 instance = new SynchronizedObjectCodeBlock2();
    Object lock1 = new Object();
    Object lock2 = new Object();
    @Override
    public void run() {
        synchronized (lock1) {
            System.out.println("我是lock1， " + Thread.currentThread().getName());
            try {
                //休眠3s，使效果更明显
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " lock1运行结束！");
        }
        synchronized (lock2) {
            System.out.println("我是lock2， " + Thread.currentThread().getName());
            try {
                //休眠3s，使效果更明显
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " lock2运行结束！");
        }
    }
    public static void main(String[] args) {
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();
        t2.start();
        while(t1.isAlive() || t2.isAlive()) {
        }
        System.out.println("Finished!");
    }
}
```

运行结果：

```
我是lock1， Thread-0


Thread-0 lock1运行结束！
我是lock2， Thread-0
我是lock1， Thread-1


Thread-0 lock2运行结束！
Thread-1 lock1运行结束！
我是lock2， Thread-1


Thread-1 lock2运行结束！
Finished!

Process finished with exit code 0
```

这里我把输出结果根据输出等待时间分成了四部分。

第一部分是Thread0先拿到lock1这把锁，那么thread1执行run方法时，首先进入lock1的代码块判断lock1是否可用，这里因为thread0占据了lock1,所以此时thread1进入等待状态。

第二部分，3s后，thread0执行完释放了lock1,然后立即往下执行，进入lock2的代码区域，拿到lock2这把锁，而thread1在等待thread0释放lock1后立即获取到lock1这把锁。

第三部分，3s后，thread0运行完毕释放lock2这把锁，thread1也运行完毕释放lock1这把锁，然后thread1进入lock2的代码区域。此时lock2已经被thread0释放了，所以thread1可以顺利获取到lock2,开始执行lock2的代码块。

第四部分，thread1执行完毕lock2的代码块，程序执行完毕，退出。

#### <span id = "anchor">普通方法锁</span>

这里将run方法内的代码放到一个方法中，然后使用synchronized修饰这个方法，功能与使用synchronized代码块是一样的。

```java
package demo;
public class SynchronizedObjectMethod3 implements Runnable{
    static SynchronizedObjectMethod3 instance = new SynchronizedObjectMethod3();
    public static void main(String[] args) {
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();
        t2.start();
        while(t1.isAlive() || t2.isAlive()) {
        }
        System.out.println("Finished!");
    }

    @Override
    public void run() {
        method();
    }

    public synchronized void method() {
        System.out.println("对象锁的方法修饰形式， " + Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " 运行结束！");
    }
}
```

运行结果：

```
对象锁的方法修饰形式， Thread-0

Thread-0 运行结束！
对象锁的方法修饰形式， Thread-1

Thread-1 运行结束！
Finished!

Process finished with exit code 0
```

将结果按输出延迟分成了三部分。

第一部分是t1获取到method的锁，开始执行method方法。此时t2执行到method时由于method的锁已经被占据，所有t2进入Blocked状态等待锁解除。

第二部分是t1运行3s后，解除锁，然后t2获取锁，执行method内的方法。

第三部分，t2执行完毕，程序退出。


### 类锁

> Java类可能有多个对象，但是只有一个Class对象。

> 所谓的类锁，就是不同的Class对象的锁而已。

> 类锁的效果就是在同一时刻，只能有一个对象拥有锁。

类锁分两种：

- Synchronized修饰的静态方法

- 指定锁为Class对象，synchronized(*.class)

#### <span id = "anchor1">Synchronized修饰的静态方法</span>

在使用synchronized修饰的方法不是静态方法时，不同实例的线程是可以同时访问该方法的。

```java
package demo;

public class SynchronizedClassStatic4 implements Runnable {
    static SynchronizedClassStatic4 instance1 = new SynchronizedClassStatic4();
    static SynchronizedClassStatic4 instance2 = new SynchronizedClassStatic4();

    public static void main(String[] args) {
        Thread t1 = new Thread(instance1);
        Thread t2 = new Thread(instance2);
        t1.start();
        t2.start();
        while(t1.isAlive() || t2.isAlive()) {
        }
        System.out.println("Finished!");
    }

    @Override
    public void run() {
        method();
    }

    public synchronized void method() {
        System.out.println("类锁的第一种形式，使用synchronized修饰static方法， " + Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " 运行结束！");
    }
}
```

运行结果：

```
类锁的第一种形式，使用synchronized修饰static方法， Thread-0
类锁的第一种形式，使用synchronized修饰static方法， Thread-1

Thread-0 运行结束！
Thread-1 运行结束！
Finished!

Process finished with exit code 0
```

可以看到两个线程同时访问到了method方法。

加上static修饰method方法后，代码如下：

```java
package demo;

public class SynchronizedClassStatic4 implements Runnable {
    static SynchronizedClassStatic4 instance1 = new SynchronizedClassStatic4();
    static SynchronizedClassStatic4 instance2 = new SynchronizedClassStatic4();

    public static void main(String[] args) {
        Thread t1 = new Thread(instance1);
        Thread t2 = new Thread(instance2);
        t1.start();
        t2.start();
        while(t1.isAlive() || t2.isAlive()) {
        }
        System.out.println("Finished!");
    }

    @Override
    public void run() {
        method();
    }

    public static synchronized void method() {
        System.out.println("类锁的第一种形式，使用synchronized修饰static方法， " + Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " 运行结束！");
    }
}
```

运行结果：

```
类锁的第一种形式，使用synchronized修饰static方法， Thread-0

Thread-0 运行结束！
类锁的第一种形式，使用synchronized修饰static方法， Thread-1

Thread-1 运行结束！
Finished!

Process finished with exit code 0
```

可以看到，当thread0获取到方法锁时，thread1是无法同时执行的，只能等thread0执行完毕释放锁后thread1才能执行。

**所以如果遇到一些情况需要在全局同步某个方法，而不仅仅是一个对象的层面或者是一个小范围时，就应该使用static修饰和保护被同步的方法。**

#### 指定锁为Class对象

使用synchronized修饰Class，上代码吧。

```java
package demo;

public class SynchronizedClassClass5 implements Runnable {
    static SynchronizedClassClass5 instance1 = new SynchronizedClassClass5();
    static SynchronizedClassClass5 instance2 = new SynchronizedClassClass5();

    public static void main(String[] args) {
        Thread t1 = new Thread(instance1);
        Thread t2 = new Thread(instance2);
        t1.start();
        t2.start();
        while(t1.isAlive() || t2.isAlive()) {
        }
        System.out.println("Finished!");
    }

    @Override
    public void run() {
        method();
    }

    public void method() {
        synchronized (SynchronizedClassClass5.class) {
            System.out.println("类锁的第二种形式，使用synchronized修饰Class， " + Thread.currentThread().getName());
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " 运行结束！");
        }
    }
}

```

运行结果：

```
类锁的第二种形式，使用synchronized修饰Class， Thread-0

Thread-0 运行结束！
类锁的第二种形式，使用synchronized修饰Class， Thread-1

Thread-1 运行结束！
Finished!

Process finished with exit code 0
```

可以看到，这里是对SynchronizedClassClass5的Class加锁了，所有SynchronizedClassClass5的实例都会使用这把锁，在t1获取到锁的时候t2进入Blocked状态，t1执行完后t2再执行。

如果把SynchronizedClassClass5.class改成this，那么instance1和instance2就不是共用一把锁啦，他们可以同时执行method方法。将method改为如下代码：

```java
public void method() {
    synchronized (this) {
        System.out.println("类锁的第二种形式，使用synchronized修饰Class， " + Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " 运行结束！");
    }
}
```

最后执行结果：

```
类锁的第二种形式，使用synchronized修饰Class， Thread-0
类锁的第二种形式，使用synchronized修饰Class， Thread-1

Thread-0 运行结束！
Thread-1 运行结束！
Finished!

Process finished with exit code 0
```

印证了我们的猜想。

## 不使用并发手段那个例子的解决方法

1. 使用对象锁中的同步代码块锁，在run方法中加入synchronized同步代码块，即：

    ```java
    @Override
    public void run() {
        synchronized (this) {
            for (int j = 0; j < 100000; j++) {
                i++;
            }
        }
    }
    ```

2. 使用对象锁中的普通方法锁，在run方法前面加上synchronized关键字修饰，即：

    ```java
    @Override
    public synchronized void run() {
        for (int j = 0; j < 100000; j++) {
            i++;
        }
    }
    ```

3. 使用类锁中的修饰静态方法，即：

    ```java
    @Override
    public void run() {
        method();
    }

    public synchronized static void method() {
        for (int j = 0; j < 100000; j++) {
            i++;
        }
    }
    ```

4. 使用类锁中的修饰Class对象的方法，即：

    ```java
    @Override
    public void run() {
        synchronized (DisappearedRequest1.class) {
            for (int j = 0; j < 100000; j++) {
                i++;
            }
        }
    }
    ```


## 常见问题

1. 两个线程同时访问一个对象的同步方法。

使用synchronized修饰普通方法模式修饰的是this对象。

两个线程访问同一个对象的同步方法时，会是互斥的。因为两个线程来自同一个实例，一个线程获取到这个锁后另一个线程对这个锁也是可见的。最后的结果就是当线程1获取到锁后，线程2进入Blocked状态，等线程1执行完该同步方法释放锁后，线程2才能开始执行同步方法。参考上面的[普通方法锁](#anchor)相关的内容。

2. 两个线程访问的是两个对象的同步方法。

两个线程在访问同一个同步方法时，不会存在互斥，也就是两个方法会同时执行，互不影响。因为两个线程来自不同实例，他们采用的锁对象不是同一个。一个线程获取到锁后跟另一个线程没有任何关系，所以另一线程也会继续执行，最后的结果就是两个线程都同时执行。可以参考上面的[Synchronized修饰的静态方法](#anchor1)中不加static修饰method的情况。

3. 两个线程访问的是synchronized的静态方法。

当两个线程访问synchronized修饰的静态方法时，不管这两个线程是不是同一个实例创建的，只要他们是同一个类的实例，他们就会反问到同一个锁，所以是互斥的。可以参考上面的[Synchronized修饰的静态方法](#anchor1)中加上static修饰method的情况。

4. 同时访问同步方法和非同步方法

先上代码：

```java
package demo;

public class SynchronizedYesAndNo6 implements Runnable{

    static SynchronizedYesAndNo6 instance = new SynchronizedYesAndNo6();

    public static void main(String[] args) {
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();
        t2.start();
        while(t1.isAlive() || t2.isAlive()) {
        }
        System.out.println("Finished!");
    }

    @Override
    public void run() {
        if (Thread.currentThread().getName().equals("Thread-0")) {
            method1();
        } else {
            method2();
        }
    }

    public synchronized void method1() {
        System.out.println("这是加锁的方法， " + Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " 运行结束！");
    }

    public void method2() {
        System.out.println("这是没有加锁的方法， " + Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " 运行结束！");
    }
}
```

该实例中两个线程皆由同一个实例创建，run方法中，让thread0调用method1，让thread1调用method2，运行结果如下：

```
这是没有加锁的方法， Thread-1
这是加锁的方法， Thread-0

Thread-1 运行结束！
Thread-0 运行结束！
Finished!

Process finished with exit code 0
```

由于method1是加锁的，所以thread0运行到method1时，获取到锁，然后进入休眠状态。method2没有锁，thread1在执行method2时没有任何阻碍，直接执行，所以看起来的效果就是thread1和thread2同时执行了，因为二者一个有锁一个没锁，互不影响。

5. 访问同一个对象的不同的普通同步方法

~~将第四问代码中的method2改成同步方法就是该问题的测试代码，我比较懒，就不重新创建代码了，直接将method2改为~~ , 算了，还是贴全一点吧：

```java
package demo;

public class SynchronizedDifferentMthod7 implements Runnable{

    static SynchronizedDifferentMthod7 instance = new SynchronizedDifferentMthod7();

    public static void main(String[] args) {
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();
        t2.start();
        while(t1.isAlive() || t2.isAlive()) {
        }
        System.out.println("Finished!");
    }

    @Override
    public void run() {
        if (Thread.currentThread().getName().equals("Thread-0")) {
            method1();
        } else {
            method2();
        }
    }

    public synchronized void method1() {
        System.out.println("这是第一个加锁的方法， " + Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " 运行结束！");
    }

    public synchronized void method2() {
        System.out.println("这是第二个加锁的方法， " + Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " 运行结束！");
    }
}

```

运行代码，结果是：

```
这是第一个加锁的方法， Thread-0

Thread-0 运行结束！
这是第二个加锁的方法， Thread-1

Thread-1 运行结束！
Finished!

Process finished with exit code 0
```

由于两个线程是同一个实例创建，而且synchronized修饰的是普通方法，也就相当与修饰的是普通的this对象，所以这两个线程访问到两个方法时碰到的锁是同一个锁，最终执行时就是谁先获取到锁谁先执行。

6. 同时访问静态synchronized和非静态synchronized方法

贴代码：

```java
package demo;

public class SynchronizedStaticAndNormal8 implements Runnable{
    static SynchronizedStaticAndNormal8 instance = new SynchronizedStaticAndNormal8();

    public static void main(String[] args) {
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();
        t2.start();
        while(t1.isAlive() || t2.isAlive()) {
        }
        System.out.println("Finished!");
    }

    @Override
    public void run() {
        if (Thread.currentThread().getName().equals("Thread-0")) {
            method1();
        } else {
            method2();
        }
    }

    public synchronized static void method1() {
        System.out.println("这是第一个静态加锁的方法， " + Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " 运行结束！");
    }

    public synchronized void method2() {
        System.out.println("这是第二个非静态加锁的方法， " + Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " 运行结束！");
    }
}
```

运行结果：

```
这是第一个静态加锁的方法， Thread-0
这是第二个非静态加锁的方法， Thread-1

Thread-0 运行结束！
Thread-1 运行结束！
Finished!

Process finished with exit code 0
```

这是因为使用synchronized修饰静态方法method1时，它的锁对象是该实例类的Class对象。而使用synchronized修饰的非静态方法method2时，它的锁对象是this对象也就是该实例对象本身，method1和method2两个触发的锁是不同的锁，所以二者互补影响，同时执行。

7. 方法抛出异常后，会释放锁吗？

会的，设置一个这样的场景验证：两个线程同时运行，第一个线程抛出异常后，看第二个线程是否会立即进入同步方法，如果能成功进入，那就意味着锁已经释放。

上代码：

```java
package demo;

public class SynchronizedException9 implements Runnable {

    static SynchronizedException9 instance = new SynchronizedException9();

    public static void main(String[] args) {
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();
        t2.start();
        while(t1.isAlive() || t2.isAlive()) {
        }
        System.out.println("Finished!");
    }

    @Override
    public void run() {
        if (Thread.currentThread().getName().equals("Thread-0")) {
            method1();
        } else {
            method2();
        }
    }

    public synchronized void method1() {
        System.out.println("这是第一个方法， " + Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
            throw new Exception();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " 运行结束！");
    }

    public synchronized void method2() {
        System.out.println("这是第二个方法， " + Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " 运行结束！");
    }
}
```

运行结果：

```
这是第一个方法， Thread-0

java.lang.Exception
	at demo.SynchronizedException9.method1(SynchronizedException9.java:30)
	at demo.SynchronizedException9.run(SynchronizedException9.java:20)
	at java.lang.Thread.run(Thread.java:748)
Thread-0 运行结束！
这是第二个方法， Thread-1

Thread-1 运行结束！
Finished!

Process finished with exit code 0
```

可以看到，第一个方法抛出异常后，第二个方法能够正常获取锁然后执行method2,说明抛出异常后会释放try代码块中获取的锁。这里释放锁的操作是jvm自动完成，后面会有介绍里面的详细原理。

### 7中常见问题总结（3个核心思想）

1. 一把锁同时只能被一个线程获取，没拿到锁的线程必须等待其他线程释放锁才能获取到锁，对应问题1,5

2. 每个实例都对应有自己的一把锁，不同实例之间互不影响，但是有例外情况：锁的对象是Class以及synchronized修饰的是static方法的时候，所有对象共用一把类锁，对应2,3,4,6情况。

3. 无论方法是正常执行完毕还是抛出异常而结束，都会释放锁，对应问题7

**终极一问： 如果在synchronized修饰的方法中调用了普通方法（没有synchronized修饰），那么这个被synchronized修饰的方法还是线程安全的吗？**

答案是：NO，因为一旦调用了不加锁的方法（不用synchronized修饰），那么其他线程也可能会同时调用那个不加锁的方法，导致当前被synchronized修饰的方法变成线程不安全方法。


## Synchronized性质

### 可重入

> 可重入指的是同一线程的外层函数获得锁后，内层函数可以直接再次获取该锁。

好处是：避免死锁，提升封装性。

假设有两个方法fun1,fun2，fun1中调用fun2,他们都被synchronized修饰，如果线程1执行到了fun1，并且获得了fun1的锁。假设synchronized不具有可重入性，由于fun2的锁跟fun1的锁是同一把锁，所以线程访问到fun2时会等待锁的解除，而fun1却一直在等待fun2的结束，最后就是fun2一直等待fun1释放锁，而fun1却一直在等待fun2执行完成，所就会导致死锁的发生。

另外可重入性避免了一次次解锁加锁操作，提升了封装性。

粒度：线程而非调用（用3种情况说明和pthread的区别）。只要在当前线程中获取到了一个锁那么这个锁可以被当前线程的其他部分使用，不需要重新获取和释放锁。

#### 情况1：证明同一个方法是可重入的

先从一个synchronized方法调用自身说起，如果能够调用成功则说明synchronized修饰的方法具有可重入性，如果进入死锁状态，则说明synchronized修饰的方法不具有可重入性。代码如下：

```java
package demo;

public class SynchronizedRecursion10 {

    int a = 0;

    public static void main(String[] args) {
        SynchronizedRecursion10 synchronizedRecursion10 = new SynchronizedRecursion10();
        synchronizedRecursion10.method1();
    }

    private synchronized void method1() {
        System.out.println("method1, a = " + a);
        if (a ==0) {
            a++;
            method1();
        }
    }
}
```

运行结果：

```
method1, a = 0
method1, a = 1

Process finished with exit code 0
```

程序正确调用了自身，说明synchronized修饰的方法是具有可重入性的。

#### 情况2：证明可重入不要求是同一个方法。

建立两个synchronized修饰的方法method1和method2,使用method1调用method2,如果能够调用成功，那么可以证明可重入性要求调用的是同一个方法，如果不能调用成功而进入死锁状态，则说明可重入不要求同一个方法。代码如下：

```java
package demo;

public class SynchronizedOtherMethod11 {
    public synchronized void method1() {
        System.out.println("mtheod1");
        method2();
    }

    public synchronized void method2() {
        System.out.println("method2");
    }

    public static void main(String[] args) {
        SynchronizedOtherMethod11 synchronizedOtherMethod11 = new SynchronizedOtherMethod11();
        synchronizedOtherMethod11.method1();
    }
}
```

运行结果：

```
mtheod1
method2

Process finished with exit code 0
```

结果表明可重入不要求是同一个方法。

#### 情况3：证明可重入不要求是同一个类中的。

使用一个子类继承父类的方法，该方法在子类和父类中都用synchronized修饰，然后在子类中调用父类的方法，看是否能调用成功，如果能调用成功，则说明可重入不要求是同一个类中，代码如下：

```java
package demo;

public class SynchronizedSuperClass12 {

    public synchronized void doSomthing() {
        System.out.println("我是父类方法");
    }
}

class TestClass extends SynchronizedSuperClass12 {
    public synchronized void doSomthing() {
        System.out.println("我是子类方法");
        super.doSomthing();
    }

    public static void main(String[] args) {
        TestClass testClass = new TestClass();
        testClass.doSomthing();
    }
}
```

运行结果：

```
我是子类方法
我是父类方法

Process finished with exit code 0
```

### 不可中断

> 一旦一个锁被其他线程获取了，如果还想获得，只能选择等待其他线程释放锁或者进入阻塞状态，直到其他线程释放锁。如果其他线程永远不释放锁，那么就会永远等待下去。

相对来说，Lock类具有中断能力。首先，如果觉得等待时间太长，程序有权中断现在已经获取到锁的线程；其次，如果觉得等待时间太久不想等了，也可以选择退出等待。

## 底层原理

### 加锁和释放锁的原理

**现象：** 每个类的实例对应一把锁，每个synchronized修饰的方法都必须首先获得调用该方法的类的实例的锁才能执行，否则线程就会阻塞。方法一旦执行，就会独占这把锁，直到该方法返回或者抛出异常才释放锁，然后其他线程才能获取这把锁，进入可执行状态。

这意味着如果一个对象中有synchronized修饰的方法或者代码块，要想执行这段代码就必须先获得对象的实例对象锁。如果此对象的锁已经被其他调用着占用，就必须等待锁被释放。所有的java对象都有一个互斥锁，由jvm自动获取和释放，我们只需要指定这个对象即可，至于锁的获取和释放不用关系，jvm会帮你做。

**获取和释放锁的时机：内置锁**

每一个java对象都可以作为实现同步的锁，这个锁就称为内置锁，或者叫监视器锁（Monitor Lock），线程在进入和退出同步代码块前后会自动获取和释放这个锁，无论是正常途径推出还是抛出异常推出，都会释放锁。获取内置锁的唯一途径就是进入内部同步代码块方法中。

**等价代码**

以一段代码为例，创建两个等价的方法method1和method2,method1使用synchronized修饰，method2使用Lock锁实现类似与method1的锁形式，代码如下：

```java
package demo;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class SynchronizedToLock13 {

    Lock lock = new ReentrantLock();

    public synchronized void method1() {
        System.out.println("这是synchronized锁！");
    }
    //method2与method1等价
    public void method2() {
        lock.lock();
        try {
            System.out.println("这是Lock锁!");
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        SynchronizedToLock13 synchronizedToLock13 = new SynchronizedToLock13();
        synchronizedToLock13.method1();
        synchronizedToLock13.method2();
    }
}
```

运行结果如下：

```
这是synchronized锁！
这是Lock锁!

Process finished with exit code 0
```

**深入JVM看字节码：反编译、monitor指令**

java对象的头字段中有一个表示锁状态的标记，在获取和释放锁的时候是基于monitor对象实现的，monitor主要有两个指令monitor enter（插入到方法开始的位置）和monitor exit（插入到方法结束的位置），JVM规定每一个monitorenter都会有monigtorexit与之对应，但是一个monitorenter可能会有多个monitorexit，因为退出的时机可能是多种多样的。

每个对象都有一个monitor与之关联，一旦持有了一个monitor，该对象就会处于锁定状态，当线程执行到monitor enter这个指令时，将会尝试获取这个对象所持有的monitor的所有权，也就是尝试获取对象的锁。

反编译

创建一个实例代码命名为Decompilation14.java：

```java
package demo;

public class Decompilation14 {
    private Object object = new Object();

    public void insert(Thread thread) {
        synchronized (object) {

        }
    }
}
```

在命令行进行编译：

```
javac Decompilation14.java 

```

可以看到反编译的信息：

```
Code:
    stack=2, locals=4, args_size=2
        0: aload_0
        1: getfield      #3                  // Field object:Ljava/lang/Object;
        4: dup
        5: astore_2
        6: monitorenter
        7: aload_2
        8: monitorexit
        9: goto          17
    12: astore_3
    13: aload_2
    14: monitorexit
    15: aload_3
    16: athrow
    17: return
```

第6行的monitorenter表示进入同步代码块，获取到monitor对象也就是获取到对象的锁。第8和14行表示离开同步代码块，释放锁。

其实monitorenter和monitorexit指令在执行的时候会将对象的锁计数加1或者减1,monitor同一时刻只能被一个对象获得，线程在尝试获得与当前对象关联的monitor的所有权的时候，monitor只会发生下面三种情况：

1. monitor计数器为0,表示目前该monitor还没有被其他进程占据，当前线程会立即获得monitor对象，并把计数器加1，此后其他线程想要获取monitor的时候看到计数器为1,就知道当前对象的monitor已经被其他线程占有了。

2. 当前线程已经拿到了monitor的所有权，而且已经重入，这回导致monitor的计数器累加，计数器变为2或者3等.

3. monitor已经被其他线程持有，当前线程想要获取的时候无法获取，进入阻塞状态，直到monitor计数器变为0后再尝试获取monitor。

monitorexit的操作很简单，就是将monitor计数器减1,一旦monitor的计数器减为0,就表示当前线程不再具备monitor的所有权，也就是完全释放了锁。如果减完后，monitor计数器不是0,就表示刚才是重入进来的，当前线程还是持有这把锁。

### 可重入原理

可重入是利用了加锁次数计数器实现的，JVM负责跟踪对象被加锁的次数，也就是monitor中计数器。

线程第一次对象加锁的时候，计数器变为1,每当这个线程在次对象上再次获得锁是，计数器就加1，只有首先在这个对象上获取到锁的线程可以再次获取这把锁。

当任务离开时，计数器减1,当计数器为0的时候，锁被完全释放。

### 保证可见性的原理

Java内存模型：

![Java内存模型](https://www.wjy329.com/wp-content/uploads/image/20180905/1536152755751077.png)

各个线程会将主内存中的共享变量复制一份放到本地内存，这样做的原因是加速程序运行。但是这样做会带来一些风险，线程A要跟线程B通信需要现将数据写入主内存，然后线程B再去主内存中读取数据到本地内存B,这部分是JVM进行控制的。

![线程通信](http://ifeve.com/wp-content/uploads/2013/01/221.png)

使用synchronized修饰的代码在释放锁之前都要将线程内存中的数据写入到主内存中，在进入代码块得到锁后，被锁定的对象的数据会立即从主内存中读取到本地线程内存中。


## Synchronized的缺陷

1. 效率低。锁的释放情况少，试图获得锁的时候不能设定超时时间，不能中断一个正在试图获得锁的线程。

锁的释放情况少是因为只有在以下两种情况下才会释放锁：线程执行完代码后释放锁和代码执行过程中出现异常释放锁。

如果遇到IO阻塞，而锁又不能释放，这就会出现效率问题。

2. 不够灵活（读写锁更灵活）：加锁和释放的时机单一，每个锁仅有单一的条件(某个对象)，不够灵活。比如读写锁，它的加锁和解锁的时机是非常灵活的，在读操作的时候不加锁，只在写的时候才加锁，大大提高程序执行效率。

3. 无法知道是否成功获取到锁。

## 常见面试问题

1. 使用synchronized时需要注意些什么？

使用时需要注意锁对象不能为空，作用域不宜过大，要避免死锁。

锁对象为空是，锁的信息无法保存到对象头中，所以锁是不起任何作用的。

作用域过大时，就变成串行变成了，失去了多线程的意义。

2. 如何选择Lock和synchronized关键字？

如果可以，即不要使用Lock也不要使用synchronized，而是使用java.util.concurrent包中的各种类，更方便也更不容易出错。

如果synchronized在程序中适用，优先使用synchronized，因为这样可以减少代码的编写，也就减少了代码出错的几率。

如果特别需要用到Lock独有的特性时才考虑使用Lock。

3. 多线程访问同步方法的各种具体情况？

参考上面的各种问题。

## 思考题

1. 多个线程等待同一个synchronized锁的时候，JVM是如何选择下一个获取锁的是哪个线程？

这涉及到JVM内部对于锁的调度机制，持有锁的进程释放锁后，竞争锁的进程包括之前就在等待的，还包括现在刚刚到来的线程，谁能获取到这把锁是不公平的状态，谁都有可能获取，根据JVM版本和算法的不同而不同。

2. synchronized使得同时只有一个线程执行，性能较差，有什么办法可以提升性能？

优化使用范围，防止误操作，充分发挥多线程的功能。

使用其他类型的锁比如读写锁Lock。

3. 我想更灵活地控制锁的获取和释放（现在释放锁的时机都被规定死了），怎么办？

可以自己实现一个Lock接口，这样锁的控制和释放就完全由自己控制了，可以参考一些已经实现的已有的锁。

4. 什么是锁的升级、降级？什么是JVM里的偏斜锁、轻量级锁、重量级锁？

balabala

## 总结

一句话介绍synchronized：
> JVM会自动通过使用monitor来加锁和解锁，保证了同时只有一个线程可以执行指定代码，从而保证了线程安全，同时具有可重入和不可中断的性质。

