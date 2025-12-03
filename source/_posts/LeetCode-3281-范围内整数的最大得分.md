---
title: LeetCode 3281 | 范围内整数的最大得分
date: 2025-11-18 10:49:00
updated: 2025-11-18 10:49:00
comments: true
tags:
  - 数组
  - 二分查找
  - 贪心
  - 排序
  - 第414场周赛
categories:
  - 算法题解
  - 二分算法
  - 二分答案
  - 最大化最小值
permalink: posts/leetcode-3281/
---
{% centerquote %}
LeetCode 第 3281 题：[范围内整数的最大得分](https://leetcode.cn/problems/maximize-score-of-numbers-in-ranges/description/)。
当直接求解“最大可能得分”比较困难时，可以考虑二分答案。我们将问题转化为：是否存在一种选择方案，使得最小绝对差不小于某个值 k？
{% endcenterquote %}
<!--more-->

### 一、题目描述

给你一个整数数组 `start` 和一个整数 `d`，代表 n 个区间 `[start[i], start[i] + d]`。

你需要选择 n 个整数，其中第 `i` 个整数必须属于第 `i` 个区间。所选整数的 **得分** 定义为所选整数两两之间的 **最小** 绝对差。

返回所选整数的 **最大可能得分**。

**示例：**
```
输入： start = [6,0,3], d = 2
输出： 4
解释：
可以选择整数 8, 0 和 4 获得最大可能得分。它们分别属于区间 [6,8], [0,2], [3,5]。
排序后为 0, 4, 8，得分为 min(|4-0|, |8-4|) = 4。
```

### 二、思路分析

#### 1. 核心思想：最大化最小值 -> 二分答案

题目的目标是“最大化（最小绝对差）”，这种“最大化最小值”或者“最小化最大值”的问题是二分答案的经典应用场景。

直接计算最大得分很复杂，因为每个区间都有多种选择。我们可以换一个角度思考：给定一个得分 `k`，我们能否找到一种选择方案，使得任意两个所选整数的绝对差都 `≥ k`？

这个问题就变成了一个判定性问题，并且具有单调性：
*   如果得分 `k` 是可以达成的，那么任何小于 `k` 的得分 `k'` 也一定可以达成。
*   如果得分 `k` 无法达成，那么任何大于 `k` 的得分 `k''` 也一定无法达成。

这种单调性让我们能够通过二分查找来逼近最终的答案。我们可以在一个可能的得分范围内进行二分，找到满足条件的最大的那个得分 `k`。

#### 2. 验证函数 `check(score)` 的设计

二分答案的关键在于如何实现这个 `check(score)` 函数，即如何判断一个给定的 `score` 是否可行。

为了方便处理，我们可以首先将所有区间按照左端点 `start[i]` 进行升序排序。排序后，我们选出的 n 个数也应该是升序的，这样我们只需要保证相邻两个数之差不小于 `score` 即可。

假设我们已经为前 `i-1` 个区间选择了整数，最后一个选择的数是 `prev_x`。现在要为第 `i` 个区间 `[s_i, s_i + d]` 选择一个数 `curr_x`。这个 `curr_x` 必须满足两个条件：
1.  `curr_x >= prev_x + score` (保证与前一个数的差值)
2.  `s_i <= curr_x <= s_i + d` (保证在当前区间内)

为了让后续的选择有尽可能大的空间，我们应该贪心地选择满足条件的最小的 `curr_x`。
因此，我们选择的 `curr_x` 应该是 `max(prev_x + score, s_i)`。

选择了这个 `curr_x` 后，我们还需要验证它是否在当前区间 `[s_i, s_i + d]` 内。也就是判断 `curr_x` 是否大于 `s_i + d`。
*   如果 `curr_x > s_i + d`，说明在满足 `score` 的前提下，无法在当前区间内找到一个合法的数。因此 `score` 值太大了，`check(score)` 返回 `False`。
*   如果 `curr_x <= s_i + d`，说明这个选择是合法的。我们就把 `curr_x` 作为新的 `prev_x`，继续为下一个区间进行选择。

如果遍历完所有区间都能找到合法的 `curr_x`，那么 `check(score)` 就返回 `True`。

#### 3. 算法流程

1.  对 `start` 数组进行升序排序。
2.  确定二分查找的范围 `[left, right]`。左边界 `left` 为 0，右边界 `right` 可以取一个足够大的值，例如 `start[n-1] + d - start[0]`。
3.  在 `[left, right]` 范围内进行二分查找。
    *   取中间值 `mid`。
    *   调用 `check(mid)` 判断得分 `mid` 是否可行。
    *   如果可行 (`check(mid)` 为 `True`)，说明 `mid` 可能就是答案，或者答案更大。我们尝试更大的值，令 `left = mid`。
    *   如果不可行 (`check(mid)` 为 `False`)，说明 `mid` 太大了，答案在更小的范围里。令 `right = mid`。
4.  循环结束后，`left` 即为最大可能得分。

### 三、代码实现

```python
from typing import List
from math import inf

class Solution:
    def maxPossibleScore(self, start: List[int], d: int) -> int:
        """
        通过二分答案来解决“最大化最小值”问题。
        """
        # 1. 对区间的起始点进行排序
        start.sort()
        n = len(start)

        # 2. 定义 check 函数，判断得分 score 是否可行
        def check(score: int) -> bool:
            # prev_x 表示上一个区间选择的数，初始化为负无穷
            prev_x = -inf
            for s in start:
                # 贪心策略：为当前区间 [s, s + d] 选择满足条件的最小整数
                # 这个数必须 >= s，也必须 >= prev_x + score
                current_x = max(prev_x + score, s)
                
                # 检查这个选择的数是否超出了当前区间的右边界
                if current_x > s + d:
                    # 如果超出，说明 score 太大，无法满足
                    return False
                
                # 更新 prev_x 为当前选择的数，用于下一次迭代
                prev_x = current_x
            
            # 如果所有区间都能成功选择，说明 score 可行
            return True

        # 3. 二分查找最大可能的得分
        # left 是下界，right 是一个合理的上界
        left, right = 0, (start[-1] + d - start[0]) // (n - 1) + 1
        
        # 使用 left + 1 < right 的模板，寻找满足 check 的最大 left
        while left + 1 < right:
            mid = (left + right) // 2
            if check(mid):
                # 如果 mid 可行，尝试更大的得分
                left = mid
            else:
                # 如果 mid 不可行，缩小范围到 [left, mid]
                right = mid
        
        return left

```

### 四、复杂度分析

*   **时间复杂度**: O(N log N + N log R)。
    *   对 `start` 数组排序需要 O(N log N) 的时间。
    *   二分查找的范围为 `R`（最大可能的分数），每次 `check` 函数需要 O(N) 的时间。所以二分部分的时间是 O(N log R)。
    *   总时间复杂度为 O(N log N + N log R)。
*   **空间复杂度**: O(1) 或 O(log N)，取决于排序算法所使用的额外空间。
