---


layout: post
title: Java并发(Concurrency)之终止线程(Termination)
categories: Java
description: 讲解Java如何终止线程
keywords: Java, Concurrency, 并发, 终止线程, Termination
---

终止一个线程通常意味着在线程处理任务完成之前停掉正在做的操作，也就是放弃当前的操作。

在 Java 中有 3 种方法可以终止正在运行的线程：

1. 使用`interrupt()`方法中断线程。
2. 使用标识位。
3. 使用`stop()`方法强行终止线程，但是不推荐使用这个方法，该方法已被弃用。我们就不具体介绍了。

# 1. 中断(Interruption)

中断是一种协作机制，Java里的中断都是依赖于线程的当前中断标志信号的状态来判断的。当在一个线程里改变另一个线程的中断标志信号状态时，被中断的线程不一定要立即停止正在做的事情。相反，中断是礼貌地请求另一个线程在它愿意并且方便的时候停止它正在做的事情。

## 1.1 中断相关方法

|                 方法                  |                             说明                             |      |
| :-----------------------------------: | :----------------------------------------------------------: | ---- |
| `public static boolean interrupted()` | 测试**当前线程**是否已经中断。线程的中断状态由该方法清除。 换句话说，如果连续两次调用该方法，则第二次调用将返回false。 (在第一次调用已清除了其中断状态之后，且第二次调用检验完中断状态前，当前线程再次中断的情况除外) |      |
|   `public boolean isInterrupted()`    | 测试**指定线程**是否已经中断。线程的中断状态不受该方法的影响。 |      |
|       `public void interrupt()`       |  中断线程，本质上是改变了调用者代表的线程中的中断标志信号。  |      |

`interrupt`方法是唯一能将中断状态设置为`true`的方法。静态方法`interrupted`会将当前线程的中断状态清除，但这个方法的命名极不直观，很容易造成误解，需要特别注意。

## 1.2 中断处理时机

作为一种协作机制，不会强求被中断线程一定要在某个点进行处理。实际上，被中断线程只需在合适的时候处理即可，如果没有合适的时间点，甚至可以不处理，这时候在任务处理层面，就跟没有调用中断方法一样。“合适的时候”与线程正在处理的业务逻辑紧密相关，例如，每次迭代的时候，进入一个可能阻塞且无法中断的方法之前等，但多半不会出现在某个临界区更新另一个对象状态的时候，因为这可能会导致对象处于不一致状态。

处理时机决定着程序的效率与中断响应的灵敏性。频繁的检查中断状态可能会使程序执行效率下降，相反，检查的较少可能使中断请求得不到及时响应。如果发出中断请求之后，被中断的线程继续执行一段时间不会给系统带来灾难，那么就可以将中断处理放到方便检查中断，同时又能从一定程度上保证响应灵敏度的地方。当程序的性能指标比较关键时，可能需要建立一个测试模型来分析最佳的中断检测点，以平衡性能和响应灵敏性。

## 1.3 中断处理方式

一般说来，当可能阻塞的方法声明中有抛出`InterruptedException`则暗示该方法是可中断的，如BlockingQueue#put、BlockingQueue#take、Object#wait、Thread#sleep等，如果程序捕获到这些可中断的阻塞方法抛出的`InterruptedException`或检测到中断后，这些中断信息该如何处理？一般有以下两个通用原则：

如果遇到的是可中断的阻塞方法抛出`InterruptedException`，可以继续向方法调用栈的上层抛出该异常，如果是检测到中断，则可清除中断状态并抛出`InterruptedException`，使当前方法也成为一个可中断的方法。
若有时候不太方便在方法上抛出`InterruptedException`，比如要实现的某个接口中的方法签名上没有`throws InterruptedException`，这时就可以捕获可中断方法的`InterruptedException`并通过`Thread.currentThread.interrupt()`来重新设置中断状态。如果是检测并清除了中断状态，亦是如此。
一般的代码中，尤其是作为一个基础类库时，绝不应当吞掉中断，即捕获到`InterruptedException`后在catch里什么也不做，清除中断状态后又不重设中断状态也不抛出`InterruptedException`等。因为吞掉中断状态会导致方法调用栈的上层得不到这些信息。

当然，凡事总有例外的时候，当你完全清楚自己的方法会被谁调用，而调用者也不会因为中断被吞掉了而遇到麻烦，就可以这么做。

总得来说，就是要让方法调用栈的上层获知中断的发生。假设你写了一个类库，类库里有个方法amethod，在amethod中检测并清除了中断状态，而没有抛出`InterruptedException`，作为amethod的用户来说，他并不知道里面的细节，如果用户在调用amethod后也要使用中断来做些事情，那么在调用amethod之后他将永远也检测不到中断了，因为中断信息已经被amethod清除掉了。如果作为用户，遇到这样有问题的类库，又不能修改代码，那该怎么处理？只好在自己的类里设置一个自己的中断状态，在调用`interrupt`方法的时候，同时设置该状态，这实在是无路可走时才使用的方法。

