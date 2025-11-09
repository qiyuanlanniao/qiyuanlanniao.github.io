---
title: LeetCode 2555 | DP与SW求解“两个线段获得的最多奖品”
date: 2025-10-02 10:05:00
updated: 2025-10-02 10:05:00
comments: true
tags:
  - 动态规划
  - 滑动窗口
  - 数组
  - 前缀和
  - 二分查找
  - 第97场双周赛
categories:
  - 算法题解
  - 滑动窗口与双指针
  - 不定长滑动窗口
permalink: posts/leetcode-2555/
---
{% centerquote %}
LeetCode 第 2555 题：[两个线段获得的最多奖品](https://leetcode.cn/problems/maximize-win-from-two-segments/description/)。
该题要求放置两个固定长度的线段来覆盖最多的点。直接寻找两个线段的最优位置组合非常复杂，但如果我们固定一个，再去优化另一个，问题就豁然开朗了。
{% endcenterquote %}
<!--more-->
### 问题解读

题目给了我们一个非递减的整数数组 `prizePositions`，代表 X 轴上奖品的位置，以及一个整数 `k`。我们需要在 X 轴上放置**两个**长度为 `k` 的线段，目标是让这两个线段覆盖的奖品总数达到最大。线段的端点必须是整数，并且它们可以重叠。

举个例子，对于 `prizePositions = [1,1,2,2,3,3,5]`, `k = 2`：
*   我们的目标是找到两个长度为 2 的线段，覆盖最多的奖品。
*   一个最优解是选择线段 `[1, 3]` 和 `[3, 5]`。
    *   线段 `[1, 3]` 覆盖了位置在 1, 2, 3 的所有奖品，共 6 个。
    *   线段 `[3, 5]` 覆盖了位置在 3, 5 的所有奖品，共 3 个。
    *   但是因为位置 3 的两个奖品被重复计算了，我们应该看它们覆盖的奖品集合。`[1,3]` 覆盖 `{1,1,2,2,3,3}`，`[3,5]` 覆盖 `{3,3,5}`。总共覆盖的奖品是 `{1,1,2,2,3,3,5}`，共 7 个。
*   题目要求的就是这个最大值 7。

直接去枚举两个线段的所有可能位置，复杂度太高。我们需要一种更高效的策略。

### 核心思路：动态规划与滑动窗口的结合

当需要同时优化两个变量（这里是两个线段的位置）时，一个常见的降维打击思路是：**先固定一个，再优化另一个**。

假设我们正在考虑第二个线段的位置。当我们用一个滑动窗口 `[j, i]` 来代表第二个线段覆盖的奖品时（即 `prizePositions[i] - prizePositions[j] <= k`），这个线段本身能获得的奖品数是固定的。为了让总数最大化，我们只需要为它搭配一个**在它左侧**的最佳第一线段。

这个“左侧最佳”正是动态规划可以发挥作用的地方。我们可以预先计算出对于任意位置 `x`，在 `x` 的左侧区域内，用**一个**线段能获得的最大奖品数是多少。

于是，整个解法分为清晰的两步：

1.  **预计算（第一遍滑动窗口）**：
    我们创建一个 `pre_max` 数组，其中 `pre_max[i]` 存储在 `prizePositions[0...i]` 这个前缀范围内，使用**一个**线段能获得的最大奖品数。这个数组可以通过一次滑动窗口遍历来高效计算。

2.  **最终求解（第二遍滑动窗口）**：
    我们再次使用一个滑动窗口 `[j, i]` 来枚举所有可能的**第二个线段**。对于每一个确定的第二个线段（覆盖 `i-j+1` 个奖品），我们利用 `pre_max` 数组，瞬间就能查到在它左边（即 `j` 之前）的最佳第一线段能覆盖多少奖品（`pre_max[j-1]`）。两者相加，就是当前组合下的总奖品数。我们遍历所有可能的第二个线段，就能找到全局最优解。

![Diagram Explanation](https://assets.leetcode.com/uploads/2023/01/21/screenshot-2023-01-22-at-000100.png)*上图形象地展示了第二个线段（蓝色）和其左侧最优的第一线段（绿色）的组合*

### 算法详解

1.  **第一步：预计算 `pre_max` 数组**
    *   初始化一个与 `prizePositions` 等长的 `pre_max` 数组，所有元素为 0。
    *   使用一个滑动窗口 `[left, right]` 从左到右遍历 `prizePositions`。
    *   对于每个 `right`，我们向右移动 `left` 指针，直到窗口满足 `prizePositions[right] - prizePositions[left] <= k`。
    *   此时，窗口 `[left, right]` 覆盖的奖品数为 `right - left + 1`。
    *   我们更新 `pre_max[right]`。它的值应该等于“在 `right` 之前能获得的最大奖品数 (`pre_max[right-1]`)”和“当前窗口覆盖的奖品数”中的较大者。即 `pre_max[right] = max(pre_max[right-1], right - left + 1)`。这确保了 `pre_max[i]` 存储的是 `0` 到 `i` 范围内的最优解。

2.  **第二步：计算最终结果**
    *   初始化最终答案 `ans = 0`。
    *   再次使用一个滑动窗口 `[j, i]` 从左到右遍历 `prizePositions`，这个窗口代表**第二个线段**。
    *   对于每个 `i`，同样移动 `j` 以维持窗口 `prizePositions[i] - prizePositions[j] <= k`。
    *   第二个线段覆盖的奖品数为 `prizes_second = i - j + 1`。
    *   第一个线段必须在第二个线段的左侧。它能获得的最大奖品数我们已经算好了，就是 `pre_max[j-1]`。（需要注意边界，如果 `j=0`，则左侧没有空间，奖品数为 0）。
    *   计算当前组合的总奖品数 `total = prizes_second + (pre_max[j-1] if j > 0 else 0)`。
    *   用 `total` 更新全局最大值 `ans`。

3.  **返回结果**
    *   遍历结束后，`ans` 即为所求。

### Python 代码实现

```python
from typing import List

class Solution:
    def maximizeWin(self, prizePositions: List[int], k: int) -> int:
        n = len(prizePositions)
        if n == 0:
            return 0

        # 步骤 1: 预计算 pre_max 数组
        # pre_max[i] 表示在 prizePositions[0...i] 范围内，用一个线段能获得的最大奖品数
        pre_max = [0] * n
        left = 0
        for right in range(n):
            # 维持窗口大小 prizePositions[right] - prizePositions[left] <= k
            while prizePositions[right] - prizePositions[left] > k:
                left += 1
            
            current_prizes = right - left + 1
            if right > 0:
                pre_max[right] = max(pre_max[right - 1], current_prizes)
            else:
                pre_max[right] = current_prizes
        
        # 步骤 2: 第二遍滑动窗口，计算最终结果
        # 窗口 [j, i] 代表第二个线段
        ans = 0
        j = 0
        for i in range(n):
            while prizePositions[i] - prizePositions[j] > k:
                j += 1
            
            # 第二个线段覆盖的奖品数
            prizes_second_segment = i - j + 1
            
            # 第一个线段在 j 左侧能获得的最大奖品数
            prizes_first_segment = 0
            if j > 0:
                prizes_first_segment = pre_max[j - 1]
            
            # 更新全局最大值
            ans = max(ans, prizes_first_segment + prizes_second_segment)
            
        return ans
```

### 复杂度分析

*   **时间复杂度**: O(N)。其中 N 是 `prizePositions` 的长度。我们对数组进行了两次独立的遍历，每次遍历中的左右指针都只单向移动，因此每次遍历都是线性的。
*   **空间复杂度**: O(N)。我们使用了一个与输入等长的 `pre_max` 数组来存储中间结果。

### 总结

本题是一个非常典型的“通过预计算降低复杂度”的案例。直接处理两个相互影响的变量很困难，但通过动态规划的思想，将其中一个变量的最优解预计算出来，再与另一个变量结合，问题就从 O(N^2) 级别的复杂度降低到了 O(N)。这种“先算一部分，再用算出的结果去算整体”的模式，是解决许多复杂数组和区间问题的金钥匙。
