---
title: LeetCode 632 | 滑动窗口求解“最小区间”
date: 2025-10-13 10:05:00
updated: 2025-10-13 10:05:00
comments: true
tags:
  - 滑动窗口
  - 排序
  - 哈希表
  - 堆（优先队列）
  - 贪心
  - 数组
categories:
  - 算法题解
  - 滑动窗口与双指针
permalink: posts/leetcode-632/
---
{% centerquote %}
LeetCode 第 632 题：[最小区间](https://leetcode.cn/problems/smallest-range-covering-elements-from-k-lists/)。
该题要求在 k 个排序列表中寻找一个最小的数值区间，这个区间需要包含来自每个列表的至少一个元素。这是一个经典的多路数据处理问题，通过将所有元素“压平”并结合滑动窗口，可以巧妙地解决。
{% endcenterquote %}
<!--more-->
### 问题解读

题目给了我们 `k` 个非递减排序的整数列表 `nums`。我们需要找到一个最小的数值区间 `[a, b]`，要求这个区间必须包含来自**每一个**列表的至少一个数字。

“最小”区间的定义有两层：
1.  首先比较区间的长度 `b - a`，长度越小，区间就越小。
2.  如果两个区间的长度相等，那么起始点 `a` 较小的区间更小。

例如，对于 `nums = [[4,10,15,24,26], [0,9,12,20], [5,18,22,30]]`：
*   区间 `[20, 24]` 是一个有效的覆盖区间，因为它包含了来自列表1的 `24`、列表2的 `20` 和列表3的 `22`。
*   这个区间的长度是 4。
*   我们可以找到其他有效区间，如 `[0, 5]` 覆盖了 `4, 0, 5`，长度为 5。
*   经过比较，`[20, 24]` 是所有有效区间中长度最小的，因此是最终答案。

直接在 `k` 个列表中寻找这样的区间非常困难，因为我们不知道区间的端点应该如何选择。

### 核心思路：排序 + 滑动窗口

这个问题的核心难点在于数据分散在 `k` 个不同的列表中，难以统一处理。一个常见的“降维”思路是，将多路数据合并成一路，从而应用更成熟的单数组算法。

关键的洞察在于：**最优区间的两个端点 `a` 和 `b`，必然是输入数据中存在的某两个数**。

基于此，我们可以：
1.  **扁平化数据**：将所有 `k` 个列表中的所有数字合并到一个单一的列表中。为了后续判断区间有效性，我们不仅要存储数值，还要存储它来自哪个列表。因此，我们创建一个 `(数值, 原始列表索引)` 的元组列表。
2.  **排序**：对这个合并后的列表，按照数值进行升序排序。
3.  **滑动窗口**：现在问题转化为了一个更经典的形式——在排序后的列表中，寻找一个**最短的子数组（窗口）**，这个子数组中包含了来自全部 `k` 个不同原始列表的元素。这正是滑动窗口算法的用武之地。

我们用 `left` 和 `right` 两个指针来定义窗口。我们不断向右移动 `right` 来扩张窗口，直到窗口内的元素满足“覆盖所有k个列表”的条件。一旦满足，我们就记录下这个有效区间，并尝试从左侧移动 `left` 指针来收缩窗口，寻找一个可能更小的有效区间。


*上图形象地展示了在合并排序后的列表上，一个有效的滑动窗口如何覆盖了来自3个不同列表的元素。*

### 算法详解

1.  **数据预处理：扁平化与排序**
    *   初始化一个空列表 `merged_list`。
    *   遍历输入的 `k` 个列表，将每个数字 `val` 和它所属的列表索引 `i` 作为一个元组 `(val, i)` 添加到 `merged_list` 中。
    *   调用排序函数，对 `merged_list` 按元组的第一个元素（即数值）进行升序排序。

2.  **滑动窗口遍历**
    *   初始化左指针 `left = 0`，以及一个最终答案区间 `ans`（可以初始化为一个无限大的区间）。
    *   为了高效地判断窗口有效性，我们使用一个哈希表 `counts` 来记录窗口内每个列表的元素数量，并用一个变量 `covered_lists` 记录当前已覆盖的列表数量。
    *   使用 `right` 指针从 `0` 到 `len(merged_list) - 1` 进行遍历，代表窗口的右边界。
    *   对于 `right` 指向的元素 `(val, list_idx)`:
        *   **扩展窗口**：将其加入窗口。在 `counts` 中将 `list_idx` 的计数加一。如果这个计数从 0 变为 1，说明我们刚刚覆盖了一个新的列表，因此 `covered_lists` 加 1。
    *   **检查并收缩窗口**：
        *   使用一个 `while` 循环检查 `covered_lists` 是否等于 `k`。如果是，说明当前窗口 `[left, right]` 是一个有效的候选区间。
        *   获取当前区间的左右端点值 `merged_list[left][0]` 和 `merged_list[right][0]`，计算区间长度。
        *   如果当前区间比已记录的 `ans` 区间更小，则更新 `ans`。
        *   **收缩窗口**：尝试将 `left` 指针右移。在 `counts` 中将 `left` 指向元素的列表索引的计数减一。如果计数变为 0，说明我们失去了一个列表的覆盖，`covered_lists` 减 1。此时 `while` 循环将终止。

3.  **返回结果**
    *   当 `right` 指针遍历完整个列表后，`ans` 中存储的就是最终的最小区间。

### Python 代码实现

```python
import collections
from typing import List

class Solution:
    def smallestRange(self, nums: List[List[int]]) -> List[int]:
        # 步骤 1: 创建并排序合并列表
        merged_list = []
        for i, lst in enumerate(nums):
            for val in lst:
                merged_list.append((val, i))
        
        merged_list.sort(key=lambda x: x[0])

        k = len(nums)
        left = 0
        list_counts = collections.defaultdict(int)
        lists_covered = 0

        min_range_size = float('inf')
        result_start, result_end = 0, 0

        # 步骤 2: 滑动窗口
        for right in range(len(merged_list)):
            val_right, list_idx_right = merged_list[right]

            # 扩展窗口
            if list_counts[list_idx_right] == 0:
                lists_covered += 1
            list_counts[list_idx_right] += 1

            # 当窗口有效时，检查并尝试收缩
            while lists_covered == k:
                val_left, list_idx_left = merged_list[left]

                # 检查是否为更小的区间
                current_range_size = val_right - val_left
                if current_range_size < min_range_size:
                    min_range_size = current_range_size
                    result_start = val_left
                    result_end = val_right
                
                # 收缩窗口
                list_counts[list_idx_left] -= 1
                if list_counts[list_idx_left] == 0:
                    lists_covered -= 1
                
                left += 1

        return [result_start, result_end]
```

### 复杂度分析

*   **时间复杂度**: O(N log N)，其中 N 是所有列表中元素的总数。主要开销在于对 `merged_list` 的排序。后续的滑动窗口遍历是 O(N)，因为 `left` 和 `right` 指针都只遍历列表一次。
*   **空间复杂度**: O(N)，用于存储 `merged_list` 和哈希表 `counts`。

### 总结

本题展示了如何通过数据结构的转换，将一个看似复杂的多列表问题简化为单数组上的经典算法问题。扁平化和排序是处理多个有序序列的常用技巧，它为滑动窗口等高效算法的应用铺平了道路。此外，对于此类问题，还存在一种使用最小堆（Min-Heap）的更优解法，其时间复杂度为 O(N log k)，在 `k` 远小于 `N` 时更具优势，也值得进一步学习。
