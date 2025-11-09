---
title: LeetCode 275 | H 指数 II
date: 2025-10-27 11:30:00
updated: 2025-10-27 11:30:00
comments: true
tags:
  - 数组
  - 二分查找
categories:
  - 算法题解
  - 二分算法
  - 二分答案
  - 求最大
permalink: posts/leetcode-275/
---
{% centerquote %}
LeetCode 第 275 题：[H 指数 II](https://leetcode.cn/problems/h-index-ii/description/)。
本文将深入探讨解决此问题的四种不同二分查找区间写法。
{% endcenterquote %}
<!--more-->

### 问题解读

我们需要计算一位研究者的 h 指数。h 指数的定义是：一名科研人员的 n 篇论文中，有**至少** `h` 篇论文分别被引用了**至少** `h` 次。

题目给出了一个整数数组 `citations`，其中 `citations[i]` 是第 `i` 篇论文的被引次数，并且该数组已经**按非降序（升序）排列**。我们的目标是设计一个时间复杂度为对数级别的算法来找出这个 h 指数。

例如，对于 `citations = [0, 1, 3, 5, 6]`：
- 我们有 5 篇论文。
- 检查 h=3：是否有至少 3 篇论文的引用次数都 >= 3？
  - 引用次数最高的 3 篇论文是 `[3, 5, 6]`。
  - 这三篇论文的引用次数都满足 >= 3 的条件。所以 h=3 是一个可能的答案。
- 检查 h=4：是否有至少 4 篇论文的引用次数都 >= 4？
  - 引用次数最高的 4 篇论文是 `[1, 3, 5, 6]`。
  - 其中 `1 < 4` 且 `3 < 4`，不满足条件。
- 因此，最大的满足条件的 h 值是 3，所以 h 指数就是 3。

### 核心思路：在“答案”上进行二分查找

直接寻找这个 h 指数似乎需要逐个尝试。但我们可以观察到答案 `h` 具有**单调性**：
- 如果一个值 `h` 满足 h 指数的定义，那么所有小于 `h` 的值（例如 `h-1`）也一定满足。
- 如果一个值 `h` 不满足 h 指数的定义，那么所有大于 `h` 的值（例如 `h+1`）也一定不满足。

这种单调性是应用二分查找的完美信号。我们可以不直接在 `citations` 数组的索引上进行二分，而是在**可能的答案 `h`** 上进行二分。`h` 的取值范围是 `[0, n]`，其中 `n` 是论文总数。

我们的目标就变成了：在 `[0, n]` 这个答案空间中，找到满足条件的最大的 `h`。

为了实现二分查找，我们需要一个**判定函数** `check(h)`，来判断任意一个猜测的 `h` 值是否满足条件。
- **如何判定 `h` 是否可行？**
  - 根据定义，我们需要检查是否存在 `h` 篇论文，其引用次数都大于或等于 `h`。
  - 由于 `citations` 数组是升序的，引用次数最高的 `h` 篇论文必然是数组末尾的 `h` 个元素。
  - 我们只需要检查这 `h` 篇论文中被引次数最少的那一篇，即 `citations[n-h]`，是否满足 `>= h`。
  - 如果 `citations[n-h] >= h`，那么它后面的所有论文引用次数也都大于等于 `h`，总共就有 `h` 篇满足条件。
  - 因此，判定条件可以简化为 `citations[n-h] >= mid` (其中 `mid` 是我们猜测的 `h` 值)。

有了这个 O(1) 的判定函数，我们就可以在 `[0, n]` 的答案空间内高效地进行二分查找了。

### 四种二分算法的 Python 代码实现

下面将展示四种经典的二分查找区间写法，它们在边界处理和循环条件上略有不同，但最终都能得到正确答案。

#### 解法一：闭区间 `[left, right]`

这是最常见的一种写法，搜索区间两端都包含在内。

```python
from typing import List

class Solution:
    def hIndex(self, citations: List[int]) -> int:
        n = len(citations)
        # 在区间 [left, right] 内询问
        left = 0
        right = n
        while left <= right:  # 区间不为空
            # 循环不变量：
            # left-1 的回答一定为「是」（即 h=left-1 满足条件）
            # right+1 的回答一定为「否」（即 h=right+1 不满足条件）
            mid = (left + right) // 2
            if mid == 0: # h=0 恒成立，特殊处理或调整 left 初始值
                left = mid + 1
                continue
            
            # 检查 h=mid 是否可行：
            # 引用次数最多的 mid 篇论文，其引用次数均需 >= mid
            # 相当于检查第 n-mid 篇论文（引用最少的）是否 citations[n-mid] >= mid
            if citations[n - mid] >= mid:
                # mid 可行，尝试更大的 h
                left = mid + 1  # 询问范围缩小到 [mid+1, right]
            else:
                # mid 不可行，需要减小 h
                right = mid - 1  # 询问范围缩小到 [left, mid-1]
        
        # 循环结束后 right 等于 left-1
        # 根据循环不变量，right 现在是最大的回答为「是」的数
        return right
```**逻辑解读**：当 `check(mid)` 成功时，意味着 `mid` 是一个潜在的答案，但我们想找最大的 `h`，所以我们向右搜索 `[mid+1, right]`；反之，`mid` 太大了，向左搜索 `[left, mid-1]`。循环结束时 `left = right + 1`，`right` 指向的是最后一个满足条件的 `h`。

---

#### 解法二：左闭右开区间 `[left, right)`

这种写法中，`right` 边界是“开”的，即不包含在搜索区间内。`right` 通常被看作是第一个“不满足”条件的位置。

```python
class Solution:
    def hIndex(self, citations: List[int]) -> int:
        n = len(citations)
        # 在区间 [left, right) 内询问
        left = 0
        right = n + 1
        while left < right:  # 区间不为空
            # 循环不变量：
            # left-1 的回答一定为「是」
            # right 的回答一定为「否」
            mid = (left + right) // 2
            if mid == 0: # h=0 恒成立
                left = mid + 1
                continue

            # 引用次数最多的 mid 篇论文，引用次数均 >= mid
            if citations[n - mid] >= mid:
                left = mid + 1  # 询问范围缩小到 [mid+1, right)
            else:
                right = mid  # 询问范围缩小到 [left, mid)
        
        # 循环结束时 left == right，它指向第一个回答为「否」的数
        # 根据循环不变量，left-1 现在是最大的回答为「是」的数
        return left - 1
```
**逻辑解读**：当 `check(mid)` 成功时，我们更新 `left = mid + 1`，继续向右探索。当 `check(mid)` 失败时，`mid` 本身就是一个“不满足”的值，所以我们更新 `right = mid`，将这个“第一个不满足”的边界向左移动。循环结束时 `left` 就是那个第一个不满足条件的 `h`，因此 `left - 1` 就是答案。

---

#### 解法三：左开右闭区间 `(left, right]`

与上一种相反，这种写法的 `left` 边界是“开”的。

```python
class Solution:
    def hIndex(self, citations: List[int]) -> int:
        n = len(citations)
        # 在区间 (left, right] 内询问
        left = 0
        right = n
        while left < right:  # 区间不为空
            # 循环不变量：
            # left 的回答一定为「是」
            # right+1 的回答一定为「否」
            mid = (left + right + 1) // 2  # 向上取整，避免死循环
            
            # 引用次数最多的 mid 篇论文，引用次数均 >= mid
            if citations[n - mid] >= mid:
                left = mid  # 询问范围缩小到 (mid, right]
            else:
                right = mid - 1  # 询问范围缩小到 (left, mid-1]
        
        # 循环结束时 left == right
        # 根据循环不变量，left 现在是最大的回答为「是」的数
        return left
```
**逻辑解读**：这里 `mid` 的计算方式是 `(left + right + 1) // 2` 向上取整，这是为了防止当 `left = right - 1` 且 `check` 成功时，`mid` 仍等于 `left` 导致无限循环。当 `check(mid)` 成功时，`mid` 成为新的“最后一个满足条件”的候选，所以 `left = mid`；反之，则收缩右边界 `right = mid - 1`。循环结束时 `left` 即为答案。

---

#### 解法四：开区间 `(left, right)`

这种写法保证 `left` 和 `right` 之间始终有空间，循环条件是 `left + 1 < right`。

```python
class Solution:
    def hIndex(self, citations: List[int]) -> int:
        n = len(citations)
        # 在区间 (left, right) 内询问
        left = 0
        right = n + 1
        while left + 1 < right:  # 区间不为空
            # 循环不变量：
            # left 的回答一定为「是」
            # right 的回答一定为「否」
            mid = (left + right) // 2
            
            # 引用次数最多的 mid 篇论文，引用次数均 >= mid
            if citations[n - mid] >= mid:
                left = mid  # 询问范围缩小到 (mid, right)
            else:
                right = mid  # 询问范围缩小到 (left, mid)
        
        # 循环结束后 left + 1 == right
        # 根据循环不变量，left 现在是最大的回答为「是」的数
        return left
```
**逻辑解读**：`left` 始终维护“已知的满足条件的最大值”，而 `right` 始终维护“已知的第一个不满足条件的值”。循环不断地用 `mid` 来更新这两个边界，直到它们相邻。最终的答案就是 `left`。

### 复杂度分析

*   **时间复杂度**: O(log N)
    *   `N` 是论文的总数 `len(citations)`。
    *   二分查找在大小为 `N+1` 的答案空间 `[0, N]` 中进行。
    *   每次迭代，判定函数 `check(mid)` 的时间复杂度为 O(1)，因为它只需要访问数组中的一个元素。
    *   因此，总的时间复杂度为 O(log N)。
*   **空间复杂度**: O(1)
    *   我们只使用了有限的几个变量（`left`, `right`, `mid`, `n`），没有使用与输入大小成比例的额外空间。

### 总结

H 指数 II 是一个典型的“二分答案”问题。当问题的解具有单调性，并且我们可以高效地对任意一个猜测的解进行“判定”时，就可以考虑使用二分查找。本文通过四种不同的区间写法，展示了二分查找在实现细节上的灵活性。理解每种写法的循环不变量是掌握二分查找、避免边界错误的关键。
