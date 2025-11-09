---
title: LeetCode 3488 | BS解决“距离最小相等元素查询”
date: 2025-10-20 15:20:00
updated: 2025-10-20 15:20:00
comments: true
tags:
  - 数组
  - 哈希表
  - 二分查找
  - 第441场周赛
categories:
  - 算法题解
  - 二分算法
  - 二分查找
permalink: posts/leetcode-3488/
---
{% centerquote %}
LeetCode 第 3488 题：[距离最小相等元素查询](https://leetcode.cn/problems/closest-equal-element-queries/)。
本题巧妙地结合了哈希表预处理和二分查找，高效解决环形数组中的距离查询问题，是“预处理+查询”思想的经典体现。
{% endcenterquote %}
<!--more-->
### 问题解读

题目给了我们一个 **环形** 数组 `nums` 和一个查询数组 `queries`。对于 `queries` 中的每一个下标 `queries[i]`，我们需要在 `nums` 数组中找到另一个下标 `j`，满足 `nums[j]` 与 `nums[queries[i]]` 的值相等，并且 `j` 与 `queries[i]` 之间的距离是所有满足条件的下标中最小的。

这里的 **距离** 需要特别注意，因为数组是环形的。两个下标 `i` 和 `j` 在一个长度为 `n` 的环形数组中的距离是 `min(|i - j|, n - |i - j|)`。如果找不到任何其他具有相同值的下标，那么该查询的结果就是 -1。

我们最后需要返回一个答案数组 `answer`，其中 `answer[i]` 对应第 `i` 个查询的结果。

举个例子，`nums = [1,3,1,4,1,3,2]`, `queries = [0,3,5]`, 数组长度 `n=7`：
*   对于查询 `queries[0] = 0`：
    *   `nums[0]` 的值是 1。 `nums` 中其他值为 1 的下标有 2 和 4。
    *   到下标 2 的距离是 `min(|0-2|, 7-|0-2|) = min(2, 5) = 2`。
    *   到下标 4 的距离是 `min(|0-4|, 7-|0-4|) = min(4, 3) = 3`。
    *   最小距离是 2。
*   对于查询 `queries[1] = 3`：
    *   `nums[3]` 的值是 4。 `nums` 中没有其他值为 4 的元素。
    *   结果为 -1。
*   对于查询 `queries[2] = 5`：
    *   `nums[5]` 的值是 3。`nums` 中另一个值为 3 的下标是 1。
    *   距离是 `min(|5-1|, 7-|5-1|) = min(4, 3) = 3`。
*   因此，最终答案是 `[2, -1, 3]`。

### 核心思路：预处理 + 二分查找

一个直接的想法是，对每个查询 `q_idx`，我们都遍历整个 `nums` 数组，找出所有值与 `nums[q_idx]` 相等的下标，然后逐一计算环形距离并取最小值。如果 `queries` 的长度是 Q，`nums` 的长度是 N，这种暴力解法的时间复杂度是 O(Q * N)，在 N 和 Q 都达到 10^5 的情况下，将会超时。

为了优化查询，我们可以对 `nums` 数组进行 **预处理**。

整个优化的思路分为两步：

1.  **预处理与分组**：我们可以使用一个哈希表（在 Python 中是字典），将 `nums` 中相同值的元素的 **下标** 聚合在一起。哈希表的键（key）是 `nums` 中的值，值（value）是一个包含该值所有出现位置下标的列表。因为我们是按顺序遍历 `nums` 来构建这个哈希表的，所以每个下标列表天然就是 **有序的**。

2.  **高效查询**：完成预处理后，对于每个查询的下标 `q_idx`，我们先找到它的值 `val = nums[q_idx]`。然后，我们从哈希表中取出 `val` 对应的有序下标列表 `indices`。问题就转化成了：**在一个排好序的数组 `indices` 中，找到离 `q_idx` 最近的两个邻居**。
    这正是二分查找的用武之地。我们可以通过二分查找，在 `indices` 列表中快速定位到 `q_idx` 自己的位置。它的前一个元素和后一个元素（需要考虑环绕情况）就是距离最近的候选者。

通过这种方式，每次查询我们不再需要遍历整个 `nums` 数组，而只需要在对应的小得多的下标列表上进行一次二分查找，时间复杂度从 O(N) 降到了 O(log K)（K 是某个值出现的次数，K ≤ N），显著提高了算法的整体效率。

### 算法详解

1.  **预处理 `nums` 数组**
    *   创建一个哈希表 `value_to_indices`，用于存储值到其下标列表的映射。
    *   遍历 `nums` 数组，对于每个元素 `nums[i]`，将其下标 `i` 添加到 `value_to_indices[nums[i]]` 对应的列表中。

2.  **处理 `queries` 数组**
    *   创建一个空的答案数组 `ans`。
    *   获取 `nums` 的长度 `n`，这在计算环形距离时会用到。
    *   遍历 `queries` 中的每一个 `q_idx`：
        *   获取其值 `val = nums[q_idx]` 和对应的下标列表 `indices = value_to_indices[val]`。
        *   如果 `indices` 的长度小于等于 1，说明没有其他相等的元素，将 -1 添加到 `ans` 中。
        *   否则，使用二分查找（在 Python 中是 `bisect.bisect_left`）在 `indices` 中找到 `q_idx` 的位置 `pos`。
        *   `q_idx` 在 `indices` 中的逻辑前驱下标是 `prev_idx = indices[(pos - 1 + len(indices)) % len(indices)]`。
        *   `q_idx` 在 `indices` 中的逻辑后继下标是 `next_idx = indices[(pos + 1) % len(indices)]`。这里的取模运算优雅地处理了列表首尾的环绕情况。
        *   分别计算 `q_idx` 到 `prev_idx` 和 `next_idx` 的环形距离。
            *   `dist_prev = min(abs(q_idx - prev_idx), n - abs(q_idx - prev_idx))`
            *   `dist_next = min(abs(q_idx - next_idx), n - abs(q_idx - next_idx))`
        *   将 `min(dist_prev, dist_next)` 添加到 `ans` 数组中。

3.  **返回结果**
    *   遍历结束后，返回 `ans` 数组。

### Python 代码实现

```python
import collections
import bisect
from typing import List

class Solution:
    def solveQueries(self, nums: List[int], queries: List[int]) -> List[int]:
        n = len(nums)
        
        # 步骤 1: 预处理 nums 数组
        # 使用哈希表存储每个值及其出现的所有下标
        value_to_indices = collections.defaultdict(list)
        for i, num in enumerate(nums):
            value_to_indices[num].append(i)

        ans = []
        # 步骤 2: 处理 queries 数组
        for q_idx in queries:
            val = nums[q_idx]
            indices = value_to_indices[val]

            # 如果该值只出现一次，没有其他相等元素
            if len(indices) <= 1:
                ans.append(-1)
                continue

            # 使用二分查找在下标列表中定位当前查询的下标
            # bisect_left 会找到 q_idx 在列表中的位置
            pos = bisect.bisect_left(indices, q_idx)

            # 找到逻辑上的前一个和后一个下标
            # (pos - 1 + len(indices)) % len(indices) 是一种处理负数取模的通用技巧
            prev_idx = indices[(pos - 1 + len(indices)) % len(indices)]
            # (pos + 1) % len(indices) 用于处理列表末尾的环绕
            next_idx = indices[(pos + 1) % len(indices)]

            # 计算到前一个和后一个元素的环形距离
            dist_prev = min(abs(q_idx - prev_idx), n - abs(q_idx - prev_idx))
            dist_next = min(abs(q_idx - next_idx), n - abs(q_idx - next_idx))

            # 取两者中的较小值
            ans.append(min(dist_prev, dist_next))
            
        # 步骤 3: 返回结果
        return ans

```

### 复杂度分析

*   **时间复杂度**: O(N + Q * log N)。
    *   `N` 是 `nums` 的长度, `Q` 是 `queries` 的长度。
    *   构建哈希表 `value_to_indices` 需要 O(N) 的时间。
    *   对于每个查询，哈希表查找的平均时间是 O(1)，二分查找的时间是 O(log K)，其中 K 是相同元素的数量（K ≤ N）。因此，处理所有查询的总时间是 O(Q * log N)。
*   **空间复杂度**: O(N)。
    *   在最坏的情况下（`nums` 中所有元素都不同），哈希表 `value_to_indices` 需要存储 N 个键和 N 个单元素列表，总空间消耗为 O(N)。

### 总结

本题是“预处理 + 高效查询”设计模式的一个绝佳范例。当面对需要对一个静态数据集进行多次查询的场景时，我们应首先考虑是否可以通过一次性的预处理（如排序、分组、构建特定数据结构等）来优化后续的查询操作。哈希表用于快速分组，而二分查找则利用了数据的有序性，两者的结合为解决这类问题提供了强大而高效的工具。
