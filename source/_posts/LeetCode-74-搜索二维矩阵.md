---
title: LeetCode 74 | 搜索二维矩阵
date: 2025-12-01 09:32:00
updated: 2025-12-01 09:32:00
comments: true
tags:
  - 二分查找
  - 数组
  - 矩阵
categories:
  - 算法题解
  - 二分算法
  - 其他
permalink: posts/leetcode-74/
---
{% centerquote %}
LeetCode 第 74 题：[搜索二维矩阵](https://leetcode.cn/problems/search-a-2d-matrix/)。
利用矩阵“行首大于前一行行尾”的特性，将其逻辑上展开为一个有序的一维数组，直接使用二分查找即可解决。
{% endcenterquote %}
<!--more-->

### 问题描述

给你一个满足下述两条属性的 `m x n` 整数矩阵：

1.  每行中的整数从左到右按非严格递增顺序排列。
2.  每行的第一个整数大于前一行的最后一个整数。

给你一个整数 `target` ，如果 `target` 在矩阵中，返回 `true` ；否则，返回 `false` 。

**示例 1：**
![示例1](https://assets.leetcode.com/uploads/2020/10/05/mat.jpg)
```text
输入：matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 3
输出：true
```

**示例 2：**
```text
输入：matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 13
输出：false
```

**提示：**
*   m == matrix.length
*   n == matrix[i].length
*   1 <= m, n <= 100
*   -10^4 <= matrix[i][j], target <= 10^4

### 解题思路

#### 1. 矩阵特性分析

题目给出的两个条件非常关键：
*   行内递增。
*   下一行的第一个元素大于上一行的最后一个元素。

这意味着，如果我们把每一行首尾相连拼接起来，这个矩阵在逻辑上就是一个**严格单调递增的一维数组**。

例如矩阵：
```text
[[1, 3, 5, 7],
 [10, 11, 16, 20],
 [23, 30, 34, 60]]
```
展开后变为：
`[1, 3, 5, 7, 10, 11, 16, 20, 23, 30, 34, 60]`

#### 2. 坐标映射

既然可以看作一维数组，我们就可以直接对在这个逻辑一维数组上进行**二分查找**。

假设矩阵有 `m` 行 `n` 列，总元素个数为 `m * n`。
对于范围在 `[0, m * n - 1]` 内的一维数组索引 `idx`，可以通过以下公式映射回二维矩阵的坐标 `(row, col)`：

*   `row = idx / n` (整除)
*   `col = idx % n` (取模)

#### 3. 二分查找实现

我们使用左开右开区间 `(l, r)` 的二分模板：
*   初始左边界 `l = -1`
*   初始右边界 `r = m * n`
*   当 `l + 1 < r` 时循环，计算 `mid`，映射坐标取值并与 `target` 比较。

### 代码实现

{% tabs code_block %}
<!-- tab Python -->
```python
class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        # 获取矩阵的行数 m 和列数 n
        m,n=len(matrix),len(matrix[0])
        
        # 将二维矩阵视为长度为 m*n 的一维有序数组
        # 定义二分查找的区间为左开右开 (-1, m*n)
        l,r = -1,m*n
        
        while l+1<r:
            mid= (l+r)//2
            
            # 核心步骤：坐标映射
            # 将一维索引 mid 转换为二维坐标 [row][col]
            # 行索引 = mid // 列数
            # 列索引 = mid % 列数
            x = matrix[mid//n][mid%n]
            
            if x==target:
                return True
            
            # 标准二分逻辑
            if x<target:
                # 目标值在右侧（较大值方向）
                l=mid
            else:
                # 目标值在左侧（较小值方向）
                r = mid
                
        return False
```
<!-- endtab -->
<!-- tab Go -->
```go
func searchMatrix(matrix [][]int, target int) bool {
    // 获取矩阵维度
    m,n:=len(matrix),len(matrix[0])
    
    // 初始化二分查找的左右边界
    // 使用开区间 (-1, m*n)，对应虚拟一维数组的索引范围
    l,r:=-1,m*n
    
    for l+1<r{
        mid :=l+(r-l)/2
        
        // 坐标转换：将一维索引 mid 映射回二维矩阵坐标
        // 行下标 = mid / 列数
        // 列下标 = mid % 列数
        x:=matrix[mid/n][mid%n]
        
        if x==target{
            return true
        }
        
        if x<target{
            // 当前值小于目标值，说明目标在右半部分，移动左边界
            l = mid
        }else{
            // 当前值大于目标值，说明目标在左半部分，移动右边界
            r = mid
        }
    }
    return false
}
```
<!-- endtab -->
{% endtabs %}

### 复杂度分析

*   **时间复杂度**: `O(log(mn))`
    我们将二维矩阵视作长度为 `mn` 的一维数组进行二分查找，每次比较操作大大缩小搜索范围，时间复杂度为对数级别。
    
*   **空间复杂度**: `O(1)`
    只使用了常数个变量（`l`, `r`, `mid`, `x` 等）来存储索引和中间值，没有使用额外的线性空间。
