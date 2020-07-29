---
title: leetcode-数组
date: 2020-7-25 10:36:42
categories: 
- leetcode
tag:
- java
---



# 1.两数之和

1.

两遍哈希表

一个简单的实现使用了两次迭代。在第一次迭代中，我们将每个元素的值和它的索引添加到表中。然后，在第二次迭代中，我们将检查每个元素所对应的目标元素（target - nums[i]target−nums[i]）是否存在于表中。注意，该目标元素不能是 nums[i]nums[i] 本身！

```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();

    for(int i = 0; i < nums.length; i++){
        map.put(nums[i],i);
    }

    for(int i = 0; i < nums.length; i++){
        int complement = target - nums[i];
        if (map.containsKey(complement) && map.get(complement) !=i){
            return  new int[]{i,map.get(complement)};
        }
    }
    throw new IllegalArgumentException("No two sum solution");
}
```

一遍哈希表

我们可以一次完成。在进行迭代并将元素插入到表中的同时，我们还会回过头来检查表中是否已经存在当前元素所对应的目标元素。如果它存在，那我们已经找到了对应解，并立即将其返回。

```java
public int[] twoSum2(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();

    for(int i = 0; i < nums.length; i++){
        int complement = target - nums[i];
        if (map.containsKey(complement)) {
            return new int[]{map.get(complement),i};
        }
        map.put(nums[i],i);
    }
    throw new IllegalArgumentException("No two sum solution");
}
```