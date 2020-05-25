---
layout: post
title: Java之并发(Concurrency)
categories: Java
description: 讲解Java中的并发
keywords: Java, concurrency, 并发
---

有一些编程问题，我们用顺序执行的程序就能解决。然而，还有一些问题，我们使用并发编程更方便，甚至于必须使用并发编程才能解决。并发程序能极大提升程序的性能，还能为一些问题提供一个模型，更便于解决。

随之而来的问题就是并发是一个非常深奥的话题，实际上，即使我们写出了一个高性能的并发程序，有的时候还是有一些很难发现的错误，时而出现，时而消失，让我们非常难解决。

在Java中使用多线程来实现程序的并发执行，我们先看看什么是线程，再逐步了解Java的并发编程。

# 1. 线程与并发

Java的并发离不开线程，想了解线程又离不开进程和并行。在介绍线程和进程等概念之前，先举个例子来方便我们理解线程和进程的关系和区别。

我们都有过排队买麦当劳的经历，每家店的柜台都有多个机器可以点单，每个机器前都有许多人排队，可实际上只有一个后厨在做汉堡，我们吃的汉堡都要等后厨做出来才行，不管你在哪里排队都要等前面人的餐做好，才能做你的。这个模型就像线程的运作方式。每个点餐的机器都像一个线程，多个机器就是多线程，它们处理不同客人的订单，比起一台点餐机点单，也就是单线程，确实提高了效率，可实际上后厨只有一个，点完单子后还是要等后厨按点单的先后顺序一个一个做好才可以，比起单一窗口点餐，效率有提高，类比为**并发**。

效率更高的方式是在旁边再开一家麦当劳，这样又多了一个后厨，两家一起上，第二家麦当劳还有许多点餐机，效率能进一步提高，这就是多进程的执行方式也就是**并行**。一家麦当劳就像一个进程，点单机就像线程，一个进程有多个线程，提高程序执行效率。

回到我们的编程上，我们定义：

**进程(Process)**：进程是一个具有一定独立功能的程序在一个数据集上的一次动态执行的过程，是操作系统进行**资源分配**和**CPU调度**的基本单位，是应用程序运行的载体。

- 程序的一次执行过程
- 是正在运行程序的抽象
- 系统资源以进程为单位分配，如内存、文件等等，每个具有独立的地址空间
- 操作系统将CPU调度给需要的进程

**线程(Thread)**：线程是程序执行中一个单一的顺序控制流程，是**程序执行**的基本单位。一个进程可以有一个或多个线程，各个线程之间共享程序的内存空间(也就是所在进程的内存空间)。有时将线程称为**轻量级进程**。

**并行(Parallelism)**：指在同一时刻，有多条指令在多个处理器上同时执行。所以无论从微观还是从宏观来看，二者都是一起执行的。

![并行](/images/posts/java/concurrency_2.png)

**并发(Concurrency)**：指在同一时刻只能有一条指令执行，但多个进程指令被快速的轮换执行，使得在宏观上具有多个进程同时执行的效果，但在微观上并不是同时执行的，只是把时间分成若干段，使多个进程快速交替的执行。

![并发](/images/posts/java/concurrency_1.png)

# 2. 创建Java的线程

通过第一部分我们知道并发，也就是多线程程序并不是真正意义上的同一时刻执行多个任务，而是通过给不同的任务分配一点利用CPU的时间，从而快速地切换任务达到宏观上的同时执行多个任务。但是大部分情况下，我们不需要考虑这个问题。

我们使用多线程是为了让每个线程执行不同的任务，在Java中有两种创建新任务的方式，分别是继承`Thread`类和实现`Runnable`接口。

## 2.1 继承`Thread`类

我们创建一个类继承`Thread`类，并重写`run()`方法就能定义一个任务，再调用`start()`方法就能开启线程，例如：

