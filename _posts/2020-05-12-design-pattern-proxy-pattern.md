---
layout: post
title: 设计模式之代理模式(Proxy Pattern)
categories: DesignPattern
description: 代理模式介绍
keywords: DesignPattern, 设计模式
---

代理模式(Proxy Pattern)又叫委托模式，是为某个对象提供一个代理对象，并且由代理对象控制对原对象的访问。类似于我们生活中的中介。

举个例子，比如一个公司的CEO，他一般都会有一个秘书，秘书会代替CEO处理一些日常事务，让CEO能更专注于公司的重大决策而不用太操心杂事，比如时间表排期啊，端茶倒水这种。

# 1. 代理模式的实现

为了保持行为的一致性，代理类和委托类通常会实现相同的接口，所以在访问者看来两者没有丝毫的区别。通过代理类这中间一层，能有效控制对委托类对象的直接访问，也可以很好地隐藏和保护委托类对象，同时也为实施不同控制策略预留了空间，从而在设计上获得了更大的灵活性。更通俗的说，代理解决的问题是：当两个类需要通信时，引入第三方代理类，将两个类的关系解耦，让我们只了解代理类即可，而且代理的出现还可以让我们完成与另一个类之间的关系的统一管理。但是切记，代理类和委托类要实现相同的接口，因为代理真正调用的还是委托类的方法。

Java的代理模式有三种实现方式，分别是静态代理，动态代理和CGLIB代理。

## 1.1 静态代理

由程序员创建或特定工具自动生成源代码，再对其进行编译。在程序运行之前，代理类.class文件就已经被创建，代理类和委托类的关系在运行前就确定。

### 1.1.1 创建接口

```java
public interface WandaEmployee {
	
	void work();

}
```

### 1.1.2 创建一个实现类

```java
public class WangZong implements WandaEmployee{

	@Override
	public void work() {
		// TODO Auto-generated method stub
		System.out.println("制定一个小目标");
	}

}
```

### 1.1.3 创建一个代理类

```java
public class WangZongProxy implements WandaEmployee{
	
	private WandaEmployee WandaEmployee;
	
	public WangZongProxy(WandaEmployee WandaEmployee) {
		this.WandaEmployee = WandaEmployee;
	}

	@Override
	public void work() {
		// TODO Auto-generated method stub
		System.out.println("给王总纸和笔");
		WandaEmployee.work();
		System.out.println("收起纸和笔");
	}

}
```

### 1.1.4 测试

```java
public class ProxyDemo {
	public static void main(String[] args) {
		WandaEmployee wangZong = new WangZong();
		WandaEmployee wangzongProxy  = new WangZongProxy(wangZong);
		wangzongProxy.work();
	}
}

```

### 1.1.5 静态代理总结

- 优点：可以做到在符合开闭原则的情况下对目标对象进行功能扩展。业务类只需要关注业务逻辑本身，保证了业务类的重用性。这是代理的共有优点。

- 缺点：我们得为每一个服务都得创建代理类，工作量太大，不易管理。同时接口一旦发生改变，代理类也得相应修改。代理对象只服务于一种类型的对象，如果要服务多类型的对象。势必要为每一种对象都进行代理，静态代理在程序规模稍大时就无法胜任了。

## 1.2 动态代理

从静态代理会发现——每个代理类只能为一个接口服务，这样程序开发中必然会产生许多的代理类。在动态代理中我们不再需要再手动的创建代理类，我们只需要编写一个动态处理器就可以了。真正的代理对象由JDK再运行时为我们动态的来创建。

### 1.2.1 创建接口

```java
public interface WandaEmployee {
	
	void work();

}
```

### 1.2.2 创建一个实现类

```java
public class WangZong implements WandaEmployee{

	@Override
	public void work() {
		// TODO Auto-generated method stub
		System.out.println("制定一个小目标");
	}

}
```

### 1.2.3 创建动态代理类

动态代理类只能代理接口（不支持抽象类），代理类都需要实现`InvocationHandler`接口，实现`invoke`方法。`invoke`方法就是调用被代理接口的所有方法时需要调用的，返回的值是被代理接口的一个实现类。

```java
public class DynamicProxy implements InvocationHandler{
	
	private Object object;
	
	public DynamicProxy(final Object object) {
		         this.object = object;
	     }

	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		// TODO Auto-generated method stub
		System.out.println("给王总纸和笔");
		Object result = method.invoke(object, args);
		System.out.println("收起纸和笔");
		return result;
	}
	
}
```

被代理对象target通过参数传递进来，我们通过`target.getClass().getClassLoader()`获取ClassLoader对象，然后通过`target.getClass().getInterfaces()`获取它实现的所有接口，然后将target包装到实现了InvocationHandler接口的ProxyHandler实现对象中。通过newProxyInstance函数我们就获得了一个动态代理对象。

