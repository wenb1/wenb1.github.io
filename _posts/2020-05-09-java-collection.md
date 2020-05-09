---
layout: post
title: Java的集合(collections)
categories: Java
description: 讲解Java中的集合
keywords: Java, collections, 集合
---

Java的集合是非常重要的概念之一，Java本身的collection库和map库提供了一些存放对象的容器供我们使用。这篇文章以《**Java编程思想**》(**Thinking in Java**)为引子，同时再掺杂些自己的理解。

## 1. 集合的定义

集合实际就是一个能存放和操作一组对象的容器。

集合的结构图如下：

![集合结构图](/images/posts/java/collection_1.PNG)

从图中我们能看到，集合包含两类，一类是collection，另一类是map，可以看到collection实现了Iterator接口，而map并没有实现Iterator接口。collection和map实际也是两个接口，定义了集合的规范。

## 2. 集合的功能

collection集合实现的功能：

![collection功能](/images/posts/java/collection_2.PNG)

注意到collection接口并没有定义用于随机访问的get()方法，这是因为collection里包含了堆(set)，而堆有自己的一套内部顺序，这让随机访问没有意义。

map集合实现的功能：

![collection功能](/images/posts/java/collection_3.PNG)

![collection功能](/images/posts/java/collection_4.PNG)

## 3. 重要集合介绍

### 3.1 list

list的使用非常简单，一般用add()插入对象，用get()取出对象，用iterator()创建一个iterator。以下代码展示了如何使用list，包括LinkedList和ArrayList。

```java
public class Lists {
  private static boolean b;
  private static String s;
  private static int i;
  private static Iterator<String> it;
  private static ListIterator<String> lit;
  public static void basicTest(List<String> a) {
    a.add(1, "x"); // Add at location 1
    a.add("x"); // Add at end
    // Add a collection:
    a.addAll(Countries.names(25));
    // Add a collection starting at location 3:
    a.addAll(3, Countries.names(25));
    b = a.contains("1"); // Is it in there?
    // Is the entire collection in there?
    b = a.containsAll(Countries.names(25));
    // Lists allow random access, which is cheap
    // for ArrayList, expensive for LinkedList:
    s = a.get(1); // Get (typed) object at location 1
    i = a.indexOf("1"); // Tell index of object
    b = a.isEmpty(); // Any elements inside?
    it = a.iterator(); // Ordinary Iterator
    lit = a.listIterator(); // ListIterator
    lit = a.listIterator(3); // Start at loc 3
    i = a.lastIndexOf("1"); // Last match
    a.remove(1); // Remove location 1
    a.remove("3"); // Remove this object
    a.set(1, "y"); // Set location 1 to "y"
    // Keep everything that's in the argument
    // (the intersection of the two sets):
    a.retainAll(Countries.names(25));
    // Remove everything that's in the argument:
    a.removeAll(Countries.names(25));
    i = a.size(); // How big is it?
    a.clear(); // Remove all elements
  }
  public static void iterMotion(List<String> a) {
    ListIterator<String> it = a.listIterator();
    b = it.hasNext();
    b = it.hasPrevious();
    s = it.next();
    i = it.nextIndex();
    s = it.previous();
    i = it.previousIndex();
  }
  public static void iterManipulation(List<String> a) {
    ListIterator<String> it = a.listIterator();
    it.add("47");
    // Must move to an element after add():
    it.next();
    // Remove the element after the newly produced one:
    it.remove();
    // Must move to an element after remove():
    it.next();
    // Change the element after the deleted one:
    it.set("47");
  }
  public static void testVisual(List<String> a) {
    print(a);
    List<String> b = Countries.names(25);
    print("b = " + b);
    a.addAll(b);
    a.addAll(b);
    print(a);
    // Insert, remove, and replace elements
    // using a ListIterator:
    ListIterator<String> x = a.listIterator(a.size()/2);
    x.add("one");
    print(a);
    print(x.next());
    x.remove();
    print(x.next());
    x.set("47");
    print(a);
    // Traverse the list backwards:
    x = a.listIterator(a.size());
    while(x.hasPrevious())
      printnb(x.previous() + " ");
    print();
    print("testVisual finished");
  }
  // There are some things that only LinkedLists can do:
  public static void testLinkedList() {
    LinkedList<String> ll = new LinkedList<String>();
    ll.addAll(Countries.names(25));
    print(ll);
    // Treat it like a stack, pushing:
    ll.addFirst("one");
    ll.addFirst("two");
    print(ll);
    // Like "peeking" at the top of a stack:
    print(ll.getFirst());
    // Like popping a stack:
    print(ll.removeFirst());
    print(ll.removeFirst());
    // Treat it like a queue, pulling elements
    // off the tail end:
    print(ll.removeLast());
    print(ll);
  }
  public static void main(String[] args) {
    // Make and fill a new list each time:
    basicTest(
      new LinkedList<String>(Countries.names(25)));
    basicTest(
      new ArrayList<String>(Countries.names(25)));
    iterMotion(
      new LinkedList<String>(Countries.names(25)));
    iterMotion(
      new ArrayList<String>(Countries.names(25)));
    iterManipulation(
      new LinkedList<String>(Countries.names(25)));
    iterManipulation(
      new ArrayList<String>(Countries.names(25)));
    testVisual(
      new LinkedList<String>(Countries.names(25)));
    testLinkedList();
  }
} /* (Execute to see output) *///:~
```

### 3.2 Set

