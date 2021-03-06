---
layout: post
title: 选择排序(Selection Sort)
categories: Algorithm
description: 讲解选择排序
keywords: Algorithm，算法，选择排序
---

选择排序算法的实现思路有点类似插入排序，也分已排序区间和未排序区间。但是选择排序每次会从未排序区间中找到最小的元素，将其放到已排序区间的末尾。

![选择排序1](/images/posts/algorithms/selection_sort_1.jpg)

动态示意图：

![选择排序2](/images/posts/algorithms/selection_sort_2.gif)

实现代码如下：

```java
public class SelectionSort {
    public static void selectionSort(int[] array){
        int length=array.length;

        if(length<=1)
            return;

        for(int i=0;i<length;i++){
            int min=i;
            for(int j=i+1;j<length;j++){
                if(array[j]<array[min]){
                    min=j;
                }
            }
            int temp=array[i];
            array[i]=array[min];
            array[min]=temp;
        }
    }
}
```

选择排序空间复杂度为 O(1)，是一种**原地排序**算法。选择排序的最好情况时间复杂度、最坏情况和平均情况时间复杂度都为 O(n^2)。

选择排序是一种不稳定的排序算法。从前面画的那张图中，可以看出，选择排序每次都要找剩余未排序元素中的最小值，并和前面的元素交换位置，这样破坏了稳定性。比如 5，8，5，2，9 这样一组数据，使用选择排序算法来排序的话，第一次找到最小元素 2，与第一个 5 交换位置，那第一个 5 和中间的 5 顺序就变了，所以就不稳定了。正是因此，相对于冒泡排序和插入排序，选择排序就稍微逊色了。