---
title: LeetCode LCP 08 | 前缀和与二分查找解决“剧情触发时间”
date: 2025-10-23 09:20:00
updated: 2025-10-23 09:20:00
comments: true
tags:
  - 数组
  - 排序
  - 二分查找
categories:
  - 算法题解
  - 二分算法
  - 二分查找
permalink: posts/leetcode-lcp-08/
---
{% centerquote %}
LeetCode LCP 08 题：[剧情触发时间](https://leetcode.cn/problems/ju-qing-hong-fa-shi-jian/)。
当游戏属性随时间单调递增时，寻找满足条件的“最早时刻”是一个经典的二分查找应用场景。通过预计算前缀和，我们可以将每次判断的复杂度降为常数级别，从而高效地解决问题。
{% endcenterquote %}
<!--more-->
### 问题解读

题目模拟了一个游戏过程，其中有三种核心属性：文明等级（C）、资源储备（R）和人口数量（H）。所有属性初始值（第 0 天）都为 0。

游戏每天都会进行，三种属性会根据一个给定的 `increase` 数组增长。`increase[i]` 代表第 `i+1` 天 C, R, H 的增量。

同时，有一系列剧情，它们的触发条件由 `requirements` 数组给出。每个 `requirement[j]` 包含一个 `[c, r, h]` 的阈值，当玩家的属性 C, R, H 同时大于或等于这些阈值时，该剧情就会被触发。

我们的任务是，对于每一个剧情，计算它被触发的最早是哪一天。如果某个剧情永远无法被触发，则记为 -1。

例如，`increase = [[2,8,4],[2,5,0],[10,9,8]]`，`requirements = [[2,11,3]]`：
*   初始状态 (第 0 天): C=0, R=0, H=0。
*   第 1 天结束: C=2, R=8, H=4。此时不满足 R>=11。
*   第 2 天结束: C=2+2=4, R=8+5=13, H=4+0=4。此时 C>=2, R>=11, H>=3 全部满足。
*   因此，该剧情的触发时间是第 2 天。

### 核心思路：前缀和 + 二分查找

一个朴素的解法是：对每一个 `requirement`，我们都从第 1 天开始，一天天模拟属性的增长，直到满足条件为止。如果 `increase` 的长度为 D，`requirements` 的长度为 Q，这种暴力解法的时间复杂度大约是 O(Q * D)，在 D 和 Q 都达到 10^5 的情况下，计算量过大，会导致超时。

我们可以进行优化。注意到，对于任何一个剧情，它能否在第 `k` 天被触发，取决于到第 `k` 天为止累积的 C, R, H 总量。这个“累积总量”可以通过**前缀和**来快速计算。我们可以先对 `increase` 数组预处理，得到一个 `prefix_sum` 数组，其中 `prefix_sum[k]` 存储了到第 `k` 天结束时 C, R, H 的总值。这样，查询任意一天的属性状态就变成了 O(1) 的操作。

更重要的是，随着天数 `d` 的增加，C, R, H 的总量是**单调不减**的。这意味着，如果一个剧情在第 `d` 天可以被触发，那么在 `d` 之后的所有天也一定可以被触发。这个单调性是使用**二分查找**的关键信号。

因此，对于每个 `requirement`，问题就转化为：**在 `[0, 1, ..., D]` 这些天中，找到满足触发条件的最小天数 `d`。** 这是一个典型的二分查找“寻找下界”的问题。

优化的思路分为两步：
1.  **预计算前缀和**：遍历一次 `increase` 数组，计算出每一天结束时 C, R, H 的累积总量，存入前缀和数组。
2.  **二分查找**：对每一个 `requirement`，在 `[0, D]` 的天数范围内进行二分查找，快速定位到最早的触发时间。

### 算法详解

1.  **预处理 `increase` 数组**
    *   创建一个前缀和数组 `prefix_sum`，长度比 `increase` 大 1。`prefix_sum[i]` 用于存储到第 `i` 天结束时的总属性值。
    *   `prefix_sum[0]` 初始化为 `[0, 0, 0]`，代表第 0 天的初始状态。
    *   遍历 `increase` 数组，从 `i = 1` 到 `D`，计算 `prefix_sum[i] = prefix_sum[i-1] + increase[i-1]`。

2.  **处理 `requirements` 数组**
    *   创建一个空的答案数组 `ans`。
    *   遍历 `requirements` 中的每一个 `req = [c, r, h]`：
        *   首先做一个快速判断：如果到游戏最后一天，总属性值都无法满足 `req`，那么这个剧情永远不会触发。即 `prefix_sum[D]` 的三项均小于 `req` 的对应项，则直接将 -1 加入答案，并继续下一个 `req`。
        *   设定二分查找的范围 `left = 0`, `right = D`。我们的目标是在这个范围内找到满足条件的最小天数。
        *   进入二分循环 (`while left <= right`)：
            *   取中间天数 `mid = left + (right - left) // 2`。
            *   检查在第 `mid` 天结束时，属性是否满足 `req`，即 `prefix_sum[mid][0] >= c`, `prefix_sum[mid][1] >= r`, 并且 `prefix_sum[mid][2] >= h`。
            *   如果满足条件：说明第 `mid` 天或更早的天数是可能的答案。我们记录下 `mid`，并尝试在更早的时间里寻找，即 `right = mid - 1`。
            *   如果不满足条件：说明第 `mid` 天太早了，需要更多时间积累属性，因此需要在之后的时间里寻找，即 `left = mid + 1`。
        *   循环结束后，记录下的那个满足条件的 `mid` 就是最早的触发时间。将其加入 `ans` 数组。
        *   特殊情况：对于 `[0,0,0]` 的需求，第 0 天即可满足，需要单独处理或确保二分查找的逻辑能覆盖。

3.  **返回结果**
    *   遍历结束后，返回 `ans` 数组。

### Python 代码实现
```python
from typing import List

class Solution:
    def getTriggerTime(self, increase: List[List[int]], requirements: List[List[int]]) -> List[int]:
        n = len(increase)
        
        # 步骤 1: 预处理 increase 数组，计算前缀和
        # prefix_sum[i] 表示第 i 天结束时的总属性 (i 从 1 开始)
        # prefix_sum[0] 表示第 0 天的初始状态
        prefix_sum = [[0, 0, 0] for _ in range(n + 1)]
        for i in range(n):
            prefix_sum[i+1][0] = prefix_sum[i][0] + increase[i][0]
            prefix_sum[i+1][1] = prefix_sum[i][1] + increase[i][1]
            prefix_sum[i+1][2] = prefix_sum[i][2] + increase[i][2]
            
        ans = []
        
        # 步骤 2: 处理 requirements 数组
        for req in requirements:
            c, r, h = req[0], req[1], req[2]
            
            # 特殊情况：0 需求
            if c == 0 and r == 0 and h == 0:
                ans.append(0)
                continue
                
            # 剪枝：如果最后一天都无法满足，则永远无法触发
            if prefix_sum[n][0] < c or prefix_sum[n][1] < r or prefix_sum[n][2] < h:
                ans.append(-1)
                continue
            
            # 二分查找：在 [1, n] 的天数范围内寻找最早的触发时间
            left, right = 1, n
            res = -1 # 用于记录满足条件的最早天数
            
            while left <= right:
                mid = (left + right) // 2
                # 检查第 mid 天的属性是否满足要求
                if prefix_sum[mid][0] >= c and prefix_sum[mid][1] >= r and prefix_sum[mid][2] >= h:
                    # 如果满足，mid 是一个可能的答案，尝试寻找更早的
                    res = mid
                    right = mid - 1
                else:
                    # 如果不满足，说明 mid 太早，需要在之后的天数里找
                    left = mid + 1
            
            ans.append(res)
            
        # 步骤 3: 返回结果
        return ans
```

### 复杂度分析

*   **时间复杂度**: O(D + Q * log D)
    *   `D` 是 `increase` 的长度 (总天数)，`Q` 是 `requirements` 的长度。
    *   计算前缀和数组需要遍历一次 `increase`，复杂度为 O(D)。
    *   对于 `Q` 个剧情中的每一个，我们都执行一次二分查找。查找的范围是 `[1, D]`，所以单次二分查找的复杂度是 O(log D)。
    *   总时间复杂度为 O(D + Q * log D)。
*   **空间复杂度**: O(D)
    *   我们需要一个额外的前缀和数组 `prefix_sum` 来存储 `D+1` 天的属性状态。
    *   返回的答案数组需要 O(Q) 的空间，但通常不计入额外空间复杂度。因此，主要额外空间开销是前缀和数组。

### 总结

本题是“预处理 + 二分查找”模式的又一个绝佳示例。题目的核心特征是状态（玩家属性）随一个维度（时间）的推移而单调变化。当我们需要在这个单调变化的序列中寻找第一个满足特定条件的点时，二分查找就是最高效的算法。通过前缀和将“检查任意一点状态”的成本从 O(D) 降至 O(1)，是让二分查找得以高效应用的关键所在。