```java
public class MyThread extends Thread {
    protected int countDown=10;
    private static int taskCount=0;
    private final int id=taskCount++; //不同实例的id

    public MyThread(){
    }

    public MyThread(int countDown){
        this.countDown=countDown;
    }

    public String status(){
        return "#"+id+"("+(countDown>0?countDown:"Liftoff!")+"),";
    }
    @Override
    public void run(){
        // run方法中一般都会有一个循环，有时是无限循环，等待被一些条件打断停止，否则会无限循环
        while (countDown-->0){
            System.out.println(status());
            Thread.yield(); //告诉线程调度器，这个线程已经完成任务了，CPU可以切换执行其它任务
        }
    }
}
```

```java
public class MyThreadTest {
    public static void main(String[] args) {
        MyThread thread=new MyThread();
        thread.start();
    }
}
```

## 2.2 实现`Runnable`接口

我们只需要实现`Runnable`接口并且重写`run()`方法就能定义一个任务，例如：

```java
public class LiftOff implements Runnable {
    protected int countDown=10;
    private static int taskCount=0;
    private final int id=taskCount++; //创建不同实例赋予的id

    public LiftOff(){
    }

    public LiftOff(int countDown){
        this.countDown=countDown;
    }

    public String status(){
        return "#"+id+"("+(countDown>0?countDown:"Liftoff!")+"),";
    }

    @Override
    public void run() {
        // run方法中一般都会有一个循环，有时是无限循环，等待被一些条件打断停止，否则会无限循环
        while (countDown-->0){
            System.out.println(status());
            Thread.yield(); //告诉线程调度器，这个线程已经完成任务了，CPU可以切换执行其它任务
        }
    }
}
```

创建好了任务之后，我们就需要一个线程去执行它。具体方法是，调用`Thread`类的构造器。把实现了`Runnable`接口的实例作为参数放到`Thread`构造方法中，创建新线程，再调用`start()`方法开启线程，例如：

```java
public class BasicThreads {
    public static void main(String[] args) {
        Thread t=new Thread(new LiftOff());
        t.start();
        System.out.println("Waiting for LiftOff");
    }
}
/**运行结果：
 * Waiting for LiftOff
 * #0(9),
 * #0(8),
 * #0(7),
 * #0(6),
 * #0(5),
 * #0(4),
 * #0(3),
 * #0(2),
 * #0(1),
 * #0(Liftoff!),
 */
```

从打印出的结果我们可以看到，即使`t.start()`方法在打印方法之前调用，实际执行的顺序其实是先打印再调用`t.start()`。原因是主线程又开启了一个线程去执行`LifeOff`类中复写的`run()`方法，而主线程完成打印的时间快去另一个线程循环的时间，所以打印顺序没有按照代码的先后顺序执行。

实际上，线程执行程序的顺序和代码顺序没有必然联系。

## 2.3 两种方法的比较

我们从源码查看`Thread`类：

```java
public class Thread implements Runnable{
    
    public Thread() {
        init(null, null, "Thread-" + nextThreadNum(), 0);
    }
    
    public Thread(Runnable target) {
        init(null, target, "Thread-" + nextThreadNum(), 0);
    }
    ...
    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
    ...
}
```

从`Thread`类源码能看出，`Thread`类实现了`Runnable`接口，并复写了`run()`方法，虽然复写了，但实际上并没有实现任何功能。第一种方法继承`Thread`类并直接复写了`run()`方法，从而能调用无参构造器直接开启新线程。而第二种方法调用了`Thread`类的第二个构造方法，把任务交给新线程，从而开启线程。

`Thread`类和`Runnable`接口之间在使用上也是有区别的，如果一个类继承 `Thread`类，则不适合于多个线程共享资源，而实现了`Runnable`接口，就可以方便地实现资源的共享。而实现接口也可以避免单继承的局限性。

## 2.4 线程的返回值 



**参考文章**：

[并发和并行的区别 (杰哥长得帅)](https://www.jianshu.com/p/cbf9588b2afb)

[进程与线程 (浅浅念)](https://www.cnblogs.com/qianqiannian/p/7010909.html)

[Java多线程看这一篇就足够了（吐血超详细总结）(Java团长)](https://www.cnblogs.com/java1024/p/11950129.html)