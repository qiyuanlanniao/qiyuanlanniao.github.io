---
title: LeetCode 878 | 第 N 个神奇数字
date: 2025-11-24 09:15:00
updated: 2025-11-24 09:15:00
comments: true
tags:
  - 二分查找
  - 数学
  - 第95场周赛
categories:
  - 算法题解
  - 二分算法
  - 二分答案
  - 第K小/大
permalink: posts/leetcode-878/
---
{% centerquote %}
LeetCode 第 878 题：[第 N 个神奇数字](https://leetcode.cn/problems/nth-magical-number/)。
这是一道标准的“二分答案”题目，核心在于结合“容斥原理”来计算区间内满足条件的数的个数。
{% endcenterquote %}
<!--more-->

### 问题描述

一个正整数如果能被 `a` 或 `b` 整除，那么它是神奇的。

给定三个整数 `n`, `a`, `b`，返回第 `n` 个神奇的数字。因为答案可能很大，所以返回答案 **对 10^9 + 7 取模** 后的值。

**示例 1：**
```text
输入：n = 1, a = 2, b = 3
输出：2
```

**示例 2：**
```text
输入：n = 4, a = 2, b = 3
输出：6
```

**提示：**
*   1 <= n <= 10^9
*   2 <= a, b <= 4 * 10^4

### 解题思路

#### 1. 为什么暴力法不可行？

题目要求找到第 `n` 个神奇数字。如果 `n` 很大（例如 10^9），直接从 1 开始遍历整数并判断是否能被 `a` 或 `b` 整除，时间复杂度将非常高，肯定会超时。我们需要一种比线性遍历更高效的方法。

#### 2. 二分答案

我们可以发现一个单调性规律：
*   随着数字 `x` 的增大，小于等于 `x` 的神奇数字的个数 **只增不减**。

基于这个性质，我们可以使用 **二分查找** 来定位第 `n` 个神奇数字。
我们需要找到一个最小的整数 `x`，使得“小于等于 `x` 的神奇数字个数”恰好大于或等于 `n`。

**搜索范围：**
*   下界：`min(a, b)`（第一个神奇数字）。
*   上界：`n * min(a, b)`（最坏情况全是 `min(a, b)` 的倍数）。

#### 3. 容斥原理计算个数

二分的核心在于 `check(mid)` 函数：如何快速计算小于等于 `mid` 的正整数中，有多少个神奇数字？

这就用到了 **容斥原理**。
集合 A：能被 `a` 整除的数。
集合 B：能被 `b` 整除的数。
我们需要求 A 和 B 的并集大小。

公式如下：
```text
cnt = (mid // a) + (mid // b) - (mid // lcm(a, b))
```

其中：
*   `mid // a` 是能被 `a` 整除的数的个数。
*   `mid // b` 是能被 `b` 整除的数的个数。
*   `mid // lcm(a, b)` 是既能被 `a` 又能被 `b` 整除的数的个数（即它们的最小公倍数的倍数）。

我们需要减去重复计算的部分，即减去最小公倍数的倍数个数。

### 代码实现

{% tabs code_block %}
<!-- tab Python -->
```python
import math

class Solution:
    def nthMagicalNumber(self, n: int, a: int, b: int) -> int:
        # 计算最小公倍数
        # math.lcm 是 Python 3.9+ 引入的
        lcm = math.lcm(a, b)
        
        # 二分查找范围
        # 左边界 l 初始为一个不可能满足条件的值 (比第一个解还小)
        # 右边界 r 初始为一个必然满足条件的值
        l, r = min(a, b) + n - 2, min(a, b) * n
        
        # 使用 l + 1 < r 的开区间写法
        while l + 1 < r:
            mid = (l + r) // 2
            # 容斥原理计算小于等于 mid 的神奇数字个数
            # count = mid // a + mid // b - mid // lcm
            if mid // a + mid // b - mid // lcm >= n:
                r = mid
            else:
                l = mid
                
        return r % (10**9 + 7)
```
<!-- endtab -->
<!-- tab Go -->
```go
func nthMagicalNumber(n int, a int, b int) int {
    // 计算最小公倍数 LCM
    // 公式：lcm(a, b) = (a * b) / gcd(a, b)
    // 先除后乘可以防止中间结果溢出（虽然本题数据范围较小，但这是一种好习惯）
    lcm := a / gcd(a, b) * b
    
    // 二分查找范围
    // 这里的初始范围是基于题目特性的优化
    l, r := min(a, b)+n-2, min(a, b)*n
    
    for l+1 < r {
        mid := l + (r-l)/2
        // 容斥原理
        // 检查小于等于 mid 的神奇数字个数是否 >= n
        if mid/a + mid/b - mid/lcm >= n {
            r = mid
        } else {
            l = mid
        }
    }
    // 结果取模
    return r % 1_000_000_007
}

// 辅助函数：最大公约数
func gcd(a, b int) int {
    for a != 0 {
        a, b = b%a, a
    }
    return b
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```
<!-- endtab -->
{% endtabs %}

### 复杂度分析

*   **时间复杂度**: `O(log(n * min(a, b)))`
    二分的范围上限大约是 `n * min(a, b)`。由于是二分查找，时间复杂度是对数级别的。在 `check` 函数中，计算 GCD 和 LCM 的时间相对于二分过程几乎可以忽略不计。对于本题的数据范围，计算量非常小。
    
*   **空间复杂度**: `O(1)`
    我们只使用了常数个变量来存储 `l`, `r`, `mid`, `lcm` 等，不需要额外的数组空间。
