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

## 2.1 HashMap

从源码我们能看到底层是维护一个`Node<K,V>[]`类型的变量`table`，`table`实际是一个数组，里面存放的是`Node<K,V>`对像，而`Node<K,V>`对象可以作为节点连接其他`Node<K,V>`节点形成一个链表。所以说HashMap的实现方式其实是数组+链表的形式。

### 2.1.1 HashMap的构造方法

```java
/**
     * The next size value at which to resize (capacity * load factor).
     *
     * @serial
     */
    // (The javadoc description is true upon serialization.
    // Additionally, if the table array has not been allocated, this
    // field holds the initial array capacity, or zero signifying
    // DEFAULT_INITIAL_CAPACITY.)
    int threshold;

    /**
     * The load factor for the hash table.
     *
     * @serial
     */
    final float loadFactor;
/**
     * The maximum capacity, used if a higher value is implicitly specified
     * by either of the constructors with arguments.
     * MUST be a power of two <= 1<<30.
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * The load factor used when none specified in constructor.
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
/**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and load factor.
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }

    /**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and the default load factor (0.75).
     *
     * @param  initialCapacity the initial capacity.
     * @throws IllegalArgumentException if the initial capacity is negative.
     */
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    /**
     * Constructs an empty <tt>HashMap</tt> with the default initial capacity
     * (16) and the default load factor (0.75).
     */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }

    /**
     * Constructs a new <tt>HashMap</tt> with the same mappings as the
     * specified <tt>Map</tt>.  The <tt>HashMap</tt> is created with
     * default load factor (0.75) and an initial capacity sufficient to
     * hold the mappings in the specified <tt>Map</tt>.
     *
     * @param   m the map whose mappings are to be placed in this map
     * @throws  NullPointerException if the specified map is null
     */
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
```

从源码，我们能看到第一个构造方法`public HashMap(int initialCapacity, float loadFactor)`需要初始容量和负载作为参数初始化一个HashMap。`public HashMap(int initialCapacity)`则是调用第一构造方法并用默认的负载作为参数。无参构造方法`public HashMap()`则是使用默认的容量，并直接给`loadFactor`赋值一个默认的负载。`public HashMap(Map<? extends K, ? extends V> m)`这个构造方法的参数是另一个map，并把这个map里的键值对放到要创建的新map中。

2.1.1 HashMap的构造方法