---
title: LeetCode 930 | 滑动窗口求解“和相同的二元子数组”
date: 2025-10-15 14:12:00
updated: 2025-10-15 14:12:00
comments: true
tags:
  - 滑动窗口
  - 数组
  - 前缀和
  - 哈希表
  - 第108场周赛
categories:
  - 算法题解
  - 滑动窗口与双指针
  - 不定长滑动窗口
permalink: posts/leetcode-930/
---
{% centerquote %}
LeetCode 第 930 题：[和相同的二元子数组](https://leetcode.cn/problems/binary-subarrays-with-sum/)。
该题要求统计和为特定值的二元子数组个数。直接用滑动窗口处理“和恰好等于 goal”的条件，在有连续 0 的情况下会比较棘手。但如果我们换一个角度，问题就迎刃而解了。
{% endcenterquote %}
<!--more-->
### 问题解读

题目给了我们一个只包含 0 和 1 的二元数组 `nums` 和一个整数 `goal`。我们需要统计出，有多少个**连续子数组**，其内部所有元素之和恰好等于 `goal`。

举个例子，对于 `nums = [1,0,1,0,1]`, `goal = 2`：
*   满足条件的子数组有 `[1,0,1]` (从索引0开始), `[1,0,1,0]` (从索引0开始), `[0,1,0,1]` (从索引1开始), `[1,0,1]` (从索引2开始)。等等，例子中的解释是 `[1,0,1]`, `[1,0,1]`, `[0,1,0,1]`。让我们重新看一下，`[1,0,1,0]` 的和也是 2。啊，示例的解释是 `[1,0,1]`、`[1,0,1,0]` 的和是2， `[0,1,0,1]` 的和是2，最后一个 `[1,0,1]` 是指从索引2开始的那个。所以总共有4个。
*   `[1,0,1]` (和为2)
*   `[1,0,1,0]` (和为2)
*   `[0,1,0,1]` (和为2)
*   `[1,0,1]` (从索引2开始，和为2)
*   因此，最终输出是 4。

直接去枚举所有子数组的复杂度是 O(N^2)，在 `3 * 10^4` 的数据规模下会超时，我们需要更高效的算法。

### 核心思路：巧妙转换为“最多为 K”

滑动窗口是解决子数组问题的利器。但如果我们尝试维护一个窗口，使其内部的和**恰好**等于 `goal`，会遇到麻烦。比如窗口 `[1,0,1]` 的和是 2，我们找到了一个解。当窗口向右扩展，加入一个 0 变成 `[1,0,1,0]`，和仍然是 2，这也是一个解。如果数组中有更多的连续 0，情况会变得更复杂，统计逻辑会很繁琐。

解决这个问题的金钥匙是**转换问题**。与其求“和恰好为 goal”，我们可以求“和最多为 goal”的子数组个数，再减去“和最多为 goal-1”的子数组个数。它们的差值，正好就是“和恰好为 goal”的子数组个数。

**`count(sum == goal) = count(sum <= goal) - count(sum <= goal - 1)`**

而求解“和最多为 k”的子数组个数，是滑动窗口的经典应用场景，逻辑非常清晰。

我们可以实现一个辅助函数 `atMost(k)`，用滑动窗口计算和小于等于 `k` 的子数组数量。

### 算法详解

1.  **实现 `atMost(k)` 辅助函数**
    *   我们使用 `left` 和 `right` 两个指针来定义一个滑动窗口 `[left, right]`。
    *   初始化 `left = 0`, `current_sum = 0`, `count = 0`。
    *   `right` 指针从左到右遍历整个数组，不断扩大窗口。
    *   每当 `right` 移动一步，将 `nums[right]` 加入 `current_sum`。
    *   检查 `current_sum` 是否大于 `k`。如果大于，就说明窗口需要收缩。我们不断地将 `left` 指针右移，并从 `current_sum` 中减去 `nums[left]`，直到 `current_sum <= k` 为止。
    *   在任意一步，当窗口 `[left, right]` 满足 `sum <= k` 时，所有以 `right` 为右端点，且左端点在 `[left, right]` 区间内的子数组，它们的和也都小于等于 `k`。这样的子数组共有 `right - left + 1` 个。我们将这个数目累加到总数 `count` 中。
    *   `right` 遍历结束后，返回 `count`。

2.  **计算最终结果**
    *   调用我们写好的辅助函数，分别计算 `atMost(goal)` 和 `atMost(goal - 1)`。
    *   最终的结果就是 `atMost(goal) - atMost(goal - 1)`。

### Python 代码实现

```python
from typing import List

class Solution:
    def numSubarraysWithSum(self, nums: List[int], goal: int) -> int:
        
        def atMost(k: int) -> int:
            """
            一个辅助函数，用来计算和最多为 k 的子数组数量。
            """
            if k < 0:
                return 0
            
            left, current_sum, count = 0, 0, 0
            for right in range(len(nums)):
                current_sum += nums[right]
                
                # 如果当前窗口的和大于 k，则从左边收缩窗口
                while current_sum > k:
                    current_sum -= nums[left]
                    left += 1
                
                # 以 right 为右端点的、和小于等于 k 的子数组个数为 right - left + 1
                count += (right - left + 1)
                
            return count

        # 原问题答案 = atMost(goal) - atMost(goal - 1)
        return atMost(goal) - atMost(goal - 1)

```

### 复杂度分析

*   **时间复杂度**: O(N)。其中 N 是 `nums` 的长度。`atMost` 函数中的 `left` 和 `right` 指针都只会从头到尾移动一次，因此是 O(N) 的。我们调用了两次该函数，所以总时间复杂度是 O(N) + O(N) = O(N)。
*   **空间复杂度**: O(1)。我们只使用了常数个额外变量。

### 补充：前缀和 + 哈希表

本题还有另一种非常经典的解法：前缀和 + 哈希表。
思路是：
1. 我们要求的是满足 `sum(nums[i:j+1]) == goal` 的 `(i, j)` 对的数量。
2. 使用前缀和 `prefix_sum`，这个条件可以改写为 `prefix_sum[j] - prefix_sum[i-1] == goal`。
3. 进一步变形得到 `prefix_sum[i-1] == prefix_sum[j] - goal`。
4. 于是，我们可以遍历数组，在计算出每个位置 `j` 的前缀和 `prefix_sum[j]` 的同时，去一个哈希表中查找，之前出现过多少次值为 `prefix_sum[j] - goal` 的前缀和。将这个次数累加到结果中，然后将当前的前缀和 `prefix_sum[j]` 存入哈希表。
这种方法的时间和空间复杂度分别为 O(N) 和 O(N)。

### 总结

本题展示了解决“恰好为 K”这类计数问题的强大技巧——将其转化为两个“最多为 K”问题的差。这个思想不仅适用于滑动窗口，也适用于其他许多算法场景。它能将一个逻辑复杂的计数问题，分解为两个逻辑清晰的子问题，从而大大简化代码的实现难度和出错率，是算法工具箱中非常实用的一招。
