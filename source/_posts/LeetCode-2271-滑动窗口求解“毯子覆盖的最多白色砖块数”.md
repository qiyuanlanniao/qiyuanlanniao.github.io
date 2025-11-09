---
title: LeetCode 2271 | 滑动窗口求解“毯子覆盖的最多白色砖块数”
date: 2025-09-30 08:00:00
updated: 2025-09-30 08:00:00
comments: true
tags:
  - 贪心
  - 数组
  - 二分查找
  - 前缀和
  - 排序
  - 滑动窗口
categories:
  - 算法题解
  - 滑动窗口与双指针
  - 不定长滑动窗口
permalink: posts/leetcode-2271/
---
{% centerquote %}
LeetCode 第 2271 题：[毯子覆盖的最多白色砖块数](https://leetcode.cn/problems/maximum-white-tiles-covered-by-a-carpet/)。
此题的核心在于如何高效地计算一个固定长度的“毯子”在任意位置能够覆盖的最大瓷砖数。通过将问题巧妙转化为“端点对齐”，并结合“排序”与“滑动窗口”，我们可以找到优雅的线性时间解法。
{% endcenterquote %}
<!--more-->
### 问题解读

题目给了我们一个二维数组 `tiles`，其中每个 `tiles[i] = [li, ri]` 代表一段连续的白色瓷砖。同时，我们还有一个长度为 `carpetLen` 的毯子。我们的目标是，将这块毯子放置在某个位置，使其能够覆盖的白色瓷砖数量达到最大，并返回这个最大值。

我们来分析一下这个目标：
*   我们有一块固定长度的毯子，可以放置在数轴的任何位置。
*   我们需要找到一个最佳的起始点，使得区间 `[start, start + carpetLen - 1]` 覆盖的白色瓷砖总数最多。
*   一个非常关键的观察是：**毯子的最优摆放位置，其左端点或右端点必然与某块瓷砖的端点对齐**。为什么呢？试想，如果一个最优位置的毯子，其两端都没有与任何瓷砖的端点对齐，我们总可以将它向左或向右平移，直到有一个端点对齐为止，而这个过程中覆盖的瓷砖数不会减少。

基于这个观察，我们可以简化问题：我们只需要考虑所有“毯子右端点与某块瓷砖的右端点对齐”的情况，并从中找出最优解即可。（选择左端点对齐也是同理的）。

举个例子，对于 `tiles = [[1,5], [10,11]], carpetLen = 6`：
*   如果我们让毯子右端点与 `tiles[0]` 的右端点 `5` 对齐，毯子覆盖的区间是 `[5 - 6 + 1, 5] = [0, 5]`。这个区间覆盖了 `[1,5]` 这一整块瓷砖，长度为 5。
*   如果我们让毯子右端点与 `tiles[1]` 的右端点 `11` 对齐，毯子覆盖的区间是 `[11 - 6 + 1, 11] = [6, 11]`。这个区间覆盖了 `[10,11]` 这一整块瓷砖，长度为 2。
*   比较可知，最大覆盖数为 5。

### 核心思路：排序 + 滑动窗口

这类在连续区间上求解最值的问题，通常是滑动窗口算法的绝佳应用场景。为了让窗口能够顺利地在瓷砖上“滑动”，**先按瓷砖的起始位置排序**是必不可少的一步。

**为什么需要排序？**
排序后，我们可以从左到右依次处理每一块瓷砖，保证了我们的窗口指针不会来回跳跃。这使得我们能够维护一个包含“可能被当前毯子覆盖的”瓷砖集合的窗口。

算法的动态过程如下：
1.  **排序**：首先对 `tiles` 数组按照每块瓷砖的起始位置进行升序排序。
2.  **窗口设定**：我们使用 `left` 和 `right` 两个指针来维护一个窗口 `tiles[left...right]`。
3.  **窗口扩张**：`right` 指针不断向右移动，考察以 `tiles[right]` 为最右侧瓷砖的覆盖情况。
4.  **成本计算与窗口收缩**：
    *   我们假设毯子的**右端点**与 `tiles[right]` 的**右端点**对齐。那么毯子的起始位置就是 `carpet_start = tiles[right][1] - carpetLen + 1`。
    *   此时，窗口内的瓷砖 `tiles[left]` 可能已经完全处在这块毯子的左侧（即 `tiles[left][1] < carpet_start`），对当前的覆盖没有任何贡献。
    *   如果出现这种情况，我们就需要收缩窗口，将 `left` 指针向右移动，并将 `tiles[left]` 的长度从当前覆盖总数中减去，直到 `tiles[left]` 至少有一部分能被当前毯子覆盖。
5.  **结果更新**：
    *   在窗口调整完毕后，窗口 `[left...right]` 内的所有瓷砖都与当前毯子有交集。
    *   但需要特别注意，最左侧的 `tiles[left]` 可能只有**一部分**被覆盖。我们需要计算出它“伸出”毯子左侧的部分，并从总覆盖数中减去。
    *   用这个精确的覆盖数来更新全局的最大覆盖数。

通过这个“扩张-计算-收缩”的循环，我们就能在 O(N) 的时间内（排序后）找到答案。

### 算法详解

1.  **排序**：
    *   对输入数组 `tiles` 按起始位置 `l_i` 进行升序排序。

2.  **初始化**：
    *   左指针 `left = 0`。
    *   记录最大覆盖数 `max_coverage = 0`。
    *   记录当前窗口内所有瓷砖的总长度 `current_coverage = 0`。

3.  **遍历与扩张**：
    *   用右指针 `right` 从 `0` 到 `n-1` 遍历排序后的 `tiles` 数组。
    *   在循环的每一步，首先将新瓷砖 `tiles[right]` 的完整长度加入 `current_coverage`。

4.  **收缩**：
    *   确定当前毯子的理论起始位置 `carpet_start = tiles[right][1] - carpetLen + 1`。
    *   使用一个 `while` 循环检查窗口左边界的有效性。条件是：`tiles[left][1] < carpet_start`。
    *   只要条件成立，说明 `tiles[left]` 完全在毯子左侧，需要将其移出窗口。
    *   收缩操作：从 `current_coverage` 中减去 `tiles[left]` 的长度，然后将 `left` 指针右移一位。

5.  **计算精确覆盖并更新结果**：
    *   `while` 循环结束后，`tiles[left]` 是第一个与毯子有交集的瓷砖。
    *   计算 `tiles[left]` 可能伸出毯子左侧的长度（即未被覆盖的部分）：`lost_coverage = max(0, carpet_start - tiles[left][0])`。
    *   当前毯子位置的实际覆盖数是 `current_coverage - lost_coverage`。
    *   用这个实际覆盖数更新全局最大值：`max_coverage = max(max_coverage, current_coverage - lost_coverage)`。

6.  **返回**：
    *   `right` 指针遍历完整个数组后，`max_coverage` 中存储的就是最终答案。

### Python 代码实现

```python
class Solution:
    def maximumWhiteTiles(self, tiles: List[List[int]], carpetLen: int) -> int:
        # 步骤 1: 排序是关键前提
        tiles.sort()
        
        n = len(tiles)
        # 步骤 2: 初始化指针和状态变量
        left = 0
        max_coverage = 0
        current_coverage = 0  # 窗口内瓷砖的总长度
        
        # 步骤 3: 遍历数组，right 指针用于扩张窗口
        for right in range(n):
            start_r, end_r = tiles[right]
            current_coverage += (end_r - start_r + 1)
            
            # 确定当前毯子的理论起始位置
            carpet_start = end_r - carpetLen + 1
            
            # 步骤 4: 当窗口左边界不满足条件时，进行收缩
            while tiles[left][1] < carpet_start:
                start_l, end_l = tiles[left]
                current_coverage -= (end_l - start_l + 1)
                left += 1
            
            # 步骤 5: 计算精确覆盖并更新结果
            # 此时，tiles[left] 可能被部分覆盖，计算其未被覆盖的长度
            lost_coverage = 0
            if tiles[left][0] < carpet_start:
                lost_coverage = carpet_start - tiles[left][0]

            # 用 (窗口内总长度 - 左侧未覆盖长度) 更新最大值
            max_coverage = max(max_coverage, current_coverage - lost_coverage)
            
        return max_coverage
```

### 复杂度分析

*   **时间复杂度**: O(N log N)。其中 N 是 `tiles` 数组的长度。算法的瓶颈在于初始的排序步骤，其时间复杂度为 O(N log N)。后续的滑动窗口遍历中，`left` 和 `right` 指针都只会从头到尾移动一次，所以是 O(N) 的。因此，总的时间复杂度为 O(N log N)。
*   **空间复杂度**: O(log N) 或 O(N)。这取决于编程语言中排序算法的实现。例如，Python 的 Timsort 在最坏情况下需要 O(N) 的空间，而一些快速排序的实现会使用 O(log N) 的递归栈空间。如果我们不考虑排序所用的空间，则算法本身只使用了常数个额外变量，空间复杂度为 O(1)。

### 总结

本题是“排序 + 滑动窗口”组合拳的又一个典型应用。解决这类问题的关键路径非常清晰：
1.  **发现简化问题的关键性质**：通过分析题目，找到一个可以简化搜索范围的策略。在本题中，这个策略就是“最优毯子的端点必然与某块瓷砖的端点对齐”。
2.  **排序**：为了方便地处理区间，排序几乎是必须的预处理步骤。
3.  **定义窗口与状态**：明确滑动窗口的左右边界代表什么，以及需要维护哪些状态变量来高效判断窗口的有效性（本题中是 `current_coverage`）。
4.  **处理边界情况**：滑动窗口问题中，最精髓也最容易出错的地方，就是对窗口边界处部分元素的处理（本题中是对 `tiles[left]` 的部分覆盖计算）。
