---


layout: post
title: Java并发(Concurrency)之死锁(Deadlock)
categories: Java
description: 讲解Java的死锁
keywords: Java, Concurrency, 并发, 死锁, Deadlock
---

所谓死锁，是指多个进程在运行过程中因争夺资源而造成的一种僵局，当进程处于这种僵持状态时，若无外力作用，它们都将无法再向前推进。举个生活中的例子，交易的时候，买方要卖方先交货才肯交钱，而卖方要买方先交钱在交货，因为两方都占有对方需要的资源但都不肯让步，就会产生死锁的情况。



# 1. 死锁产生的原因

## 1.1 系统资源竞争

通常系统中拥有的不可剥夺资源，其数量不足以满足多个进程运行的需要，使得进程在运行过程中，会因争夺资源而陷入僵局，如磁带机、打印机等。只有对不可剥夺资源的竞争才可能产生死锁，对可剥夺资源的竞争是不会引起死锁的。

## 1.2 进程推进顺序非法

进程在运行过程中，请求和释放资源的顺序不当，也同样会导致死锁。例如，并发进程 P1、P2分别保持了资源R1、R2，而进程P1申请资源R2，进程P2申请资源R1时，两者都会因为所需资源被占用而阻塞。

Java中死锁最简单的情况是，一个线程T1持有锁L1并且申请获得锁L2，而另一个线程T2持有锁L2并且申请获得锁L1，因为默认的锁申请操作都是阻塞的，所以线程T1和T2永远被阻塞了。导致了死锁。这是最容易理解也是最简单的死锁的形式。但是实际环境中的死锁往往比这个复杂的多。可能会有多个线程形成了一个死锁的环路，比如：线程T1持有锁L1并且申请获得锁L2，而线程T2持有锁L2并且申请获得锁L3，而线程T3持有锁L3并且申请获得锁L1，这样导致了一个锁依赖的环路：T1依赖T2的锁L2，T2依赖T3的锁L3，而T3依赖T1的锁L1。从而导致了死锁。

从上面两个例子中，我们可以得出结论，产生死锁可能性的最根本原因是：**线程在获得一个锁L1的情况下再去申请另外一个锁L2，也就是锁L1想要包含了锁L2，也就是说在获得了锁L1，并且没有释放锁L1的情况下，又去申请获得锁L2，这个是产生死锁的最根本原因**。另一个原因是**默认的锁申请操作是阻塞的**。

## 1.3 死锁产生的必要条件

产生死锁必须同时满足以下四个条件，只要其中任一条件不成立，死锁就不会发生。

1. 互斥条件：进程要求对所分配的资源（如打印机）进行排他性控制，即在一段时间内某资源仅为一个进程所占有。此时若有其他进程请求该资源，则请求进程只能等待。
2. 不剥夺条件：进程所获得的资源在未使用完毕之前，不能被其他进程强行夺走，即只能由获得该资源的进程自己来释放（只能是主动释放)。
3. 请求和保持条件：进程已经保持了至少一个资源，但又提出了新的资源请求，而该资源已被其他进程占有，此时请求进程被阻塞，但对自己已获得的资源保持不放。
4. 循环等待条件：存在一种进程资源的循环等待链，链中每一个进程已获得的资源同时被链中下一个进程所请求。即存在一个处于等待状态的进程集合{Pl, P2, ..., pn}，其中Pi等 待的资源被P(i+1)占有（i=0, 1, ..., n-1)，Pn等待的资源被P0占有，如图1所示。

直观上看，循环等待条件似乎和死锁的定义一样，其实不然。按死锁定义构成等待环所要求的条件更严，它要求Pi等待的资源必须由P(i+1)来满足，而循环等待条件则无此限制。 例如，系统中有两台输出设备，P0占有一台，PK占有另一台，且K不属于集合{0, 1, ..., n}。

Pn等待一台输出设备，它可以从P0获得，也可能从PK获得。因此，虽然Pn、P0和其他 一些进程形成了循环等待圈，但PK不在圈内，若PK释放了输出设备，则可打破循环等待, 如图2-16所示。因此循环等待只是死锁的必要条件。

![死锁1](/images/posts/java/concurrency_7.jpg)![死锁2](/images/posts/java/concurrency_8.jpg)

下面再来通俗的解释一下死锁发生时的条件：

1. 互斥条件：一个资源每次只能被一个进程使用。独木桥每次只能通过一个人。
2. 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。乙不退出桥面，甲也不退出桥面。
3. 不剥夺条件: 进程已获得的资源，在未使用完之前，不能强行剥夺。甲不能强制乙退出桥面，乙也不能强制甲退出桥面。
4. 循环等待条件：若干进程之间形成一种头尾相接的循环等待资源关系。如果乙不退出桥面，甲不能通过，甲不退出桥面，乙不能通过。

