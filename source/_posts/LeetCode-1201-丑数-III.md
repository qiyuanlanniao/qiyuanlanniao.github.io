---
title: LeetCode 1201 | 丑数 III
date: 2025-11-25 09:16:00
updated: 2025-11-25 09:16:00
comments: true
tags:
  - 二分查找
  - 数学
  - 组合数学
  - 数论
  - 第155场周赛
categories:
  - 算法题解
  - 二分算法
  - 二分答案
  - 第K小/大
permalink: posts/leetcode-1201/
---
{% centerquote %}
LeetCode 第 1201 题：[丑数 III](https://leetcode.cn/problems/ugly-number-iii/description/)。
核心思路依然是 **二分答案** 结合 **容斥原理**，区别在于变量从两个增加到了三个。
{% endcenterquote %}
<!--more-->

### 问题描述

给你四个整数：`n` 、`a` 、`b` 、`c` ，请你设计一个算法来找出第 `n` 个丑数。
丑数是可以被 `a` 或 `b` 或 `c` 整除的 **正整数** 。

**示例 1：**
```text
输入：n = 3, a = 2, b = 3, c = 5
输出：4
解释：丑数序列为 2, 3, 4, 5, 6, 8, 9, 10... 其中第 3 个是 4。
```

**示例 2：**
```text
输入：n = 4, a = 2, b = 3, c = 4
输出：6
解释：丑数序列为 2, 3, 4, 6, 8, 9, 10, 12... 其中第 4 个是 6。
```

**提示：**
*   `1 <= n, a, b, c <= 10^9`
*   `1 <= a * b * c <= 10^18`
*   本题结果在 `[1, 2 * 10^9]` 的范围内

### 解题思路

#### 1. 二分答案

与“第 N 个神奇数字”一样，丑数的分布具有单调性：
*   如果一个数字 `x` 是第 `k` 个丑数，那么小于 `x` 的丑数肯定少于 `k` 个，大于 `x` 的丑数肯定多于 `k` 个。
*   我们可以二分枚举答案 `x`，判断 `[1, x]` 范围内有多少个丑数。如果个数 `>= n`，说明答案在左半区间（尝试更小的 `x`）；否则在右半区间。

**搜索范围：**
*   下界：`1`（或者 `min(a, b, c)`）。
*   上界：题目提示结果在 `2 * 10^9` 范围内，可以直接设为 `2 * 10^9`。

#### 2. 集合的三维容斥原理

我们需要计算 `[1, mid]` 范围内，能被 `a` 或 `b` 或 `c` 整除的数的个数。
设：
*   集合 A：能被 `a` 整除的数。
*   集合 B：能被 `b` 整除的数。
*   集合 C：能被 `c` 整除的数。

我们需要求的是集合的并集大小 `|A ∪ B ∪ C|`。
根据 **容斥原理**：
`|A ∪ B ∪ C| = |A| + |B| + |C| - |A ∩ B| - |A ∩ C| - |B ∩ C| + |A ∩ B ∩ C|`

转换成具体的计算公式：
1.  `mid // a`：能被 `a` 整除的个数。
2.  `mid // lcm(a, b)`：既能被 `a` 又能被 `b` 整除的个数（即 `a` 和 `b` 的最小公倍数的倍数）。
3.  `mid // lcm(a, b, c)`：同时能被 `a, b, c` 整除的个数。

最终公式为：
```text
count = mid/a + mid/b + mid/c 
      - mid/lcm(a,b) - mid/lcm(a,c) - mid/lcm(b,c) 
      + mid/lcm(a,b,c)
```

#### 3. 最小公倍数 (LCM) 计算

*   两个数的 LCM：`lcm(a, b) = (a * b) / gcd(a, b)`
*   三个数的 LCM：`lcm(a, b, c) = lcm(lcm(a, b), c)`

注意：题目提示 `a * b * c` 可能达到 `10^18`，在计算 LCM 时要注意由乘法引起的溢出风险。Python 自动处理大整数，Go 语言需要使用 `int64`（不过本题结果范围在 `int` 内，中间计算 LCM 建议使用 64 位整数）。

### 代码实现

{% tabs code_block %}
<!-- tab Python -->
```python
import math

class Solution:
    def nthUglyNumber(self, n: int, a: int, b: int, c: int) -> int:
        # 预先计算两两的 LCM 和三者的 LCM
        # 避免在二分查找中重复计算
        ab = math.lcm(a,b)
        ac = math.lcm(a,c)
        bc = math.lcm(b,c)
        abc = math.lcm(ab,c)

        # 定义 check 函数：判定丑数个数是否 >=n 
        def check(x:int)->bool:
            # 容斥原理公式
            # + 单个
            count = x // a + x // b + x // c
            # - 两两交集
            count -= (x // lcm_ab + x // lcm_ac + x // lcm_bc)
            # + 三者交集
            count += x // lcm_abc
            return cnt>=n

        # 二分查找
        l,r = 0,2*10**9+1
        while l+1<r:
            mid = (l+r)//2
            if check(mid):
                r = mid
            else:
                l = mid
        return r
```
<!-- endtab -->
<!-- tab Go -->
```go
func nthUglyNumber(n int, a int, b int, c int) int {
    // 转换为 int64 防止 LCM 计算过程中溢出
    // 虽然题目结果在 int 范围，但 a*b*c 可能达到 10^18
    nA, nB, nC := int64(a), int64(b), int64(c)
    
    // 预计算最小公倍数
    lcmAB := lcm(nA, nB)
    lcmAC := lcm(nA, nC)
    lcmBC := lcm(nB, nC)
    lcmABC := lcm(lcmAB, nC)
    
    // 二分查找范围 [1, 2*10^9]
    // 使用 int64 进行二分
    var l, r int64 = 1, 2000000000
    target := int64(n)
    
    for l <= r {
        mid := l + (r-l)/2
        // 容斥原理计算数量
        count := mid/nA + mid/nB + mid/nC - mid/lcmAB - mid/lcmAC - mid/lcmBC + mid/lcmABC
        
        if count >= target {
            r = mid - 1
        } else {
            l = mid + 1
        }
    }
    
    return int(l)
}

// 辅助函数：最大公约数
func gcd(a, b int64) int64 {
    for a != 0 {
        a, b = b%a, a
    }
    return b
}

// 辅助函数：最小公倍数
func lcm(a, b int64) int64 {
    return (a / gcd(a, b)) * b
}
```
<!-- endtab -->
{% endtabs %}

### 复杂度分析

*   **时间复杂度**: `O(log(Range))`
    这里 `Range` 取决于题目给出的结果上限 `2 * 10^9`。每次 `check` 操作仅涉及常数次加减乘除运算。由于预先计算了 LCM，二分过程非常快。
    
*   **空间复杂度**: `O(1)`
    只使用了常数个变量来存储 LCM 值和二分边界。
