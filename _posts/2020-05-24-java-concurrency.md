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

## 1.1 并发编程的三个概念

1. **原子性**：即一个操作或者多个操作，要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。

   举个例子，假如为一个32位的变量赋值过程不具备原子性的话，会发生什么后果？

   ```java
   i = 9;
   ```

   假若一个线程执行到这个语句时，我暂且假设为一个32位的变量赋值包括两个过程：为低16位赋值，为高16位赋值。那么就可能发生一种情况：当将低16位数值写入之后，突然被中断，而此时又有一个线程去读取i的值，那么读取到的就是错误的数据。

2. **可见性**：当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

   举个例子：

   ```java
   //线程1执行的代码
   int i = 0;
   i = 10;
   //线程2执行的代码
   j = i;
   ```

   假若执行线程1的是CPU1，执行线程2的是CPU2。由上面的分析可知，当线程1执行 `i =10`这句时，会先把`i`的初始值加载到CPU1的高速缓存中，然后赋值为10，那么在CPU1的高速缓存当中`i`的值变为10了，却没有立即写入到主存当中。此时线程2执行` j = i`，它会先去主存读取i的值并加载到CPU2的缓存当中，注意此时内存当中`i`的值还是0，那么就会使得`j`的值为0，而不是10。

   这就是可见性问题，线程1对变量`i`修改了之后，线程2没有立即看到线程1修改的值。

3. **有序性**：即程序执行的顺序按照代码的先后顺序执行。

   举个简单的例子，看下面这段代码：

   ```java
   int i = 0;              
   boolean flag = false;
   i = 1;                //语句1  
   flag = true;          //语句2
   ```

   上面代码定义了一个`int`型变量，定义了一个`boolean`类型变量，然后分别对两个变量进行赋值操作。从代码顺序上看，语句1是在语句2前面的，那么JVM在真正执行这段代码的时候会保证语句1一定会在语句2前面执行吗？不一定，为什么呢？这里可能会发生**指令重排序(Instruction Reorder)**。

   下面解释一下什么是指令重排序，一般来说，处理器为了提高程序运行效率，可能会对输入代码进行优化，它不保证程序中各个语句的执行先后顺序同代码中的顺序一致，但是它会保证程序最终执行结果和代码顺序执行的结果是一致的。

   比如上面的代码中，语句1和语句2谁先执行对最终的程序结果并没有影响，那么就有可能在执行过程中，语句2先执行而语句1后执行。

   但是要注意，虽然处理器会对指令进行重排序，但是它会保证程序最终结果会和代码顺序执行结果相同。

   虽然重排序不会影响单个线程内程序执行的结果，但是会对多线程有影响。

   举个例子：

   ```java
   //线程1:
   context = loadContext();   //语句1
   inited = true;             //语句2
    
   //线程2:
   while(!inited ){
     sleep()
   }
   doSomethingwithconfig(context);
   ```

   上面代码中，由于语句1和语句2没有数据依赖性，因此可能会被重排序。假如发生了重排序，在线程1执行过程中先执行语句2，而此时线程2会以为初始化工作已经完成，那么就会跳出while循环，去执行`doSomethingwithconfig(context)`方法，而此时context并没有被初始化，就会导致程序出错。

   也就是说，要想并发程序正确地执行，必须要保证**原子性**、**可见性**以及**有序性**。只要有一个没有被保证，就有可能会导致程序运行不正确。

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

## 2.5 线程的返回值与`Callable`

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

# 4. 线程的基本方法

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
public static native void sleep(long millis) throws InterruptedException
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

`yield()`的作用是：暂停当前正在执行的线程对象，并执行其他线程。具体是让**运行中**的线程回到**就绪态**，以允许具有相同优先级的其他线程获得运行机会。因此，使用`yield()`的目的是让相同优先级的线程之间能适当的轮转执行。但是，实际中无法保证`yield()`达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。

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

 在一个线程中，我们可以调用另一个线程的`join()`方法，作用是当前线程放弃执行，进入**等待**状态或者**超时等待**状态，直到调用`join()`方法的线程执行完才返回执行当前线程。

