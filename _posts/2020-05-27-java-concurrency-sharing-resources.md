---
layout: post
title: Java并发(Concurrency)之资源共享(Sharing Resources)
categories: Java
description: 讲解Java中的并发
keywords: Java, concurrency, 并发
---

单线程程序因为只

在多线程程序里，我们需要考虑多个线程共享资源引发的问题。在生活中，就像两辆车抢一个车位，两个人同时进一个门一样。

# 资源共享

## 1 共享资源访问问题

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

## 2 解决共享资源竞争

我们可以通过加锁来解决这个问题。一个线程在使用共享资源的时候需要锁住资源，其它线程就需要等待这个线程解锁之后才能使用共享资源。我们通过一些机制来实现一次只能有一个线程使用共享资源，实现互斥(mutual exclusion)。锁的运用保证了前面提到的**原子性**，因为占有锁，所以其它线程会被阻塞，从而保证不被打断。

举个例子，公共厕所就像互斥资源，一个人在使用厕所的时候会锁上门，而其它人会先敲敲门来看看里面有没有人，如果有人的话，就在外面排队，等到里面的人出来再进去锁住门，阻止其它人进入。

Java有一种内置的方法来加锁，那就是`synchronized`关键字，也就是**隐式锁**。还有另一种锁，`Lock`类，也叫**显式锁**。

### 2.1 `synchronized`

当我们使用了`synchronized`关键字，线程会先查看锁是否可用，如果可用则获得锁，执行代码，之后再释放锁。

共享资源一般以对象的形式存在，也可能是一个文件，I/O端口等等。我们把共享资源放入一个对象，使用共享资源的方法加上`synchronized`关键字。如果一个线程在使用`synchronized`标记的方法，想使用这个方法的其它线程就会被阻塞直到锁释放。`synchronized`可以在方法上使用，也可以在代码块上使用。

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

### 2.2 临界区(Critical sections)

有时，你只想阻止多线程同时进入方法的一部分，而不是整个方法。这部分你想隔离出来的代码叫做临界区(Critical sections)。我们可以使用`synchronized`创建出临界区。`synchronized`在这里用来标明是哪个对象的锁创建出的临界区，例如：

```java
synchronized(syncObject){
	// This code can be accessed
    // by only one task at a time
}
```

这也被叫做**同步块(synchronized block)**，在线程进入到同步代码块之前，需要获得`syncObject`对象的锁。如果有其它线程获得了锁，则这个线程需要等到锁释放才能进入同步块。

`synchronized`除了可以限制其它对象，还可以限制当前对象，用法是：

```java
//同步代码块，锁住该类的实例对象
synchronized(this){
}
```

使用同步块锁住类实例对象与使用`synchronized`作用于类的普通方法实际是一样的，只是形式不同。

`synchronized`也可以用来锁住类对象：

```java
//同步块，锁住类对象
synchronized(Demo.class){
}
```

使用同步块锁住类对象与使用`synchronized`作用`static`方法是一样的。

### 2.3 `Lock`类     

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

### 2.4 可重入锁(ReentrantLock)

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

### 2.5 读写锁(ReadWriteLock)

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

## 3 `volatile`

`volatile`是Java提供的一种轻量级的同步机制。Java语言包含两种内在的同步机制：同步块(或方法)和 `volatile` 变量，相比于`synchronized`(`synchronized`通常称为重量级锁)，`volatile`更轻量级，因为它不会引起线程上下文的切换和调度。但是`volatile`变量的同步性较差(有时它更简单并且开销更低)，而且其使用也更容易出错。

一旦一个共享变量(类的成员变量、类的静态成员变量)被`volatile`修饰之后，那么就具备了两层语义：

1. 保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。
2. 禁止进行指令重排序。

如果多个线程共用一个变量，那么，这个变量就需要使用`volatile`修饰，或者使用`synchronized`来保护这个变量。如果一个变量被`synchronized`的方法或者代码块保护起来，就不一定需要`volatile`了。

`synchronized`还是我们的首选。

### 3.1 `volatile`与可见性

`volatile`对可见性的支持，举个例子：

```java
//线程1
boolean stop = false;
while(!stop){
    doSomething();
}
 
//线程2
stop = true;
```

这段代码是很典型的一段代码，很多人在中断线程时可能都会采用这种标记办法。但是事实上，这段代码会完全运行正确么？即一定会将线程中断么？不一定，也许在大多数时候，这个代码能够把线程中断，但是也有可能会导致无法中断线程(虽然这个可能性很小，但是只要一旦发生这种情况就会造成死循环了)。

下面解释一下这段代码为何有可能导致无法中断线程。在前面已经解释过，每个线程在运行过程中都有自己的工作内存，那么线程1在运行的时候，会将stop变量的值拷贝一份放在自己的工作内存当中。

那么当线程2更改了`stop`变量的值之后，但是还没来得及写入主存当中，线程2转去做其他事情了，那么线程1由于不知道线程2对`stop`变量的更改，因此还会一直循环下去。

但是用`volatile`修饰之后就变得不一样了：

第一：使用`volatile`关键字会强制将修改的值立即写入主存；

第二：使用`volatile`关键字的话，当线程2进行修改时，会导致线程1的工作内存中缓存变量`stop`的缓存行无效（反映到硬件层的话，就是CPU的L1或者L2缓存中对应的缓存行无效）；

第三：由于线程1的工作内存中缓存变量`stop`的缓存行无效，所以线程1再次读取变量`stop`的值时会去主存读取。

