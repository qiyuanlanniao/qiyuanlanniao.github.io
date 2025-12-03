---
title: LeetCode 1515 | 服务中心的最佳位置
date: 2025-12-01 09:10:00
updated: 2025-12-01 09:10:00
comments: true
tags:
  - 几何
  - 数学
  - 数组
  - 随机化
  - 第197场周赛
categories:
  - 算法题解
  - 二分算法
  - 三分法
permalink: posts/leetcode-1515/
---
{% centerquote %}
LeetCode 第 1515 题：[服务中心的最佳位置](https://leetcode.cn/problems/best-position-for-a-service-centre/)。
我们需要找到一个点，使其到所有给定点的欧几里得距离之和最小。
{% endcenterquote %}
<!--more-->

### 问题描述

一家快递公司希望在新城市建立新的服务中心。公司统计了该城市所有客户在二维地图上的坐标 `positions`，其中 `positions[i] = [xi, yi]`。

请你为服务中心选址 `[xcentre, ycentre]`，使服务中心到所有客户的 **欧几里得距离的总和最小**。

返回该最小距离总和。答案与真实值误差在 `10^-5` 之内的将被视作正确答案。

**示例 1：**
```text
输入：positions = [[0,1],[1,0],[1,2],[2,1]]
输出：4.00000
解释：选 [1, 1] 作为新中心的位置，到每个客户的距离都是 1，总和为 4。
```

### 解题思路

#### 1. 几何重心 vs 几何中位数

*   **几何重心（Centroid）**：是所有点坐标的平均值 `(sum(x)/n, sum(y)/n)`。它最小化的是到所有点的**距离平方和**。
*   **几何中位数（Geometric Median）**：最小化的是到所有点的**欧几里得距离之和**。

本题要求的是后者。不幸的是，当点的数量 `n > 4` 时，不存在通用的封闭公式（即无法直接用公式算出来）。我们需要使用**迭代近似**的方法。

#### 2. 凸函数的性质

目标函数 `f(x, y) = Σ sqrt((xi-x)^2 + (yi-y)^2)` 是一个**凸函数**。
这意味着：
1.  它形状像一个碗，只有一个最低点（局部最小值即全局最小值）。
2.  我们可以使用**梯度下降（Gradient Descent）**或**爬山法（Hill Climbing）**等迭代算法，从任意初始点出发，一步步逼近最低点。

#### 3. 求解方法

*   **Python (SciPy)**: 题目允许使用数值优化库，Python 的 `scipy.optimize` 模块提供了现成的最小化函数求解器，非常适合处理这类非线性优化问题。
*   **Go (手动迭代)**: 由于标准库通常不包含此类高级数学优化器，我们可以手写一个简单的**步长衰减法**（类似梯度下降或三分法的变种）：
    1.  **初始化**：将中心点设为所有点的重心（平均值），这是一个很好的起始猜测。
    2.  **迭代**：尝试向 上、下、左、右 四个方向移动。
    3.  **贪心策略**：如果移动后的新位置能使总距离变小，就更新中心点到新位置。
    4.  **衰减步长**：如果四个方向都不能使距离变小，说明当前步长太大，需要减小步长（例如除以 2）以进行微调。
    5.  **终止**：当步长非常小（如小于 `1e-7`）时停止。

### 代码实现

{% tabs code_block %}
<!-- tab Python -->
```python
from scipy.optimize import minimize
from math import dist
from typing import List

class Solution:
    def getMinDistSum(self, positions: List[List[int]]) -> float:
        # 使用 SciPy 的 minimize 函数进行数值优化
        # minimize 寻找目标函数的最小值
        # 参数 1: lambda t ... : 目标函数。输入是点 t (x, y)，输出是 t 到所有 positions 的欧几里得距离之和
        # 参数 2: (0, 0) : 初始猜测值 (x0, y0)。虽然重心是更好的猜测，但 (0,0) 对于凸函数也能收敛
        # 返回值: minimize 返回一个对象，['fun'] 属性包含了目标函数的最小值
        return minimize(lambda t: sum([dist(p, t) for p in positions]), (0, 0))['fun']
```
<!-- endtab -->
<!-- tab Go -->
```go
import (
    "math"
)

func getMinDistSum(positions [][]int) float64 {
    // 辅助函数：计算点 (xc, yc) 到所有点的欧几里得距离之和
    getSum := func(xc, yc float64) float64 {
        sum := 0.0
        for _, pos := range positions {
            // dist = sqrt((x1-x2)^2 + (y1-y2)^2)
            sum += math.Sqrt(math.Pow(float64(pos[0])-xc, 2) + math.Pow(float64(pos[1])-yc, 2))
        }
        return sum
    }

    // 1. 初始化中心点 (x, y) 为几何重心（平均值）
    // 这是一个很好的起始位置，比 (0,0) 更接近答案
    x, y := 0.0, 0.0
    n := float64(len(positions))
    for _, pos := range positions {
        x += float64(pos[0])
        y += float64(pos[1])
    }
    x /= n
    y /= n

    // 2. 迭代参数设置
    // 初始步长，由于坐标范围是 0-100，步长设为 1 比较合理
    step := 1.0 
    // 精度控制，题目要求 10^-5，我们设置得更小一点以保证准确度
    epsilon := 1e-7 
    // 衰减率，每次无法移动时步长缩小的比例
    decay := 0.5 

    // 定义四个移动方向：左、右、下、上
    dirs := [4][2]float64{{-1, 0}, {1, 0}, {0, -1}, {0, 1}}

    // 计算当前的最小距离和
    minDist := getSum(x, y)

    // 3. 开始迭代，直到步长小于阈值
    for step > epsilon {
        foundBetter := false
        
        // 尝试往四个方向走一步
        for _, d := range dirs {
            nx := x + d[0]*step
            ny := y + d[1]*step
            
            newDist := getSum(nx, ny)
            
            // 如果新位置的距离和更小，则移动中心点
            if newDist < minDist {
                minDist = newDist
                x = nx
                y = ny
                foundBetter = true
                // 这里可以选择 break 也可以不 break
                // 不 break 意味着一轮可能移动多次，通常不 break 收敛更快
                break 
            }
        }
        
        // 如果四个方向都不能优化结果，说明当前步长太大，需要缩小步长
        if !foundBetter {
            step *= decay
        }
    }

    return minDist
}
```
<!-- endtab -->
{% endtabs %}

### 复杂度分析

*   **时间复杂度**:
    *   **Python**: 取决于 SciPy 内部优化算法（通常是 BFGS 或类似算法），一般迭代次数不多。
    *   **Go**: `O(N * K)`，其中 `N` 是点的数量，`K` 是迭代次数。迭代次数与 `step` 从 1 衰减到 `1e-7` 的过程有关，约为 `log(1/epsilon)`。由于目标函数是凸的，收敛速度非常快。
*   **空间复杂度**: `O(1)`，只需要常数级别的变量存储坐标和距离。

### 注意事项

1.  **Python 环境**: `scipy` 是第三方科学计算库，在标准的 LeetCode 在线环境中**通常不可用**（除非是特定的支持全库的比赛环境）。Python 解法在本地环境或支持 `scipy` 的 OJ 上非常方便，但在标准 LeetCode 环境下，建议使用类似 Go 语言解法中的模拟退火或梯度下降逻辑来实现 Python 版本。
2.  **局部最优即全局最优**: 因为距离和函数是**凸函数**，所以我们不需要担心算法陷入局部最优解，简单的贪心移动策略配合步长衰减一定能找到全局最优解。
