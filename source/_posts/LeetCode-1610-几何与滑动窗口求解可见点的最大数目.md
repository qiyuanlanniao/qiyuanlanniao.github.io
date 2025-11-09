---
title: LeetCode 1610 | 几何与滑动窗口：求解可见点的最大数目
date: 2025-10-03 10:08:00
updated: 2025-10-03 10:08:00
comments: true
tags:
  - 几何
  - 数学
  - 排序
  - 数组
  - 滑动窗口
  - 第209场周赛
categories:
  - 算法题解
  - 滑动窗口与双指针
  - 不定长滑动窗口
permalink: posts/leetcode-1610/
---
{% centerquote %}
LeetCode 第 1610 题：[可见点的最大数目](https://leetcode.cn/problems/maximum-number-of-visible-points/)。
这道题将一个几何问题巧妙地包装起来，表面上看是在一个二维平面内旋转视野，但其核心可以转化为一个在一维数组上处理循环问题的经典滑动窗口场景。
{% endcenterquote %}
<!--more-->
### 问题解读

题目给了我们一组二维点 `points`，一个观察者的位置 `location`，以及一个视野角度 `angle`。我们可以站在 `location` 原地旋转，视野范围是一个宽度为 `angle` 的扇形区域。我们需要找到一个最佳的朝向，使得能看到的点的数量最多。

有几个关键点需要注意：
1.  与观察者位置重合的点，永远都是可见的。
2.  点不会遮挡其他点。
3.  我们需要求解的是**最多**能同时看到多少个点。

直接去模拟旋转并计算每个角度下可见点的数量，无疑是低效且复杂的。我们需要一种更数学化、更高效的方法来解决这个问题。

### 核心思路：坐标转换与滑动窗口

这个问题的核心在于，将二维的坐标问题，转化为一维的角度问题。

1.  **坐标系中心化**：首先，我们将观察者的位置 `location` 视为坐标原点 `(0,0)`。所有 `points` 都需要进行相应的平移，即 `(px, py)` 变为 `(px - loc_x, py - loc_y)`。

2.  **计算极角**：对于每一个平移后的点，我们可以计算它相对于新原点（即观察者位置）和正东方向（x轴正方向）的夹角。在编程中，`math.atan2(y, x)` 函数是完成这项工作的完美工具，它可以精确地计算出 `(-π, π]` 范围内的弧度。我们再将弧度转换为 `(-180, 180]` 范围内的角度。

3.  **处理循环问题**：现在，所有的点都被映射到了一系列角度上。我们的视野是一个宽度为 `angle` 的“窗口”，可以在 `[-180, 180]` 这个角度圈上滑动。这里最大的挑战是**循环性**。例如，如果 `angle` 是 90 度，一个从 350 度开始的视野会覆盖到 `[350, 360]` 和 `[0, 80]` 这个范围。直接在数组上处理这种跨越边界的情况非常麻烦。

    **关键技巧**：为了打破这个循环，我们可以将角度数组“线性化”。首先对所有角度进行排序。然后，我们将每个角度值加上 360 度，并将这些新值追加到原数组的末尾。例如，一个排序后的角度数组 `[10, 150, 350]` 会被扩展为 `[10, 150, 350, 370, 510, 710]`。
    
    这样做之后，一个从 350 度到 10 度的视野，就等价于在扩展数组中一个从 350 度到 370 度 (`10 + 360`) 的连续区间。问题就转化成了一个标准的滑动窗口问题。

4.  **应用滑动窗口**：在新生成的、线性且有序的角度数组上，我们寻找一个窗口，其宽度（即 `angles[right] - angles[left]`）不超过 `angle`，并使其包含的元素数量最多。

最终，滑动窗口找到的最大点数，加上一开始就统计好的、与观察者位置重合的点数，就是我们的答案。

### 算法详解

1.  **预处理**：
    *   创建一个空列表 `angles` 用于存放所有点的角度。
    *   初始化一个计数器 `same_location_points = 0`。
    *   遍历所有 `points`，如果一个点与 `location` 重合，则 `same_location_points` 加一。

2.  **计算并转换角度**：
    *   对于不与 `location` 重合的点，将其坐标相对于 `location` 平移。
    *   使用 `math.atan2(y, x)` 计算角度（弧度制），然后用 `math.degrees()` 转换为角度制。将结果存入 `angles` 列表。

3.  **排序与数组扩展**：
    *   对 `angles` 列表进行升序排序。
    *   为了处理视野跨越 0/360 度的情况，遍历排序后的 `angles` 列表，将每个角度 `a` 加上 360 后，作为一个新元素追加到 `angles` 列表的末尾。

4.  **滑动窗口**：
    *   初始化左指针 `left = 0` 和最大可见点数 `max_visible = 0`。
    *   用右指针 `right` 从 `0` 开始遍历扩展后的 `angles` 数组。
    *   在循环中，检查当前窗口 `angles[left]` 到 `angles[right]` 的角度差是否大于 `angle`。
    *   如果 `angles[right] - angles[left] > angle`，说明窗口过大，需要收缩。我们将 `left` 指针向右移动，直到窗口重新满足条件。
    *   在每次移动 `right` 之后（并可能收缩 `left` 之后），当前窗口 `s[left:right+1]` 是一个有效的视野范围。我们用它的长度 `right - left + 1` 来更新 `max_visible`。

5.  **计算最终答案**：
    *   遍历结束后，`max_visible` 就是在非重合点中能看到的最大数量。
    *   最终结果是 `max_visible + same_location_points`。

### Python 代码实现

```python
import math
from typing import List

class Solution:
    def visiblePoints(self, points: List[List[int]], angle: int, location: List[int]) -> int:
        
        angles = []
        same_location_points = 0
        loc_x, loc_y = location[0], location[1]

        # 步骤 1 & 2: 处理重合点并计算所有其他点的角度
        for p_x, p_y in points:
            if p_x == loc_x and p_y == loc_y:
                same_location_points += 1
            else:
                # 使用 atan2 计算弧度，再转换为角度
                rad = math.atan2(p_y - loc_y, p_x - loc_x)
                angles.append(math.degrees(rad))
        
        # 步骤 3: 排序并扩展数组以处理循环
        angles.sort()
        n = len(angles)
        for i in range(n):
            angles.append(angles[i] + 360.0)
            
        max_visible = 0
        left = 0
        
        # 如果除了重合点外没有其他点
        if not angles:
            return same_location_points

        # 步骤 4: 滑动窗口
        for right in range(len(angles)):
            # 如果窗口角度范围超过 angle，收缩左边界
            while angles[right] - angles[left] > angle:
                left += 1
            
            # 更新最大可见点数
            max_visible = max(max_visible, right - left + 1)
                
        # 步骤 5: 最终结果是窗口内的最大点数加上重合点数
        return max_visible + same_location_points
```

### 复杂度分析

*   **时间复杂度**: O(N log N)。其中 N 是点的数量。算法的瓶颈在于对所有点的角度进行排序。计算角度和滑动窗口部分都是 O(N) 的。
*   **空间复杂度**: O(N)。我们需要一个列表来存储 N 个点的角度，并且为了处理循环问题，这个列表的长度会翻倍。

### 总结

本题是一个很好的例子，展示了如何将一个几何问题抽象和转化为我们熟悉的算法模型。核心的转化步骤有两步：
1.  **降维**：通过计算极角，将二维坐标问题转化为一维的角度数组问题。
2.  **线性化**：通过排序和扩展数组，将一个循环数组上的问题转化为一个普通线性数组上的问题。

完成这两步转化后，问题就变成了一个可以用滑动窗口模板轻松解决的经典问题。这个“扩展数组以处理循环”的技巧在很多涉及环形数组的问题中都非常有用。
