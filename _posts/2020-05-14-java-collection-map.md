---
layout: post
title: Java的集合(collections)之Map
categories: Java
description: 讲解Java中的Map
keywords: Java, collections, 集合，map
---

Map实际是存储一系列键值对映射的集合，通过键，我们能找到对应的值。这种一一对应的例子，在生活中也有很多，比如车牌号和车，通过唯一的车牌号能找到对应的车辆信息。

# 1. Map的特点

通过之前的博客，我们知道集合分两种，一种是collection分支，另一种则是map分支。collection分支中的list也好，set也好，都是存储一个对象的，只要把要存储的对象扔进去就好，而map分支则需要存储键值对，换句话说也就是要存两个值。map还有如下几个特点：

1. map不能存储重复的键，并且一个键只能对应最多一个值
2. map的顺序取决于具体map的实现类，TreeMap和LinkedHashMap有顺序，而HashMap则没有顺序可言

[Map的官方文档](https://docs.oracle.com/javase/8/docs/api/java/util/Map.html)

# 2. Map的实现类

Map的类继承关系如下：

![map结构图](/images/posts/java/collection_2.png)

## 2.1 HashMap

直接贴一篇美团技术团队的博客，从源码分析，非常详细。

[Java 8系列之重新认识HashMap](https://tech.meituan.com/2016/06/24/java-hashmap.html)