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

## 2.4 使用`Executors`

Java SE5的`java.util.concurrent`包提供了`Executors`简化线程的启动，我们就不需要创建一个`Thread`对象去开启线程，而是通过`Executors`来帮助我们执行任务。例如：

```java
public class CachedThreadPool {
    public static void main(String[] args) {
        //CachedThreadPool需要多少线程就会创建多少线程，当旧线程执行完任务会重新使用旧线程
        ExecutorService exec= Executors.newCachedThreadPool();
        for(int i=0;i<5;i++){
            exec.execute(new LiftOff());
        }
        //阻止新任务执行
        exec.shutdown();
    }
}
```

`ExecutorService`知道如何去执行任务。一般情况下，一个`Executor`就足够管理所有的任务了。我们还可以把`newCachedThreadPool()`替换为`newFixedThreadPool(5)`，它能规定使用多少个线程，我们只需要一次性把所有线程创建出来，不用担心有更多的线程创建从而导致浪费时间。

## 2.5 线程的返回值 

不管是继承`Thread`类还是实现`Runnable`接口的方法，都没有返回值。而如果需要线程完成任务时返回值，我们需要实现`Callable`接口，而不是`Runnable`。

`Callable`接口是在Java SE5中引入的，通过`call()`方法和泛型来返回特定类型的值，而不是之前介绍的`run()`方法，而且必须配合`ExecutorService`的`submit()`方法来调用。例如：

```java
public class TaskWithResult implements Callable<String> {
    private int id;

    public TaskWithResult(int id){
        this.id=id;
    }
    
    @Override
    public String call() throws Exception {
        return "result of TashWithResult "+id;
    }
}
```

```java
public class CallableDemo {
    public static void main(String[] args) {
        ExecutorService exec= Executors.newCachedThreadPool();
        ArrayList<Future<String>> results=new ArrayList<>();
        for(int i=0;i<10;i++)
            results.add(exec.submit(new TaskWithResult(i)));
        for(Future<String> fs:results){
            try {
                System.out.println(fs.get());
            }catch (InterruptedException | ExecutionException e){
                System.out.println(e);
                return;
            }finally {
                exec.shutdown();
            }
        }
    }
}
```

`submit()`方法执行完之后返回一个`Future`对象，在我们这个例子中是`Future<String>`对象。我们还可以调用`Future`对象的`isDone()`方法查看线程是否完成。如果任务完成，可以调用`get()`方法获得执行结果，如果任务没有完成而调用了`get()`方法，`get()`方法会阻塞直到有返回结果。

# 3. Java线程的状态

Java中的线程有六种状态，分别是**创建(New)**，**运行(Runnable)**，**阻塞(Blocked)**，**等待(Waiting)**，**超时等待(Timed Waiting)**和**终止(Terminated)**。

- 创建(New)：新创建出一个线程，还没有调用`start()`方法，只作为一个对象存在

- 运行(Runnable)：Java线程中将就绪(Ready)和运行中(Running)两种状态笼统的称为“运行”。调用`start()`后线程就会进入就绪状态。该状态的线程位于可运行线程池中，等待被线程调度器选中，获取CPU的使用权，此时处于就绪状态(Ready)。就绪状态的线程在获得CPU时间片后变为运行中状态(Running)。

  ![运行态](/images/posts/java/concurrency_3.png)

- 阻塞(Blocked)：当一个处于就绪态的线程不能立刻执行它的任务，而它需要暂时等待其它任务执行完毕。

- 等待(Waiting)：当前线程在执行的任务需要其它任务优先执行。处于这种状态的线程不会被分配CPU执行时间，它们要等待被显式地唤醒，否则会处于无限期等待的状态。

- 超时等待(Timed Waiting)：处于这种状态的线程不会被分配CPU执行时间，不过无须无限期等待被其他线程显示地唤醒，在达到一定时间后它们会自动唤醒。

- 终止(Terminated)：当任务执行完或者因为错误，线程就会进入终止态。线程一旦进入终止态就不能复生。在一个终止的线程上调用`start()`方法，会抛出`java.lang.IllegalThreadStateException`异常。

  线程终止的原因有：

  - 线程正常终止：`run()`方法执行到`return`语句返回；
  - 线程意外终止：`run()`方法因为未捕获的异常导致线程终止；
  - 对某个线程的`Thread`实例调用`stop()`方法强制终止（强烈不推荐使用）。