# 2. 哲学家就餐问题

哲学家就餐问题是1965年由Dijkstra提出的一种线程同步的问题。

问题描述：一圆桌前坐着5位哲学家，两个人中间有一只筷子，桌子中央有面条。哲学家思考问题，当饿了的时候拿起左右两只筷子吃饭，必须拿到两只筷子才能吃饭。上述问题会产生死锁的情况，当5个哲学家都拿起自己右手边的筷子，准备拿左手边的筷子时产生死锁现象。

```java
public class Chopstick {
    private boolean taken=false;

    public synchronized void take() throws InterruptedException {
        while(taken){
            wait();
        }
        taken=true;
    }

    public synchronized void drop(){
        taken=false;
        notifyAll();
    }
}
```

```java
public class Philosopher implements Runnable{
    private Chopstick left;
    private Chopstick right;
    private final int id;
    private final int ponderFactor;
    private Random rand=new Random(47);

    private void pause() throws InterruptedException {
        if(ponderFactor==0) return;
        TimeUnit.MILLISECONDS.sleep(rand.nextInt(ponderFactor * 250));
    }

    public Philosopher(Chopstick left, Chopstick right, int ident, int ponder){
        this.left=left;
        this.right=right;
        id=ident;
        ponderFactor=ponder;
    }

    @Override
    public void run(){
        try{
            while(!Thread.interrupted()){
                System.out.println(this+" "+"thinking");
                pause();
                System.out.println(this+" "+"grabbing right");
                right.take();
                System.out.println(this+" "+"grabbing left");
                left.take();
                System.out.println(this+" "+"eating");
                pause();
                right.drop();
                left.drop();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public String toString(){
        return "Philosopher "+id;
    }
}
```

```java
public class DeadlockingDiningPhilosophers{

    public static void main(String[] args) throws Exception{
        int ponder=5;
        if(args.length>0){
            ponder=Integer.parseInt(args[0]);
        }
        int size=5;
        if(args.length>1){
            size=Integer.parseInt(args[1]);
        }
        ExecutorService exec= Executors.newCachedThreadPool();
        Chopstick[] sticks=new Chopstick[size];

        for(int i=0;i<size;i++){
            sticks[i]=new Chopstick();
        }

        for(int i=0;i<size;i++){
            exec.execute(new Philosopher(sticks[i], sticks[(i+1)%size], i, ponder));
        }

        if(args.length==3 && args[2].equals("timeout"))
            TimeUnit.SECONDS.sleep(5);
        else{
            System.out.println("Press 'Enter' to quit");
            System.in.read();
        }
        exec.shutdownNow();
    }
}
```

只有四个条件全部成立的时候才会出现死锁问题，所以我们只需要解决四个条件中的一个就能避免死锁。因为在我们的实现中哲学家是先拿右边的筷子再拿左边的筷子，那么就会出现前面提到的情况，当所有哲学家都拿了右手边的筷子就会产生死锁。

我们可以通过改变第四个条件，循环等待来解决这个问题，也就是最后一个哲学家可以先拿左边的筷子，再拿右边的，这样就解决了循环等待的问题。

```java
public class FixedDiningPhilosophers {
    public static void main(String[] args) throws Exception{
        int ponder=5;
        if(args.length>0){
            ponder=Integer.parseInt(args[0]);
        }
        int size=5;
        if(args.length>1){
            size=Integer.parseInt(args[1]);
        }
        ExecutorService exec= Executors.newCachedThreadPool();
        Chopstick[] sticks=new Chopstick[size];

        for(int i=0;i<size;i++){
            sticks[i]=new Chopstick();
        }

        for(int i=0;i<size;i++){
            if(i<(size-1))
                exec.execute(new Philosopher(sticks[i], sticks[i+1], i, ponder));
            else
                exec.execute(new Philosopher(sticks[0], sticks[i], i, ponder));
        }

        if(args.length==3 && args[2].equals("timeout"))
            TimeUnit.SECONDS.sleep(5);
        else{
            System.out.println("Press 'Enter' to quit");
            System.in.read();
        }
        exec.shutdownNow();
    }
}
```

------

**参考文章：**

[Java多线程：死锁 (平凡希)](https://www.cnblogs.com/xiaoxi/p/8311034.html)

[JAVA多线程学习--哲学家就餐问题 (qiuhuilu)](https://www.cnblogs.com/vettel/p/3438257.html)