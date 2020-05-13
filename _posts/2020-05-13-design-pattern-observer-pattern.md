---
layout: post
title: 设计模式之观察者模式(Observer Pattern)
categories: DesignPattern
description: 观察者模介绍
keywords: DesignPattern, 设计模式
---

观察者模式定义一系列对象之间的一对多关系，当一个对象改变、更新状态时，依赖它的都会收到通知改变或者更新。它是一种**行为模式**。

举个例子，在街边卖小吃的小商小贩可能需要一个通风报信的人，城管来了就赶快收摊。再比如，上自习课玩手机的同学，就需要有人提醒他们老师来了，及时把手机收起来免得被没收。这些都是生活中观察者模式的例子。

那个通风报信的人叫做被观察者，在java的具体实现中我们称为主题(subject)，小摊小贩和玩手机的同学叫做观察者。被观察者需要通知观察者。

# 1. 观察者模式实现

![观察者模式示意图](/images/posts/designpattern/observer_pattern_diagram.PNG)

其中，Subject类是主题，它把所有对观察者对象的引用文件存在了一个集合里，每个主题都可以有任何数量的观察者。抽象主题提供了一个接口，可以增加和删除观察者对象；Observer类是抽象观察者，为所有的具体观察者定义一个接口，在得到主题的通知时更新自己；ConcreteSubject类是具体主题，将有关状态存入具体观察者对象，在具体主题内部状态改变时，给所有登记过的观察者发出通知；ConcreteObserver是具体观察者，实现抽象观察者角色所要求的更新接口，以便使本身的状态与主题的状态相协同。

## 1.1 创建Subject接口

Subject其实就是被观察者，`notifyObservers()`负责通知观察者做出改变，也就是俗话说的通风报信。

```java
public interface Subject {
	
	//注册观察者
	void registerObserver(Observer observe);
	//解除绑定观察者
	void unRegisterObserver(Observer observe);
	//更新观察者的数据
	void notifyObservers();

}
```

## 1.2 创建Observer接口

```java
public interface Observer {
	//更新数据
    void update(String action);
}
```

## 1.3 创建Subject的具体实现类

```java
public class WatchDog implements Subject{

	private ArrayList arrayObserve;

	   private String action;

	   public WatchDog() {
	       arrayObserve = new ArrayList();
	   }
	 @Override
	   public void registerObserver(Observer observer) {
	       //将观察者添加到列表中
	       arrayObserve.add(observer);
	   }

	   @Override
	   public void unRegisterObserver(Observer observer) {
	       int i = arrayObserve.indexOf(observer);
	       if (i >= 0) {
	           //将观察者从列表中解除
	           arrayObserve.remove(i);
	       }

	   }
	   //通知所以观察者数据更新了
	   @Override
	   public void notifyObservers() {

	       for (int i = 0; i < arrayObserve.size(); i++) {
	           Observer o = (Observer) arrayObserve.get(i);
	           o.update("收起手机!!");
	       }
	   }

}
```

## 1.4 创建Observer的实现类

```java
public class StudentObserver implements Observer{

    //将主题当成观察者的属性
    private Subject watchDog;

    public StudentObserver(Subject watchdog) {
        this.watchDog = watchdog;
        //注册该观察者
        watchdog.registerObserver(this);
    }

    @Override
    public void update(String action) {
        System.out.println(action);
    }
}
```

## 1.5 测试

```java
public class ObserverDemo {
	public static void main(String[] args) {

        Subject watchdog = new WatchDog();
        StudentObserver student1 = new StudentObserver(watchdog);
        StudentObserver student2 = new StudentObserver(watchdog);
        System.out.println("老师来了，同学们快收起手机");
        watchdog.notifyObservers();
    }
}
```

# 2. 观察者模式分析

## 2.1 优点

- 观察者和被观察者是抽象耦合的
- 建立了一套触发机制

## 2.2 缺点

- 如果一个被观察者对象有很多的直接和间接的观察者的话，将所有的观察者都通知到会花费很多时间
- 如果观察者和观察目标间有循环依赖，可能导致系统崩溃
- 没有相应的机制让观察者知道所观察的目标对象是怎么发生变化的

## 2.3 使用场景

- 关联行为场景
- 事件多级触发场景
- 跨系统的消息变换场景，如消息队列的处理机制



**参考文章**

[观察者模式 (eirunye)](https://www.jianshu.com/p/4be5de46ec63)

[简说设计模式——观察者模式 (JAdam)](https://www.cnblogs.com/adamjwh/p/10913660.html)