`join()`方法有三种形式：

```java
public final synchronized void join(long millis) throws InterruptedException
```

```java
public final synchronized void join(long millis, int nanos) throws InterruptedException
```

```java
public final void join() throws InterruptedException
```

使用如下：

```java
public class PrintTask implements Runnable{
    private String name;

    public PrintTask(String name){
        this.name=name;
    }

    @Override
    public void run() {
        for(int i=0;i<1000;i++){
            System.out.println(name + "在运行");
        }
    }
}
```

```java
public class JoinTest {
    public static void main(String[] args) throws InterruptedException {
        Runnable newPrintTask=new PrintTask("Thread1");
        Runnable newPrintTask1=new PrintTask("Thread2");

        Thread t1=new Thread(newPrintTask);
        Thread t2=new Thread(newPrintTask1);

        t1.start();
        /**
         * 程序在main线程中调用t1线程的join方法，则main线程放弃cpu控制权，并返回t1线程继续执行直到线程t1执行完毕
         * 所以结果是t1线程执行完后，才到主线程执行，相当于在main线程中同步t1线程，t1执行完了，main线程才有执行的机会
         */
        t1.join();
        t2.start();

    }
}
```

运行结果是Thread1全部执行完之后，Thread2才开始执行。

在A线程中调用了B线程的`join()`方法时，表示只有当B线程执行完毕时，A线程才能继续执行。注意，这里调用的`join()`方法是没有传参的，`join()`方法其实也可以传递一个参数给它的，规定等待的时间。

例如：

```java
public class JoinTest {
    public static void main(String[] args) throws InterruptedException {
        Runnable newPrintTask=new PrintTask("Thread1");
        Runnable newPrintTask1=new PrintTask("Thread2");

        Thread t1=new Thread(newPrintTask);
        Thread t2=new Thread(newPrintTask1);

        t1.start();
        t1.join(10);
        t2.start();

    }
}
```

结果是前10ms执行的是Thread1，之后是Thread1和Thread2交替执行。

所以，`join()`方法中如果传入参数，则表示这样的意思：如果A线程中掉用B线程的`join(10)`，则表示A线程会等待B线程执行10毫秒，10毫秒过后，A、B线程并行执行。需要注意的是，JDK规定，`join(0)`的意思不是A线程等待B线程0秒，而是A线程等待B线程无限时间，直到B线程执行完毕，即`join(0)`等价于`join()`。

我们只有在一个线程需要等待另一个线程的任务执行完的时候才需要使用`join()`方法。比如，线程A需要等待线程B打开文件之后才能执行。

# 5. 资源共享

在多线程程序里，我们需要考虑多个线程共享资源引发的问题。在生活中，就像两辆车抢一个车位，两个人同时进一个门一样。

## 5.1 共享资源访问问题

我们先看一个例子，我们创建两个对象，一个对象负责制造偶数，而另一个对象负责使用这些偶数。

```java
public abstract class IntGenerator {
    private volatile boolean canceled=false;
    public abstract int next();

    //改变标识位
    public void cancel(){
        canceled=true;
    }
    //查看是否被取消
    public boolean isCanceled(){
        return canceled;
    }
}
```

```java
public class EvenChecker implements Runnable{
    private IntGenerator generator;
    private final int id;

    public EvenChecker(IntGenerator g, int ident){
        generator=g;
        id=ident;
    }

    @Override
    public void run() {
        while (!generator.isCanceled()){
            int val=generator.next();
            if(val%2!=0){
                System.out.println(val+" not even! ");
                generator.cancel();
            }
        }
    }

    public static void test(IntGenerator gp, int count){
        System.out.println("Press control-c to exit");
        ExecutorService exec= Executors.newCachedThreadPool();
        for(int i=0;i<count;i++){
            exec.execute(new EvenChecker(gp, i));
        }
        exec.shutdown();
    }

    public static void test(IntGenerator gp){
        test(gp, 10);
    }
}
```

