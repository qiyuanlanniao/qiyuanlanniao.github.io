---
title: LeetCode 3116 | 单面值组合的第 K 小金额
date: 2025-11-29 17:22:00
updated: 2025-11-29 17:22:00
comments: true
tags:
  - 位运算
  - 数组
  - 数学
  - 二分查找
  - 组合数学
  - 数论
  - 第393场周赛
categories:
  - 算法题解
  - 二分算法
  - 二分答案
  - 第K小/大
permalink: posts/leetcode-3116/
---
{% centerquote %}
LeetCode 第 3116 题：[单面值组合的第 K 小金额](https://leetcode.cn/problems/kth-smallest-amount-with-single-denomination-combination/)。
一道结合了“二分答案”与“容斥原理”的经典数论题目。
{% endcenterquote %}
<!--more-->

### 问题描述

给你一个整数数组 `coins` 表示不同面额的硬币，另给你一个整数 `k`。
你有无限量的每种面额的硬币。但是，你 **不能** 组合使用不同面额的硬币。
也就是说，如果你选择面额为 `c` 的硬币，你只能制造 `c, 2c, 3c, ...` 这些金额。
返回使用这些硬币能制造的 **第 k 小** 的金额。

**示例 1：**
```text
输入：coins = [3,6,9], k = 3
输出：9
解释：
3元硬币产生：3, 6, 9, 12...
6元硬币产生：6, 12, 18...
9元硬币产生：9, 18, 27...
去重合并后：3, 6, 9, 12...
第3小的是9。
```

**提示：**
*   1 <= coins.length <= 15
*   1 <= coins[i] <= 25
*   1 <= k <= 2 * 10^9
*   coins 包含两两不同的整数。

### 解题思路

#### 1. 预处理：去重与排序

首先观察示例 1：`coins = [3, 6, 9]`。
*   `6` 是 `3` 的倍数，所有能由 `6` 产生的金额（6, 12...）一定都能由 `3` 产生。
*   同理，`9` 也是 `3` 的倍数。
*   因此，集合 `[3, 6, 9]` 等价于 `[3]`。

为了减少容斥原理计算时的集合数量（指数级复杂度），我们首先应该对 `coins` 进行排序，并移除那些“本身是其他较小面额倍数”的硬币。

#### 2. 二分答案

类似于求“第 K 小”的题目，通常具有单调性：
*   如果金额 `m` 能够覆盖至少 `k` 个生成数，那么比 `m` 大的金额也能覆盖至少 `k` 个。
*   我们需要找到满足 `count(m) >= k` 的最小 `m`。

**搜索范围：**
*   下界：`k`（假设最小面额是1）。
*   上界：`min(coins) * k`（最坏情况只使用最小面额）。

#### 3. 容斥原理

核心难点在于 `check(m)` 函数：计算小于等于 `m` 的金额中有多少个是给定硬币的倍数。
假设去重后的硬币集合为 `A`，我们需要计算 $|A_0 \cup A_1 \cup \dots \cup A_{n-1}|$。

根据容斥原理：
`Count = Σ(奇数个集合的交集大小) - Σ(偶数个集合的交集大小)`

*   集合 `coin_i` 在范围 `m` 内的元素个数为 `m // coin_i`。
*   多个集合的交集，即这些硬币的 **最小公倍数 (LCM)** 的倍数个数：`m // lcm(c1, c2, ...)`。

由于 `coins` 长度最多为 15，我们可以预处理所有子集的 LCM。
使用位掩码（Bitmask）`1` 到 `(1 << n) - 1` 来表示所有子集。

### 代码实现

{% tabs code_block %}
<!-- tab Python -->
```python
import math
from bisect import bisect_left
from typing import List

class Solution:
    def findKthSmallest(self, coins: List[int], k: int) -> int:
        coins.sort()
        a = []
        # 预处理：去除倍数关系的冗余硬币
        # 例如 [2, 4] -> [2]，因为 4 的倍数一定是 2 的倍数
        for x in coins:
            if all(x % y for y in a):
                a.append(x)

        # 预处理所有子集的 LCM
        # subset_lcm[mask] 存储掩码为 mask 的子集的最小公倍数
        n = len(a)
        subset_lcm = [1] * (1 << n)
        
        # 动态规划计算子集 LCM
        # 外层遍历每一个硬币，内层遍历已有的掩码
        for i, x in enumerate(a):
            bit = 1 << i
            for mask in range(bit):
                # 新的 LCM = lcm(旧子集 LCM, 新加入的硬币)
                subset_lcm[bit | mask] = math.lcm(subset_lcm[mask], x)

        # 二分查找的 check 函数
        def check(m: int) -> bool:
            cnt = 0
            # 遍历所有非空子集
            for i in range(1, len(subset_lcm)):  
                # 容斥原理：
                # 奇数个元素的集合（bit_count 为奇数）：加
                # 偶数个元素的集合（bit_count 为偶数）：减
                if i.bit_count() % 2:
                    cnt += m // subset_lcm[i]
                else:
                    cnt -= m // subset_lcm[i]
            return cnt >= k
            
        # 二分查找左边界
        # 搜索范围从 0 到 a[0] * k
        # key=check 会寻找第一个使 check(m) 为 True 的位置
        return bisect_left(range(a[0] * k), True, k, key=check)
```
<!-- endtab -->
<!-- tab Go -->
```go
import (
	"math/bits"
	"slices"
	"sort"
)

func findKthSmallest(coins []int, k int) int64 {
	slices.Sort(coins)
	// 预处理：去除冗余硬币
	// 使用切片过滤技巧，a 存储过滤后的硬币
	a := coins[:0]
next:
	for _, x := range coins {
		for _, y := range a {
			if x%y == 0 {
				continue next
			}
		}
		a = append(a, x)
	}

	// 预处理所有子集的 LCM
	subsetLcm := make([]int, 1<<len(a))
	subsetLcm[0] = 1
	for i, x := range a {
		bit := 1 << i
		// 遍历当前已生成的子集掩码
		for mask, l := range subsetLcm[:bit] {
			subsetLcm[bit|mask] = lcm(l, x)
		}
	}

	// 优化：预处理符号
	// 根据容斥原理，偶数个元素的子集需要减去，奇数个需要加上
	// 这里直接将偶数个元素的 LCM 设为负数，方便后续只做加法
	for i := range subsetLcm {
		if bits.OnesCount(uint(i))%2 == 0 {
			subsetLcm[i] *= -1
		}
	}

	// 二分查找
	// sort.Search 查找 [0, n) 范围内第一个使函数返回 true 的值
	ans := sort.Search(a[0]*k, func(m int) bool {
		cnt := 0
		// 遍历除空集外的所有子集 LCM
		for _, l := range subsetLcm[1:] {
			// 如果 l 是正数（奇数个元素），加
			// 如果 l 是负数（偶数个元素），相当于减去绝对值
			cnt += m / l
		}
		return cnt >= k
	})
	return int64(ans)
}

func gcd(a, b int) int {
	for a != 0 {
		a, b = b%a, a
	}
	return b
}

func lcm(a, b int) int {
	if a == 0 || b == 0 {
		return 0
	}
	return a / gcd(a, b) * b
}
```
<!-- endtab -->
{% endtabs %}

### 复杂度分析

假设 `coins` 去重后的长度为 `M` (M <= 15)。

*   **时间复杂度**: 
    *   **预处理 LCM**: `O(2^M)`。我们需要遍历所有可能的子集来计算 LCM。
    *   **二分查找**: 搜索范围上限约为 `min(coins) * k`，二分次数为 `O(log(min(coins) * k))`。
    *   **Check 函数**: 每次 check 需要遍历 `2^M` 个子集。
    *   总时间复杂度：`O(2^M · log(C·K))`。对于 `M=15`，`2^15 ≈ 3.2万`，二分约 60 次，总计算量在 200万次左右，完全可以通过。
    
*   **空间复杂度**: `O(2^M)`
    需要一个数组来存储所有子集的 LCM 值。
