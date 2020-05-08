---
layout: post
title: 《深入浅出 Springboot 2.x》读书笔记
categories: Springboot
description: 了解springboot的事务
keywords: Springboot, Transaction, 事务
---

# 第6章 聊聊数据库事务处理

Spring事务有两种实现方式: 

- 编程式事务: 使用TransactionTemplate或者底层的PlatformTransactionManager (<span style="color:red;">这种方式已经被基本淘汰</span>)
- 声明式事务

## 6.1 JDBC的数据库事务
JDBC中使用Connection对象来处理事务

```java
Connection  conn = null; //创建连接
try{
     conn = dataSource.getConnection(); //获取连接
     conn.setAutoCommit(false); //开启手动事务
     ......
     conn.commit(); //手动提交事务
     
} catch（){
    conn.rollback(); //事务回滚
}finally {
    conn.setAutoCommit(true); //开启自动事务
    conn.close(); //关闭连接
}
```
JDBC中大量try...catch...finally的三段式事务非常冗余。而Spring的AOP很好地解决了这个问题，AOP允许我们把公共的代码抽取出来，单独实现。

## 6.2 Spring声明式事务的使用
### 6.2.1 Spring声明式数据库事务约定
声明式事务使用@Transactional进行标注，这个注解可以修饰类和方法，用在类上时，所有的public，非static方法都将启用事务，spring事务的约定如下图所示：

![spring事务约定](/images/posts/springboot/chapter6_1.png)

### 6.2.2 @Transactional的配置项
Spring的事务允许进行许多配置，比如传播行为，隔离级别，事务超时，只读事务，回滚设置。

传播行为和隔离级别是很重要的内容，在后面会详细讲述，这里先介绍除此以外的配置。

- **事务超时**: 指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。在 TransactionDefinition 中以 int 来表示超时时间，单位是秒。默认设置为底层事务系统的超时值，如果底层数据库事务系统没有设置超时值，那么就是none，没有超时限制

	```java
	@Transactional(timeout = 100)
	public String example(){
		...
	}
	```
- **只读事务**: 只读事务只能在代码只读取数据但是不改变数据的情况，只读事务可以用来优化

    ```java
    @Transactional(readOnly = true)
    public String example(){
    	...
    }
    ```
    
- **事务回滚**: 通过设置回滚属性可以指定在什么异常下依旧提交事务，在什么异常下回滚事务

	```java
	@Transactional(rollbackFor = InvalidAmountException.class)
	public String example(){
		...
	}
	```
  
  
    Spring允许进行的配置:
  
  | Property               | Type                                                                | Description                                                                |
|------------------------|---------------------------------------------------------------------|----------------------------------------------------------------------------|
| value                  | String                                                              | Optional qualifier specifying the transaction manager to be used.          |
| propagation            | enum: Propagation                                                   | Optional propagation setting.                                              |
| isolation              | enum: Isolation                                                     | Optional isolation level.                                                  |
| readOnly               | boolean                                                             | Read/write vs. read-only transaction                                       |
| timeout                | int (in seconds granularity)                                        | Transaction timeout.                                                       |
| rollbackFor            | Array of Class objects, which must be derived from  Throwable.      | Optional array of exception classes that must cause rollback.              |
| rollbackForClassName   | Array of class names. Classes must be derived from  Throwable.      | Optional array of names of exception classes that must cause rollback.     |
| noRollbackFor          | Array of Class objects, which must be derived from  Throwable.      | Optional array of exception classes that must not cause rollback.          |
| noRollbackForClassName | Array of String class names, which must be derived from  Throwable. | Optional array of names of exception classes that must not cause rollback. |

### 6.2.3 Spring事务管理器
上述事务流程中，事务的打开，回滚和提交是由事务管理器来完成的。在Spring中，事务管理器的顶层接口是PlatformTransactionManager，当我们引入不同框架时会，会引入许多不同的事务管理器，在Mybatis中，最常用到的事务管理器是DataSourceTransactionManager。

