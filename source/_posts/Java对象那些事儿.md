---
title: java对象那些事儿
date: 2020-05-13 21:15:42
tags:
---

- Java 对象的创建过程 半初始化

- DLC 与 volatile 问题 指令重排

- 对象在内存中的布局 （对象与数组存储不同）

- 对象头具体包括什么（markword klasspointer synchronized 锁信息）

- 对象怎么定位？（直接、间接）

- 对象怎么分配？（栈上、线程本地、Eden、old）

- Object o = new Object() 在内存中占用多少字节？

## Java 对象创建过程

用一个最简单的小程序说明：

```java
public class T {
    int m = 8;

    public static void main(String[] args) {
        T t = new T();
    }
}
```

编译后生成的字节码部分文件内容如下：

```java
public static main([Ljava/lang/String;)V
   L0
    LINENUMBER 5 L0
    NEW T
    DUP
    INVOKESPECIAL T.<init> ()V
    ASTORE 1
   L1
    LINENUMBER 6 L1
    RETURN
   L2
    LOCALVARIABLE args [Ljava/lang/String; L0 L2 0
    LOCALVARIABLE t LT; L1 L2 1
    MAXSTACK = 2
    MAXLOCALS = 2
```

对象创建分三步：

1. 申请内存 `NEW T` 

2. 调用构造方法，赋初始值 `INVOKESPECIAL T.<init> ()V`

3. 建立关联，将变量 `t` 与创建的对象关联起来  `ASTORE 1`


对象的创建过程是半初始化的，先申请内存，然后再赋值，最后再将变量与申请的内存空间关联起来。

## DCL 问题和 volatile 是否有必要

首先看一个基础的设计模式 -- 单例模式，最简单的创建单例方法：

### 单例模式是实现方式一

```java
public class Mgr01 {
    private static final Mgr01 INSTANCE = new Mgr01();

    private Mgr01() {}

    public static Mgr01 getInstance() {return INSTANCE;}

    public void m() {
        System.out.println("m");
    }

    public static void main(String[] args) {
        Mgr01 m1 = Mgr01.getInstance();
        Mgr01 m2 = Mgr01.getInstance();
        System.out.println(m1 == m2);
    }
}
```

这里虽然实现了单例的要求，但是还是有点问题的，比如对象在使用前直接就创建了出来，如果对象比较占用内存空间，岂不是浪费资源吗？所以可以改进一下，等需要使用的时候再创建对象。

### 单例模式实现方式二

```java
public class Mgr03 {
    private static Mgr03 INSTANCE;

    private Mgr03() {}

    public static Mgr03 getInstance() {
        if (INSTANCE == null) {
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            INSTANCE = new Mgr03();
        }
        return INSTANCE;
    }

    public void m() {
        System.out.println("m");
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() ->
                    System.out.println(Mgr03.getInstance().hashCode())
            ).start();
        }
    }
}
```

这种实现方式解决了方式一中的问题，也就是避免程序在一开始执行就创建对象实例，只有在要使用对象的时候才创建。但是这种实现方式还是有个问题，那就是线程不安全，多个线程同时调用 `getInstance()` 方法时，在判断 `INSTANCE` 变量是否为 null 时可能出现多个线程中的判断条件都成立的情况，因此会出现多次创建实例的情况，这就不是单例了。

下面是改进多线程不安全问题后的实现方式。

### 单例模式实现方式三

```java
public class Mgr04 {
    private static Mgr04 INSTANCE;

    private Mgr04() {}

    public static synchronized Mgr04 getInstance() {
        if (INSTANCE == null) {
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            INSTANCE = new Mgr04();
        }

        return INSTANCE;
    }

    public void m() {
        System.out.println("m");
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() ->
                    System.out.println(Mgr04.getInstance().hashCode())
            ).start();
        }
    }
}
```

线程不安全的原因是多个线程同时访问到 `getInstance()` 方法时没有做相应的处理，这里用最简单的 `synchronized` 加锁来处理多线程同步问题。

这里已经解决了多线程不安全的问题，不过如果要深究这个 `synchronized` 加的地方是否合理的话，我们或许有更合理的处理办法，因为这里是将整个方法都加锁了，当然对于当前的程序来说没什么问题，但是如果方法内有比较耗时的业务逻辑或者处理过程，而且这部分逻辑是不需要加锁的，那我们当前这个方法锁的粒度就太大了，可能会对系统性能有较大影响，这时就可以考虑将锁的位置加到方法内部可能处理并发安全问题的代码块上。

改进后的代码实现看下面。

### 单例模式实现方式四

```java
public class Mgr05 {
    private static Mgr05 INSTANCE;

    private Mgr05() {}

    public static Mgr05 getInstance() {
        if (INSTANCE == null) {
            synchronized (Mgr05.class) {
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                INSTANCE = new Mgr05();
            }
        }
        return INSTANCE;
    }

    public void m() {
        System.out.println("m");
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() ->
                    System.out.println(Mgr05.getInstance().hashCode())
            ).start();
        }
    }
}
```

