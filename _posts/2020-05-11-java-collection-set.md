---
layout: post
title: Java的集合(collections)之Set
categories: Java
description: 讲解Java中的Set
keywords: Java, collections, 集合，set
---

Set类似于一个罐子，我们只要把元素扔进去就能存起来，但要注意的是set不允许有相同的元素而且没有顺序可言。比如一个装满糖的罐子，我们不能给这些罐子里的糖排一个顺序。

虽然表面上没有顺序，但是Set有一个内部的顺序要维护，对于Integer和String这类Java定义好的类，我们不需要担心，但是如果你自己要定义一个类出来，就需要注意set是如何维护这个顺序的了。不同的set有不同的维护方法，因此，不同的set实现类不仅有不同的方法，在能放进去的对象类型也有所不同。

# 1. Set简介

先把set的[Set的官方文档](https://docs.oracle.com/javase/8/docs/api/java/util/Set.html)拿出来，从文档里我们能看到给set的定义是 A collection that contains no duplicate elements. More formally, sets contain no pair of elements `e1` and `e2` such that `e1.equals(e2)`, and at most one null element. As implied by its name, this interface models the mathematical *set* abstraction.

Set具体有三个实现类，HashSet，TreeSet，LinkedHashSet，它们的特点如下：

| Set (interface) | Each element that you add to the Set must be unique; otherwise, the Set doesn’t add the duplicate element. Elements added to a Set must at least define equals( ) to establish object uniqueness. Set has exactly the same interface as Collection. The Set interface does not guarantee that it will maintain its elements in any particular order. |
| --------------- | ------------------------------------------------------------ |
| HashSet*        | For Sets where fast lookup time is important. Elements must also define hashCode( ). |
| TreeSet         | An ordered Set backed by a tree. This way, you can extract an ordered sequence from a Set. Elements must also implement the Comparable interface. |
| LinkedHashSet   | Has the lookup speed of a HashSet, but internally maintains the order in which you add the elements (the insertion order) using a linked list. Thus, when you iterate through the Set, the results appear in insertion order. Elements must also define hashCode( ). |

如果没有其它约束条件，一般使用HashSet，因为它的速度比较快。

Set的一般使用：

```java
class SetType {
  int i;
  public SetType(int n) { i = n; }
  public boolean equals(Object o) {
    return o instanceof SetType && (i == ((SetType)o).i);
  }
  public String toString() { return Integer.toString(i); }
}

class HashType extends SetType {
  public HashType(int n) { super(n); }
  public int hashCode() { return i; }
}

class TreeType extends SetType
implements Comparable<TreeType> {
  public TreeType(int n) { super(n); }
  public int compareTo(TreeType arg) {
    return (arg.i < i ? -1 : (arg.i == i ? 0 : 1));
  }
}

public class TypesForSets {
  static <T> Set<T> fill(Set<T> set, Class<T> type) {
    try {
      for(int i = 0; i < 10; i++)
          set.add(
            type.getConstructor(int.class).newInstance(i));
    } catch(Exception e) {
      throw new RuntimeException(e);
    }
    return set;
  }
  static <T> void test(Set<T> set, Class<T> type) {
    fill(set, type);
    fill(set, type); // Try to add duplicates
    fill(set, type);
    System.out.println(set);
  }
  public static void main(String[] args) {
    test(new HashSet<HashType>(), HashType.class);
    test(new LinkedHashSet<HashType>(), HashType.class);
    test(new TreeSet<TreeType>(), TreeType.class);
    // Things that don't work:
    test(new HashSet<SetType>(), SetType.class);
    test(new HashSet<TreeType>(), TreeType.class);
    test(new LinkedHashSet<SetType>(), SetType.class);
    test(new LinkedHashSet<TreeType>(), TreeType.class);
    try {
      test(new TreeSet<SetType>(), SetType.class);
    } catch(Exception e) {
      System.out.println(e.getMessage());
    }
    try {
      test(new TreeSet<HashType>(), HashType.class);
    } catch(Exception e) {
      System.out.println(e.getMessage());
    }
  }
} /* Output: (Sample)
[2, 4, 9, 8, 6, 1, 3, 7, 5, 0]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
[9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
[9, 9, 7, 5, 1, 2, 6, 3, 0, 7, 2, 4, 4, 7, 9, 1, 3, 6, 2, 4, 3, 0, 5, 0, 8, 8, 8, 6, 5, 1]
[0, 5, 5, 6, 5, 0, 3, 1, 9, 8, 4, 2, 3, 9, 7, 3, 4, 4, 0, 7, 1, 9, 6, 2, 1, 8, 2, 8, 6, 7]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
java.lang.ClassCastException: SetType cannot be cast to java.lang.Comparable
java.lang.ClassCastException: HashType cannot be cast to java.lang.Comparable
*///:~

```

# 2. Set各种实现类

## 2.1 HashSet详解

先说说HashSet的构造方法:

```java
private transient HashMap<E,Object> map;

/**
     * Constructs a new, empty set; the backing <tt>HashMap</tt> instance has
     * default initial capacity (16) and load factor (0.75).
     */
    public HashSet() {
        map = new HashMap<>();
    }

    /**
     * Constructs a new set containing the elements in the specified
     * collection.  The <tt>HashMap</tt> is created with default load factor
     * (0.75) and an initial capacity sufficient to contain the elements in
     * the specified collection.
     *
     * @param c the collection whose elements are to be placed into this set
     * @throws NullPointerException if the specified collection is null
     */
    public HashSet(Collection<? extends E> c) {
        map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c);
    }

    /**
     * Constructs a new, empty set; the backing <tt>HashMap</tt> instance has
     * the specified initial capacity and the specified load factor.
     *
     * @param      initialCapacity   the initial capacity of the hash map
     * @param      loadFactor        the load factor of the hash map
     * @throws     IllegalArgumentException if the initial capacity is less
     *             than zero, or if the load factor is nonpositive
     */
    public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<>(initialCapacity, loadFactor);
    }

    /**
     * Constructs a new, empty set; the backing <tt>HashMap</tt> instance has
     * the specified initial capacity and default load factor (0.75).
     *
     * @param      initialCapacity   the initial capacity of the hash table
     * @throws     IllegalArgumentException if the initial capacity is less
     *             than zero
     */
    public HashSet(int initialCapacity) {
        map = new HashMap<>(initialCapacity);
    }

    /**
     * Constructs a new, empty linked hash set.  (This package private
     * constructor is only used by LinkedHashSet.) The backing
     * HashMap instance is a LinkedHashMap with the specified initial
     * capacity and the specified load factor.
     *
     * @param      initialCapacity   the initial capacity of the hash map
     * @param      loadFactor        the load factor of the hash map
     * @param      dummy             ignored (distinguishes this
     *             constructor from other int, float constructor.)
     * @throws     IllegalArgumentException if the initial capacity is less
     *             than zero, or if the load factor is nonpositive
     */
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
```

可以看到HashSet有许多构造方法，不过底层还是用了HashMap来实现。HashSet保证去重的方法是把每一个要添加的元素作为HashMap的键值(key)插入到HashMap中，因为HashMap不能有重复的键值，自然保证了HashSet中元素的唯一。

再来看看HashSet的添加方法：

```java
// Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();
    
/**
     * Adds the specified element to this set if it is not already present.
     * More formally, adds the specified element <tt>e</tt> to this set if
     * this set contains no element <tt>e2</tt> such that
     * <tt>(e==null&nbsp;?&nbsp;e2==null&nbsp;:&nbsp;e.equals(e2))</tt>.
     * If this set already contains the element, the call leaves the set
     * unchanged and returns <tt>false</tt>.
     *
     * @param e element to be added to this set
     * @return <tt>true</tt> if this set did not already contain the specified
     * element
     */
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
```

我们可以看到，在HashSet里添加值的时候，要添加的值作为键存储在HashMap中，而HashMap真正的值却是一个没有意义的object对象。

我们也可以从添加方法中理解官方定义中说null值可以被存在HashSet中，但是只允许存储一个。

HashSet的删除方法：

```java
// Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();

/**
     * Removes the specified element from this set if it is present.
     * More formally, removes an element <tt>e</tt> such that
     * <tt>(o==null&nbsp;?&nbsp;e==null&nbsp;:&nbsp;o.equals(e))</tt>,
     * if this set contains such an element.  Returns <tt>true</tt> if
     * this set contained the element (or equivalently, if this set
     * changed as a result of the call).  (This set will not contain the
     * element once the call returns.)
     *
     * @param o object to be removed from this set, if present
     * @return <tt>true</tt> if the set contained the specified element
     */
    public boolean remove(Object o) {
        return map.remove(o)==PRESENT;
    }
```

HashSet的删除方法也是调用HashMap的删除方法来完成删除的。

## 2.1 TreeSet详解

## 2.4 特殊的SortedSet：

SortedSet比较特殊，它继承了Set接口，不同的是，它的元素是有序的，所以它能实现一些普通set无法实现的功能。SortedSet的实现类就是TreeSet。

[SortedSet的官方文档](https://docs.oracle.com/javase/8/docs/api/java/util/SortedSet.html)

使用方法如下:

```java
public class SortedSetDemo {
  public static void main(String[] args) {
    SortedSet<String> sortedSet = new TreeSet<String>();
    Collections.addAll(sortedSet,
      "one two three four five six seven eight"
        .split(" "));
    print(sortedSet);
    String low = sortedSet.first();
    String high = sortedSet.last();
    print(low);
    print(high);
    Iterator<String> it = sortedSet.iterator();
    for(int i = 0; i <= 6; i++) {
      if(i == 3) low = it.next();
      if(i == 6) high = it.next();
      else it.next();
    }
    print(low);
    print(high);
    print(sortedSet.subSet(low, high));
    print(sortedSet.headSet(high));
    print(sortedSet.tailSet(low));
  }
} /* Output:
[eight, five, four, one, seven, six, three, two]
eight
two
one
two
[one, seven, six, three]
[eight, five, four, one, seven, six, three]
[one, seven, six, three, two]
*///:~
```

