---
title: LeetCode 3134 | 找出唯一性数组的中位数
date: 2025-11-30 17:00:00
updated: 2025-11-30 17:00:00
comments: true
tags:
  - 二分查找
  - 滑动窗口
  - 数组
  - 哈希表
  - 第395场周赛
categories:
  - 算法题解
  - 二分算法
  - 二分答案
  - 第K小/大
permalink: posts/leetcode-3134/
---
{% centerquote %}
LeetCode 第 3134 题：[找出唯一性数组的中位数](https://leetcode.cn/problems/find-the-median-of-the-uniqueness-array/)。
我们需要在 O(N²) 的子数组空间中，高效地找到第 K 小的唯一性计数。
{% endcenterquote %}
<!--more-->

### 问题描述

给定一个整数数组 `nums`。
数组的 **唯一性数组** 是一个数组，包含 `nums` 的所有非空子数组中不同元素的个数。
换句话说，对于 `nums` 的每一个子数组，计算其不同元素的个数，将这些值收集起来形成一个新的数组。

需要返回这个 **唯一性数组** 的 **中位数**。

注意：
*   子数组是数组中连续的一段。
*   如果数组长度为偶数，中位数是排序后中间两个数的较小者（即第 `(total + 1) / 2` 个数，向上取整的逻辑）。

**示例 1：**
```text
输入：nums = [1, 2, 3]
输出：1
解释：
子数组及其不同元素个数：
[1]: 1
[2]: 1
[3]: 1
[1, 2]: 2
[2, 3]: 2
[1, 2, 3]: 3
唯一性数组为 [1, 1, 1, 2, 2, 3]。
排序后为 [1, 1, 1, 2, 2, 3]，中位数是 1。
```

**提示：**
*   `1 <= nums.length <= 10^5`
*   `1 <= nums[i] <= 10^5`

### 解题思路

#### 1. 问题转化：二分答案

直接暴力生成所有子数组的唯一性值需要 O(N²) 的时间，这在 `N = 10^5` 的数据规模下是不可接受的。
我们需要找到一个数 `x`，使得唯一性数组中 **小于等于** `x` 的元素个数至少占总数的一半。

这就具备了单调性：
*   如果我们允许子数组包含更多种类的元素（即 `x` 变大），那么满足条件的子数组数量一定会增加或持平。

因此，我们可以对答案（即子数组中不同元素的个数）进行 **二分查找**。
答案的范围是 `[1, N]`（或者更精确地说是 `[1, len(set(nums))]`）。

#### 2. 中位数的定义

对于长度为 `n` 的数组，非空子数组的总数 `Total` 为 `n * (n + 1) / 2`。
题目要求的中位数，本质上是求第 `k` 小的数，其中 `k = (Total + 1) / 2`（整数除法）。

我们需要找到最小的 `limit`，使得：
**不同元素个数 ≤ limit 的子数组数量 ≥ k**

#### 3. 计数检查：滑动窗口（双指针）

核心问题变成了：如何快速计算“不同元素个数 ≤ limit”的子数组有多少个？
我们可以使用 **滑动窗口** 在 O(N) 时间内完成统计：

1.  维护一个窗口 `[l, r]` 和一个哈希表 `freq` 记录窗口内元素的出现次数。
2.  遍历右端点 `r`，将 `nums[r]` 加入窗口。
3.  如果窗口内不同元素的个数（即 `len(freq)`）超过了 `limit`，则移动左端点 `l`，直到窗口恢复合法。
4.  对于当前固定的右端点 `r`，所有以 `r` 结尾且起始位置在 `[l, r]` 之间的子数组都是合法的。这样的子数组个数为 `r - l + 1`。
5.  累加这些个数。

### 代码实现

{% tabs code_block %}
<!-- tab Python -->
```python
class Solution:
    def medianOfUniquenessArray(self, nums: List[int]) -> int:
        n = len(nums)
        # 计算子数组总数 n*(n+1)/2，并确定中位数的位置 k
        # 这里使用的是向上取整的逻辑，即第 (Total + 1) // 2 小的数
        k = (n * (n + 1) // 2 + 1) // 2

        # 核心校验函数：检查是否存在至少 k 个子数组，
        # 其不同元素个数小于等于 upper
        def check(upper: int) -> bool:
            cnt = l = 0
            freq = defaultdict(int)
            for r, in_ in enumerate(nums):
                freq[in_] += 1  # 移入右端点，更新计数
                
                # 当窗口内不同元素的个数超过上限时，收缩左边界
                while len(freq) > upper:  # 窗口内元素过多
                    out = nums[l]
                    freq[out] -= 1  # 移出左端点
                    if freq[out] == 0:
                        del freq[out] # 如果次数归零，从 map 中移除，len(freq) 会减少
                    l += 1
                
                # 对于当前的右端点 r，合法的左端点范围是 [l, r]
                # 因此以 r 结尾且满足条件的子数组个数为 r - l + 1
                cnt += r - l + 1
                
                # 剪枝：如果计数已经达到 k，提前返回 True
                if cnt >= k:
                    return True
            return False

        # 在 [1, len(set(nums))] 范围内二分查找
        # bisect_left 会寻找第一个使得 check(val) 为 True 的值
        # range 的范围需要覆盖可能的解空间
        return bisect_left(range(len(set(nums))), True, 1, key=check)
```
<!-- endtab -->
<!-- tab Go -->
```go
func medianOfUniquenessArray(nums []int) int {
	n := len(nums)
    // 计算中位数是第 k 小的数
    // 总子数组数为 n*(n+1)/2
	k := (n*(n+1)/2 + 1) / 2
    
    // 使用 sort.Search 进行二分查找
    // Search(n) 会在 [0, n) 范围内查找，我们查找的实际范围与数组长度相关
    // 但答案上限是不同元素的个数，这里用 n-1 作为搜索空间的基数
	ans := 1 + sort.Search(n-1, func(upper int) bool {
		upper++ // sort.Search 的参数是从 0 开始的，实际含义需要 +1
		cnt := 0
		l := 0
		freq := map[int]int{} // 哈希表记录窗口内元素频率
		for r, in := range nums {
			freq[in]++ // 移入右端点
            
            // 如果窗口内不同元素数量超过 upper，则收缩左端点
			for len(freq) > upper { // 窗口内元素过多
				out := nums[l]
				freq[out]-- // 移出左端点
				if freq[out] == 0 {
					delete(freq, out) // 彻底移除元素，确保 len(freq) 正确
				}
				l++
			}
            
            // 累加当前右端点 r 对应的合法子数组数量
			cnt += r - l + 1 // 右端点固定为 r 时，有 r-l+1 个合法左端点
            
            // 如果计数满足要求，返回 true，二分查找会尝试更小的值
			if cnt >= k {
				return true
			}
		}
		return false
	})
	return ans
}
```
<!-- endtab -->
{% endtabs %}

### 复杂度分析

*   **时间复杂度**: O(N * log(M))
    *   其中 N 是数组 `nums` 的长度，M 是数组中不同元素的个数（最坏情况下 M = N）。
    *   二分查找进行 O(log M) 次迭代。
    *   每次 `check` 函数中使用滑动窗口遍历整个数组，时间复杂度为 O(N)。
    *   总复杂度为 O(N log M)。
    
*   **空间复杂度**: O(M)
    *   主要消耗在于滑动窗口中的哈希表 `freq`，其大小最多存储 M 个不同的元素。
