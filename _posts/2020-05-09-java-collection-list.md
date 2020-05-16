---
layout: post
title: Java集合(collections)之List
categories: Java
description: 讲解Java中的List
keywords: Java, collections, 集合，list
---

Java的集合是非常重要的概念之一，Java本身的collection库和map库提供了一些存放对象的容器供我们使用。这篇文章以《**Java编程思想**》(**Thinking in Java**)为引子，同时再掺杂些自己的理解。

# 1. 集合的定义

集合实际就是一个能存放和操作一组对象的容器。

集合的结构图如下：

![集合结构图](/images/posts/java/collection_1.PNG)

从图中我们能看到，集合包含两类，一类是collection，另一类是map，可以看到collection继承了了Iterable接口，而map并没有继承Iterable接口。collection和map实际也是两个接口，定义了集合的规范。

# 2. 集合的功能

**collection接口规定的功能**：

[collection官方文档](https://docs.oracle.com/javase/8/docs/api/java/util/Collection.html)

注意到collection接口并没有定义用于随机访问的get()方法，这是因为collection里包含了堆(set)，而堆有自己的一套内部顺序，这让随机访问没有意义。

**map接口规定的功能**：

[map官方文档](https://docs.oracle.com/javase/8/docs/api/java/util/Map.html)

# 3. List介绍

list接口继承了collection接口，并提供了一些具体的实现类，比如LinkedList和ArrayList。一般list的使用非常简单，用add()插入对象，用get()取出对象，用iterator()创建一个iterator。下面，我们具体讲解LinkedList和ArrayList。

### 3.1 LinkedList详解

在数据结构中，我们学过单向链表和双向链表。在Java中链表(LinkedList)的具体实现是双向链表还是单向链表？

链表是由节点(node)串联起来的，通过节点，我们可以知道这个链表是双向还是单向，Java在链表中的节点定义如下：

```java
private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

由此可见，节点中有两个引用，所以Java的链表是一个双向链表。因此，Java中的链表具有双向链表的优势，存储不需要分配连续的内存空间，插入和删除非常快速，但是查找很慢。

LinkedList有两个构造方法:

```java
/**
     * Constructs an empty list.
     */
    public LinkedList() {
    }

    /**
     * Constructs a list containing the elements of the specified
     * collection, in the order they are returned by the collection's
     * iterator.
     *
     * @param  c the collection whose elements are to be placed into this list
     * @throws NullPointerException if the specified collection is null
     */
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
```

LinkedList具体实现的功能在[LinkedList官方文档](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html)中。

### 3.2 ArrayList详解

ArrayList 是数组队列，相当于动态数组。与Java中的数组相比，它的容量能动态增长。

通过源码，我们可以看到，ArrayList的实现其实是依赖一个数组(array)，通过维护这个数组来维护整个ArrayList。既然底层是数组，那么如何动态扩容呢？**源码是这样做的**：

```java
/**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
	
	private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

	private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }

	private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
	/**
     * The maximum size of array to allocate.
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
/**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

根据这一系列源码，我们能看到，扩容的过程如下：

1. 如果一开始我们调用了无参的ArrayList构造方法，那它底层的数组长度就是0。之后，我们如果想要添加一个新元素，显然长度为0的数组是不能存放任何元素的，所以我们就需要构建一个新的数组。源码是这样做的：先比较规定的默认长度，也就是10，与添加这个元素之后的长度，也就是1，之后取最大的数作为数组长度，也就是10。所以无参构建ArrayList，添加一个元素之后的底层数组长度是10。
2. 之后如果需要存放的元素数目超过了10，比如11，我们把11叫做minCapacity，也就是需要存放所有元素所需要的最小容量。如果发现容量不够了，需要扩容到原来容量的3/2倍。如果扩容之后，扩容之后的长度还是比minCapacity小，那么我们就把minCapacity定为扩充容量。另一种情况是，需要的容量比规定的数组最大容量还要大，那么我们就把最大的整数值作为扩充容量。

我们知道对于数组，它的优势在于随机访问，也就是给定一个索引，可以使用O(1)的时间访问索引对应的值。缺点也很明显，对于在非末尾的指定位置添加和删除这两种情况，需要整体移动添加位置和删除位置后的元素。

**源码的在指定索引位置添加元素**：

```java
/**
     * A version of rangeCheck used by add and addAll.
     */
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
/**
     * Constructs an IndexOutOfBoundsException detail message.
     * Of the many possible refactorings of the error handling code,
     * this "outlining" performs best with both server and client VMs.
     */
    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }
/**
     * Inserts the specified element at the specified position in this
     * list. Shifts the element currently at that position (if any) and
     * any subsequent elements to the right (adds one to their indices).
     *
     * @param index index at which the specified element is to be inserted
     * @param element element to be inserted
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
```

我们可以看到，凡是添加方法都要考虑扩容问题，因为这个方法还指定了索引位置，所以还要考虑索引越界的问题。在通过了检查之后，源码调用了System.arraycopy()方法，整体平移了一个单位索引后面的元素。再在指定位置插入元素。

**源码的删除方法**：

```java
/**
     * Removes the element at the specified position in this list.
     * Shifts any subsequent elements to the left (subtracts one from their
     * indices).
     *
     * @param index the index of the element to be removed
     * @return the element that was removed from the list
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }

    /**
     * Removes the first occurrence of the specified element from this list,
     * if it is present.  If the list does not contain the element, it is
     * unchanged.  More formally, removes the element with the lowest index
     * <tt>i</tt> such that
     * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>
     * (if such an element exists).  Returns <tt>true</tt> if this list
     * contained the specified element (or equivalently, if this list
     * changed as a result of the call).
     *
     * @param o element to be removed from this list, if present
     * @return <tt>true</tt> if this list contained the specified element
     */
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
/*
     * Private remove method that skips bounds checking and does not
     * return the value removed.
     */
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```

1. 在指定位置删除元素和在指定位置添加元素很类似，不同的是在移动完所有元素后，最后一个元素要交给GC线程回收。

2. 对于删除一个元素的方法，我们注意到null也是可以被删除的，我们之前看的删除过程被封装到了fastRemove()方法里。

ArrayList的实现的具体实现的方法在[ArrayList的官方文档](https://docs.oracle.com/javase/8/docs/api/java/util/ArrayList.html)中。
