---
title: LeetCode 4 | 寻找两个正序数组的中位数
date: 2025-12-05 11:30:00
updated: 2025-12-05 11:30:00
comments: true
tags:
  - 二分查找
  - 分治
  - 数组
categories:
  - 算法题解
  - 二分算法
  - 其他
permalink: posts/leetcode-4/
---
{% centerquote %}
LeetCode 第 4 题：[寻找两个正序数组的中位数](https://leetcode.cn/problems/median-of-two-sorted-arrays/)。
题目要求算法的时间复杂度为 O(log (m+n))。通常涉及数组的题目如果要求对数级复杂度，核心思路大概率是 **二分查找**。
{% endcenterquote %}
<!--more-->

### 问题描述

给定两个大小分别为 `m` 和 `n` 的正序（从小到大）数组 `nums1` 和 `nums2`。请你找出并返回这两个正序数组的 **中位数**。

算法的时间复杂度应该为 O(log (m+n)) 。

**示例 1：**
```text
输入：nums1 = [1,3], nums2 = [2]
输出：2.00000
解释：合并数组 = [1,2,3] ，中位数 2
```

**示例 2：**
```text
输入：nums1 = [1,2], nums2 = [3,4]
输出：2.50000
解释：合并数组 = [1,2,3,4] ，中位数 (2 + 3) / 2 = 2.5
```

**提示：**
*   nums1.length == m
*   nums2.length == n
*   0 <= m <= 1000
*   0 <= n <= 1000
*   1 <= m + n <= 2000
*   -10^6 <= nums1[i], nums2[i] <= 10^6

### 解题思路

#### 1. 核心思想：划分数组

要在两个有序数组中找到中位数，本质上是要将两个数组分别切成两部分（左半部分和右半部分），满足以下两个条件：
1.  **数量平衡**：左半部分的总元素个数等于右半部分（或者比右半部分多 1 个）。
2.  **交叉有序**：左半部分的所有元素都要小于等于右半部分的所有元素。即：`Left_Part_Max ≤ Right_Part_Min`。

#### 2. 处理边界问题的技巧（哨兵法）

在常规解法中，处理数组下标越界（例如切分点在数组开头或结尾）是非常繁琐的。
本题解采用了一种巧妙的 **预处理** 方式：
*   在两个数组的首尾分别加上 **负无穷 (-inf)** 和 **正无穷 (inf)**。
*   这样不仅避免了下标越界的判断，还保证了数组长度永远够用，逻辑更加统一。

#### 3. 二分查找切分点

我们只需要对长度较短的那个数组进行二分查找。假设我们在短数组 `A` 中切一刀，下标为 `i`，那么为了满足“数量平衡”，长数组 `B` 的切分位置 `j` 也就确定了：`j = (m + n + 1) / 2 - i`

我们需要找到一个 `i`，使得 `A[i] ≤ B[j+1]` 且 `B[j] ≤ A[i+1]`。
由于数组是有序的，我们只需要通过二分调整 `i` 的位置即可。

### 代码实现

{% tabs code_block %}
<!-- tab Python -->
```python
import math
from typing import List

class Solution:
    def findMedianSortedArrays(self, a: List[int], b: List[int]) -> float:
        # 确保 a 是较短的数组，这样可以缩短二分查找的区间，优化性能
        if len(a) > len(b):
            a, b = b, a
        
        m, n = len(a), len(b)
        
        # 技巧：在数组首尾添加无穷小和无穷大作为哨兵
        # 这样可以避免处理边界情况（如切分点在数组最左或最右侧时）
        # inf 表示正无穷，-inf 表示负无穷
        inf = float('inf')
        a = [-inf] + a + [inf]
        b = [-inf] + b + [inf]
        
        # 二分查找的范围：在处理后的数组 a 中寻找分割点
        # l 和 r 是基于开区间的二分查找 (l, r)
        l, r = 0, m + 1
        
        # 开始二分查找
        # 目标是找到一个位置 i，使得 a[i] <= b[j+1]
        while l + 1 < r:
            i = (l + r) // 2
            # 计算 b 数组对应的分割点 j
            # (m + n + 1) // 2 是左半部分需要的总元素个数（不含哨兵的逻辑）
            # 这里的下标计算考虑了前面的哨兵位
            j = (m + n + 1) // 2 - i
            
            # 判断交叉条件：a 的左侧最大值是否小于等于 b 的右侧最小值
            if a[i] <= b[j+1]:
                l = i # i 可能是合法的，尝试向右找更大的 i
            else:
                r = i # i 太大了，需要向左找
        
        # 循环结束时，l 就是我们要找的分割点 i
        i = l
        j = (m + n + 1) // 2 - i
        
        # max1：左半部分的最大值（取 a[i] 和 b[j] 较大的那个）
        max1 = max(a[i], b[j])
        # min2：右半部分的最小值（取 a[i+1] 和 b[j+1] 较小的那个）
        min2 = min(a[i+1], b[j+1])
        
        # 如果总长度是奇数，中位数就是左半部分的最大值
        # 如果总长度是偶数，中位数是左侧最大值和右侧最小值的平均数
        return max1 if (m + n) % 2 else (max1 + min2) / 2
```
<!-- endtab -->
<!-- tab Go -->
```go
import (
    "math"
)

func findMedianSortedArrays(nums1 []int, nums2 []int) float64 {
    // 确保 nums1 是较短的数组
    if len(nums1) > len(nums2) {
        nums1, nums2 = nums2, nums1
    }

    m, n := len(nums1), len(nums2)

    // 构造带哨兵的新数组
    // 使用 int64 的极值来模拟 inf 和 -inf
    // 注意：题目范围值在 -10^6 到 10^6 之间，所以使用 int 极值是安全的
    a := make([]int, 0, m+2)
    a = append(a, math.MinInt) // -inf
    a = append(a, nums1...)
    a = append(a, math.MaxInt) // inf

    b := make([]int, 0, n+2)
    b = append(b, math.MinInt) // -inf
    b = append(b, nums2...)
    b = append(b, math.MaxInt) // inf

    // 二分查找范围
    l, r := 0, m+1

    for l+1 < r {
        i := (l + r) / 2
        // 计算 nums2 对应的分割点 j
        j := (m+n+1)/2 - i

        // 检查左边部分是否小于等于右边部分
        if a[i] <= b[j+1] {
            l = i
        } else {
            r = i
        }
    }

    i := l
    j := (m+n+1)/2 - i

    // 获取左半部分的最大值
    max1 := max(a[i], b[j])
    // 获取右半部分的最小值
    min2 := min(a[i+1], b[j+1])

    // 根据总长度奇偶性返回结果
    if (m+n)%2 != 0 {
        return float64(max1)
    }
    return float64(max1+min2) / 2.0
}

// 辅助函数：求最大值
func max(x, y int) int {
    if x > y {
        return x
    }
    return y
}

// 辅助函数：求最小值
func min(x, y int) int {
    if x < y {
        return x
    }
    return y
}
```
<!-- endtab -->
{% endtabs %}

### 复杂度分析

*   **时间复杂度**: `O(log (min(m, n)))`
    我们只对长度较短的数组进行二分查找，循环次数与短数组长度的对数成正比。
    
*   **空间复杂度**: `O(m + n)`
    为了简化边界判断，本解法创建了包含哨兵的新数组，使用了线性的额外空间。
