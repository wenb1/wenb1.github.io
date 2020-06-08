---
layout: post
title: 希尔排序(Shell Sort)
categories: Algorithm
description: 讲解希尔排序
keywords: Algorithm，算法，希尔排序
---

希尔排序是基于插入排序的。插入排序对于大的无序数组来说非常慢，因为要移动的元素只能和相邻的元素交换。比如，最小的元素在数组的最末尾，那么要进行n-1次交换才能把它移动到最前端。

希尔排序是插入排序的拓展，它通过允许元素与较远的另一个元素交换，形成一个相对有序的数组，之后再进行插入排序从而提高速度。

主要思想是重新排列数组，达到每h间隔的元素都是有序的。h可以先选一个很大的值，之后再慢慢减小h的值。

动态示意图：

![希尔排序1](/images/posts/algorithms/shell_sort_1.gif)

实现：

```java
public class ShellSort {
    public static void shellSort(int[] array){
        int length=array.length;
        int h=1;

        if(length<=1)
            return;

        while(h<length/3)
            h=h*3+1;

        while(h>=1){
            for(int i=h;i<length;i++){
                int value=array[i];
                int j=i-h;
                for(;j>=0;j=j-h){
                    if(array[j]>value){
                        array[j+h]=array[j];
                    }else
                        break;
                }
                array[j+h]=value;
            }
            h=h/3;
        }
    }
}
```

最好情况的时间复杂度是O(nlogn)，最坏为O(n^2)。