## 6.3 隔离级别
### 6.3.1 数据库事务的知识
事务的隔离级别是老生常谈的问题了，在数据库知识中是重中之重的概念之一。

先引入事务的4大属性ACID:

- **原子性(Atomic)**: 事务中包含的操作被看作一个整体的业务单元，这个业务单元中的操作要么全部成功，要么全部失败，不会出现部分成功，部分失败的场景。
- **一致性(Consistency)**: 事务在完成时，必须使所有的数据都保持一致状态，在数据库中所有的修改都基于事务，保证了数据的完整性。
- **隔离性(Isolation)**: 先举个例子，在多个应用程序线程同时访问同一数据时，数据库同样的数据就会在各个不同的事务中被访问，这样会产生丢失更新。为了压制丢失更新的产生，数据库定义了隔离级别的概念，通过它的选择，可以在不同程度上压制丢失更新的发生。
- **持久性(Durability)**: 事务结束后，所有的数据会固化到一个地方，如保存到磁盘当中，即使断电重启后也可以提供给应用程序访问。

隔离性比较难理解，下面主要介绍隔离性。

**第一类丢失更新**

第一类丢失更新就是由于一个事务的回滚另一个事务的提交而引发的数据不一致的情况。例子：

![第一类丢失更新](/images/posts/springboot/chapter6_2.png)

由上图可见，T5时刻回滚，把事务2的修改覆盖了。这种由于回滚导致的数据不一致已经被大部分数据库克服了，所以我们不需要担心这种情况的发生。

**第二类丢失更新**

多个事务都提交引发的丢失更新是第二类丢失更新。例子：

![第一类丢失更新](/images/posts/springboot/chapter6_3.png)

在T5时刻，事务1提交事务时无法感知事务2的操作，最后导致库存数量不正确。为了解决这种问题，数据库提出了事务的隔离级别。

### 6.3.2 详解隔离级别

**1. 读未提交(read uncommitted)**

读未提交(read uncommitted)是最低的隔离级别，其含义是允许一个事务读取另外一个事务没有提交的数据。未提交读是一种危险的隔离级别，所以一般在我们实际的开发中应用不广， 但是它的优点在于并发能力高，适合那些对数据一致性没有要求而追求高并发的场景，它的最大坏处是出现脏读。

脏读例子：

![读未提交例子](/images/posts/springboot/chapter6_4.png)

为了克服脏读，数据库隔离级别还提供了读已提交(read commited)的级别

**2. 读已提交(read committed)**

读已提交(read committed)隔离级别， 是指一个事务只能读取另外一个事务已经提交的数据，
不能读取未提交的数据。

克服脏读例子：

![读未提交例子](/images/posts/springboot/chapter6_5.png)

虽然克服了脏读的问题，但又产生了另一个问题，不可重复读：

![不可重复读例子](/images/posts/springboot/chapter6_6.png)

为了解决不可重复读，数据库又提供了另一个隔离级别，可重复度。

**3. 可重复读(repeatable read)**

可重复读的目标是克服读写提交中出现的不可重复读的现象，因为在读写提交的时候，可能出现一些值的变化， 影响当前事务的执行，这个时候数据库提出了可重复读的隔离级别，这样就能够克服不可重复读的现象。

克服不可重复读例子：

![克服不可重复读例子](/images/posts/springboot/chapter6_7.png)

虽然克服了不可重复读的情况，但是又会引发新的问题，这就是幻读。

幻读例子：

![幻读例子](/images/posts/springboot/chapter6_8.png)

可重复读和幻读，是比较难以理解的内容， 这里稍微论述一下。首先这里的笔数不是数据库存储的值，而是一个统计值，商品库存则是数据库存储的值，这一点是要注意的。也就是幻读不是针对一条数据库记录而言，而是多条记录，例如， 这51 笔交易笔数就是多条数据库记录统计出来的。而可重复读是针对数据库的单一条记录，例如，商品的库存是以数据库里面的一条记录存储的，它可以产生可重复读，而不能产生幻读。

**4. 串行化(serializable)**