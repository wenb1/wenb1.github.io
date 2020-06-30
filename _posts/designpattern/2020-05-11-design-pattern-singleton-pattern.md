---
layout: post
title: 设计模式之单例模式(Singleton Pattern)
categories: DesignPattern
description: 单例模式介绍
keywords: DesignPattern, 设计模式
---

单例模式(Singleton pattern)，也叫单子模式，是一种常用的**创建型模式**。单例模式，顾名思义，就是只有一个实例，该类负责创建自己的对象，同时确保只有一个对象被创建，并为整个系统提供一个全局访问点 (向整个系统提供这个实例)。许多时候，整个系统只需要拥有一个的全局对象，这样有利于我们协调系统整体的行为。在Java，一般常用在工具类的实现或创建对象需要消耗资源。

# 1. 单例模式的实现

单例模式就像茴香豆的茴字一样，有多种写法。单例模式一共有5种实现方法：懒汉式，饿汉式，双重锁模式，静态内部类单例模式和枚举单例模式。

## 1.1 单例模式实现特点

- 类构造器私有
- 持有自己类型的属性
- 对外提供获取实例的静态方法

## 1.2 实现具体方法

### 1.2.1 懒汉式

```java
public class Singleton{
    private static Singleton instance;
    private Singleton(){} //私有化构造器，确保类不能被实例化
    public static Singleton getInstance(){ //对外提供获取实例的静态方法
        // 只有在需要创建实例的时候才会创建实例
        if(instance == null){
            instance = new Signleton();
        }
        return instance;
    }
}
```

懒汉式实现线程不安全，延迟加载，只有在需要的时候才会创建实例，所以看起来比较“懒”。

上面发生非线程安全的一个显著原因是，会有多个线程同时进入 `if (singleton2 == null) {…}` 语句块的情形发生。当这种这种情形发生后，该单例类就会创建出多个实例，违背单例模式的初衷。因此，传统的懒汉式单例是非线程安全的。

**延迟加载**：等到真正使用的时候才去创建实例，不用时不去主动创建

**立即加载**：在类加载初始化的时候就主动创建实例

### 1.2.2 饿汉式

```java
public class Singleton{
    private static Singleton instance = new Singleton();
    private Singleton(){} //私有化构造器，确保类不能被实例化
    public static Singleton getInstance(){ //对外提供获取实例的静态方法
        return instance;
    }
}
```

线程安全，比较常用，但因为不管需不需要都会创建出一个实例，容易产生垃圾。

这里解释下为什么线程安全，我们知道静态变量其实是优先于类被创建的，实际即使这个类在将要被实例化之前，静态变量已经被创建出来并被赋值了。所以，不管多少个线程来使用这个方法，实例都会在每个线程要创建实例之前初始化。

### 1.2.3 双重锁模式

```java
public class Singleton {  
    private volatile static Singleton singleton;  
    private Singleton (){}  
    public static Singleton getSingleton() {  
    if (singleton == null) {  
        synchronized (Singleton.class) {  
        if (singleton == null) {  
            singleton = new Singleton();  
        }  
        }  
    }  
    return singleton;  
    }  
}
```

双重检查模式，进行了两次的判断，第一次是为了避免不要的上锁，第二次是为了进行同步，避免多线程问题。由于`singleton=new Singleton()`对象的创建在JVM中可能会进行重排序，在多线程访问下存在风险，使用`volatile`修饰`signleton`实例变量有效，解决该问题。

### 1.2.4 静态内部类

```java
public class Singleton { 
    private Singleton(){
    }
      public static Singleton getInstance(){  
        return Inner.instance;  
    }  
    private static class Inner {  
        private static final Singleton instance = new Singleton();  
    }  
} 

```

只有第一次调用`getInstance()`方法时，虚拟机才加载 Inner 并初始化instance ，只有一个线程可以获得对象的初始化锁，其他线程无法进行初始化，保证对象的唯一性。目前此方式是所有单例模式中最推荐的模式，但具体还是根据项目选择。

### 1.2.5 枚举单例模式

```java
public enum Singleton  {
    INSTANCE 
 
    //doSomething 该实例支持的行为
      
    //可以省略此方法，通过Singleton.INSTANCE进行操作
    public static Singleton get Instance() {
        return Singleton.INSTANCE;
    }
}
```

默认枚举实例的创建是线程安全的，并且在任何情况下都是单例。实际上

- 枚举类隐藏了私有的构造器
- 枚举类的域 是相应类型的一个实例对象

# 2. 单例模式分析

## 2.1 单例模式优点

我们从单例模式的定义和实现，可以知道单例模式具有以下几个优点：

- 在内存中只有一个对象，节省内存空间
- 避免频繁的创建销毁对象，可以提高性能
- 避免对共享资源的多重占用，简化访问
- 为整个系统提供一个全局访问点

## 2.2 单例模式的使用场景

由于单例模式具有以上优点，并且形式上比较简单，所以是日常开发中用的比较多的一种设计模式，其核心在于为整个系统提供一个唯一的实例，其应用场景包括但不仅限于以下几种：

- 有状态的工具类对象；
- 频繁访问数据库或文件的对象；



**参考文章**：

[java单例模式](https://blog.csdn.net/czqqqqq/article/details/80451880)

[设计模式之单例模式](https://www.jianshu.com/p/3bfd916f2bb2)