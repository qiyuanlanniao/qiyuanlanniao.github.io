---
title: LeetCode 1439 | 有序矩阵中的第 k 个最小数组和
date: 2025-11-27 10:00:00
updated: 2025-11-27 10:00:00
comments: true
tags:
  - 二分查找
  - 数组
  - 矩阵
  - 堆（优先队列）
  - 第187场周赛
categories:
  - 算法题解
  - 二分算法
  - 二分答案
  - 第K小/大
permalink: posts/leetcode-1439/
---
{% centerquote %}
LeetCode 第 1439 题：[有序矩阵中的第 k 个最小数组和](https://leetcode.cn/problems/find-the-kth-smallest-sum-of-a-matrix-with-sorted-rows/)。
由于矩阵行有序，我们可以利用 **BS** 来确定答案，结合 **DFS** 进行验证。
{% endcenterquote %}
<!--more-->

### 问题描述

给你一个 `m * n` 的矩阵 `mat`，以及一个整数 `k`，矩阵中的每一行都以非递减的顺序排列。

你可以从每一行中选出 1 个元素形成一个数组。返回所有可能数组中的第 `k` 个 **最小** 数组和。

**示例 1：**
```text
输入：mat = [[1,3,11],[2,4,6]], k = 5
输出：7
解释：从每一行中选出一个元素，前 k 个和最小的数组分别是：
[1,2], [1,4], [3,2], [3,4], [1,6]。其中第 5 个的和是 7 。
```

**提示：**
*   `m == mat.length`
*   `n == mat.length[i]`
*   `1 <= m, n <= 40`
*   `1 <= k <= min(200, n ^ m)`
*   `1 <= mat[i][j] <= 5000`
*   `mat[i]` 是一个非递减数组

### 解题思路

#### 1. 为什么选择二分答案？

我们需要找到所有可能的数组和中第 `k` 小的那个和。
如果我们把所有可能的数组和从小到大排列，设第 `k` 小的和为 `target`，那么我们可以发现一个规律：
*   对于任意小于 `target` 的数值 `x`，能组成的数组和小于等于 `x` 的数量一定小于 `k`。
*   对于任意大于等于 `target` 的数值 `x`，能组成的数组和小于等于 `x` 的数量一定大于等于 `k`。

这满足**单调性**，因此我们可以对“数组和”这个值进行二分查找。

#### 2. 确定二分范围

*   **下界 (Left)**：每行取最小的元素（即第一列），求和。这是所有组合中最小的和 `sl`。
*   **上界 (Right)**：每行取最大的元素（即最后一列），求和。这是所有组合中最大的和 `sr`。

答案一定在 `[sl, sr]` 之间。

#### 3. Check 函数的设计 (DFS)

二分的核心在于 `check(s)` 函数：判断是否存在至少 `k` 个数组组合，其和小于等于 `s`。

如果我们要枚举所有组合，复杂度是指数级的，无法接受。但我们只需要知道 **数量是否达到 k**。
我们可以使用 DFS 回溯来统计：
1.  从最后一行开始往前递归（或从第一行开始，效果一样）。
2.  为了加速，我们不传递当前的累加和，而是传递 **剩余的额度**。
    *   初始额度 = `s` - `sl` (当前二分值减去最小基础和)。
    *   这相当于我们在计算除了每行选第一个数（基准）之外，额外增加的增量能否控制在限制内。
3.  **剪枝策略**：
    *   如果在某一行，选择某个元素后，增量已经超过了剩余额度，由于行是有序的，后面的元素更大，肯定也不满足，直接 `break`。
    *   **数量限制**：维护一个全局计数器（或引用变量）`lk`，初始为 `k`。每找到一个合法组合，`lk` 减 1。当 `lk` 降为 0 时，说明我们已经找到了 `k` 个满足条件的组合，直接返回 `True`。

### 代码实现

{% tabs code_block %}
<!-- tab Python -->
```python
class Solution:
    def kthSmallest(self, mat: List[List[int]], k: int) -> int:
        # check 函数：判断小于等于 s 的数组和的数量是否 >= k
        def check(s: int) -> bool:
            # lk 用于计数，还需要找到多少个满足条件的组合
            lk = k
            
            # DFS 搜索
            # i: 当前处理的行索引
            # s: 当前剩余的可支配和（相对于每行最小值的增量）
            def dfs(i: int, s: int) -> bool:
                # Base case: 成功遍历完所有行（i < 0），说明找到一个合法组合
                if i < 0:
                    nonlocal lk
                    lk -= 1
                    return lk == 0 # 如果找到了 k 个，返回 True
                
                # 遍历当前行的元素
                for x in mat[i]:
                    # 剪枝：如果当前元素带来的增量 (x - mat[i][0]) 超过了剩余预算 s
                    # 由于 mat[i] 是有序的，后面的元素肯定也超，直接 break
                    if x - mat[i][0] > s:
                        break
                    
                    # 递归处理上一行
                    # 这里的 s - (x - mat[i][0]) 是减去当前元素相对于该行最小值的增量
                    if dfs(i - 1, s - (x - mat[i][0])):
                        return True # 如果子递归已经找到了足够的数量，向上返回 True
                
                return False

            # 从最后一行开始 DFS，初始预算是 s - sl
            # s 是二分的猜测值，sl 是所有行最小值的和
            return dfs(len(mat) - 1, s - sl)

        # 计算可能的最小和 sl 和最大和 sr
        sl = sum(row[0] for row in mat)
        sr = sum(row[-1] for row in mat)
        
        # 二分查找
        # range(sl, sr) 生成从 sl 到 sr-1 的序列
        # bisect_left 会寻找第一个使得 check(val) 为 True 的位置
        # 结果需要加上 sl 偏移量
        # 注意：如果 range 中所有值都 check 失败，bisect_left 返回数组长度，即 sr - sl
        # 此时结果为 sl + (sr - sl) = sr，逻辑是自洽的
        return sl + bisect_left(range(sl, sr), True, key=check)
```
<!-- endtab -->
<!-- tab Go -->
```go
import (
    "sort"
)

func kthSmallest(mat [][]int, k int) int {
    sl, sr := 0, 0
    // 计算理论最小和 sl 和最大和 sr
    for _, row := range mat {
        sl += row[0]
        sr += row[len(row)-1]
    }

    // 二分查找
    // sort.Search(n, f) 在 [0, n) 范围内查找使得 f(i) 为 true 的最小 i
    // 这里的范围长度是 sr - sl，代表从最小和开始的增量范围
    return sl + sort.Search(sr-sl, func(s int) bool {
        // s 代表当前二分尝试的“额外增量”值
        // 实际尝试的数组和 target = sl + s
        
        lk := k // 剩余需要寻找的组合数量
        
        var dfs func(int, int) bool
        dfs = func(i, s int) bool {
            // Base case: 所有行都选择完毕
            if i < 0 {
                lk--
                return lk == 0 // 如果找到了 k 个，返回 true
            }
            
            // 遍历第 i 行的元素
            for _, x := range mat[i] {
                // 剪枝：如果当前元素相对于该行最小值的增量大于剩余预算 s
                // 则该行后续元素也一定不满足，直接 break
                if x-mat[i][0] > s {
                    break
                }
                
                // 递归进入上一行 (i-1)
                // 预算减去当前选择带来的增量
                if dfs(i-1, s-(x-mat[i][0])) {
                    return true
                }
            }
            return false
        }
        
        // 从最后一行开始，初始预算就是二分传入的 s
        return dfs(len(mat)-1, s)
    })
}
```
<!-- endtab -->
{% endtabs %}

### 复杂度分析

*   **时间复杂度**: `O(m * k * log(Range))`
    *   二分查找的范围是 `Range = sr - sl`，大约为 `m * 5000`，二分次数为 `log(Range)`。
    *   在每次 `check` 中，我们使用 DFS 寻找不超过 `k` 个合法组合。虽然最坏情况下 DFS 复杂度较高，但由于我们限制了找到 `k` 个就停止，且利用了排序进行剪枝，实际运行效率很高。这里的有效搜索节点数与 `k` 和 `m` 相关。
*   **空间复杂度**: `O(m)`
    *   主要消耗在于 DFS 的递归调用栈，深度为 `m`。
