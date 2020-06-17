---
layout: post
title: 归并排序(Merge Sort)
categories: Algorithm
description: 讲解归并排序
keywords: Algorithm，算法，归并排序
---

归并排序的核心思想还是蛮简单的。如果要排序一个数组，我们先把数组从中间分成前后两部分，然后对前后两部分分别排序，再将排好序的两部分合并在一起，这样整个数组就都有序了。

![归并排序1](/images/posts/algorithms/merge_sort_1.jpg)

动态示意图如下：

![归并排序1](/images/posts/algorithms/merge_sort_2.gif)

归并排序使用的就是分治思想。分治，顾名思义，就是分而治之，将一个大问题分解成小的子问题来解决。小的子问题解决了，大问题也就解决了。从刚才的描述，你有没有感觉到，分治思想跟我们前面讲的递归思想很像。是的，分治算法一般都是用递归来实现的。分治是一种解决问题的处理思想，递归是一种编程技巧，这两者并不冲突。

现在看看如何用递归代码来实现归并排序。

首先，写递归代码的技巧就是，分析得出递推公式，然后找到终止条件，最后将递推公式翻译成递归代码。所以，要想写出归并排序的代码，我们先写出归并排序的递推公式。

```java
递推公式：
merge_sort(p…r) = merge(merge_sort(p…q), merge_sort(q+1…r))

终止条件：
p >= r 不用再继续分解
```

merge_sort(p…r) 表示，给下标从 p 到 r 之间的数组排序。我们将这个排序问题转化为了两个子问题，merge_sort(p…q) 和 merge_sort(q+1…r)，其中下标 q 等于 p 和 r 的中间位置，也就是 (p+r)/2。当下标从 p 到 q 和从 q+1 到 r 这两个子数组都排好序之后，我们再将两个有序的子数组合并在一起，这样下标从 p 到 r 之间的数据就也排好序了。

实现如下：

```java
public class MergeSort {
    public static void mergeSort(int[] array){
        sort(array, 0, array.length-1);
    }

    private static void sort(int[] array, int lo, int hi){

        if(lo>=hi)
            return;

        int mid=(lo+hi)/2;
        sort(array, lo, mid);
        sort(array, mid+1, hi);
        merge(array, lo, mid, hi);
    }

    private static void merge(int[] array, int lo, int mid, int hi){
        int[] aux=new int[array.length];
        int i=lo, j=mid+1;

        for(int k=lo;k<=hi;k++){
            aux[k]=array[k];
        }

        for(int k=lo;k<=hi;k++){
            if(i>mid)
                array[k]=aux[j++];
            else if(j>hi)
                array[k]=aux[i++];
            else if(aux[j]<aux[i])
                array[k]=aux[j++];
            else
                array[k]=aux[i++];
        }
    }
}

```

`merge`这个函数的作用就是，将已经有序的 A[p…q]和 A[q+1…r]合并成一个有序的数组，并且放入 A[p…r]。

归并排序稳不稳定关键要看`merge()`函数，也就是两个有序子数组合并成一个有序数组的那部分代码。在合并的过程中，如果 A[p…q]和 A[q+1…r]之间有值相同的元素，那我们可以像伪代码中那样，先把 A[p…q]中的元素放入 tmp 数组。这样就保证了值相同的元素，在合并前后的先后顺序不变。所以，归并排序是一个**稳定的排序算法**。

