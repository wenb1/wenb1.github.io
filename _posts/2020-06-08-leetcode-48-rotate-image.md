---
layout: post
title: 48. Rotate Image
tags: Array
description: 48.Rotate Image
keywords: Array，算法，leetcode
---

You are given an *n* x *n* 2D matrix representing an image. Rotate the image by 90 degrees (clockwise).

# 解法一

先按对角线交换元素，之后再交换对称列。

![48-1](/images/posts/leetcode/48_1.jpg)

```java
class Solution {
    public void rotate(int[][] matrix) {
        int length=matrix[0].length;
        
        for(int i=0;i<length;i++){
            int j=i+1;
            while(j<length){
                int temp=matrix[i][j];
                matrix[i][j]=matrix[j][i];
                matrix[j][i]=temp;
                j++;
            }
        }
        
        for(int i=0;i<length/2;i++){
            for(int k=0;k<length;k++){
                int temp=matrix[k][i];
                matrix[k][i]=matrix[k][length-1-i];
                matrix[k][length-1-i]=temp;
            }
        }
    }
}
```

时间复杂度：O（n²）

空间复杂度：O（1）

------

**参考文章**：

https://leetcode.wang/leetCode-48-Rotate-Image.html