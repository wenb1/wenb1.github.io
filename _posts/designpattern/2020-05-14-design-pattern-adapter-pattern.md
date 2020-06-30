---
layout: post
title: 设计模式之适配器模式(Adapter Pattern)
categories: DesignPattern
description: 适配器模式介绍
keywords: DesignPattern, 设计模式
---

适配器模式(Adapter Pattern)是将一个接口转换成客户希望的另一个接口，使接口不兼容的那些类可以一起工作。在适配器模式中，我们通过增加一个新的适配器类来解决接口不兼容的问题，使得原本没有任何关系的类可以协同工作。适配器模式是一种**结构型模式**。

比如MacBook电脑，只有Type-C接口，我想用USB接口的时候就要买个转接器。像这种情况，生活中遇见的还挺多的。

# 1. 适配器模式的实现

Java中适配器模式有两种，类适配器和对象适配器

## 1.1 类适配器实现

类适配器主要是使用继承的方式连接两个接口。

### 1.1.1 定义一个接口

```java
public interface MP4{
    void play();
}
```

### 1.1.2 定义这个接口的实现类

```java
public class ExpensiveMP4 implement MP4{
    public void play(){
            // TODO
    }
}
```

### 1.1.3 定义另外一个接口

```java
public interface Player{
      void action();
}
```

当你发现实际需要的`action()`方法和`play()`方法是一样的，我们就不需要重新再写一个`action()`方法，只需要想办法让它们适配就行了。

### 1.1.4 类适配器

```java
public class ExpensiveAdapter extends ExpensiveMP4 implement Player{
    public void action(){
        play();
    }
}
```

这样我们就能在`action()`方法中调用`play()`，不必再重复`play()`方法。

## 1.2 对象适配器实现

上面的类适配器用的是“继承”的方式去连接，这里的对象适配器用的是“组合”的方式。也就是说，我们可以在适配器里创建`ExpensiveMP4`对象，然后再调用它的`play()`方法。

### 1.2.1 初步实现方式

```java
public class PlayerAdapter implement Player{
    public ExpensiveMP4 expensiveMP4;
    
    public PlayerAdapter (ExpensiveMP4 expensiveMP4){
        this.expensiveMP4 = expensiveMP4;
    }     

    public void action(){
        if(expensiveMP4  != null){
             expensiveMP4 .play();
        }
    }

}
```

### 1.2.2 优化

```java
public class PlayerAdapter implement Player{
    public MP4 mp4;
    
    public PlayerAdapter (MP4 mp4){
        this.mp4 = mp4;
    }     

    public void action(){
        if(mp4!= null){
             mp4.play();
        }
    }

}
```

我们使用接口进一步扩大适配器的拓展性。

# 2. 适配器模式分析

## 2.1 适配器模式适用场景

- 系统需要使用现有的类，但现有的类却不兼容。
- 需要建立一个可以重复使用的类，用于一些彼此关系不大的类，并易于扩展，以便于面对将来会出现的类。
- 需要一个统一的输出接口，但是输入类型却不可预知。

## 2.2 适配器模式的优点

- 将目标类和适配者类解耦

- 增加了类的透明性和复用性，将具体的实现封装在适配者类中，对于客户端类来说是透明的，而且提高了适配者的复用性

- 灵活性和扩展性都非常好，符合开闭原则

## 2.3 适配器模式的缺点

- 对于Java、C#等不支持多重继承的语言，一次最多只能适配一个适配者类，而且目标抽象类只能为接口，不能为类，其使用有一定的局限性，不能将一个适配者类和他的子类同时适配到目标接口。

  

**参考文章**

[浅谈Java适配器模式 (键盘上的麒麟臂)](https://www.jianshu.com/p/b3a00cca10de)

[适配器模式(三种)简单使用 (Must_Do_Kaihong)](https://blog.csdn.net/u012359453/article/details/79165080)