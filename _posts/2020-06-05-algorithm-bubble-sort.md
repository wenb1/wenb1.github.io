---
layout: post
title: 冒泡排序(Bubble Sort)
categories: Algorithm
description: 讲解冒泡排序
keywords: Algorithm，算法，冒泡排序
---

冒泡排序，类似于水中冒泡，较大的数沉下去，较小的数慢慢冒起来，假设从小到大，即为较大的数慢慢往后排，较小的数慢慢往前排。

冒泡排序只会操作相邻的两个数据。每次冒泡操作都会对相邻的两个元素进行比较，看是否满足大小关系要求。如果不满足就让它俩互换。一次冒泡会让至少一个元素移动到它应该在的位置，重复 n 次，就完成了 n 个数据的排序工作。

举个例子：

假设我们要对一组4，5，6，3，2，1的数据从小到大排序。

那么第一次排序过程为：

![冒泡排序过程1](/images/posts/algorithms/bubble_sort_1.jpg)

在第一次冒泡排序之后，6就在正确的位置上了。

接下来，我们只需要再经过5次冒泡排序，整个数组就是有序的了，一共6次冒泡排序。

![冒泡排序过程2](/images/posts/algorithms/bubble_sort_2.jpg)

实际上，刚讲的冒泡过程还可以优化。当某次冒泡操作已经没有数据交换时，说明已经达到完全有序，不用再继续执行后续的冒泡操作。

这里有一张动态图说明冒泡排序的过程：

![冒泡排序过程3](/images/posts/algorithms/bubble_sort_3.gif)

我们还能注意到，第一冒泡排序一定会把最大的元素排好，放到末尾。第二次排好第二大的元素，以此类推。

具体实现代码如下：

```java
public class BubbleSort {

    // array为待排序数组
    public static void bubbleSort(int[] array){
        int length=array.length;

        if(length<=1){
            return;
        }

        for(int i=0;i<length;i++){
            // 提前退出冒泡循环的标志位
            boolean flag=false;
            for(int j=0;j<length-i-1;j++){
                if(array[j]>array[j+1]){
                    int temp=array[j];
                    array[j]=array[j+1];
                    array[j+1]=temp;
                    flag=true;
                }
            }
            if(!flag) break;
        }
    }
}
```

冒泡的过程只涉及相邻数据的交换操作，只需要常量级的临时空间，所以它的空间复杂度为 O(1)，是一个**原地排序**算法。

在冒泡排序中，只有交换才可以改变两个元素的前后顺序。为了保证冒泡排序算法的稳定性，当有相邻的两个元素大小相等的时候，我们不做交换，相同大小的数据在排序前后不会改变顺序，所以冒泡排序是**稳定排序**算法。

最好情况下，要排序的数据已经是有序的了，我们只需要进行一次冒泡操作，就可以结束了，所以最好情况时间复杂度是 O(n)。而最坏的情况是，要排序的数据刚好是倒序排列的，我们需要进行 n 次冒泡操作，所以最坏情况时间复杂度为O(n^2)。平均情况下的时间复杂度是 O(n^2)。