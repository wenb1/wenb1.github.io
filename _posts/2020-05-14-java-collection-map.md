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

## 2.2 LinkedHashMap

LinkedHashMap继承于HashMap，LinkedHashMap和HashMap的区别是，LinkedHashMap是有序的，而HashMap是无序的，LinkedHashMap的默认顺序是插入顺序。

### 2.2.1 LinkedHashMap的构造方法

```java
/**
     * Constructs an empty insertion-ordered <tt>LinkedHashMap</tt> instance
     * with the specified initial capacity and load factor.
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
    public LinkedHashMap(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
        accessOrder = false;
    }

    /**
     * Constructs an empty insertion-ordered <tt>LinkedHashMap</tt> instance
     * with the specified initial capacity and a default load factor (0.75).
     *
     * @param  initialCapacity the initial capacity
     * @throws IllegalArgumentException if the initial capacity is negative
     */
    public LinkedHashMap(int initialCapacity) {
        super(initialCapacity);
        accessOrder = false;
    }

    /**
     * Constructs an empty insertion-ordered <tt>LinkedHashMap</tt> instance
     * with the default initial capacity (16) and load factor (0.75).
     */
    public LinkedHashMap() {
        super();
        accessOrder = false;
    }

    /**
     * Constructs an insertion-ordered <tt>LinkedHashMap</tt> instance with
     * the same mappings as the specified map.  The <tt>LinkedHashMap</tt>
     * instance is created with a default load factor (0.75) and an initial
     * capacity sufficient to hold the mappings in the specified map.
     *
     * @param  m the map whose mappings are to be placed in this map
     * @throws NullPointerException if the specified map is null
     */
    public LinkedHashMap(Map<? extends K, ? extends V> m) {
        super();
        accessOrder = false;
        putMapEntries(m, false);
    }

    /**
     * Constructs an empty <tt>LinkedHashMap</tt> instance with the
     * specified initial capacity, load factor and ordering mode.
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @param  accessOrder     the ordering mode - <tt>true</tt> for
     *         access-order, <tt>false</tt> for insertion-order
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
    public LinkedHashMap(int initialCapacity,
                         float loadFactor,
                         boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }
```

可以看到，因为LinkedHashMap继承于HashMap，所以LinkedHashMap中调用了`super`也就是父类的构造方法去初始化一个LinkedHashMap。把accessOrder设置为false，这就跟存储的顺序有关了，LinkedHashMap存储数据是有序的，而且分为两种：插入顺序和访问顺序。

这里accessOrder设置为false，表示不是访问顺序而是插入顺序存储的，这也是默认值，表示LinkedHashMap中存储的顺序是按照调用put方法插入的顺序进行排序的。

### 2.2.2 LinkedHashMap的`put`方法

LinkedHashMap的源码中没有`put`方法，所以我们可以看出，它是调用的父类HashMap的put方法来实现添加的。

### 2.2.3 LinkedHashMap的`remove`方法

同样，LinkedHashMap调用的是父类HashMap的`remove`方法

### 2.2.4 LinkedHashMap的`get`方法

```java
public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) == null)
            return null;
        if (accessOrder)
            afterNodeAccess(e);
        return e.value;
    }

void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;
        if (accessOrder && (last = tail) != e) {
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
            p.after = null;
            if (b == null)
                head = a;
            else
                b.after = a;
            if (a != null)
                a.before = b;
            else
                last = b;
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p;
            ++modCount;
        }
    }
```