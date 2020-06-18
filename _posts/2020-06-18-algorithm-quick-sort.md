---
layout: post
title: 快速排序(	Quick Sort)
categories: Algorithm
description: 讲解快速排序
keywords: Algorithm，算法，快速排序
---

快排的思想是这样的：如果要排序数组中下标从 p 到 r 之间的一组数据，我们选择 p 到 r 之间的任意一个数据作为 pivot（分区点）。

我们遍历 p 到 r 之间的数据，将小于 pivot 的放到左边，将大于 pivot 的放到右边，将 pivot 放到中间。经过这一步骤之后，数组 p 到 r 之间的数据就被分成了三个部分，前面 p 到 q-1 之间都是小于 pivot 的，中间是 pivot，后面的 q+1 到 r 之间是大于 pivot 的。

![快速排序1](/images/posts/algorithms/quick_sort_1.jpg)

动态示意图如下：

![快速排序2](/images/posts/algorithms/quick_sort_2.gif)

根据分治、递归的处理思想，我们可以用递归排序下标从 p 到 q-1 之间的数据和下标从 q+1 到 r 之间的数据，直到区间缩小为 1，就说明所有的数据都有序了。

如果我们用递推公式来将上面的过程写出来的话，就是这样：

```
递推公式：
quick_sort(p…r) = quick_sort(p…q-1) + quick_sort(q+1… r)

终止条件：
p >= r
```

实现如下：

```java
public class QuickSort {
    public static void quickSort(int[] array){
        sort(array, 0, array.length-1);
    }

    private static void sort(int[] array, int lo, int hi){
        if(lo>=hi)
            return;
        int j=partition(array, lo, hi);
        sort(array, lo, j-1);
        sort(array, j+1, hi);
    }

    private static int partition(int[] array, int lo, int hi){
        int i=lo, j=hi+1;
        int v=array[lo];
        while (true){
            while(array[++i]<v){
                if(i==hi)
                    break;
            }
            while(v<array[--j]){
                if(j==lo)
                    break;
            }
            if(i>=j)
                break;
            exch(array, i, j);
        }
        exch(array, lo, j);
        return j;
    }

    private static void exch(int[] a, int i, int j) {
        int swap = a[i];
        a[i] = a[j];
        a[j] = swap;
    }
}

```

归并排序中有一个 merge() 合并函数，我们这里有一个 partition() 分区函数。partition() 分区函数实际上我们前面已经讲过了，就是随机选择一个元素作为 pivot（一般情况下，可以选择 p 到 r 区间的最后一个元素），然后对 A[p…r]分区，函数返回 pivot 的下标。

分区的处理有点类似选择排序。我们通过游标 i 把 A[p…r-1]分成两部分。A[p…i-1]的元素都是小于 pivot 的，我们暂且叫它“已处理区间”，A[i…r-1]是“未处理区间”。我们每次都从未处理的区间 A[i…r-1]中取一个元素 A[j]，与 pivot 对比，如果小于 pivot，则将其加入到已处理区间的尾部，也就是 A[i]的位置。

数组的插入操作还记得吗？在数组某个位置插入元素，需要搬移数据，非常耗时。当时我们也讲了一种处理技巧，就是交换，在 O(1) 的时间复杂度内完成插入操作。这里我们也借助这个思想，只需要将 A[i]与 A[j]交换，就可以在 O(1) 时间复杂度内将 A[j]放到下标为 i 的位置。

文字不如图直观，所以我画了一张图来展示分区的整个过程。

![快速排序2](/images/posts/algorithms/quick_sort_3.jpg)

因为分区的过程涉及交换操作，如果数组中有两个相同的元素，比如序列 6，8，7，6，3，5，9，4，在经过第一次分区操作之后，两个 6 的相对先后顺序就会改变。所以，快速排序并**不是一个稳定的排序算法**。