从`EvenChecker`类中我们看到，`EvenChecker`的运行取决于`IntGenerator`，在`run()`方法中要检查`IntGenerator`的标识位。通过这种方法，线程的共享资源`IntGenerator`可以监控标识位信号来决定是否终止，消除了竞争条件(race condition)，也就是多个线程竞争同一个资源时产生的时间问题或者序列冲突。一般情况下，一个任务不能取决于另一个任务，因为任务的终止顺序是不确定的。我们这个例子通过一个任务取决于一个非任务对象来消除潜在的竞争条件。

测试一下：

```java
public class EvenGenerator extends IntGenerator{
    private int currentEvenValue=0;

    @Override
    public int next() {
        ++currentEvenValue; //危险操作
        ++currentEvenValue;
        return currentEvenValue;
    }

    public static void main(String[] args) {
        EvenChecker.test(new EvenGenerator());
    }
}
/**结果：
 * Press control-c to exit
 * 385 not even! 
 * 387 not even! 
 * 383 not even! 
 */
```

从`EvenGenerator`类的`next()`方法可以看出，我们通过累加两次`currentEvenValue`值来制造偶数，如果每个线程都能把`next()`方法执行完而不被打断，而之后的线程在一个线程执行完`next()`方法之后再使用`currentEvenValue`的话，会一直产生偶数。可从结果看到，线程并不是这样执行的。

实际上，在一个线程执行完第一个`++currentEvenValue`而没有执行第二个`++currentEvenValue`操作时，有一个线程调用了`next()`方法，这时`currentEvenValue`的值就会产生错误。

## 5.2 解决共享资源竞争

我们可以通过加锁来解决这个问题。一个线程在使用共享资源的时候需要锁住资源，其它线程就需要等待这个线程解锁之后才能使用共享资源。我们通过一些机制来实现一次只能有一个线程使用共享资源，实现互斥(mutual exclusion)。

举个例子，公共厕所就像互斥资源，一个人在使用厕所的时候会锁上门，而其它人会先敲敲门来看看里面有没有人，如果有人的话，就在外面排队，等到里面的人出来再进去锁住门，阻止其它人进入。

Java有一种内置的方法来加锁，那就是`synchronized`关键字，也就是**隐式锁**。还有另一种锁，`Lock`类，也叫**显式锁**。

### 5.2.1 `synchronized`

当我们使用了`synchronized`关键字，线程会先查看锁是否可用，如果可用则获得锁，执行代码，之后再释放锁。

共享资源一般以对象的形式存在，也可能是一个文件，I/O端口等等。我们把共享资源放入一个对象，使用共享资源的方法加上`synchronized`关键字。如果一个线程在使用`synchronized`标记的方法，想使用这个方法的其它线程就会被阻塞直到锁释放。`synchronized`可以在方法上使用，也可以在代码块上使用。

------

在方法上使用`synchronized`：

```java
public synchronized void f(){}
public synchronized void g(){}
```

事实上，所有的对象都自动有一个**对象锁(object level lock)**，它与**管程(monitor)**相关，当一个线程调用了一个对象里的`synchronized`方法，这个对象会被对象锁锁住，其它线程不能调用这个对象里其它被`synchronized`标记的方法。比如，`f()`方法和`g()`方法存在于一个对象里，当一个线程调用了`f()`方法，则其它线程既不能使用`f()`方法，也不能使用`g()`方法直到锁释放。实际是，一个对象的所有`synchronized`方法共享一把锁。所以，多个线程调用不同对象的同一`synchronized`方法时，不会被阻塞。

一个线程可以多次获得对象锁。这种情况一般发生在一个线程在获得对象锁之后多次调用对象里的其它方法。JVM会跟踪对象被锁的次数，如果对象解锁，则次数清零。这种运行方式就是可重入锁。

不同于每个对象一把的对象锁，还有一种锁，是每个类只有一把，就是**类锁(class level lock)**。类锁是`Class`对象的一部分。对象锁是用于对象实例方法，或者一个对象实例上的，类锁是用于类的静态方法或者一个类的`Class`对象上的。我们知道，类的对象实例可以有很多个，但是每个类只有一个`Class`对象，所以不同对象实例的对象锁是互不干扰的，但是每个类只有一个类锁。但是有一点必须注意的是，其实类锁只是一个概念上的东西，并不是真实存在的，它只是用来帮助我们理解锁定实例方法和静态方法的区别的。

