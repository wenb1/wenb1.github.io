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
Connection  conn = null;//创建连接
try{
     conn = dataSource.getConnection();//获取连接
     conn.setAutoCommit(false);//开启手动事务
     ......
     conn.commit();//手动提交事务
     
} catch（） {
    conn.rollback();//事务回滚
}
```
JDBC中大量try...catch...finally的三段式事务非常冗余。而Spring的AOP很好地解决了这个问题，AOP允许我们把公共的代码抽取出来，单独实现。

## 6.2 Spring声明式事务的使用
### 6.2.1 Spring声明式数据库事务约定
声明式事务使用@Transactional进行标注，这个注解可以修饰类和方法，用在类上时，所有的public，非static方法都将启用事务，spring事务的约定如下图所示：

![spring事务约定](/images/posts/springboot/chapter6_1.png)

### 6.2.2 @Transactional的配置项
Spring的事务允许进行许多配置，比如传播行为，隔离级别，超时时间，只读事务，回滚设置