### 1.2.4 测试

```java
public class ProxyDemo {
	public static void main(String[] args) {
		WandaEmployee wangZong = new WangZong();
		WandaEmployee wangzongProxy  = (WandaEmployee) Proxy.newProxyInstance(WangZong.class.getClassLoader(), new Class[]{WandaEmployee.class}, new DynamicProxy(wangZong));
		wangzongProxy.work();
	}
}
```

### 1.2.5 动态代理总结

- 优点：动态代理与静态代理相比较，最大的好处是接口中声明的所有方法都被转移到调用处理器一个集中的方法中处理（`InvocationHandler.invoke`）。这样，在接口方法数量比较多的时候，我们可以进行灵活处理，而不需要像静态代理那样每一个方法进行中转。而且动态代理的应用使我们的类职责更加单一，复用性更强。

- 缺点：Proxy已经设计得非常优美，但是还是有一点遗憾之处——**它始终无法摆脱仅支持 interface代理的束缚**，因为它的设计注定了这个遗憾。动态生成的代理类的继承关系图，已经注定有一个共同的父类叫 Proxy。Java 的继承机制注定了这些动态代理类们无法实现对 class 的动态代理，原因是多继承在 Java 中本质上就行不通。

## 1.3 CGLIB代理

JDK实现动态代理需要实现类通过接口定义业务方法，对于没有接口的类，如何实现动态代理呢，这就需要CGLib了。CGLib采用了非常底层的字节码技术，其原理是通过字节码技术为一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑。但因为采用的是继承，所以不能对final修饰的类进行代理。JDK动态代理与CGLib动态代理均是实现Spring AOP的基础。

### 1.3.1 创建接口

```java
public interface WandaEmployee {
	
	void work();

}
```

### 1.3.2 创建一个实现类

```java
public class WangZong implements WandaEmployee{

	@Override
	public void work() {
		// TODO Auto-generated method stub
		System.out.println("制定一个小目标");
	}

}
```

### 1.3.3 创建CGLIB代理类

```java
public class CglibProxy implements MethodInterceptor {
     private Object target;
     public Object getInstance(final Object target) {
         this.target = target;
         Enhancer enhancer = new Enhancer();
         enhancer.setSuperclass(this.target.getClass());
         enhancer.setCallback(this);
         return enhancer.create();
     }
 
     public Object intercept(Object object, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
    	 System.out.println("给王总纸和笔");
         Object result = methodProxy.invokeSuper(object, args);
         System.out.println("收起纸和笔");
         return result;
     }
 }
```

### 1.3.4 创建测试类

```java
public class cglibTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		WandaEmployee wangZong = new WangZong();
		CglibProxy cglibProxy = new CglibProxy();
		WangZong wangzongCglibProxy = (WangZong) cglibProxy.getInstance(wangZong);
		wangzongCglibProxy.work();

	}

}
```

### 1.3.5 CGLIB代理总结

CGLIB创建的动态代理对象比JDK创建的动态代理对象的性能更高，但是CGLIB创建代理对象时所花费的时间却比JDK多得多。所以对于单例的对象，因为无需频繁创建对象，用CGLIB合适，反之使用JDK方式要更为合适一些。同时由于CGLib由于是采用动态创建子类的方法，对于final修饰的方法无法进行代理。

# 2. 代理模式分析

## 2.1 代理模式的优点

- 职责清晰：具体角色是实现具体的业务逻辑，不用关心其他非本职责的事务，通过后期的代理完成一件事务，代码清晰。在某些情况下，一个客户类不想或者不能直接引用一个委托对象，而代理类对象可以在客户类和委托对象之间起到中介的作用，其**特征是代理类和委托类实现相同的接口**。

- 高扩展性：具体主题角色随时会发生变化，但是只要实现了接口，接口不变，代理类就可以不做任何修改继续使用，符合“**开闭原则**”。
   另外，代理类除了是客户类和委托类的中介之外，我们还可以通过给代理类增加额外的功能来扩展委托类的功能，这样做我们只需要修改代理类而不需要再修改委托类，**同样符合开闭原则**。

## 2.2 代理模式的适用场景

代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后对返回结果的处理等。代理类本身并不真正实现服务，而是通过调用委托类的相关方法，来提供特定的服务。真正的业务功能还是由委托类来实现，但是可以在业务功能执行的前后加入一些公共的服务。例如我们想给项目加入缓存、日志这些功能，我们就可以使用代理类来完成，而没必要打开已经封装好的委托类。



**参考文章**：

[设计模式---代理模式 (Jerry_1116)](https://www.cnblogs.com/daniels/p/8242592.html)

[设计模式之——代理模式 (Dan_Go)](https://www.jianshu.com/p/9cdcf4e5c27d)