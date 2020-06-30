---
layout: post
title: 设计模式之装饰者模式(Decorator Pattern)
categories: DesignPattern
description: 装饰者模式介绍
keywords: DesignPattern, 设计模式
---

装饰者模式(Decorator Pattern)就是动态地将责任附加到对象上。若要扩展功能，装饰者提供了比继承更有弹性的替代方案。在不改变原有对象的基础之上，将功能附加到对象上。是一种**结构型模式**。

这个设计模式让我想起了Netflix的剧*爱，死亡和机器人*中有一集叫*Zima blue*，讲述了一个叫Zima的机器人，从一个清洗泳池的简单机器人不断被人改装进化，变成了一个具有高级智能的艺术家机器人，最后它参透艺术的真谛是回归本真，卸掉了所有的智能设备，变成了从前那个简单的清洗泳池机器人，一直重复简单的工作和任务。这也是我最喜欢的一集。说了点题外话...

其实这个例子很好的说明了装饰者模式，我们实际是用一个原始的对象，通过装饰者，给原始对象加许多其它功能。

# 1. 装饰者模式实现

实现图：

![装饰者模式UML图](/images/posts/designpattern/decorator_pattern_diagram.png)

从UML图中可以看到这几个角色：

**Component（抽象构件）**：定义需要实现业务的抽象方法。

**ConcreteComponent（具体构件）**：实现Component接口，用于定义具体的构建对象，可以给它增加额外的职责（方法）。

**Decorator（抽象装饰类）**：实现Component接口，并创建Component实例对象。用于给具体构件(ConcreteComponent)增加职责。

**ConcreteDecorator（具体装饰类）**：抽象装饰类的子类，负责向构件添加新的职责。每一个具体装饰类都定义了一些新的行为，它可以调用在抽象装饰类中定义的方法，并可以增加新的方法用以扩充对象的行为。

我们实现装饰者模式的实例图如下：

![装饰者模式实例UML图](/images/posts/designpattern/decorator_pattern_example_diagram.png)

## 1.1 创建抽象构件

```java
public interface Shape {
	void draw();
}
```

## 1.2 创建具体构件

```java
public class Rectangle implements Shape{

	@Override
	public void draw() {
		System.out.println("Shape: Rectangle");
	}
	
}

public class Circle implements Shape{

	@Override
	public void draw() {
		System.out.println("Shape: Circle");
	}

}
```

## 1.3 创建抽象装饰者

```java
public abstract class ShapeDecorator implements Shape{
	protected Shape decoratorShape;
	
	public ShapeDecorator(Shape decoratorShape) {
		this.decoratorShape = decoratorShape;
	}

	@Override
	public void draw() {
		decoratorShape.draw();
	}

}
```

## 1.4 创建具体装饰者

```java
public class RedShapeDecorator extends ShapeDecorator{

	public RedShapeDecorator(Shape decoratorShape) {
		super(decoratorShape);
	}
	
	@Override
	public void draw() {
		decoratorShape.draw();
		setRedBorder(decoratorShape);
	}
	
	private void setRedBorder(Shape decoratedShape){
	    System.out.println("Border Color: Red");
	}
}
```

## 1.5 测试

```java
public class DecoratorDemo {
	public static void main(String[] args) {

	      Shape circle = new Circle();

	      Shape redCircle = new RedShapeDecorator(new Circle());

	      Shape redRectangle = new RedShapeDecorator(new Rectangle());
	      System.out.println("Circle with normal border");
	      circle.draw();

	      System.out.println("\nCircle of red border");
	      redCircle.draw();

	      System.out.println("\nRectangle of red border");
	      redRectangle.draw();
	   }
}
```

# 2. 装饰者模式分析

## 2.1 优点

- 继承的有力补充，比继承灵活，不改变原有对象的情况下给一个对象扩展功能。（继承在扩展功能是静态的，必须在编译时就确定好，而使用装饰者可以在运行时决定，装饰者也建立在继承的基础之上的）

- 通过使用不同装饰类以及这些类的排列组合，可以实现不同的效果。

- 符合开闭原则

## 2.2 缺点

- 装饰模式会导致设计出大量的ConcreteDecorator类，增加系统的复杂性。
- 动态装饰时，多层装饰时会更复杂。（使用继承来拓展功能会增加类的数量，使用装饰者模式不会像继承那样增加那么多类的数量但是会增加对象的数量，当对象的数量增加到一定的级别时，无疑会大大增加我们代码调试的难度）

## 2.3 适用场景

在不影响其他对象的情况下，以动态、透明的方式给单个对象添加职责。
 2.需要动态地给一个对象增加功能，这些功能也可以动态地被撤销。
 3.当不能采用继承的方式对系统进行扩充或者采用继承不利于系统扩展和维护时。

------

**参考文章**：

[Design Patterns - Decorator Pattern](https://www.tutorialspoint.com/design_pattern/decorator_pattern.htm)

[装饰者模式 (三不猴子)](https://www.jianshu.com/p/4a530a3c70af)

[设计模式之装饰者模式（Decorator Pattern） (秃头的路上)](https://www.jianshu.com/p/9922bf82be34)