各个状态的关系如下：

![状态转换](/images/posts/java/concurrency_5.png)

# 4. 线程的方法

## 4.1 线程的优先级

线程的优先级代表了不同线程对调度器的重要程度不同。尽管CPU运行线程的顺序是不定的，但是它更倾向于挑选高优先级的等待线程上CPU。这不意味着低优先级的线程就不会运行，而是不如高优先级的线程运行的频繁。

Java中的线程优先级的范围是1～10，默认的优先级是5。10极最高。我们可以通过`getPriority()`来获得线程的优先级，通过`setPriority()`更改线程优先级。示例：

```java
public class SimplePriorities implements Runnable{
    private int countDown=5;
    private volatile double d;
    private int priority;
    private int id;

    public SimplePriorities(int priority, int id){
        this.priority=priority;
        this.id=id;
    }

    public void run(){
        Thread.currentThread().setPriority(priority);
        while (true){
            for(int i=1;i<100000;i++){
                d += (Math.PI + Math.E)/ (double) i;
                if(i%1000==0)
                    Thread.yield();
            }
            System.out.println("线程的id是： "+id+" 优先级是： "+priority);
            if(--countDown==0) return;
        }
    }

    public static void main(String[] args) {
        ExecutorService exec= Executors.newCachedThreadPool();
        for(int i=0;i<5;i++){
            exec.execute(new SimplePriorities(Thread.MIN_PRIORITY, i));
        }
        exec.execute(new SimplePriorities(Thread.MAX_PRIORITY, 5));
        exec.shutdown();
    }
}
```

从运行后的结果可以看出，无论是是级别相同还是不同，线程调用都不会绝对按照优先级执行。

## 4.2 `Sleep()`

`sleep()`方法也就是睡眠可以让线程进入**超时等待**状态，在规定的时间之后，线程重新回到**运行态**等待调用。`sleep()`有两种形式：

```java
public static native void sleep(long millis) throws InterruptedException;
```

```java
public static void sleep(long millis, int nanos)
```

`millis`参数决定了线程睡眠的时间，以毫秒为单位，`nanos`代表额外的睡眠时间，以纳秒为单位。

使用如下：

```java
public class SleepingTask extends LiftOff{
    public void run(){
        while(countDown-->0){
            System.out.println(status());
            try {
                Thread.sleep(100);
                //TimeUnit.MILLISECONDS.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        ExecutorService exec= Executors.newCachedThreadPool();
        for(int i=0;i<5;i++){
            exec.execute(new SleepingTask());
        }
        exec.shutdown();
    }
}
```

## 4.3 `yield()`

`yield()`的作用是：暂停当前正在执行的线程对象，并执行其他线程。具体是让运行中的线程回到就绪态，以允许具有相同优先级的其他线程获得运行机会。因此，使用`yield()`的目的是让相同优先级的线程之间能适当的轮转执行。但是，实际中无法保证`yield()`达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。

我们修改下之前的例子，把`Thread.sleep(100)`修改成`Thread.yield()`：

```java
public class SleepingTask extends LiftOff{
    public void run(){
        while(countDown-->0){
            System.out.println(status());
            Thread.yield();
        }
    }

    public static void main(String[] args) {
        ExecutorService exec= Executors.newCachedThreadPool();
        for(int i=0;i<5;i++){
            exec.execute(new SleepingTask());
        }
        exec.shutdown();
    }
}
```

和之前`sleep()`方法的结果相比，我们看到有让步失败的情况。源码的注释中提到`yield()`并不是一个常用的方法，一般用来debug或者测试。

## 4.4 `join()`

 







**参考文章**：

[并发和并行的区别 (杰哥长得帅)](https://www.jianshu.com/p/cbf9588b2afb)

[进程与线程 (浅浅念)](https://www.cnblogs.com/qianqiannian/p/7010909.html)

[Java多线程看这一篇就足够了（吐血超详细总结）(Java团长)](https://www.cnblogs.com/java1024/p/11950129.html)

[Java线程的6种状态及切换(透彻讲解) (诚o)](https://blog.csdn.net/qq_22771739/article/details/82529874)

[廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/1252599548343744/1306580767211554)