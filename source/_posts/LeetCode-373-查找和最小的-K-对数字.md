---
title: LeetCode 373 | 查找和最小的 K 对数字
date: 2025-11-26 09:40:00
updated: 2025-11-26 09:40:00
comments: true
tags:
  - 堆（优先队列）
  - 数组
categories:
  - 算法题解
  - 二分算法
  - 二分答案
  - 第K小/大
permalink: posts/leetcode-373/
---
{% centerquote %}
LeetCode 第 373 题：[查找和最小的 K 对数字](https://leetcode.cn/problems/find-k-pairs-with-smallest-sums/)。
这是一道经典的“多路归并”问题，本质上是在一个隐式的二维矩阵中寻找第 K 小的元素。利用 **最小堆（优先队列）** 可以高效解决。
{% endcenterquote %}
<!--more-->

### 问题描述

给定两个以 **非递减顺序排列** 的整数数组 `nums1` 和 `nums2` , 以及一个整数 `k` 。

定义一对值 `(u,v)`，其中第一个元素来自 `nums1`，第二个元素来自 `nums2` 。

请找到和最小的 `k` 个数对 `(u1,v1), (u2,v2) ... (uk,vk)` 。

**示例 1:**
```text
输入: nums1 = [1,7,11], nums2 = [2,4,6], k = 3
输出: [1,2],[1,4],[1,6]
解释: 返回序列中的前 3 对数：
     [1,2],[1,4],[1,6],[7,2],[7,4],[11,2],[7,6],[11,4],[11,6]
```

**示例 2:**
```text
输入: nums1 = [1,1,2], nums2 = [1,2,3], k = 2
输出: [1,1],[1,1]
解释: 返回序列中的前 2 对数：
     [1,1],[1,1],[1,2],[2,1],[1,2],[2,2],[1,3],[1,3],[2,3]
```

**提示:**
*   1 <= nums1.length, nums2.length <= 10^5
*   -10^9 <= nums1[i], nums2[i] <= 10^9
*   nums1 和 nums2 均为 升序排列
*   1 <= k <= 10^4
*   k <= nums1.length * nums2.length

### 解题思路

#### 1. 为什么暴力法不可行？

如果直接生成所有可能的数对 `(nums1[i], nums2[j])`，总共有 `m * n` 对（其中 `m` 和 `n` 分别是数组的长度）。题目中数组长度可达 10^5，乘积可能达到 10^10，内存和时间都无法承受。
然而，我们需要找的只是 **前 k 个** 最小的对，且 `k` 的范围相对较小（最大 10^4），这提示我们应该使用与 `k` 相关的复杂度算法。

#### 2. 利用有序性与堆

由于 `nums1` 和 `nums2` 都是升序排列的，我们可以将所有可能的数对想象成一个矩阵，其中第 `i` 行第 `j` 列的元素是 `nums1[i] + nums2[j]`。
在这个矩阵中：
*   每一行从左到右是递增的。
*   每一列从上到下是递增的。

最小的元素一定是 `(nums1[0], nums2[0])`。
当我们选择了 `(nums1[i], nums2[j])` 后，下一个可能的最小元素只能是它的右边邻居 `(nums1[i], nums2[j+1])` 或者下边邻居 `(nums1[i+1], nums2[j])`。

#### 3. 优化入堆策略

为了避免重复访问和控制堆的大小，我们可以采取一种特定的扩展策略：
1.  初始时，只将 `(nums1[0] + nums2[0], 0, 0)` 放入最小堆。元组中记录的是 `(数值和, nums1下标, nums2下标)`。
2.  每次从堆中取出最小元素 `(sum, i, j)`，将其加入答案。
3.  接下来需要将该元素的潜在“后继”加入堆中。为了保证不重不漏：
    *   **向右扩展**：总是将 `(i, j+1)` 加入堆（如果 `j+1` 未越界）。
    *   **向下扩展**：只有当 `j == 0` 时，才将 `(i+1, 0)` 加入堆（如果 `i+1` 未越界）。

这种策略相当于把 `nums1` 看作行的起始，我们可以在行内自由向右走；但换行操作只允许在每一行的开头进行。这样可以保证每一对组合只被加入堆一次。

### 代码实现

{% tabs code_block %}
<!-- tab Python -->
```python
class Solution:
    def kSmallestPairs(self, nums1: List[int], nums2: List[int], k: int) -> List[List[int]]:
        # 初始只放入 (0,0) 位置的元素
        h = [(nums1[0]+nums2[0],0,0)]
        ans = []
        for _ in range(k):
            # 取出当前堆中最小的元素
            _,i,j = heappop(h)
            ans.append([nums1[i],nums2[j]])
            
            # 策略：如果当前是该行的第一个元素 (j==0)，则可以将下一行的第一个元素入堆
            # 这样保证了第一列的元素会被依次加入
            if j == 0 and i+1<len(nums1):
                heappush(h,(nums1[i+1]+nums2[0],i+1,0))
            
            # 策略：总是将当前元素的右侧元素入堆
            if j+1<len(nums2):
                heappush(h,(nums1[i]+nums2[j+1],i,j+1))
        return ans
```
<!-- endtab -->
<!-- tab Go -->
```go
func kSmallestPairs(nums1, nums2 []int, k int) [][]int {
    n, m := len(nums1), len(nums2)
    // 初始化最小堆，放入 (0,0)
    h := hp{{nums1[0] + nums2[0], 0, 0}}

    ans := make([][]int, 0, k) // 预分配空间
    for range k {
        // 弹出堆顶最小元素
        p := heap.Pop(&h).(tuple)
        i, j := p.i, p.j
        ans = append(ans, []int{nums1[i], nums2[j]})
        
        // 只有当我们在当前行的第一个位置时，才去添加下一行的起始元素
        // 这相当于负责了 "向下" 的搜索方向
        if j == 0 && i+1 < n {
            heap.Push(&h, tuple{nums1[i+1] + nums2[0], i + 1, 0})
        }
        
        // 总是尝试添加当前位置右边的元素
        // 这相当于负责了 "向右" 的搜索方向
        if j+1 < m {
            heap.Push(&h, tuple{nums1[i] + nums2[j+1], i, j + 1})
        }
    }
    return ans
}

// 定义堆所需的结构体和接口方法
type tuple struct{ s, i, j int }
type hp []tuple
func (h hp) Len() int           { return len(h) }
func (h hp) Less(i, j int) bool { return h[i].s < h[j].s }
func (h hp) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *hp) Push(v any)        { *h = append(*h, v.(tuple)) }
func (h *hp) Pop() any          { a := *h; v := a[len(a)-1]; *h = a[:len(a)-1]; return v }
```
<!-- endtab -->
{% endtabs %}

### 复杂度分析

*   **时间复杂度**: `O(k log k)`
    我们需要循环 `k` 次来找到前 `k` 个最小对。在每次迭代中，我们执行一次 `heappop` 和最多两次 `heappush`。堆中的元素数量最多不会超过 `k`（实际上更接近于 `k` 或 `nums1` 的长度，取决于扩展策略，但在最坏情况下与 `k` 同阶）。因此，每次堆操作的时间复杂度为 `O(log k)`。总时间复杂度为 `O(k log k)`。

*   **空间复杂度**: `O(k)`
    堆中最多存储 `k` 个元素（或者更准确地说是生成的候选对数量，这与 `k` 成正比）。用于存储答案的数组也需要 `O(k)` 的空间。
