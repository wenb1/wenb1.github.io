---
layout: post
title: 设计模式之简单工厂模式(Factory Pattern)
categories: DesignPattern
description: 简单工厂模式介绍
keywords: DesignPattern, 设计模式
---

简单工厂模式属于创建型模式又叫做静态工厂方法模式，它属于**类创建型模式**。在简单工厂模式中，可以根据参数的不同返回不同类的实例。
简单工厂模式专门定义一个类来负责创建其他类的实例，被创建的实例通常都具有共同的父类。

## 1. 简单工厂模式实现

### 1.1 实现结构图

![简单工厂模式结构图](/images/posts/designpattern/factory_pattern_uml_diagram.png)

我们有一个shape的接口，这个接口有三个实现类，Circle，Square，Rectangle，工厂就负责制造这三个实现类。

### 1.2 具体实现

#### 1.2.1 创建Shape接口

```java
public interface Shape {
	void draw();
}
```

#### 1.2.2 创建具体实现接口的类

```java
public class Rectangle implements Shape {

   @Override
   public void draw() {
      System.out.println("Inside Rectangle::draw() method.");
   }
}
```

```java
public class Square implements Shape {

   @Override
   public void draw() {
      System.out.println("Inside Square::draw() method.");
   }
}
```

```java
public class Circle implements Shape {

   @Override
   public void draw() {
      System.out.println("Inside Circle::draw() method.");
   }
}
```

#### 1.2.3 创建工厂

```java
public class ShapeFactory {
	
   //use getShape method to get object of type shape 
   public Shape getShape(String shapeType){
      if(shapeType == null){
         return null;
      }		
      if(shapeType.equalsIgnoreCase("CIRCLE")){
         return new Circle();
         
      } else if(shapeType.equalsIgnoreCase("RECTANGLE")){
         return new Rectangle();
         
      } else if(shapeType.equalsIgnoreCase("SQUARE")){
         return new Square();
      }
      
      return null;
   }
}
```

#### 1.2.4 使用工厂

```java
public class FactoryPatternDemo {

   public static void main(String[] args) {
      ShapeFactory shapeFactory = new ShapeFactory();

      //get an object of Circle and call its draw method.
      Shape shape1 = shapeFactory.getShape("CIRCLE");

      //call draw method of Circle
      shape1.draw();

      //get an object of Rectangle and call its draw method.
      Shape shape2 = shapeFactory.getShape("RECTANGLE");

      //call draw method of Rectangle
      shape2.draw();

      //get an object of Square and call its draw method.
      Shape shape3 = shapeFactory.getShape("SQUARE");

      //call draw method of square
      shape3.draw();
   }
}
```

## 2. 简单工厂模式分析

### 2.1 优点

- 工厂类含有必要的判断逻辑，可以决定在什么时候创建哪一个产品类的实例，客户端可以**免除直接创建产品对象的责任**，而仅仅“消费”产品；简单工厂模式通过这种做法**实现了对责任的分割**，它提供了**专门的工厂类用于创建对象**。

- 客户端无须知道所创建的具体产品类的类名，只需要知道具体产品类**所对应的参数**即可，对于一些复杂的类名，通过简单工厂模式可以减少使用者的记忆量。

- 通过引入配置文件，可以在不修改任何客户端代码的情况下更换和增加新的具体产品类，在一定程度上提高了系统的灵活性。

### 2.2 缺点

- 由于**工厂类集中了所有产品创建逻辑**，一旦不能正常工作，整个系统都要受到影响。

- 使用简单工厂模式将会增加系统中类的个数，在一定程序上增加了系统的复杂度和理解难度。

- **系统扩展困难，一旦添加新产品就不得不修改工厂逻辑，同样破坏了“开闭原则”；在产品类型较多时，有可能造成工厂逻辑过于复杂，不利于系统的扩展和维护**。

- 简单工厂模式由于使用了静态工厂方法，造成工厂角色**无法形成基于继承的等级结构**。

### 2.3 适用条件

- 工厂类负责创建的**对象比较少**：由于创建的对象较少，不会造成工厂方法中的业务逻辑太过复杂。

- 客户端只知道传入工厂类的参数，对于如何创建对象不关心：客户端既不需要关心创建细节，甚至连类名都不需要记住，**只需要知道类型所对应的参数**。

**参考文章**：

[工厂模式--简单工厂模式](https://www.jianshu.com/p/5cb52d84bd6d)

[Design Pattern - Factory Pattern](https://www.tutorialspoint.com/design_pattern/factory_pattern.htm)