静态方法使用如下：

```java
public static synchronized void f(){}
public static synchronized void g(){}
```

值得注意的是，类锁和对象锁是不冲突的。换句话说，如果一个线程调用了`synchronized`方法说明它获得了对象锁，其它线程不能调用同一对象的所有`synchronized`方法，而可以调用`static synchronized`标记的方法，也就是说，可以获得类锁。即便没有获得对象锁，也不影响获得类锁。

------

`synchronized`在代码块上使用：

```java
//同步代码块，锁住该类的实例对象
public void f(){
    synchronized(this){
	}
}
```

使用代码块锁住类实例对象与使用`synchronized`作用于普通方法实际是一样的，只是形式不同。

```java
//同步代码块，锁住类对象
public void f(){
    synchronized(Demo.class){
	}
}
```

而使用代码块锁住类对象与使用`synchronized`作用静态方法是一样的。

------

我们使用`synchronized`修改下5.1的代码：

```java
public class EvenGenerator extends IntGenerator{
    private int currentEvenValue=0;

    @Override
    public synchronized int next() {
        ++currentEvenValue; //危险操作
        ++currentEvenValue;
        return currentEvenValue;
    }

    public static void main(String[] args) {
        EvenChecker.test(new EvenGenerator());
    }
}
```

### 5.2.2 `Lock`类     

在Java SE5中，`java.util.concurrent`包提供了一种显式的互斥机制`Lock`。`Lock`对象必须被显式地定义出来，上锁，解锁。所以我们也称它为**显锁**。   

使用`Lock`改写5.1的代码：

```java
public class MutexEvenGenerator extends IntGenerator{
    private int currentEvenValue=0;
    private Lock lock=new ReentrantLock();

    @Override
    public int next() {
        lock.lock();
        try{
            ++currentEvenValue; //危险操作
            ++currentEvenValue;
            return currentEvenValue;
        }finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        EvenChecker.test(new MutexEvenGenerator());
    }
}
```

`Lock`实际是一个接口，定义了这些方法：

```java
public interface Lock {
    
    /*请求锁，实现时应该考虑能够检测出错误的使用，如避免死锁或者抛出未经检查的异常     */
    void lock();
    
	/*支持在请求锁的时候被中断     */
    void lockInterruptibly() throws InterruptedException;
	
    /*如果请求锁时锁可用则立即获取并返回true,否则立即返回false     */
    boolean tryLock();
  
    /*如果请求锁时锁可用则立即获取并返回true ，否则线程进入不可调度状态直到以下三种情况：
    *线程获取锁
    *其他线程中断当前线程
    *超时
    */
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
	
    /*释放锁     */
    void unlock();
    
	/*返回一个新的绑定到当前lock实例的Condition对象实例     */
    Condition newCondition();
}

```

相比于隐锁`synchronized`，显锁更加灵活。代码中try-finally的形式允许我们做更多的事，不同于`synchronized`遇到异常会直接抛出异常，显锁可以通过try-finally的灵活性让我们处理异常，维护系统在合适的状态。

隐锁和显锁的区别：

|          |                隐锁(synchronized)                |                          显锁(Lock)                          |
| :------: | :----------------------------------------------: | :----------------------------------------------------------: |
| 实现方式 | Java中的关键字，是由JVM来维护的。是JVM层面的锁。 | JDK5以后才出现的具体的类。使用lock是调用对应的API，是API层面的锁。 |
| 使用方式 |     系统维护，自动上锁和释放，不易出现死锁。     |               手动上锁和释放锁，容易出现死锁。               |
| 等待中断 |    不可中断的。除非抛出异常或者正常运行完成。    |                          可以中断。                          |
|  公平性  |                     非公平锁                     |                两者都可以的。默认是非公平锁。                |
| 实现机制 |                  悲观锁，独占锁                  | 乐观锁，就是每次不加锁，假设没有冲突地去执行，如果冲突了就重试，直到成功 |

