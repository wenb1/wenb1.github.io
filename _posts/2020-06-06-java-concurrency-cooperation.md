---


layout: post
title: Java并发(Concurrency)之线程协作(Cooperation)
categories: Java
description: 讲解Java的线程协作
keywords: Java, Concurrency, 并发, 线程协作, Cooperation
---

两个线程的共享资源可以使用互斥保护起来，一次只允许一个线程访问资源。接下来，就是如何使线程之间协作，多个线程一起来解决问题。有时候，一个线程的执行依赖于其它线程。就好比炒菜，要先洗菜，切菜，最后才能炒，如果使用多线程的方式来解决这个问题，就涉及到线程间的协作。

线程间的协作也是通过互斥实现的，互斥也保证了只有一个线程可以响应信号量。这样就消除了竞争条件(race condition)。在互斥之上，我们引入一个挂起线程的方式，直到某个状态改变，挂起的线程才会恢复运行。

# 1. `wait()`和`notifyAll()`

`wait()`方法可以使线程等待方法之外的条件改变。一般这个条件会被另一个线程改变。不同于`sleep()`方法的是，`wait()`方法在等待期间会释放对象锁。

等待有两种形式：**忙等待(busy waiting)**和**阻塞等待(blocking)**

**忙等待**是指线程占有CPU资源，一直测试某个条件，看是否符合条件。而**阻塞等待**会挂起线程不会占用CPU资源。

我们使用`wait()`方法实现阻塞等待，挂起线程等待满足条件。只有使用`notify()`或者`notifyAll()`唤醒线程，线程才会继续工作。

通过源码，我们能看到，`wait()`有两种形式：

```java
public final void wait() throws InterruptedException
```

```java
public final void wait(long timeout, int nanos) throws InterruptedException
```

第一种无参数的`wait()`会无限等待，直到有`notify()`或者`notifyAll()`方法唤醒。

第二种`wait()`和`sleep()`类似，都是等待规定时间后自动唤醒，但是不同于`sleep()`的是，在超时之前，线程也可以被唤醒，不一定要等到超时。

值得注意的是，`wait()`，`notify()`和`notifyAll()`是`Object`类的方法而不是`Thread`类中的。似乎很奇怪，这些明明是线程并发使用到的方法，为什么会定义在`Object`类中呢？因为这些方法需要操作对象锁，而锁存在于对象中。也正因为如此，`wait()`方法会放在`synchronized`方法中，不管这个类是继承`Thread`类还是实现`Runnable`接口。实际上，我们也只能在`synchronized`方法或者代码块中调用`wait()`，`notify()`和`notifyAll()`方法，因为涉及到对象锁的问题。如果在非`synchronized`方法中调用的话，会报`IllegalMonitorStateException`。调用`wait()`，`notify()`和`notifyAll()`方法的前提是要获得对象锁。

例如：

```java
synchronized(x){
	x.notifyAll();
}
```

如果想调用`x`的`notifyAll()`方法，首先要获得`x`的对象锁。调用`notify()`或`notifyAll()`后，当前线程不会马上释放该对象锁，要等到程序退出同步块后，当前线程才会释放锁。

举个例子：

我们有两个线程，一个给车打蜡，一个负责给蜡抛光。抛光只能在打蜡之后做，而打一层新的蜡只有在原来的蜡抛光过之后才行。实现如下：

```java
public class Car {
    private boolean waxOn=false; // 汽车打蜡的状态

    public synchronized void waxed(){
        waxOn=true;
        notifyAll();
    }

    public synchronized void buffed(){
        waxOn=false;
        notifyAll();
    }

    public synchronized void waitForWaxing() throws InterruptedException {
        while(waxOn==false){
            wait();
        }
    }

    public synchronized void waitForBuffing() throws InterruptedException {
        while(waxOn==true){
            wait();
        }
    }
}
```

```java
public class WaxOn implements Runnable{
    private Car car;

    public WaxOn(Car c){
        car=c;
    }

    @Override
    public void run() {
        try {
            while(!Thread.interrupted()){
                System.out.println("Wax On! ");
                TimeUnit.MILLISECONDS.sleep(200);
                car.waxed();
                car.waitForBuffing();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Ending Wax On task");
    }
}

```

```java
public class WaxOff implements Runnable{
    private Car car;

    public WaxOff(Car c){
        car=c;
    }
    @Override
    public void run() {
        try{
            while(!Thread.interrupted()){
                car.waitForWaxing();
                System.out.println("Wax Off! ");
                TimeUnit.MILLISECONDS.sleep(200);
                car.buffed();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Ending Wax Off task");
    }
}
```

```java
public class WaxOMatic {
    public static void main(String[] args) throws InterruptedException {
        Car car=new Car();
        ExecutorService exec= Executors.newCachedThreadPool();
        exec.execute(new WaxOff(car));
        exec.execute(new WaxOn(car));
        TimeUnit.SECONDS.sleep(5);
        exec.shutdown();
    }
}
```

