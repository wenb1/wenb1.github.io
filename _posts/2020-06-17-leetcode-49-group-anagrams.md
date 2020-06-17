---
layout: post
title: 49. Group Anagrams
tags: String
description: 49.Group Anagrams
keywords: String，算法，leetcode
---

Given an array of strings, group anagrams together.

```markdown
Input: ["eat", "tea", "tan", "ate", "nat", "bat"],
Output:
[
  ["ate","eat","tea"],
  ["nat","tan"],
  ["bat"]
]
```

# 我的解法

```java
public List<List<String>> groupAnagrams(String[] strs) {
        List<List<String>> result=new ArrayList();
        List<String> tempResult=new ArrayList();
        Set<Integer> set=new HashSet();
        String[] strs_copy=new String[strs.length];
        
        for(int i=0;i<strs.length;i++){
            strs_copy[i]=strs[i];
            char[] chars = strs[i].toCharArray();
            Arrays.sort(chars);
            strs[i] = new String(chars);
        }
        
        for(int j=0;j<strs.length;j++){
            if(!set.contains(j)){
                set.add(j);
                tempResult.add(strs_copy[j]);
            }else
                continue;
            for(int k=0;k<strs.length;k++){
                if(strs[j].equals(strs[k]) && j!=k){
                    set.add(k);
                    tempResult.add(strs_copy[k]);
                }
            }
            result.add(new ArrayList(tempResult));
            tempResult.clear();
        }
        
        return result;
        
    }
```

主要思路是把原来数组里的元素都复制到新的数组里，在复制的过程中还要对每个字符串按字母顺序排序，这样能节省一部分时间。之后再比较每个字符串就比较容易了，因为字符串已经被重新排序了，只要含有相同的字母，排序后的字母顺序都是相同的。我们找到相同的字符串，添加进去，然后把下标记录下来，避免重复。

# 解法一

将每个字符串按照字母顺序排序，这样的话就可以把 eat，tea，ate 都映射到 aet。

![49_1](/images/posts/leetcode/49_1.jpg)

这个方法是我的方法的改进，实际上也对字符串进行了排序，但是使用了hashmap和排序后的值作为键，非常巧妙。

```java
public List<List<String>> groupAnagrams(String[] strs) {
    HashMap<String, List<String>> hash = new HashMap<>();
    for (int i = 0; i < strs.length; i++) {
        char[] s_arr = strs[i].toCharArray();
        //排序
        Arrays.sort(s_arr);
        //映射到 key
        String key = String.valueOf(s_arr); 
        //添加到对应的类中
        if (hash.containsKey(key)) {
            hash.get(key).add(strs[i]);
        } else {
            List<String> temp = new ArrayList<String>();
            temp.add(strs[i]);
            hash.put(key, temp);
        }

    }
    return new ArrayList<List<String>>(hash.values()); 
}

```

------

**参考文章**：

https://leetcode.wang/leetCode-49-Group-Anagrams.html