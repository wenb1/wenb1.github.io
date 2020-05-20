---
layout: post
title: 《深入浅出 Springboot 2.x》第3章 全注解下的Spring IoC
categories: Springboot
description: 了解springboot的IoC
keywords: Springboot, IoC
---

IoC 容器是Spring 的核心，可以说Spring 是一种基于IoC容器编程的框架。Spring Boot 是基于注解的开发Spring IoC 。

# 第3章 全注解下的Spring IoC

IoC是一种通过描述来生成或者获取对象的技术，而这个技术不是Spring甚至不是Java独有的。对于Java初学者更多的时候所熟悉的是使用new关键字来创建对象，而在Spring中则不是，它是通过描述来创建对象。只是Spring Boot并不建议使用XML，而是通过注解的描述生成对象。

一个系统可以生成各种对象，井且这些对象都需要进行管理。还值得一提的是，对象之间并不是孤立的，它们之间还可能存在依赖的关系。在Spring中把每一个
需要管理的对象称为Spring Bean （简称Bean ），而Spring管理这些Bean 的容器，被我们称为SpringIoC容器（或者简称IoC 容器） 。IoC 容器需要具备两个基本的功能：

- 通过描述管理Bean ， 包括发布和获取Bean
- 通过描述完成Bean 之间的依赖关系。
  

## 3.1 IoC容器简介

Spring IoC容器是一个管理Bean的容器，在Spring的定义中，它要求所有的IoC 容器都需要实现接口BeanFactory，它是一个顶级容器接口。由于BeanFactory 的功能还不够强大，因此Spring在BeanFactory的基础上， 还设计了一个更为高级的接口ApplicationContext。它是BeanFactory的子接口之一， 在Spring的体系中BeanFactory和ApplicationContext是最为重要的接口设计，在现实中我们使用的大部分Spring IoC容器是ApplicationContext接口的实现类。

在Spring Boot当中我们主要是通过注解来装配Bean到Spring IoC容器中，为了贴近Spring Boot的需要， 这里不再介绍与XML相关的IoC容器，而主要介绍一个基于注解的IoC容器，它就是AnnotationConfigApp licationContext，从名称就可以看出它是一个基于注解的IoC 容器。

看一个例子：

```java
public class User {
    private Long id;
    private String userName;
    private String note;

    // getter and setter
}
```

```java
@Configuration
public class AppConfig{
    @Bean(name = "user")
    public User initUser(){
        User user = new User();
        user.setId(1L);
        user.setUserName("user_name_l");
        user.setNote("note_l");
        return user;
    }
}
```

@Configuration 代表这是一个Java 配置文件， Sprin g 的容器会根据它来生成IoC 容器去装配Bean; @Bean 代表将initUser方法返回的POJO 装配到IoC 容器中，而其属性name 定义这个Bean 的名称，如果没有配置它，则将方法名称“ initUser ”作为Bean 的名称保存到S pring IoC 容器中。

做好了这些，就可以使用AnnotationConfigApplicationContext来构建自己的IoC 容器：

```java
public class IocTest {
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
        User user = ctx.getBean(User.class);
        System.out.println(user.getId());
    }
}
```

代码中将Java配置文件AppConfig传递给AnnotationConfigApplicationContext的构造方法，这样它就能够读取配置了。然后将配置里面的Bean装配到IoC容器中，于是可以使用getBean方法获取对应的POJO 。

当然这只是很简单的方法，而注解@Bean也不是唯一创建Bean的方法，还有其他的方法可以让IoC容器装配Bean，而且Bean之间还有依赖的关系需要进一步处理。

## 3.2 装配你的Bean

### 3.2.1 通过扫描装配你的Bean

如果一个个的Bean使用注解@Bean注入Spring loC容器中，那将是一件很麻烦的事情。好在Spring还允许我们进行扫描装配Bean到loC容器中，对于扫描装配而言使用的注解是@Component和@ComponentScan。@Component是标明l哪个类被扫描进入Spring IoC容器，而@ComponentScan则是标明采用何种策略去扫描装配Bean。

```java
@Component("user")
public class User {
    @Value("1")
    private Long id;
    @Value("user_name_1")
    private String userName;
    @Value("note_1")
    private String note;

    // getter and setter
}
```

```java
@Configuration
@ComponentScan
public class AppConfig{
}
```

这里加入了@ComponentScan，意味着它会进行扫描，但是它只会扫描类AppConfig所在的当前包和其子包。@ComponentScan还允许我们自定义扫描的包。

测试：

```java
public class IocTest {
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
        User user = ctx.getBean(User.class);
        System.out.println(user.getId());
    }
}
```