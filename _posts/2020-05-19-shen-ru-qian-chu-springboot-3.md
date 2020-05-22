---
layout: post
title: 《深入浅出 Springboot 2.x》第3章 全注解下的Spring IoC
categories: Springboot
description: 了解springboot的IoC
keywords: Springboot, IoC
---

IoC 容器是Spring 的核心，可以说Spring 是一种基于IoC容器编程的框架。Spring Boot 是基于注解的开发Spring IoC 。

# 第3章 全注解下的Spring IoC

IoC是一种通过描述来生成或者获取对象的技术，而这个技术不是Spring甚至不是Java独有的。对于Java初学者更多的时候所熟悉的是使用`new`关键字来创建对象，而在Spring中则不是，它是通过描述来创建对象。只是Spring Boot并不建议使用XML，而是通过注解的描述生成对象。

一个系统可以生成各种对象，井且这些对象都需要进行管理。还值得一提的是，对象之间并不是孤立的，它们之间还可能存在依赖的关系。在Spring中把每一个
需要管理的对象称为Spring Bean（简称Bean ），而Spring管理这些Bean的容器，被我们称为SpringIoC容器（或者简称IoC 容器） 。IoC容器需要具备两个基本的功能：

- 通过描述管理Bean，包括发布和获取Bean
- 通过描述完成Bean之间的依赖关系。
  

## 3.1 IoC容器简介

Spring IoC容器是一个管理Bean的容器，在Spring的定义中，它要求所有的IoC 容器都需要实现接口BeanFactory，它是一个顶级容器接口。由于BeanFactory 的功能还不够强大，因此Spring在BeanFactory的基础上， 还设计了一个更为高级的接口ApplicationContext。它是BeanFactory的子接口之一， 在Spring的体系中BeanFactory和ApplicationContext是最为重要的接口设计，在现实中我们使用的大部分Spring IoC容器是ApplicationContext接口的实现类。

在Spring Boot当中我们主要是通过注解来装配Bean到Spring IoC容器中，为了贴近Spring Boot的需要，这里不再介绍与XML相关的IoC容器，而主要介绍一个基于注解的IoC容器，它就是AnnotationConfigApplicationContext，从名称就可以看出它是一个基于注解的IoC容器。

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

`@Configuration`代表这是一个Java配置文件，Spring的容器会根据它来生成IoC容器去装配Bean; `@Bean`代表将`initUser`方法返回的POJO装配到IoC容器中，而其属性name 定义这个Bean的名称，如果没有配置它，则将方法名称“ initUser ”作为Bean的名称保存到Spring IoC 容器中。

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

代码中将Java配置文件`AppConfig`传递给AnnotationConfigApplicationContext的构造方法，这样它就能够读取配置了。然后将配置里面的Bean装配到IoC容器中，于是可以使用getBean方法获取对应的POJO 。

当然这只是很简单的方法，而注解`@Bean`也不是唯一创建Bean的方法，还有其他的方法可以让IoC容器装配Bean，而且Bean之间还有依赖的关系需要进一步处理。

## 3.2 装配你的Bean

### 3.2.1 通过扫描装配你的Bean

如果一个个的Bean使用注解`@Bean`注入Spring loC容器中，那将是一件很麻烦的事情。好在Spring还允许我们进行扫描装配Bean到loC容器中，对于扫描装配而言使用的注解是`@Component`和`@ComponentScan`。`@Component`是标明哪个类被扫描进入Spring IoC容器，而`@ComponentScan`则是标明采用何种策略去扫描装配Bean。

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

这里加入了`@ComponentScan`，意味着它会进行扫描，但是它只会扫描类AppConfig所在的当前包和其子包。`@ComponentScan`还允许我们自定义扫描的包。

`@ComponentScan`可以通过配置项basePackages定义扫描的包名，在没有定义的情况下，它只会扫描当前包和其子包下的路径：还可以通过basePackageClasses定义扫描的类；其中还有includeFilters和excludeFilters，includeFilters是定义满
足过滤器(Filter)条件的Bean才去扫描， excludeFilters则是排除过滤器条件的Bean，它们都需要通过一个注解@Filter 去定义，它有一个type类型，这里可以定义为注解或者正则式等类型。

为了不装配特定的Bean，我们可以使用excludeFilters配置特定的Bean不会被装入IoC：

```java
@ComponentScan(basePackages = "dev.wenbo.learnspring", excludeFilters = {@ComponentScan.Filter(classes = {Service.class})})
```

### 3.2.2 定义第三方Bean

现实的Java的应用往往需要引入许多来自第三方的包， 并且很有可能希望把第三方包的类对象也放入到Spring IoC容器中，这时＠Bean注解就可以发挥作用了。

## 3.3 依赖注入

Bean之间的依赖，在Spring IoC的概念中，我们称为依赖注入（ Dependency Injection）。

例子：

```java
public interface Animal {
    public void use();
}

public interface Person {
    public void service();
    public void setAnimal(Animal animal);
}
```

```java
@Component
public class Dog implements Animal {
    @Override
    public void use() {
        System.out.println("狗是人类的朋友");
    }
}

@Component
public class APerson implements Person {
	
    //通过注解将Dog注入到APerson里
    @Autowired
    private Animal animal;

    @Override
    public void service() {
        this.animal.use();
    }

    @Override
    public void setAnimal(Animal animal) {
        this.animal=animal;
    }
}
```

测试：

```java
@Configuration
@ComponentScan
public class AppConfig {

}

public class IocTest {
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
        Person person = ctx.getBean(APerson.class);
        person.service();
    }
}
```

