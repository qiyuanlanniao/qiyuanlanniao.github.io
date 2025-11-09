---
title: LeetCode 1818 | 二分查找优化绝对差值和
date: 2025-10-22 20:20:00
updated: 2025-10-22 20:20:00
comments: true
tags:
  - 数组
  - 排序
  - 二分查找
  - 有序集合
  - 第235场周赛
categories:
  - 算法题解
  - 二分算法
  - 二分查找
permalink: posts/leetcode-1818/
---
{% centerquote %}
LeetCode 第 1818 题：[绝对差值和](https://leetcode.cn/problems/minimum-absolute-sum-difference/)。
本题的关键在于，将问题转化为寻找最大“收益”的单次替换。通过对`nums1`排序并利用二分查找，我们可以高效地为`nums2`中的每个元素找到最佳匹配，从而将寻找最优解的复杂度从 O(n²) 优化到 O(n log n)。
{% endcenterquote %}
<!--more-->
### 问题解读

题目给了我们两个长度相等的正整数数组 `nums1` 和 `nums2`。首先，我们需要理解“绝对差值和”的定义，即 `sum(|nums1[i] - nums2[i]|)`。

核心任务是：我们可以对 `nums1` 进行**至多一次**修改，即用 `nums1` 中的**任意**一个元素替换 `nums1` 中的某个元素，目标是让这个“绝对差值和”变得尽可能小。最后返回这个最小的和，并对 `10^9 + 7` 取余。

举个例子，`nums1 = [1,7,5]`, `nums2 = [2,3,5]`：
*   **不替换**：初始的绝对差值和是 `|1-2| + |7-3| + |5-5| = 1 + 4 + 0 = 5`。
*   **尝试替换**：我们希望通过一次替换来最大化地减小这个和。
    *   观察到 `|7-3|=4` 是最大的差值项。如果我们能将 `nums1[1]`（值为7）换成一个更接近 `nums2[1]`（值为3）的数，收益可能最大。
    *   `nums1` 中有哪些元素可选？有 `1` 和 `5`。
    *   如果用 `1` 替换 `7`，新 `nums1` 为 `[1,1,5]`。差值和变为 `|1-2| + |1-3| + |5-5| = 1 + 2 + 0 = 3`。
    *   如果用 `5` 替换 `7`，新 `nums1` 为 `[1,5,5]`。差值和变为 `|1-2| + |5-3| + |5-5| = 1 + 2 + 0 = 3`。
*   **结论**：最小的绝对差值和是 3。

### 核心思路：贪心 + 二分查找

问题的本质是找到一个最优的替换方案。对于 `n` 个位置中的每一个 `i`，我们都有机会将其 `nums1[i]` 替换掉，从而改变 `|nums1[i] - nums2[i]|` 这一项。我们的目标是让这次替换带来的**收益**（即差值和的减少量）最大化。

对于任意一个位置 `i`，其原始差值为 `diff_i = |nums1[i] - nums2[i]|`。如果我们用 `nums1` 中的某个值 `val` 来替换 `nums1[i]`，那么新的差值变为 `new_diff_i = |val - nums2[i]|`。这次替换带来的收益就是 `reduction = diff_i - new_diff_i`。

要想让 `reduction` 最大，我们就必须让 `new_diff_i` 最小。这意味着，对于当前的 `nums2[i]`，我们需要在整个 `nums1` 数组中，找到一个与它最接近的数。

**如何高效地找到这个“最接近的数”？**
如果我们对每一个 `nums2[i]` 都遍历一遍 `nums1` 来寻找最接近的值，那么单次查找的时间复杂度是 O(n)，总时间复杂度将是 O(n²)，对于 n 高达 10^5 的情况，这显然会超时。

注意到 `nums1` 数组是固定的。我们可以先对其进行**排序**。在一个有序数组中查找一个数，或者查找与它最接近的数，正是**二分查找**的经典应用场景。

因此，我们的整体策略是：
1.  **预处理**：先对 `nums1` 进行排序，得到一个有序版本 `sorted_nums1`。
2.  **计算初始值**：计算不做任何替换时的原始绝对差值和 `total_diff`。
3.  **寻找最大收益**：遍历 `i` 从 `0` 到 `n-1`，对于每一个 `nums2[i]`，利用二分查找在 `sorted_nums1` 中找到与它最接近的数。从而计算出替换 `nums1[i]` 能带来的最大收益 `max_reduction`。
4.  **计算最终结果**：最终的最小绝对差值和就是 `total_diff - max_reduction`。

### 算法详解

1.  **计算初始总和**
    *   定义模 `MOD = 10^9 + 7`。
    *   遍历 `nums1` 和 `nums2`，累加 `|nums1[i] - nums2[i]|` 得到初始总和 `total_diff`。

2.  **排序 `nums1`**
    *   创建一个 `nums1` 的副本并对其进行升序排序，得到 `sorted_nums1`。

3.  **寻找最大可减少量 (max_reduction)**
    *   初始化 `max_reduction = 0`。
    *   再次遍历两个数组，对于每个索引 `i`：
        *   获取原始差值 `original_diff = |nums1[i] - nums2[i]|`。
        *   以 `target = nums2[i]` 为目标，在 `sorted_nums1` 中执行二分查找。
        *   二分查找会返回一个插入点 `j`。`target` 最接近的两个候选值就是 `sorted_nums1[j]` (如果 `j` 没越界) 和 `sorted_nums1[j-1]` (如果 `j > 0`)。
        *   计算 `target` 与这两个候选值的差的绝对值，取其中较小的一个作为 `min_new_diff`。
        *   当前位置 `i` 能产生的最大收益为 `reduction = original_diff - min_new_diff`。
        *   更新全局最大收益：`max_reduction = max(max_reduction, reduction)`。

4.  **返回结果**
    *   最终的答案是 `(total_diff - max_reduction) % MOD`。为了防止结果为负（在某些语言的取模运算中），可以使用 `(total_diff - max_reduction + MOD) % MOD` 来保证结果非负。

### Python 代码实现

```python
import bisect
from typing import List

class Solution:
    def minAbsoluteSumDiff(self, nums1: List[int], nums2: List[int]) -> int:
        MOD = 10**9 + 7
        n = len(nums1)

        # 步骤 1: 创建 nums1 的排序副本
        sorted_nums1 = sorted(nums1)

        total_diff = 0
        # 步骤 2: 计算原始的绝对差值和
        for i in range(n):
            total_diff += abs(nums1[i] - nums2[i])

        if total_diff == 0:
            return 0

        max_reduction = 0
        # 步骤 3: 遍历查找最大的可减少量
        for i in range(n):
            original_diff = abs(nums1[i] - nums2[i])
            target = nums2[i]
            
            # 步骤 4: 使用二分查找在 sorted_nums1 中寻找最接近 target 的值
            j = bisect.bisect_left(sorted_nums1, target)
            
            min_new_diff = float('inf')
            
            # 候选1: 插入点右侧（或本身）的元素
            if j < n:
                min_new_diff = min(min_new_diff, abs(sorted_nums1[j] - target))
            
            # 候选2: 插入点左侧的元素
            if j > 0:
                min_new_diff = min(min_new_diff, abs(sorted_nums1[j-1] - target))
            
            # 计算当前替换能带来的收益
            reduction = original_diff - min_new_diff
            
            # 更新全局最大收益
            max_reduction = max(max_reduction, reduction)

        # 步骤 5: 从原始总和中减去最大收益并取模
        result = (total_diff - max_reduction + MOD) % MOD
        
        return result
```

### 复杂度分析

*   **时间复杂度**: O(n log n)。
    *   `n` 是数组的长度。
    *   对 `nums1` 排序需要 O(n log n)。
    *   计算初始的 `total_diff` 需要 O(n)。
    *   主循环执行 `n` 次，每次循环内部执行一次二分查找，需要 O(log n)。这部分总共是 O(n log n)。
    *   因此，总的时间复杂度由排序和主循环决定，为 O(n log n)。
*   **空间复杂度**: O(n)。
    *   我们需要一个额外的数组 `sorted_nums1` 来存储 `nums1` 的排序副本，占用了 O(n) 的空间。

### 总结

本题是一个很好的例子，展示了如何通过**预处理**和**数据结构/算法**的选择来优化暴力解法。通过贪心地锁定“最大化收益”这一目标，我们将一个复杂的替换问题简化为了一系列查找问题。而排序和二分查找这对经典组合，则为我们提供了高效完成查找任务的强大工具，使得算法的性能满足了题目的要求。