这里将锁的位置加在了判断条件里面，在判断需要创建对象时对创建对象的动作加锁，但是这个实现是有问题的，因为多个线程可能执行到判断条件这个地方，但是只有一个进程获得锁，其他进程阻塞到判断条件后的那个语句（这里说阻塞到一个语句可能不太合理，不过不妨碍理解），当获得锁的那个进程执行完后创建了一个实例，其他阻塞的进程争抢锁后继续创建对象实例，这样的话根本不能实现单例模式，所以这里还需要改进，在锁后面的语句再加一个判断，确认实例对象是否已经创建，如果已经创建就直接使用已经创建的那个实例，如果没创建就自己创建一个，这样就能保证只存在一个实例，也就是实现了单例模式。

改进后的代码如下：

```java
public class Mgr06 {
    private static volatile Mgr06 INSTANCE;

    private Mgr06() {
    }

    private static Mgr06 getInstance(){
        if (INSTANCE == null) {
            synchronized (Mgr06.class) {
                if (INSTANCE == null) {
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    INSTANCE = new Mgr06();
                }
            }
        }
        return INSTANCE;
    }

    public void m() {
        System.out.println("m");
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() ->
                    System.out.println(Mgr06.getInstance().hashCode())
            ).start();
        }
    }
}
```

这个最终的代码就是我们说的 DCL(Double Check Lock) 实现：先检查一次再上锁，锁内在次进行一次检查，也就是检查了两次，就是 Double Check Lock。

这里有个问题是单例前面的 `volatile` 是否有必要添加，为什么？

`volatile` 有两个作用：1.保持多个线程内存可见性；2.禁止指令重排序。

保持线程可见性这个比较好理解，就是多个线程访问到一个变量时能够保证这些线程访问到的这个变量是最新的，防止多个线程在各自的线程内存中更改了公共的变量后其他线程不可见的问题。

**那指令重排序是什么呢？**

举个栗子，CPU 要顺序执行两条指令：cmd1 和 cmd2，其中 cmd1 是去内存取数据，cmd2 是对寄存器的操作，我们知道 cpu 的执行速度是比内存快很多的，所以在 cmd1 执行取内存数据 CPU 等待返回数据的这段时间，CPU 其实是空闲的。这其实是一种资源浪费，所以有了一种机制就是指令重排，将 cmd2 先执行，再执行 cmd1，这样就在固定时间内把两条指令都执行了。

接下来就是重点回答 volatile 要不要加这个问题了，如果不加 volatile，就可能发生指令重排的问题，我们前面说过对象创建过程有三步，1. 申请内存；2. 调用构造方法，赋初始值；3. 建立关联。这三步是分别对应三条指令的，如果发生了指令重排，那么指令三就可能先于指令二执行，也就是将还未赋值的内存空间跟变量关联起来，如果此时其他线程正好也在创建对象，那么它的判断条件 `if (INSTANCE == null)` 就是正确的，然后直接返回了中间状态的对象（对象中 m 变量的值是 0 而不是我们希望的 8），这是有问题的。

## 对象在内存中的存储布局

对象内存由四部分组成：markword，class pointer，instance data，padding

![image-20200524212409677](/Users/mac/Library/Application Support/typora-user-images/image-20200524212409677.png)

先看一个程序，打印一下对象的相关信息：

```java
import org.openjdk.jol.info.ClassLayout;

public class HelloObj {
    public static void main(String[] args) {
        Object o = new Object();
        String s = ClassLayout.parseInstance(o).toPrintable();
        System.out.println(s);
    }
}
```

输出结果如下：

```
java.lang.Object object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
      4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4        (object header)                           e5 01 00 f8 (11100101 00000001 00000000 11111000) (-134217243)
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
```

其中第三和第四行叫 markword，第五行叫 class pointer，用于指向 o.class 这个对象，第六行是保持字节对齐用的，保证对象占用大小是 8  的整数倍。

```
OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
      4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
```

### 对象头具体都包含什么

**`markword，classpointer`**

简单回答就是包含了锁信息，下面以一个 synchronized 锁为例讲解一下：

```java
import org.openjdk.jol.info.ClassLayout;

public class synchronizedTest {
    public static void main(String[] args) {
        Object o = new Object();
        System.out.println(ClassLayout.parseInstance(o).toPrintable());
        synchronized (o) {
            System.out.println(ClassLayout.parseInstance(o).toPrintable());
        }
    }
}
```

将一个对象加锁前后的内存信息打印一下，输出结果如下：

```
java.lang.Object object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
      4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4        (object header)                           e5 01 00 f8 (11100101 00000001 00000000 11111000) (-134217243)
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

java.lang.Object object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           90 c9 e4 03 (10010000 11001001 11100100 00000011) (65325456)
      4     4        (object header)                           00 70 00 00 (00000000 01110000 00000000 00000000) (28672)
      8     4        (object header)                           e5 01 00 f8 (11100101 00000001 00000000 11111000) (-134217243)
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
```

对比一下前后的输出，可以看到对象头那部分发生了改变，其实就是 synchronized 起作用了。

一个刚创建的对象上锁的过程一般都是：新对象 -> 偏向锁 -> 自旋锁『无锁/lock-free/轻量级锁』-> 重量级锁。