### 3.3.1 注解`@Autowired`

`@Autowired`是我们使用得最多的注解之一， 因此在这里需要进一步地探讨它。它注入的机制最基本的一条是根据类型（ by type ）， 我们回顾IoC容器的顶级接口BeanFactory，就可以知道IoC 容器是通过getBean方法获取对应Bean的，而getBean又支持根据类型（ by type ）或者根据名称（ by name ）。

如果我们再创建一个如下所示的类：

```java
@Component
public class Cat implements Animal {
    @Override
    public void use() {
        System.out.println("猫很可爱");
    }
}
```

这时，我们有两个具体类实现了Animal接口，到运行的时候，IoC不知道应该注入Dog类还是Cat类。如果我们把代码中的animal变量重新命名为dog的话，问题就解决了。

```java
@Component
public class APerson implements Person {

    @Autowired
    private Animal dog;

    @Override
    public void service() {
        this.dog.use();
    }

    @Override
    public void setAnimal(Animal animal) {
        this.dog=animal;
    }
}
```

这里， 我们只是将属性的名称从animal修改为了dog，那么我们再测试的时候，你可以看到是采用狗来提供服务的。那是因为`@Autowired`提供这样的规则， 首先它会根据类型找到对应的Bean，如果对应类型的Bean不是唯一的，那么它会根据其属性名称和Bean的名称进行匹配。如果匹配得上，就会使用该Bean，如果还无法匹配，就会抛出异常。

这里还要注意的是`@Autowired`是一个默认必须找到对应Bean的注解，如果不能确定其标注属性一定会存在并且允许这个被标注的属性为`null` ，那么你可以配置`@Autowired`属性`required`为`false`。

### 3.3.2 消除歧义性一`@Primary`和`@Quelifier`

在上面我们发现有猫有狗的时候，为了使`@Autowired`能够继续使用，我们做了一个决定，将BussinessPerson的属性名称从animal 修改为dog。显然这是一个憋屈的做法，好好的一个动物，却被我们定义为了狗。产生注入失败的问题根本是按类型（ by type ） 查找，正如动物可以有多种类型，这样会造成Spring IoC容器注入的困扰，我们把这样的一个问题称为歧义性。

我们可以通过`@Primary`注解来修改优先权来解决这个问题，当我们有猫有狗的时候，假设这次需要使用猫，那么只需要在猫类的定义上加入`@Primary`就可以了：

```java
@Component
@Primary
public class Cat implements Animal {
    @Override
    public void use() {
        System.out.println("猫很可爱");
    }
}
```

这里的`@Primary`的含义告诉Spring IoC容器，当发现有多个同样类型的Bean时，请优先使用我进行注入，于是再进行测试时会发现，系统将用猫为你提供服务。因为当Spring进行注入的时候虽然它发现存在多个动物，但因为Cat被标注为了`@Primary`，所以优先采用Cat 的实例进行了注入，这样就通过优先级的变换使得IoC容器知道注入哪个具体的实例来满足依赖注入。

有时候`@Primary`也可以使用在多个类上，也许无论是猫还是狗都可能带上`@Primary`注解，其结果是IoC容器还是无法区分采用哪个Bean的实例进行注入， 又或者说我们需要更加灵活的机制来实现注入，那么`@Quelifier`可以满足你的这个愿望。它的配置项`value`需要一个字符串去定义，它将与`@Autowired`组合在一起，通过类型和名称一起找到Bean。

```java
@Component
public class APerson implements Person {

    @Autowired
    @Qualifier("dog")
    private Animal animal;
	...
}
```

一旦这样声明，Spring IoC将会以类型和名称去寻找对应的Bean进行注入。根据类型和名称，显然也只能找到狗为我们服务了。

### 3.3.3 带有参数的构造方法类的装配

在上面，我们都基于一个默认的情况，那就是不带参数的构造方法下实现依赖注入。但事实上，有些类只有带有参数的构造方法，于是上述的方法都不能再使用了。为了满足这个功能，我们可以使用`@Autowired`注解对构造方法的参数进行注入。

```java
@Component
public class APerson implements Person {

    private Animal animal;

    public APerson(@Autowired @Qualifier("dog") Animal animal){
        this.animal = animal;
    }
    ...
}
```

## 3.4 生命周期

Bean的生命周期过程大致分为：Bean定义，Bean的初始化，Bean的生存期和Bean的销毁4个部分。

Bean定义大致过程如下：

- Spring通过我们的配置，如`@ComponentScan`定义的扫描路径去找到带有`@Component`的类，这个过程就是一个资源定位的过程。

- 一旦找到了资源，那么它就开始解析，并且将定义的信息保存起来。注意，此时还没有初始化Bean，也就没有Bean的实例，它有的仅仅是Bean 的定义。

- 然后就会把Bean定义发布到Spring IoC容器中。此时，IoC容器也只有Bean的定义，还是没有Bean的实例生成。

完成了这3 步只是一个资源定位并将Bean的定义发布到IoC容器的过程，还没有Bean实例的生成，更没有完成依赖注入。在默认的情况下， Spring会继续去完成Bean的实例化和依赖注入，这样从IoC容器中就可以得到一个依赖注入完成的Bean 。但是，有些Bean会受到变化因素的影响，这时我们倒希望是取出Bean 的时候完成初始化和依赖注入，换句话说就是让那些Bean只是将定义发布到IoC 容器而不做实例化和依赖注入，当我们取出来的时候才做初始化和依赖注入等操作。

Spring Bean的初始化流程：

![springboot bean生命周期](/images/posts/springboot/chapter3_1.PNG)