那么在线程2修改`stop`值时(当然这里包括2个操作，修改线程2工作内存中的值，然后将修改后的值写入内存)，会使得线程1的工作内存中缓存变量`stop`的缓存行无效，然后线程1读取时，发现自己的缓存行无效，它会等待缓存行对应的主存地址被更新之后，然后去对应的主存读取最新的值。

那么线程1读取到的就是最新的正确的值。

### 3.2 `volatile`与有序性

在前面提到`volatile`关键字能禁止指令重排序，所以`volatile`能在一定程度上保证有序性。

`volatile`关键字禁止指令重排序有两层意思：

1. 当程序执行到`volatile`变量的读操作或者写操作时，在其前面的操作的更改肯定全部已经进行，且结果已经对后面的操作可见；在其后面的操作肯定还没有进行；

2. 在进行指令优化时，不能将在对`volatile`变量访问的语句放在其后面执行，也不能把`volatile`变量后面的语句放到其前面执行。

可能上面说的比较绕，举个简单的例子：

```java
//x、y为非volatile变量
//flag为volatile变量
 
x = 2;        //语句1
y = 0;        //语句2
flag = true;  //语句3
x = 4;         //语句4
y = -1;       //语句5
```

由于`flag`变量为`volatile`变量，那么在进行指令重排序的过程的时候，不会将语句3放到语句1、语句2前面，也不会讲语句3放到语句4、语句5后面。但是要注意语句1和语句2的顺序、语句4和语句5的顺序是不作任何保证的。

并且`volatile`关键字能保证，执行到语句3时，语句1和语句2必定是执行完毕了的，且语句1和语句2的执行结果对语句3、语句4、语句5是可见的。

那么我们回到前面举的一个例子：

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

 前面举这个例子的时候，提到有可能语句2会在语句1之前执行，那么久可能导致`context`还没被初始化，而线程2中就使用未初始化的`context`去进行操作，导致程序出错。

这里如果用`volatile`关键字对`inited`变量进行修饰，就不会出现这种问题了，因为当执行到语句2时，必定能保证`context`已经初始化完毕。

## 4 原子操作(Atomic operations)

### 4.1 原子操作

原子操作是指一个或者多个不可再分割的操作。这些操作的执行顺序不能被打乱，这些步骤也不可以被切割而只执行其中的一部分(不可中断性)。举个列子：

```java
//赋值是一个原子操作
int i = 1;

//非原子操作，i++是一个多步操作，而且是可以被中断的。
//i++可以被分割成3步，第一步读取i的值，第二步计算i+1；第三部将最终值赋值给i
i++;
```

在Java中，我们可以通过锁或者CAS操作来实现原子操作。锁我们之前已经介绍了，因为锁机制保证了线程完整运行完上锁的部分，所以保证了原子性，是一种原子操作。

### 4.2 CAS操作

CAS是Compare And Swap的简称，这个操作是硬件级别的操作，在硬件层面保证了操作的原子性。CAS有3个操作数，内存值V，旧的预期值A，要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。Java中的`sun.misc.Unsafe`类提供了`compareAndSwapInt`和`compareAndSwapLong`等几个方法实现CAS。

如果我们自己通过CAS编写`incrementAndGet()`，它大概长这样：

```java
public int incrementAndGet(AtomicInteger var) {
    int prev, next;
    do {
        prev = var.get();
        next = prev + 1;
    } while ( ! var.compareAndSet(prev, next));
    return next;
}
```

CAS是指，在这个操作中，如果`AtomicInteger`的当前值是`prev`，那么就更新为`next`，返回`true`。如果`AtomicInteger`的当前值不是`prev`，就什么也不干，返回`false`。通过CAS操作并配合`do ... while`循环，即使其他线程修改了`AtomicInteger`的值，最终的结果也是正确的。

### 4.3 原子类(Atomic classes)

在JDK的`java.util.concurrent.atomic`包下提供了很多基于CAS实现的原子操作类，`Atomic`类是通过无锁(lock-free)的方式实现的线程安全(thread-safe)访问：

![原子类](/images/posts/java/concurrency_6.png)

我们可以使用原子类来改写5.1的代码：

```java
public class AtomicEvenGenerator extends IntGenerator {
    private AtomicInteger currentEvenValue=new AtomicInteger(0);
    public int next(){
        return currentEvenValue.addAndGet(2);
    }

    public static void main(String[] args) {
        EvenChecker.test(new AtomicEvenGenerator());
    }
}
```

## 5 `ThreadLocal`

另一种防止线程共享资源冲突的是消除变量的共享。`ThreadLocal`机制可以为一个共享变量创造出多份存储备份，供每个线程使用。比如，有5个线程共享一个变量x，则`ThreadLocal`可以创建出5个变量x的备份，实际在每个线程操作的时候，操作的是自己本地内存中的变量。

------

**参考文章**：

[JAVA显式锁简介 (Cafebaby)](https://www.jianshu.com/p/75212b04dbfe)

[Java的锁—彻底理解重入锁(ReentrantLock) (kopshome)](https://blog.csdn.net/I_AM_KOP/article/details/80958856)

[【Java并发】ReadWriteLock读写锁的使用 (itbird01)](https://www.jianshu.com/p/9cd5212c8841)

[Java并发编程：volatile关键字解析 (Matrix 海子)](https://www.cnblogs.com/dolphin0520/p/3920373.html)

[【并发编程】Java中的原子操作 (程序员自由之路)](https://www.cnblogs.com/54chensongxia/p/11910681.html)