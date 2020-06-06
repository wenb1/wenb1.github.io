---


layout: post
title: Java并发(Concurrency)之线程协作(Cooperation)
categories: Java
description: 讲解Java的线程协作
keywords: Java, Concurrency, 并发, 线程协作, Cooperation
---

两个线程的共享资源可以使用互斥保护起来，一次只允许一个线程访问资源。接下来，就是如何使线程之间协作，多个线程一起来解决问题。有时候，一个线程的执行依赖于其它线程。就好比炒菜，要先洗菜，切菜，最后才能炒，如果使用多线程的方式来解决这个问题，就涉及到线程间的协作。

线程间的协作也是通过互斥实现的，互斥也保证了只有一个线程可以响应信号量。这样就消除了竞争条件(race condition)。在互斥之上，我们引入一个挂起线程的方式，直到某个状态改变，挂起的线程才会恢复运行。

# 1. `wait()`

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

如果想调用`x`的`notifyAll()`方法，首先要获得`x`的对象锁。

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