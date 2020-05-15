---
layout: post
title: 设计模式之策略模式(Strategy Pattern)
categories: DesignPattern
description: 策略模式介绍
keywords: DesignPattern, 设计模式
---

策略模式(Strategy Pattern)：它定义了算法家族，分别封装起来，让他们之间可以互相替换，此模式让算法的变化不会影响到使用算法的客户。

策略模式就是用来封装算法的，但在实践中，我们发现可以用它来封装几乎任何类型的规则，只要在分析过程中听到需要在不同时间应用不同的业务规则，就可以考虑使用策略模式处理这种变化的可能性。

# 1. 策略模式实现

## 1.1 策略模式实现图

![策略模式实现图](/images/posts/designpattern/strategy_pattern_uml_diagram.png)

## 1.2 策略模式具体实现

### 1.2.1 创建策略接口

```java
public interface Strategy {
   public int doOperation(int num1, int num2);
}
```

### 1.2.2 创建策略接口的实现类

```java
public class OperationAdd implements Strategy {

	@Override
	public int doOperation(int num1, int num2) {
		return num1 + num2;
	}

}

public class OperationMultiply implements Strategy {

	@Override
	public int doOperation(int num1, int num2) {
		return num1 * num2;
	}

}

public class OperationSubstract implements Strategy {

	@Override
	public int doOperation(int num1, int num2) {
		return num1 - num2;
	}

}
```

### 1.2.3 创建Context类

```java
public class Context {
	Strategy strategy;
	
	public Context(Strategy strategy) {
		this.strategy=strategy;
	}
	
	public int executeStrategy(int num1, int num2){
	      return strategy.doOperation(num1, num2);
	}

}
```

### 1.2.4 测试

```java
public class StrategyDemo {

	public static void main(String[] args) {
		Context context = new Context(new OperationAdd());		
	    System.out.println("10 + 5 = " + context.executeStrategy(10, 5));

	    context = new Context(new OperationSubstract());		
	    System.out.println("10 - 5 = " + context.executeStrategy(10, 5));

	    context = new Context(new OperationMultiply());		
	    System.out.println("10 * 5 = " + context.executeStrategy(10, 5));

	}

}
```

# 2. 策略模式分析

## 2.1 优点

- 算法可以自由切换
- 避免使用多重条件判断
- 扩展性良好
- 化了单元测试，因为每个算法都有自己的类，可以通过自己的接口单独测试

## 2.2 缺点

- 策略类会增多
- 所有策略类都需要对外暴露

## 2.3 使用场景

1. 如果在一个系统里面有许多类，它们之间的区别仅在于它们的行为，那么使用策略模式可以动态地让一个对象在许多行为中选择一种行为。 
2. 一个系统需要动态地在几种算法中选择一种。 3
3. 如果一个对象有很多的行为，如果不用恰当的模式，这些行为就只好使用多重的条件选择语句来实现。