---
title: LeetCode 1658 | 逆向思维+滑动窗口求解“将 x 减到 0 的最小操作数”
date: 2025-09-26 10:30:15
updated: 2025-09-26 10:30:15
comments: true
tags:
  - 数组
  - 哈希表
  - 二分查找
  - 滑动窗口
  - 前缀和
  - 第215场周赛
categories:
  - 算法题解
  - 滑动窗口与双指针
  - 不定长滑动窗口
permalink: posts/leetcode-1658/
---
{% centerquote %}
LeetCode 第 1658 题：[将 x 减到 0 的最小操作数](https://leetcode.cn/problems/minimum-operations-to-reduce-x-to-zero/description/)。
这道题看似复杂，因为它允许从数组的两端进行操作。但只要我们转换一下思路，它就会变成一个非常经典的滑动窗口问题。
{% endcenterquote %}
<!--more-->
### 问题解读

题目要求我们从一个整数数组 `nums` 的最左边或最右边移除元素，每次移除一个元素，它的值就从 `x` 中减去。目标是用**最少的操作次数**将 `x` 恰好减到 0。

我们来分析一下这个操作：
*   可以只从左边移除
*   可以只从右边移除
*   可以两边都移除

例如，对于 `nums = [1,1,4,2,3], x = 5`：
*   我们可以从左边移除 `1` 和 `1`，再从右边移除 `3`，总和为 `1+1+3=5`，操作 3 次。
*   我们也可以直接从右边移除 `3` 和 `2`，总和为 `3+2=5`，操作 2 次。

因为 `2 < 3`，所以最小操作数是 2。

直接模拟所有“左边取 i 个，右边取 j 个”的组合会非常低效。我们需要找到一个更优雅的切入点。

### 核心思路：逆向思维与问题转化

直接求解“两端移除的最小元素数量”比较困难，那我们不妨换个角度思考：**当我们从两端移除一部分元素后，剩下的是什么？**

答案是：**一个连续的、位于数组中间的子数组。**

我们的目标是让移除的元素之和恰好为 `x`，并且移除的元素数量最少。这等价于什么呢？

**让保留在中间的子数组最长！**

如果数组的总和是 `total_sum`，要移除的元素之和是 `x`，那么保留在中间的那个连续子数组，其元素之和就必须是 `target = total_sum - x`。



这样一来，原问题就被我们巧妙地转化成了另一个问题：
**在数组 `nums` 中，找到一个和为 `total_sum - x` 的最长的连续子数组。**

只要我们找到了这个最长的子数组，用数组的总长度 `n` 减去这个子数组的长度，就是我们要求的“最小操作数”。

而“寻找和为定值的最长连续子数组”正是一个可以用滑动窗口高效解决的经典问题！

### 算法详解

1.  **问题转化**:
    *   首先计算整个数组的总和 `total_sum`。
    *   计算出我们要在中间寻找的目标子数组的和 `target = total_sum - x`。
    *   处理一些边界情况：
        *   如果 `target < 0`，说明 `x` 本身就比数组所有元素之和还大，不可能凑成，返回 `-1`。
        *   如果 `target == 0`，说明需要移除所有元素才能使 `x` 减到 0，此时最长的中间子数组为空，但我们需要移除所有元素，答案是数组长度 `n`。

2.  **滑动窗口求解**:
    *   现在，我们在数组 `nums` 上寻找和为 `target` 的最长连续子数组。
    *   初始化左指针 `left = 0`，当前窗口内元素的和 `current_sum = 0`，以及记录最长长度的 `max_len = -1`（-1 表示尚未找到）。
    *   使用右指针 `right` 遍历数组，`right` 每向右移动一步，就将 `nums[right]` 加入 `current_sum`，这代表窗口在向右扩张。
    *   当 `current_sum > target` 时，说明当前窗口的和太大了，需要缩小。我们让 `left` 指针向右移动，并从 `current_sum` 中减去 `nums[left]`，直到 `current_sum <= target`。
    *   在每一步扩张或收缩后，如果 `current_sum == target`，我们就找到了一个符合条件的子数组。计算其长度 `right - left + 1`，并更新 `max_len`。

3.  **计算最终结果**:
    *   遍历结束后，如果 `max_len` 仍然是 `-1`，说明数组中不存在和为 `target` 的子数组，即无法通过移除操作使 `x` 减到 0，返回 `-1`。
    *   否则，最小操作数就是 `n - max_len`。

### Python 代码实现

```python
from typing import List

class Solution:
    def minOperations(self, nums: List[int], x: int) -> int:
        n = len(nums)
        total_sum = sum(nums)
        
        # 步骤 1: 问题转化，计算目标和
        target = total_sum - x
        
        # 边界情况：x 大于数组总和，无解
        if target < 0:
            return -1
        
        # 边界情况：需要移除所有元素
        if target == 0:
            return n

        # 步骤 2: 滑动窗口寻找和为 target 的最长子数组
        max_len = -1  # 记录最长子数组的长度
        current_sum = 0
        left = 0  # 窗口左边界
        
        for right in range(n):
            # 窗口右边界扩张
            current_sum += nums[right]
            
            # 当窗口和大于 target 时，收缩左边界
            while current_sum > target and left <= right:
                current_sum -= nums[left]
                left += 1
            
            # 如果窗口和等于 target，更新最大长度
            if current_sum == target:
                max_len = max(max_len, right - left + 1)

        # 步骤 3: 计算最终结果
        # 如果 max_len 未被更新，说明找不到满足条件的子数组
        if max_len == -1:
            return -1
        else:
            return n - max_len
```

### 复杂度分析

*   **时间复杂度**: O(N)。其中 N 是数组 `nums` 的长度。滑动窗口的左、右指针 `left` 和 `right` 都只会从头到尾遍历数组一次，因此总的时间复杂度是线性的。
*   **空间复杂度**: O(1)。我们只使用了常数个额外变量（如 `left`, `right`, `current_sum` 等），没有使用额外的数据结构。

### 总结

本题的核心在于**逆向思维**。当我们面对一个从两端操作、看似复杂的问题时，不妨思考一下操作的“补集”是什么。通过将“求两端最小移除数”转化为“求中间最长保留数”，问题瞬间豁然开朗，变成了一个我们非常熟悉的滑动窗口模板题。这个思路在很多算法题中都非常有效，是工具箱中必备的思维技巧。
