---
layout: post
title: 《深入浅出 Springboot 2.x》第4章 开始约定编程-Spring AOP
categories: Springboot
description: 了解springboot的AOP
keywords: Springboot, AOP
---

对于约定编程， 首先需要记住的是约定的流程是什么，然后就可以完成对应的任务，却不需要知道底层设计者是怎么将约定的内容织入对应的流程中的。

# 第4章 开始约定编程-Spring AOP

## 4.1 约定编程

先看一个约定编程的例子，之后Spring AOP的概念也就容易理解了，因为它们是异曲同工的东西。

### 4.1.1 约定

我们编写一个接口，再创建一个实现类，如下：

```java
public interface HelloService {
     void sayHello(String name);
}
```

```java
public class HelloServiceImpl implements HelloService{
    @Override
    public void sayHello(String name) {
        if(name==null || name.trim()==""){
            throw new RuntimeException("parameter is null!");
        }
        System.out.println("hello "+name);
    }
}
```

接下来定义一个拦截器接口：

```java
public interface Interceptor {
     //事前方法
     boolean before();

     // 事后方法
     void after();

    /**
     * 取代原有事件方法
     * @param invocation -- 回调参数，可以通过它的proceed方法，回调原有事件
     * @return 原有事件返回对象
     * @throws InvocationTargetException
     * @throws IllegalAccessException
     */
     Object around(Invocation invocation)
         throws InvocationTargetException, IllegalAccessException;

     // 是否返回方法。事件没有发生异常执行
     void afterReturning();

     // 事后异常方法，当事件发生异常后执行
     void afterThrowing();

     // 是否使用around方法替代原有方法
     boolean useAround();
}

```

`Invocation`对象的`proceed`方法会以反射的形式去调用原有的方法。

我们创建一个拦截器的实现类：

```java
public class MyInterceptor implements Interceptor{
    @Override
    public boolean before() {
        System.out.println("before....");
        return true;
    }

    @Override
    public void after() {
        System.out.println("after...");
    }

    @Override
    public Object around(Invocation invocation) throws Throwable {
        System.out.println("around before...");
        Object obj=invocation.proceed();
        System.out.println("around after...");
        return obj;
    }

    @Override
    public void afterReturning() {
        System.out.println("afterReturning...");
    }

    @Override
    public void afterThrowing() {
        System.out.println("afterThrowing...");
    }

    @Override
    public boolean useAround() {
        return true;
    }
}

```

这个拦截器的功能也不复杂， 接着就要谈谈我和你的约定了。约定是本节的核心，也是Spring AOP 的本质。我先提供一个类一`ProxyBean`使用，它有一个静态的(`static`)方法：

```java
public static Object getProxyBean(Object target, Interceptor interceptor)
```