在`while`循环里嵌套`wait()`方法是一种很重要的实现方式，有如下几个原因：

1. 有可能有多个线程因为同一个原因等待同一把锁，第一个被唤醒的线程可能会获得这把锁。如果是这种情况，这个线程如果没有满足适当的条件，就应该重新被挂起。

2. 有可能一个线程被唤醒，但是有另一个线程改变了一些条件，被唤醒的线程因此不能执行，它应该被重新挂起而不应该执行。
3. 也有可能是多个线程因为不同原因等待一个锁，但是`notifyAll()`会把它们都唤醒，这时候就需要检查被唤醒的线程是不是都符合条件。

我们对比下`sleep()`和`wait()`

|          | `sleep()`      | `wait()`             |
| -------- | -------------- | -------------------- |
| 所属类   | `Thread`       | `Object`             |
| 锁       | 不会释放对象锁 | 会释放对象锁         |
| 使用场景 | 适用于任何范围 | 只能在同步代码块使用 |

## 1.1 `notify`早期通知问题

`notify`通知的遗漏很容易理解，即A还没开始` wait`的时候，B已经notify了，这样，B通知是没有任何响应的，当 B退出 `synchronized`代码块后，A再开始`wait`，便会一直阻塞等待，直到被别的线程打断。比如在下面的示例代码中，就模拟出`notify`早期通知带来的问题：

```java
public class NotifyThread extends Thread{
    private String lock;

    public NotifyThread(String lock){
        this.lock=lock;
    }

    @Override
    public void run() {
        synchronized (lock){
            System.out.println(Thread.currentThread().getName() + "  进去代码块");
            System.out.println(Thread.currentThread().getName() + "  开始notify");
            lock.notify();
            System.out.println(Thread.currentThread().getName() + "  结束开始notify");
        }
    }
}
```

```java
public class WaitThread extends Thread{
    private String lock;

    public WaitThread(String lock){
        this.lock=lock;
    }

    @Override
    public void run() {
        synchronized (lock){
            System.out.println(Thread.currentThread().getName() + "  进去代码块");
            System.out.println(Thread.currentThread().getName() + "  开始wait");
            try {
                lock.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "   结束wait");
        }
    }
}
```

```java
public class EarlyNotifyDemo {

    public static void main(String[] args) throws InterruptedException {
        String lock="lock";
        Thread waitThread=new WaitThread(lock);
        Thread notifyThread=new NotifyThread(lock);

        notifyThread.start();

        Thread.sleep(100);

        waitThread.start();
    }
}
```

**解决方法：**

一般是添加一个状态标志，让`waitThread`调用`wait()`方法前先判断状态是否已经改变了没，如果通知早已发出的话，`WaitThread`就不再去`wait()`，对之前的代码进行改进：

```java
public class Flag {
    public boolean isWait=true;
}
```

```java
public class NotifyThread extends Thread{
    private String lock;
    private Flag flag;

    public NotifyThread(String lock, Flag flag){
        this.lock=lock;
        this.flag=flag;
    }

    @Override
    public void run() {
        synchronized (lock){
            System.out.println(Thread.currentThread().getName() + "  进去代码块");
            System.out.println(Thread.currentThread().getName() + "  开始notify");
            lock.notify();
            flag.isWait=false;
            System.out.println(Thread.currentThread().getName() + "  结束开始notify");
        }
    }
}
```

```java
public class WaitThread extends Thread{
    private String lock;
    private Flag flag;

    public WaitThread(String lock, Flag flag){
        this.lock=lock;
        this.flag=flag;
    }

    @Override
    public void run() {
        synchronized (lock){
            while(flag.isWait){
                System.out.println(Thread.currentThread().getName() + "  进去代码块");
                System.out.println(Thread.currentThread().getName() + "  开始wait");
                try {
                    lock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "   结束wait");
            }
        }
    }
}
```

```java
public class EarlyNotifyDemo {

    public static void main(String[] args) throws InterruptedException {
        String lock="lock";
        Flag flag=new Flag();

        Thread waitThread=new WaitThread(lock, flag);
        Thread notifyThread=new NotifyThread(lock, flag);

        notifyThread.start();

        Thread.sleep(100);

        waitThread.start();
    }
}
```

这段代码只是增加了一个`isWait`状态变量，`NotifyThread`调用`notify`方法后会对状态变量进行更新，在`WaitThread`中调用`wait`方法之前会先对状态变量进行判断，在该示例中，调用`notify`后将状态变量`isWait`改变为`false`，因此，在`WaitThread`中`while`对`isWait`判断后就不会执行`wait`方法，从而避免了Notify过早通知造成遗漏的情况。

# 2. `notify()`和`notifyAll()`

一般来说，如果多个线程在等待同一个条件，使用`notifyAll()`比`notify()`更安全。

