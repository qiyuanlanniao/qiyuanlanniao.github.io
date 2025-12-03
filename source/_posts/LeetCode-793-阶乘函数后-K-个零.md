---
title: LeetCode 793 | 阶乘函数后 K 个零
date: 2025-11-25 11:05:00
updated: 2025-11-25 11:05:00
comments: true
tags:
  - 二分查找
  - 数学
categories:
  - 算法题解
  - 二分算法
  - 二分答案
  - 第 K小/大
permalink: posts/leetcode-793/
---
{% centerquote %}
LeetCode 第 793 题：[阶乘函数后 K 个零](https://leetcode.cn/problems/preimage-size-of-factorial-zeroes-function/)。
这道题是 [172. 阶乘后的零](https://leetcode.cn/problems/factorial-trailing-zeroes/) 的逆向问题。核心在于理解阶乘末尾零的单调性，并利用二分查找来确定范围。
{% endcenterquote %}
<!--more-->

### 问题描述

f(x) 是 x! 末尾是 0 的数量。回想一下 x! = 1 * 2 * 3 * ... * x，且 0! = 1 。

例如， f(3) = 0 ，因为 3! = 6 的末尾没有 0 ；而 f(11) = 2 ，因为 11!= 39916800 末端有 2 个 0 。
给定 k，找出返回能满足 f(x) = k 的非负整数 x 的数量。

**示例 1：**
```text
输入：k = 0
输出：5
解释：0!, 1!, 2!, 3!, 和 4! 均符合 k = 0 的条件。
```

**示例 2：**
```text
输入：k = 5
输出：0
解释：没有匹配到这样的 x!，符合 k = 5 的条件。
```

**提示：**
*   0 <= k <= 10^9

### 解题思路

#### 1. 数学原理分析

首先，我们需要知道如何计算 x! 末尾 0 的个数。
末尾的 0 是由因子 2 * 5 产生的。在 1 到 x 的所有数中，因子 2 的数量远多于因子 5 的数量。因此，x! 末尾 0 的数量取决于因子 5 的数量。

根据勒让德定理（Legendre's Formula），f(x) 的计算公式如下：

```text
f(x) = floor(x / 5) + floor(x / 25) + floor(x / 125) + ...
```

简单来说，就是计算 x 中包含多少个 5，多少个 25，多少个 125……以此类推并求和。

#### 2. 函数的单调性与结果特性

观察函数 f(x)：
*   **单调性**：随着 x 增大，f(x) 的值是非递减的。
*   **阶梯状**：因为只有当 x 是 5 的倍数时，f(x) 的值才会增加。对于 x, x+1, x+2, x+3, x+4，它们的阶乘末尾 0 的个数是一样的。
*   **答案的取值**：基于上述特性，满足 f(x) = k 的 x 的数量，**要么是 5，要么是 0**。
    *   如果是 5：说明存在某个区间 [5m, 5m+4] 使得其阶乘末尾恰好有 k 个 0。
    *   如果是 0：说明 f(x) 的值直接从 k-1 跳到了 k+1（或更大），跳过了 k。这通常发生在 x 是 25, 125 等 5 的高次幂的倍数时（此时因子 5 的数量会一次性增加多个）。

#### 3. 二分查找解法

由于 f(x) 具有单调性，我们可以使用 **二分查找**。
题目要求满足 f(x) = k 的数量，这可以转化为求两个边界：
1.  满足 f(x) >= k 的最小 x（左边界）。
2.  满足 f(x) >= k + 1 的最小 x（右边界）。

最终答案即为：`search(k + 1) - search(k)`。
如果 k 存在解，差值为 5；如果不存在，差值为 0。

**搜索范围：**
*   因为 f(x) 大约等于 x/5 加上一些更小的项，所以 x 大约是 5*k。
*   考虑到 k 最大为 10^9，我们可以将二分的上界设为 5 * 10^9 左右，使用 64 位整数（long long 或 Python int）不会溢出。

### 代码实现

{% tabs code_block %}
<!-- tab Python -->
```python
class Solution:
    def preimageSizeFZF(self, k: int) -> int:
        # 计算 x! 末尾有多少个 0
        # 逻辑：f(x) = x//5 + x//25 + x//125 ...
        def zeta(x):
            res = 0
            while x:
                x //= 5
                res += x
            return res
        
        # 二分查找：寻找满足 zeta(x) >= target 的最小 x
        def search_min_x(target):
            # 搜索范围 [0, 5*target] 足够覆盖
            # 当 target 很大时，上界稍微放宽一点保证覆盖
            l, r = 0, 5 * target + 1
            while l < r:
                mid = (l + r) // 2
                if zeta(mid) < target:
                    l = mid + 1
                else:
                    r = mid
            return l
        
        # 逻辑：(满足 f(x) >= k+1 的最小 x) - (满足 f(x) >= k 的最小 x)
        # 如果 k 存在解，差值为 5；否则为 0
        return search_min_x(k + 1) - search_min_x(k)
```
<!-- endtab -->
<!-- tab Go -->
```go
func preimageSizeFZF(k int) int {
    // 辅助函数：计算 x! 末尾 0 的个数
    // 逻辑：count = x/5 + x/25 + x/125 ...
    zeta := func(x int) int {
        res := 0
        for x > 0 {
            x /= 5
            res += x
        }
        return res
    }

    // 二分查找：寻找满足 zeta(x) >= t 的最小 x
    search := func(t int) int {
        // 搜索范围：下界 0，上界 5*t + 1
        // 当 t=10^9 时，5*t 约为 50亿，在 64位 int 范围内
        l, r := 0, 5*t+1
        for l < r {
            mid := l + (r-l)/2
            if zeta(mid) < t {
                l = mid + 1
            } else {
                r = mid
            }
        }
        return l
    }

    // 利用差分思想：
    // 寻找 f(x)=k 的数量等价于：
    // (f(x) >= k+1 的起点) - (f(x) >= k 的起点)
    return search(k+1) - search(k)
}
```
<!-- endtab -->
{% endtabs %}

### 复杂度分析

*   **时间复杂度**: O(log^2 k)
    *   二分查找的范围大约是 5k，迭代次数为 O(log k)。
    *   每次迭代中计算 `zeta(mid)` 需要 O(log_5 mid) 的时间，也就是 O(log k)。
    *   总复杂度为 O(log k * log k)。对于 k = 10^9，计算量非常小，执行速度很快。
    
*   **空间复杂度**: O(1)
    *   只使用了常数个变量。