## 1.4 中断响应

程序里发现中断后该怎么响应？这就得视实际情况而定了。有些程序可能一检测到中断就立马将线程终止，有些可能是退出当前执行的任务，继续执行下一个任务……作为一种协作机制，这要与中断方协商好，当调用interrupt会发生些什么都是事先知道的，如做一些事务回滚操作，一些清理工作，一些补偿操作等。若不确定调用某个线程的interrupt后该线程会做出什么样的响应，那就不应当中断该线程。

## 1.5 不同状态的线程对中断的响应

### 1.5.1 NEW和TERMINATED

在新生态时，线程只是被创建出来，还未启动，所以调用中断不会产生任何影响。而在终止态时，线程已经死亡，所以中断也不会起作用。

测试NEW状态：

```java
public class TestInterruptionDemo {
    public static void main(String[] args) {
        Thread testThread=new TestThread();

        System.out.println(testThread.getState());
        System.out.println(testThread.isInterrupted());
        testThread.interrupt();
        System.out.println(testThread.isInterrupted());
    }
}
```

测试TERMINATED状态：

```java
public class TestInterruptionDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread testThread=new TestThread();
        testThread.start();

        testThread.join();
        System.out.println(testThread.getState());
        System.out.println(testThread.isInterrupted());
        testThread.interrupt();
        System.out.println(testThread.isInterrupted());
    }
}
```

### 1.5.2 RUNNABLE

如果线程处于运行状态，那么该线程的状态就是RUNNABLE，但是不一定所有处于RUNNABLE状态的线程都能获得CPU运行，在某个时间段，只能由一个线程占用CPU，那么其余的线程虽然状态是RUNNABLE，但是都没有处于运行状态。而我们处于RUNNABLE状态的线程在遭遇中断操作的时候只会设置该线程的中断标志位，并不会让线程实际中断，想要发现本线程已经被要求中断了则需要用程序去判断。

测试：

```java
public class TestThread extends Thread{
    @Override
    public void run() {
        while (true){
            // System.out.println("这是测试线程");
        }
    }
}
```

```java
public class TestInterruptionDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread testThread=new TestThread();
        testThread.start();

        System.out.println(testThread.getState());
        System.out.println(testThread.isInterrupted());
        testThread.interrupt();
        System.out.println(testThread.isInterrupted());
        System.out.println(testThread.getState());
    }
}
/*结果：
 RUNNABLE
 false
 true
 RUNNABLE
 */
```

可以看到在我们启动线程之后，线程状态变为RUNNABLE，中断之后输出中断标志，显然中断位已经被标记，但是当我们再次输出线程状态的时候发现，线程仍然处于RUNNABLE状态。很显然，处于RUNNBALE状态下的线程即便遇到中断操作，会设置中断标志位但不一定会立刻终止线程。

### 1.5.3 BLOCKED

```java
public class Task implements Runnable{

    public synchronized void doSomething(){
        while(true){

        }
    }

    @Override
    public void run(){
        doSomething();
    }
}
```

```java
public class TestInterruptionDemo {
    public static void main(String[] args) throws InterruptedException {
        Runnable task=new Task();

        Thread testThread1=new Thread(task);
        testThread1.start();

        Thread testThread2=new Thread(task);
        testThread2.start();

        Thread.sleep(1000);
        System.out.println("testThread1: "+testThread1.getState());
        System.out.println("testThread2: "+testThread2.getState());
        testThread2.interrupt();
        System.out.println(testThread2.isInterrupted());
        System.out.println("testThread2: "+testThread2.getState());
    }
}

/** result：
 testThread1: RUNNABLE
 testThread2: BLOCKED
 true
 testThread2: BLOCKED
 */
```

# 2. 使用标识位

在 run() 方法执行完毕后，该线程就终止了。但是在某些特殊的情况下，run() 方法会被一直执行；比如在服务端程序中可能会使用 `while(true) { ... }` 这样的循环结构来不断的接收来自客户端的请求。此时就可以用修改标志位的方式来结束 run() 方法。

```java
public class ServerThread extends Thread {
    //volatile修饰符用来保证其它线程读取的总是该变量的最新的值
    public volatile boolean exit = false; 

    @Override
    public void run() {
        ServerSocket serverSocket = new ServerSocket(8080);
        while(!exit){
            serverSocket.accept(); //阻塞等待客户端消息
            ...
        }
    }
    
    public static void main(String[] args) {
        ServerThread t = new ServerThread();
        t.start();
        ...
        t.exit = true; //修改标志位，退出线程
    }
}
```

------

**参考文章:**

[Java线程的中断 (JeansPocket)](https://blog.csdn.net/rr123rrr/article/details/77897294)

[并发基础（八） java线程的中断机制 (潜龙在渊)](https://www.cnblogs.com/jinggod/p/8486096.html)

[Java并发之线程中断 (Single_Yam)](https://www.cnblogs.com/yangming1996/p/7612653.html)