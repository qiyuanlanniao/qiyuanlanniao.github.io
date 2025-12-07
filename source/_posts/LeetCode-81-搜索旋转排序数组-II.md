---
title: LeetCode 81 | 搜索旋转排序数组 II
date: 2025-12-04 10:04:00
updated: 2025-12-04 10:04:00
comments: true
tags:
  - 二分查找
  - 数组
categories:
  - 算法题解
  - 二分算法
  - 其他
permalink: posts/leetcode-81/
---
{% centerquote %}
LeetCode 第 81 题：[搜索旋转排序数组 II](https://leetcode.cn/problems/search-in-rotated-sorted-array-ii/)。
该题难点在于数组中可能包含 **重复元素**，这破坏了原本二分查找中严格的单调性判断。当左右边界和中间值相等时，我们无法确定哪一半是有序的，必须退化为线性查找。
{% endcenterquote %}
<!--more-->

### 问题描述

已知存在一个按非降序排列的整数数组 `nums`，数组中的值不必互不相同。

在传递给函数之前，`nums` 在预先未知的某个下标 `k`（`0 <= k < nums.length`）上进行了 **旋转**，使数组变为 `[nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]`（下标 从 0 开始 计数）。

例如， `[0,1,2,4,4,4,5,6,6,7]` 在下标 5 处经旋转后可能变为 `[4,5,6,6,7,0,1,2,4,4]`。

给你 **旋转后** 的数组 `nums` 和一个整数 `target`，请你编写一个函数来判断给定的目标值是否存在于数组中。如果 `nums` 中存在这个目标值 `target`，则返回 `true`，否则返回 `false`。

**示例 1：**
```text
输入：nums = [2,5,6,0,0,1,2], target = 0
输出：true
```

**示例 2：**
```text
输入：nums = [2,5,6,0,0,1,2], target = 3
输出：false
```

**提示：**
*   1 <= nums.length <= 5000
*   -10^4 <= nums[i] <= 10^4
*   题目数据保证 nums 在预先未知的某个下标上进行了旋转
*   -10^4 <= target <= 10^4

### 解题思路

#### 1. 旋转数组的特性

旋转后的数组会被分割成两段有序区间：
1.  **左半段（上半段）**：数值较大，且 `nums[i] >= nums[0]`。
2.  **右半段（下半段）**：数值较小，且 `nums[i] <= nums[n-1]`。

#### 2. 处理重复元素（核心难点）

在没有重复元素的第 33 题中，我们可以通过比较 `nums[mid]` 和 `nums[r]`（或 `nums[l]`）来确定 `mid` 落在左半段还是右半段。

但是，当 `nums[mid] == nums[r]` 时，我们无法区分 `mid` 是在左半段（例如 `[2, 2, ..., 2, 0, 1]`）还是在右半段（例如 `[2, 0, 1, 2, ..., 2]`）。
此时，唯一的办法是 **缩小区间**，将右边界 `r` 向左移动一位 (`r -= 1`)，跳过这个重复值，然后再继续二分。这会导致最坏情况下的时间复杂度退化为 O(N)。

#### 3. 二分逻辑（check 函数）

我们将二分判定逻辑封装在 `check(i)` 函数中，该函数用于判断 **目标值 target 是否可能落在左侧区间 [l, mid] 中**（即是否应该让 `r = mid`）。

判定依据如下：
*   **Case 1: `nums[mid] > nums[r]`**
    说明 `mid` 落在 **左半段**（数值较大的那一段）。
    *   如果 `target > nums[r]`（说明 target 也在左半段）且 `nums[mid] >= target`，则 target 在 `mid` 的左侧，我们需要向左收缩 (`r = mid`)。
    
*   **Case 2: `nums[mid] <= nums[r]`**
    说明 `mid` 落在 **右半段**（数值较小的那一段）或者区间本身是有序的。
    *   如果 `target > nums[r]`，说明 target 实际上是在左半段（数值非常大），而当前 `mid` 在右半段，所以 target 肯定在 `mid` 的左侧（逻辑上的左侧，其实是回绕过去了），需要向左收缩。
    *   或者，如果 `nums[mid] >= target`，说明 target 在右半段且比 `mid` 小，也需要向左收缩。

### 代码实现

这里采用了开区间二分模板 `(l, r)`，循环条件为 `l + 1 < r`。

{% tabs code_block %}
<!-- tab Python -->
```python
from typing import List

class Solution:
    def search(self, nums: List[int], target: int) -> bool:
        # check 函数判断是否需要向左收缩区间 (让 r = mid)
        def check(i: int) -> bool:
            x = nums[i]
            # 情况 1: mid 在左半段（大数值段）
            # 因为旋转特性，左半段的所有值都应该大于 nums[r]（除非没旋转，但逻辑通用）
            if x > nums[r]:
                # 如果 target 也在左半段 (target > nums[r]) 并且 target 小于等于 x
                # 说明 target 在 [l...mid] 范围内
                return target > nums[r] and x >= target
            
            # 情况 2: mid 在右半段（小数值段）
            # 此时我们要找的情况是：什么时候 target 会在 mid 的左边？
            # 1. target 实际上在左半段 (target > nums[r])，而 mid 在右半段，所以 target 在 mid 左侧
            # 2. target 在右半段，且 x >= target，说明 target 在 [l...mid] 范围内
            return target > nums[r] or x >= target
        
        # 定义二分查找的左右边界（开区间写法）
        # l 是左边界的左侧一位，r 是右边界
        l, r = -1, len(nums) - 1
        
        while l + 1 < r:
            mid = (l + r) // 2
            
            # 核心去重逻辑：
            # 如果中点值等于右边界值，无法判断在左段还是右段
            # 只能线性排除右边界
            if nums[mid] == nums[r]:
                r -= 1
            # 如果 check 返回 True，说明 target 可能在左侧，收缩右边界
            elif check(mid):
                r = mid
            # 否则，target 在右侧，收缩左边界
            else:
                l = mid
                
        # 循环结束后，r 指向的位置就是可能的答案
        # 检查 nums[r] 是否等于 target
        return nums[r] == target
```
<!-- endtab -->
<!-- tab Go -->
```go
func search(nums []int, target int) bool {
    // 定义 check 函数 logic
    // 返回 true 表示 target 可能在 [l, mid] 范围内，需要 r = mid
    check := func(i int, r int) bool {
        x := nums[i]
        // Case 1: mid 在左半段 (值较大的一侧)
        if x > nums[r] {
            // target 必须也在左半段，且小于等于 mid 的值
            return target > nums[r] && x >= target
        }
        // Case 2: mid 在右半段 (值较小的一侧)
        // 满足以下任一条件则说明 target 在 mid 的左侧：
        // 1. target > nums[r]: target 实际上在左半段，而 mid 在右半段
        // 2. x >= target: target 在右半段且小于等于 mid
        return target > nums[r] || x >= target
    }

    // 初始化左右指针，使用开区间 (l, r] 的思路
    // l 初始为 -1, r 初始为 len-1
    l, r := -1, len(nums)-1

    for l+1 < r {
        mid := l + (r-l)/2
        
        // 如果 mid 和 r 指向的值相同，无法判断区间有序性
        // 此时只能将 r 向左移动一位，退化为线性操作
        if nums[mid] == nums[r] {
            r--
        } else if check(mid, r) {
            // 满足 check 条件，说明 target 在左侧，移动 r
            r = mid
        } else {
            // 否则 target 在右侧，移动 l
            l = mid
        }
    }

    // 最终检查 r 指向的位置是否为 target
    // 注意：如果是空数组或越界情况需额外处理，但题目保证 1 <= nums.length
    return nums[r] == target
}
```
<!-- endtab -->
{% endtabs %}

### 复杂度分析

*   **时间复杂度**:
    *   **平均情况**: `O(log N)`。当数组中重复元素较少时，主要进行二分查找。
    *   **最坏情况**: `O(N)`。当数组中包含大量重复元素（例如 `[1, 1, 1, ..., 1]`）时，每次 `nums[mid] == nums[r]` 都会导致 `r` 仅减小 1，退化为线性遍历。
*   **空间复杂度**: `O(1)`。仅使用了常数个变量存储指针和中间值。
```