接下来，我们介绍`Lock`的具体实现类。

### 5.2.3 可重入锁(ReentrantLock)

可重入锁简单理解就是对同一个线程而言，它可以重复的获取锁。例如这个线程可以连续获取两次锁，但是释放锁的次数也一定要是两次。我们之前提到`synchronized`也是一种可重入锁。我们改写5.1的代码使用的就是可重入锁。

**线程中断响应**

如果线程阻塞于`synchronized`，那么要么获取到锁，继续执行，要么一直等待。重入锁提供了另一种可能，就是中断线程。如果某一线程A正在执行锁中的代码，另一线程B正在等待获取该锁，可能由于等待时间过长，线程B不想等待了，想先处理其他事情，我们可以让它中断自己或者在别的线程中中断它，这种就是可中断锁。

**有限时间的等待锁**

顾名思义，简单理解就是在指定的时间内如果拿不到锁，则不再等待锁。当持有锁的线程出问题导致长时间持有锁的时候，不可能让其他线程永远等待其释放锁。

**公平锁与非公平锁**

当一个线程释放锁时，其他等待的线程则有机会获取锁，如果是公平锁，则分先来后到的获取锁，如果是非公平锁则谁抢到锁算谁的，这就相当于排队买东西和不排队买东西是一个道理。`synchronized`是非公平锁。

可重入锁`ReentrantLock`是可以设置公平性的：

```java
// 通过传入一个布尔值来设置公平锁，为true则是公平锁，false则为非公平锁
public ReentrantLock(boolean fair) {
	sync = fair ? new FairSync() : new NonfairSync();
}
```

构建一个公平锁需要维护一个有序队列，如果实际需求用不到公平锁则不需要使用公平锁。

### 5.2.4 读写锁(ReadWriteLock)

`synchronized`存在明显的一个性能问题就是读与读之间互斥，简言之就是，我们编程想要实现的最好效果是，可以做到读和读互不影响，读和写互斥，写和写互斥，提高读写的效率。

`java.util.concurrent`包中`ReadWriteLock`是一个接口，主要有两个方法：

```java
public interface ReadWriteLock {
    /**
     * Returns the lock used for reading.
     *
     * @return the lock used for reading
     */
    Lock readLock();

    /**
     * Returns the lock used for writing.
     *
     * @return the lock used for writing
     */
    Lock writeLock();
}
```

`ReadWriteLock`管理一组锁，一个是只读的锁，一个是写锁。Java并发库中`ReetrantReadWriteLock`实现了`ReadWriteLock`接口并添加了可重入的特性。

**公平锁与非公平锁**

支持公平与非公平（默认）的锁获取方式，吞吐量非公平优先于公平。

**锁降级**

要实现一个读写锁，需要考虑很多细节，其中之一就是锁升级和锁降级的问题。什么是升级和降级呢？

在不允许中间写入的情况下，写入锁可以降级为读锁吗？读锁是否可以升级为写锁，优先于其他等待的读取或写入操作？简言之就是说，锁降级：从写锁变成读锁；锁升级：从读锁变成写锁。`ReentrantReadWriteLock`支持锁降级但不支持锁升级。

**读锁和读锁的使用**

```java
public class ReadWriteLockTest {

    public static void get(Thread thread) {
        ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
        lock.readLock().lock();
        System.out.println(thread.getName() + ":start time:" + System.currentTimeMillis());
        for (int i = 0; i < 5; i++) {
            try {
                Thread.sleep(20);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(thread.getName() + ":正在进行读操作……");
        }
        System.out.println(thread.getName() + ":读操作完毕！");
        System.out.println(thread.getName() + ":end time:" + System.currentTimeMillis());
        lock.readLock().unlock();
    }

    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                get(Thread.currentThread());
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                get(Thread.currentThread());
            }
        }).start();
    }
}

```

Thread-0和Thread-1交替执行，由此可见`ReentrantReadWriteLock`读锁使用共享模式，即：同时可以有多个线程并发地读数据。

**读锁和写锁穿插使用**

