---
title: LeetCode 2106 | 滑动窗口求解“摘水果”
date: 2025-10-01 08:08:00
updated: 2025-10-01 08:08:00
comments: true
tags:
  - 数组
  - 二分查找
  - 滑动窗口
  - 前缀和
  - 第 271 场周赛
categories:
  - 算法题解
  - 滑动窗口与双指针
  - 不定长滑动窗口
permalink: posts/leetcode-2106/
---
{% centerquote %}
LeetCode 第 2106 题：[摘水果](https://leetcode.cn/problems/maximum-fruits-harvested-after-at-most-k-steps/)。
该题要求我们在有限的步数内，规划一条路径以摘取最大数量的水果。路径的选择（先左还是先右）似乎复杂。实际上可以通过巧妙地定义移动成本，将其转化为一个经典的滑动窗口问题。
{% endcenterquote %}
<!--more-->
### 问题解读

题目给了我们一个无限长的 x 轴，上面分布着一些水果。我们用一个数组 `fruits` 来描述，其中 `fruits[i] = [positioni, amounti]` 表示在位置 `positioni` 有 `amounti` 个水果。这个数组已经按位置升序排列。

我们从一个给定的 `startPos` 出发，总共可以移动最多 `k` 步。目标是规划一条路径，使得摘到的水果总数最多。

举个例子，对于 `fruits = [[2,8],[6,3],[8,6]]`, `startPos = 5`, `k = 4`：
*   我们的目标是在 4 步内，从位置 5 出发，摘到最多的水果。
*   一种可行的方案是：向右走到位置 6 (1 步)，摘 3 个水果。再向右走到位置 8 (2 步)，摘 6 个水果。总步数是 (8 - 5) = 3，小于 4。总水果数是 3 + 6 = 9。
*   另一种方案：向左走到位置 2 (3 步)，摘 8 个水果。总步数是 (5 - 2) = 3，小于 4。总水果数是 8。
*   我们能找到的最优解是先到 6 再到 8，共摘 9 个水果。

直接模拟所有可能的路径组合（比如左走 `i` 步，右走 `k-i` 步）会非常复杂，且效率低下。

### 核心思路：滑动窗口与成本计算

首先，一个关键的观察是：**我们最优的采摘方案一定是采摘 x 轴上一个连续区间内的所有水果**。因为如果我们在路径上跳过了一个水果点，我们完全可以在不增加额外成本的情况下把它摘掉（因为我们已经路过了）。

因此，问题就转化成了：**寻找一个连续的水果区间 `[left, right]`，使得从 `startPos` 出发，访问完这个区间内所有水果的总步数不超过 `k`，并且这个区间的水果总量最大。**

这正是滑动窗口算法的用武之地。我们可以把 `[left, right]` 看作一个在 `fruits` 数组上滑动的窗口。

这个问题的核心难点在于如何计算访问一个区间 `[left_pos, right_pos]` 所需的**最小步数**。从 `startPos` 出发，要覆盖这个区间，无非就两种回头路策略：

1.  **先去区间的左端点，再一路走到右端点**：步数为 `(startPos - left_pos) + (right_pos - left_pos)`。
2.  **先去区间的右端点，再一路走到左端点**：步数为 `(right_pos - startPos) + (right_pos - left_pos)`。

当然，如果 `startPos` 本身就在区间的某一边，就不需要走回头路了：
*   如果 `startPos` 在区间左侧 (`startPos <= left_pos`)，我们只能一路向右，成本是 `right_pos - startPos`。
*   如果 `startPos` 在区间右侧 (`startPos >= right_pos`)，我们只能一路向左，成本是 `startPos - left_pos`。
*   如果 `startPos` 在区间内部 (`left_pos < startPos < right_pos`)，我们就需要从上面两种回头路策略中选择步数更少的那一种。

一旦我们确定了这个成本计算方法，就可以套用滑动窗口模板了：
*   用 `right` 指针扩大窗口。
*   当窗口的移动成本超过 `k` 时，用 `left` 指针收缩窗口。
*   在每一步都用当前有效窗口的水果总量更新最大值。

### 算法详解

1.  **滑动窗口初始化**：
    *   左指针 `left = 0`。
    *   记录最大水果数 `max_fruits = 0`。
    *   记录当前窗口内的水果总数 `current_sum = 0`。

2.  **遍历与窗口扩张**：
    *   用右指针 `right` 从 `0` 遍历到 `n-1`（`n` 是 `fruits` 的长度）。
    *   在循环的每一步，将 `fruits[right]` 的水果数量加入 `current_sum`。

3.  **成本判断与窗口收缩**：
    *   扩张后，计算覆盖当前窗口 `[fruits[left]...fruits[right]]` 所需的最小步数 `dist`。
    *   我们使用一个 `while` 循环来检查和调整窗口：
        *   只要 `dist > k`，就说明当前窗口太“远”或太“宽”，需要从左侧收缩。
        *   将左侧水果 `fruits[left]` 的数量从 `current_sum` 中减去，然后将 `left` 指针右移。
        *   这个 `while` 循环会一直执行，直到窗口重新满足步数限制 (`dist <= k`) 为止。

4.  **更新结果**：
    *   在每次 `right` 指针移动后，并且在（可能发生的）收缩操作完成后，当前的窗口 `[left...right]` 一定是有效的。
    *   我们用 `current_sum` 来更新 `max_fruits`。

5.  **返回最终答案**：
    *   遍历结束后，`max_fruits` 就是我们能找到的最大水果总数。

### Python 代码实现

```python
class Solution:
    def maxTotalFruits(self, fruits: List[List[int]], startPos: int, k: int) -> int:
        """
        使用滑动窗口解决摘水果问题。
        我们维护一个水果区间 [left, right] 作为窗口。
        随着 right 指针向右移动来扩大窗口，我们检查覆盖当前窗口所需的步数是否超过 k。
        如果超过，我们就移动 left 指针来收缩窗口，直到满足步数限制为止。
        在每一步，我们都用当前有效窗口内的水果总数来更新最大值。
        """
        
        left = 0
        max_fruits = 0
        current_sum = 0
        n = len(fruits)

        # right 指针用于扩大窗口
        for right in range(n):
            right_pos, amount = fruits[right]
            current_sum += amount

            # 当窗口不满足步数限制时，从左边收缩窗口
            while left <= right:
                left_pos = fruits[left][0]
                
                # 计算访问 [left_pos, right_pos] 区间的成本
                dist = 0
                # 情况1: startPos 在整个区间的左侧或重合于左端点，只能向右走
                if startPos <= left_pos:
                    dist = right_pos - startPos
                # 情况2: startPos 在整个区间的右侧或重合于右端点，只能向左走
                elif startPos >= right_pos:
                    dist = startPos - left_pos
                # 情况3: startPos 在区间内部，需要走回头路
                else:
                    # 策略A: 先左后右 vs 策略B: 先右后左
                    dist = min(startPos - left_pos, right_pos - startPos) + (right_pos - left_pos)

                # 如果所需步数大于 k，则收缩窗口
                if dist > k:
                    current_sum -= fruits[left][1]
                    left += 1
                # 否则，当前窗口有效，停止收缩
                else:
                    break
            
            # 用当前有效窗口的水果总数更新最大值
            max_fruits = max(max_fruits, current_sum)
            
        return max_fruits
```

### 复杂度分析

*   **时间复杂度**: O(N)。其中 N 是 `fruits` 数组的长度。滑动窗口的左指针 `left` 和右指针 `right` 都只会从头到尾各自扫描一遍数组，每个元素最多被访问两次。
*   **空间复杂度**: O(1)。我们只使用了常数个额外变量来存储指针和计数值。

### 总结

本题的核心在于将一个路径规划问题，转化为寻找**最优连续区间**的问题。其难点在于为这个“区间”（即我们的滑动窗口）建立一个正确的成本模型。一旦我们分析清楚了从起点出发覆盖一个区间的两种基本走法（先左后右 vs 先右后左），并处理好边界情况，剩下的就是滑动窗口算法的经典应用了。这个思路将复杂路径问题简化为线性扫描，是算法优化中的一个重要思想。
