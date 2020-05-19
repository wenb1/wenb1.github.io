---
layout: post
title: 设计模式之抽象工厂模式(Abstract Factory Pattern)
categories: DesignPattern
description: 抽象工厂模式介绍
keywords: DesignPattern, 设计模式
---

抽象工厂模式(Abstract Factory Pattern)提供一个创建一系列相关或相互依赖对象的接口，而无须指定它们具体的类。它是一种**创建型模式**。

抽象工厂起源于以前对不同操作系统的图形化解决方案，如不同操作系统中的按钮和文本框控件其实现不同，展示效果也不一样，对于每一个操作系统，其本身就构成一个产品类，而按钮和文本框控件也构成一个产品类，两种产品类两种变化，各自有自己的特性。

看定义依然不知道什么意思。举个例子，抽象工厂就像一套标准，比如说手机代工厂富士康，那么富士康做手机的工厂有一套自己的标准，比如要生产摄像头，生产扬声器，生产电子元器件等等这一系列工厂生产标准。但是工厂又有不同，代工苹果手机的工厂标准和代工三星手机的工厂标准肯定不同，毕竟要生产两款手机，就是两款产品。这就像一个更具体的工厂，但建成的工厂标准还是富士康那套标准，还是要生产摄像头，扬声器什么的。

# 1. 抽象工厂模式实现

抽象工厂模式包含了几个角色：

AbstractFactory：用于声明生成抽象产品的方法

ConcreteFactory：实现了抽象工厂声明的生成抽象产品的方法，生成一组具体产品，这些产品构成了一个产品族，每一个产品都位于某个产品等级结构中；

AbstractProduct：为每种产品声明接口，在抽象产品中定义了产品的抽象业务方法；

Product：定义具体工厂生产的具体产品对象，实现抽象产品接口中定义的业务方法。

我们还需要提及一个产品等级与产品族的概念，一个制造苹果手机的工厂要能生产苹果的摄像头，也能生产苹果的屏幕等等，相当于一个工厂能生产苹果手机的所有零件，如下图所示：

![产品图](/images/posts/designpattern/abstract_factory_product.png)

好比图中的方块代表摄像头，圆圈代表屏幕，椭圆代表扬声器等等，而许多个方块代表不同种类手机的摄像头。

## 1.1 定义抽象产品族

```java
public abstract class Camera {
	public abstract void makeCamera();
}
```

```java
public abstract class Screen {
	public abstract void makeScreen();
}
```

## 1.2 实现具体的产品

```java
public class iPhoneCamera extends Camera {

	@Override
	public void makeCamera() {
		System.out.println("make iPhone camera");
	}

}
```

```java
public class iPhoneScreen extends Screen {

	@Override
	public void makeScreen() {
		System.out.println("make iPhone screen");
	}

}
```

## 1.3 定义抽象工厂

```java
public abstract class PhoneFactory {
	public abstract Camera makeCamera();
	public abstract Screen makeScreen();
}
```

## 1.4 实现具体工厂

```java
public class iPhoneFactory extends PhoneFactory {

	@Override
	public Camera makeCamera() {
		return new iPhoneCamera();
	}

	@Override
	public Screen makeScreen() {
		return new iPhoneScreen();
	}

}
```

## 1.5 测试

```java
public class AbstractFactoryDemo {

	public static void main(String[] args) {
		PhoneFactory phoneFactory = new iPhoneFactory();
		Screen iPhoneScreen = phoneFactory.makeScreen();
		Camera iPhoneCamera = phoneFactory.makeCamera();
		iPhoneScreen.makeScreen();
		iPhoneCamera.makeCamera();
	}

}
```

# 2. 抽象工厂模式分析

### 2.1 优点

-  具体产品在应用层代码隔离，无须关心创建细节
- 将一系列的产品族统一到一起创建

### 2.2 缺点

- 规定了所有可能被创建的产品集合，产品族中扩展新的产品困难，需要修改抽象工厂的接口
- 增加了系统的抽象性和理解难度

### 2.3 适用场景

- 客户端（应用层）不依赖于产品类实例如何被创建、实现等细节
- 强调一系列相关的产品对象（属于同一产品族）一起使用创建对象需要大量重复的代码
- 提供一个产品类的库，所有的产品以同样的接口出现，从而使客户端不依赖于具体实现

**参考文章**

[抽象工厂模式 (南小夕)](https://www.jianshu.com/p/7a56b7bafbb9)

[设计模式：抽象工厂模式，结合类图秒懂！(鄙人薛某)](https://blog.csdn.net/yeyazhishang/article/details/95173103)