```java
public class ReadWriteLockTest {
    public static ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    public static void main(String[] args) {
        //同时读、写
        ExecutorService service = Executors.newCachedThreadPool();
        service.execute(new Runnable() {
            @Override
            public void run() {
                readFile(Thread.currentThread());
            }
        });
        service.execute(new Runnable() {
            @Override
            public void run() {
                writeFile(Thread.currentThread());
            }
        });
    }

    // 读操作
    public static void readFile(Thread thread) {
        lock.readLock().lock();
        boolean readLock = lock.isWriteLocked();
        if (!readLock) {
            System.out.println("当前为读锁！");
        }
        try {
            for (int i = 0; i < 5; i++) {
                try {
                    Thread.sleep(20);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(thread.getName() + ":正在进行读操作……");
            }
            System.out.println(thread.getName() + ":读操作完毕！");
        } finally {
            System.out.println("释放读锁！");
            lock.readLock().unlock();
        }
    }

    // 写操作
    public static void writeFile(Thread thread) {
        lock.writeLock().lock();
        boolean writeLock = lock.isWriteLocked();
        if (writeLock) {
            System.out.println("当前为写锁！");
        }
        try {
            for (int i = 0; i < 5; i++) {
                try {
                    Thread.sleep(20);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(thread.getName() + ":正在进行写操作……");
            }
            System.out.println(thread.getName() + ":写操作完毕！");
        } finally {
            System.out.println("释放写锁！");
            lock.writeLock().unlock();
        }
    }
}
```

从运行结果看到，一个线程执行完成后，另一个线程才执行，所以读锁和写锁是互斥的。

**写锁和写锁的使用**

```java
public class ReadWriteLockTest {
    public static ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    public static void main(String[] args) {
        //同时写
        ExecutorService service = Executors.newCachedThreadPool();
        service.execute(new Runnable() {
            @Override
            public void run() {
                writeFile(Thread.currentThread());
            }
        });
        service.execute(new Runnable() {
            @Override
            public void run() {
                writeFile(Thread.currentThread());
            }
        });
    }

    // 写操作
    public static void writeFile(Thread thread) {
        lock.writeLock().lock();
        boolean writeLock = lock.isWriteLocked();
        if (writeLock) {
            System.out.println("当前为写锁！");
        }
        try {
            for (int i = 0; i < 5; i++) {
                try {
                    Thread.sleep(20);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(thread.getName() + ":正在进行写操作……");
            }
            System.out.println(thread.getName() + ":写操作完毕！");
        } finally {
            System.out.println("释放写锁！");
            lock.writeLock().unlock();
        }
    }
}

```

从结果我们能看出，写锁之间也是互斥的。即一个线程在做写操作的时候，另一线程不能同时做写操作。

## 5.3 `volatile`





------

**参考文章**：

[《Java编程思想》(Thinking in Java)]()

[并发和并行的区别 (杰哥长得帅)](https://www.jianshu.com/p/cbf9588b2afb)

[进程与线程 (浅浅念)](https://www.cnblogs.com/qianqiannian/p/7010909.html)

[Java多线程看这一篇就足够了（吐血超详细总结）(Java团长)](https://www.cnblogs.com/java1024/p/11950129.html)

[Java线程的6种状态及切换(透彻讲解) (诚o)](https://blog.csdn.net/qq_22771739/article/details/82529874)

[廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/1252599548343744/1306580767211554)

[java 线程方法join的简单总结 (coder-lcp)](https://www.cnblogs.com/lcplcpjava/p/6896904.html)

[JAVA显式锁简介 (Cafebaby)](https://www.jianshu.com/p/75212b04dbfe)

[Java的锁—彻底理解重入锁(ReentrantLock) (kopshome)](https://blog.csdn.net/I_AM_KOP/article/details/80958856)

[【Java并发】ReadWriteLock读写锁的使用 (itbird01)](https://www.jianshu.com/p/9cd5212c8841)

[Java并发编程：volatile关键字解析 (Matrix 海 子)](https://www.cnblogs.com/dolphin0520/p/3920373.html)