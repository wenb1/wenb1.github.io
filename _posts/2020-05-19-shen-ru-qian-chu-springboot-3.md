---
layout: post
title: 《深入浅出 Springboot 2.x》第3章 全注解下的Spring IoC
categories: Springboot
description: 了解springboot的IoC
keywords: Springboot, IoC
---

IoC 容器是Spring 的核心，可以说Spring 是一种基于IoC容器编程的框架。Spring Boot 是基于注解的开发Spring IoC 。

# 第3章 全注解下的Spring IoC

IoC 是一种通过描述来生成或者获取对象的技术，而这个技术不是Spring 甚至不是Java 独有的。对于Java 初学者更多的时候所熟悉的是使用new 关键字来创建对象，而在Spring 中则不是，它是通过描述来创建对象。只是Spring Boot 并不建议使用XML ，而是通过注解的描述生成对象。

一个系统可以生成各种对象，井且这些对象都需要进行管理。还值得一提的是，对象之间并不是孤立的，它们之间还可能存在依赖的关系。在Spring中把每一个
需要管理的对象称为Spring Bean （简称Bean ），而Spring 管理这些Bean 的容器，被我们称为SpringIoC 容器（或者简称IoC 容器） 。IoC 容器需要具备两个基本的功能：

- 通过描述管理Bean ， 包括发布和获取Bean
- 通过描述完成Bean 之间的依赖关系。
  

## 3.1 IoC容器简介

Spring IoC 容器是一个管理Bean 的容器，在Spring 的定义中，它要求所有的IoC 容器都需要实现接口BeanFactory ，它是一个顶级容器接口。

由于BeanFactory 的功能还不够强大，因此Spring 在BeanFactory 的基础上， 还设计了一个更为高级的接口ApplicationContext 。它是BeanFactory 的子接口之一， 在Spring 的体系中BeanFactory和ApplicationContext 是最为重要的接口设计，在现实中我们使用的大部分Spring IoC 容器是ApplicationContext 接口的实现类。