使用`notify()`比起`notifyAll()`是一种优化。`notify()`会唤醒多个等待锁的线程中的一个，所以要确定被唤醒的是正确的线程。使用`notify()`要确保所有线程在等待同一条件，因为如果有线程在等待不同的条件，你就不能确定被唤醒的就是需要的线程。如果使用`notify()`，只有一个线程能被唤醒。

要注意的是，`notifyAll()`不是唤醒所有调用了`wait()`方法的线程，而是所有调用了`wait()`并且等待同一把锁的线程。

例如：

```java
public class Blocker {
    synchronized void waitingCall(){
        try{
            while(!Thread.interrupted()){
                wait();
                System.out.println(Thread.currentThread()+" ");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    synchronized void prod(){notify();}

    synchronized void prodAll(){notifyAll();}
}
```

```java
public class Task implements Runnable{
    static Blocker blocker=new Blocker();

    @Override
    public void run() {
        blocker.waitingCall();
    }
}
```

```java
public class Task2 implements Runnable{
    static Blocker blocker=new Blocker();

    @Override
    public void run() {
        blocker.waitingCall();
    }
}
```

```java
public class NotifyVsNotifyAll {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService exec= Executors.newCachedThreadPool();
        for(int i=0;i<5;i++){
            exec.execute(new Task());
        }
        exec.execute(new Task2());
        Timer timer=new Timer();
        timer.scheduleAtFixedRate(new TimerTask() {
            boolean prod=true;
            @Override
            public void run() {
                if(prod){
                    System.out.println("\nnotify()");
                    Task.blocker.prod();
                    prod=false;
                }else{
                    System.out.println("\nnotifyAll()");
                    Task.blocker.prodAll();
                    prod=true;
                }

            }
        }, 400,400);
        TimeUnit.SECONDS.sleep(5);
        timer.cancel();
        System.out.println("\nTimer canceled");
        TimeUnit.MILLISECONDS.sleep(500);
        System.out.println("Task2.blocker.prodAll()");
        Task2.blocker.prodAll();
        TimeUnit.MILLISECONDS.sleep(500);
        System.out.println("\nShutting down");
        exec.shutdownNow();
    }
}

```

# 3. 生产者(Producer)和消费者(Consumer)

一个餐厅有一个厨师和一个服务员。服务员必须需要等待厨师做好饭才能给客人上菜。当厨师准备好了饭菜，厨师会通知服务员，服务员才能上菜然后继续等待。这个例子就好比线程间的协作。厨师是生产者，服务员是消费者。

实现：

```java
public class Meal {
    private final int orderNum;
    public Meal(int orderNum){
        this.orderNum=orderNum;
    }
    public String toString(){
        return "Meal "+orderNum;
    }
}
```

```java
public class WaitPerson implements Runnable{
    private Restaurant restaurant;
    public WaitPerson(Restaurant r){
        restaurant=r;
    }

    public void run(){
        try{
            while(!Thread.interrupted()){
                synchronized (this){
                    while(restaurant.meal==null)
                        wait();
                }
                System.out.println("WaitPerson got "+restaurant.meal);
                synchronized (restaurant.chef){
                    restaurant.meal=null;
                    restaurant.chef.notifyAll();
                }
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class Chef implements Runnable{
    private Restaurant restaurant;
    private int count=0;

    public Chef(Restaurant r){
        restaurant=r;
    }

    public void run(){
        try{
            while (!Thread.interrupted()){
                synchronized (this){
                    while (restaurant.meal!=null)
                        wait();
                }
                if(++count==10){
                    System.out.println("Out of food, closing");
                    restaurant.exec.shutdownNow();
                }
                System.out.println("Order up!");
                synchronized (restaurant.waitPerson){
                    restaurant.meal=new Meal(count);
                    restaurant.waitPerson.notifyAll();
                }
                TimeUnit.MILLISECONDS.sleep(100);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class Restaurant {
    Meal meal;
    ExecutorService exec=Executors.newCachedThreadPool();
    WaitPerson waitPerson=new WaitPerson(this);
    Chef chef=new Chef(this);

    public Restaurant(){
        exec.execute(chef);
        exec.execute(waitPerson);
    }

    public static void main(String[] args) {
        new Restaurant();
    }
}
```

我们可以看到，因为一开始的时候`meal`是`null`值，所以`waitPerson`会调用`wait()`方法，等待被`chef`唤醒。而`Chef`因为`meal`值为`null`，之后会获得`waitPerson`的对象锁赋值给`meal`一个`Meal`对象，之后唤醒`waitPerson`。相应的，`waitPerson`会获得`Chef`的对象锁，消费`meal`对象，之后再唤醒`chef`。这样就实现了两个线程之间的配合。



------

**参考文章**

[一篇文章，让你彻底弄懂生产者--消费者问题 (你听___)](https://www.jianshu.com/p/e29632593057)