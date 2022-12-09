---
title: LeetCode189
date: 2022-12-08 14:54:23
tags:
- LeetCode
categories:
- xieajiu
description: "[轮转数组](https://leetcode.cn/problems/rotate-array/)环状替换方法"
---

#### 题目介绍

给你一个数组，将数组中的元素向右轮转 `k` 个位置，其中 `k` 是非负数。

```
输入: nums = [1,2,3,4,5,6,7], k = 3
输出: [5,6,7,1,2,3,4]
解释:
向右轮转 1 步: [7,1,2,3,4,5,6]
向右轮转 2 步: [6,7,1,2,3,4,5]
向右轮转 3 步: [5,6,7,1,2,3,4]
```

#### 环状替换解决方案

变量解释：

> `arrayLength`数组长度
>
> `k`移动步长
>
> `startIndex` 开始下标（默认从0开始）
>
> `moveCount`移动次数

先思考最简单的情况，就是当`k=1`的时候。对于任意数组只需以此顺移即可，如下图：

{% asset_img 001.png 示意图 %}

数组元素一共移动了`4`次，此时下标正好移动了开始位置，且数据所有元素都已经移动。然后考虑下特殊情况，就是当(`startIndex` + `moveCount` * `k`) % `arrayLength` = `startIndex`，且`moveCount` < `arrayLength`，意味着一次循环不足以移动所有数组元素，此时`startIndex`需要`+1`，如下图：

{% asset_img 002.png 示意图 %}

直到所有元素都被移动完成，即`moveCount == arrayLength`。

代码如下：

```java
public class Solution {

    /**
     * 轮转数组
     * @param nums 数组
     * @param k 步长
     */
    public void rotate(int[] nums, int k) {
        // 跳转次数
        k = k % nums.length;
        /**
         * next --> 下个元素的坐标
         * moveCount --> 循环的次数
         * temp --> 迁移的值
         * startIndex --> 起始下标
         */
        int moveCount = 0;
        int startIndex = 0;
        for (int next = 0, temp = nums[0]; moveCount < nums.length; moveCount++) {
            // 当前元素要移动下一个地方的下标
            next = (next + k) % nums.length;
            // 使用异或运算对两个变量的值进行替换
            temp = temp ^ nums[next];
            nums[next] = temp ^ nums[next];
            temp = temp ^ nums[next];
            if (next % nums.length == startIndex && startIndex < nums.length - 1) {
                // 特殊情况，一次循环不足以遍历所有元素，开始下标 +1，进行下次循环
                next = ++startIndex;
                temp = nums[next];
            }
        }
    }
}
```

