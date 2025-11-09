---
title: LeetCode 34 | 二分寻找元素的起始与结束位置
date: 2025-10-18 12:05:00
updated: 2025-10-18 12:05:00
comments: true
tags:
  - 数组
  - 二分查找
categories:
  - 算法题解
  - 二分算法
  - 二分查找
permalink: posts/leetcode-34/
---
{% centerquote %}
LeetCode 第 34 题：[在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)。
本题是二分查找算法的经典应用。标准的二分查找只能告诉我们元素是否存在，而这道题要求我们精确定位目标值连续出现的“左边界”和“右边界”。这需要我们对二分查找的细节进行巧妙的改造。
{% endcenterquote %}
<!--more-->
### 问题解读

题目给定一个按非递减顺序（即升序）排列的整数数组 `nums` 和一个目标值 `target`。我们需要找到 `target` 在数组中出现的起始位置和结束位置。

核心要求：
1.  如果 `target` 存在，返回一个包含起始和结束索引的数组 `[start, end]`。
2.  如果 `target` 不存在，返回 `[-1, -1]`。
3.  算法的时间复杂度必须是 **O(log n)**，这强烈暗示了我们必须使用二分查找。

例如：
*   对于 `nums = [5,7,7,8,8,10]`, `target = 8`，`8` 第一次出现在索引 3，最后一次出现在索引 4，所以返回 `[3,4]`。
*   对于 `nums = [5,7,7,8,8,10]`, `target = 6`，`6` 不存在于数组中，所以返回 `[-1,-1]`。

### 核心思路：寻找边界

直接用一次二分查找找到 `target` 是不够的，因为我们无法确定找到的是第一个、最后一个还是中间的某一个。

一个直观的想法是进行两次二分查找：
1.  一次查找 `target` 的**第一个**出现位置（左边界）。
2.  另一次查找 `target` 的**最后一个**出现位置（右边界）。

一个更优雅且统一的思路是，将问题转化为寻找“**下界（Lower Bound）**”。所谓“下界”，指的是数组中第一个大于或等于目标值的元素的位置。

*   `target` 的**起始位置**，恰好就是 `target` 的下界。
*   `target` 的**结束位置**，可以通过寻找 `target + 1` 的下界来间接得到。`target + 1` 的下界索引减去 1，就是 `target` 的最后一个出现位置。

例如，在 `[5,7,7,8,8,10]` 中：
*   `target = 8` 的下界是索引 3。
*   `target + 1 = 9` 的下界是索引 5（第一个 `>=9` 的元素是 `10`，其索引为 5）。
*   因此，`8` 的结束位置就是 `5 - 1 = 4`。

这样，我们就把问题统一为了实现一个可靠的“下界”二分查找函数。二分查找的实现有多种区间定义方式，常见的有“左闭右闭”、“左闭右开”和“全开”区间，它们在循环条件和边界更新上略有不同，但都能解决问题。下面我们分别探讨这三种写法的实现。

### 算法详解与代码实现

#### 解法一：闭区间写法 `[l, r]`

这是最常见和直观的写法。搜索区间 `[l, r]` 的定义是两端都包含，因此循环条件是 `l <= r`。

*   **初始化**：`l = 0`, `r = len(nums) - 1`。
*   **循环条件**：`while l <= r`，当 `l > r` 时，区间为空，循环结束。
*   **逻辑**：
    *   `nums[mid] >= target`：`mid` 可能是我们寻找的下界，或者下界在 `mid` 的左侧。因此，我们不能排除 `mid`，需要向左收缩搜索范围，令 `r = mid - 1`。
    *   `nums[mid] < target`：`mid` 一定不是下界，下界必然在 `mid` 的右侧。令 `l = mid + 1`。
*   **返回值**：循环结束后，`l` 指向的就是第一个大于或等于 `target` 的位置。

**Python 代码实现**
```python
from typing import List

class Solution:
    # 闭区间 [l, r] 写法
    def lb(self, nums: List[int], target: int) -> int:
        l, r = 0, len(nums) - 1
        while l <= r:
            mid = (l + r) // 2
            if nums[mid] >= target:
                r = mid - 1
            else:
                l = mid + 1
        return l  # 最终 l 就是下界

    def searchRange(self, nums: List[int], target: int) -> List[int]:
        # 寻找 target 的下界作为起始位置
        s = self.lb(nums, target)
        
        # 如果起始位置越界，或者该位置的值不为 target，说明 target 不存在
        if s == len(nums) or nums[s] != target:
            return [-1, -1]
        
        # 寻找 target+1 的下界，其前一个位置就是 target 的结束位置
        e = self.lb(nums, target + 1) - 1
        
        return [s, e]
```

#### 解法二：左闭右开区间写法 `[l, r)`

这种写法的搜索区间 `[l, r)` 左端包含，右端不包含。这在处理边界时非常方便，也是很多语言标准库（如 C++ STL）中 `lower_bound` 的实现方式。

*   **初始化**：`l = 0`, `r = len(nums)`。注意 `r` 的初始值。
*   **循环条件**：`while l < r`，当 `l == r` 时，区间 `[l, l)` 为空，循环结束。
*   **逻辑**：
    *   `nums[mid] >= target`：`mid` 可能是下界，或者下界在 `mid` 左侧。由于右边界 `r` 是开区间，我们可以安全地将 `r` 设置为 `mid`，即新的搜索区间是 `[l, mid)`。
    *   `nums[mid] < target`：`mid` 以及其左侧所有元素都小于 `target`，下界一定在 `mid` 右侧。令 `l = mid + 1`。
*   **返回值**：循环结束后，`l` 和 `r` 相遇，指向的就是下界。

**Python 代码实现**
```python
from typing import List

class Solution:
    # 左闭右开区间 [l, r) 写法
    def lb(self, nums: List[int], target: int) -> int:
        l, r = 0, len(nums)
        while l < r:
            mid = (l + r) // 2
            if nums[mid] >= target:
                r = mid
            else:
                l = mid + 1
        return l # 最终 l (或 r) 就是下界

    def searchRange(self, nums: List[int], target: int) -> List[int]:
        s = self.lb(nums, target)
        if s == len(nums) or nums[s] != target:
            return [-1, -1]
        e = self.lb(nums, target + 1) - 1
        return [s, e]
```

#### 解法三：开区间写法 `(l, r)`

这种写法将 `l` 和 `r` 视为“哨兵”，搜索区间是它们之间的 `(l, r)`。

*   **初始化**：`l = -1`, `r = len(nums)`。
*   **循环条件**：`while l + 1 < r`，确保 `l` 和 `r` 之间至少有一个元素，当它们相邻时循环结束。
*   **逻辑**：
    *   `nums[mid] >= target`：`mid` 可能是下界，或者下界在 `mid` 左侧。我们将右哨兵移动到 `mid`，即 `r = mid`。
    *   `nums[mid] < target`：`mid` 一定不是下界，下界在 `mid` 右侧。我们将左哨兵移动到 `mid`，即 `l = mid`。
*   **返回值**：循环结束后，`l` 和 `r` 相邻，而 `r` 正是我们寻找的下界。

**Python 代码实现**
```python
from typing import List

class Solution:
    # 开区间 (l, r) 写法
    def lb(self, nums: List[int], target: int) -> int:
        l, r = -1, len(nums)
        while l + 1 < r:
            mid = (l + r) // 2
            if nums[mid] >= target:
                r = mid
            else:
                l = mid
        return r # 最终 r 就是下界

    def searchRange(self, nums: List[int], target: int) -> List[int]:
        s = self.lb(nums, target)
        if s == len(nums) or nums[s] != target:
            return [-1, -1]
        e = self.lb(nums, target + 1) - 1
        return [s, e]
```

### 复杂度分析

*   **时间复杂度**: **O(log N)**。我们调用了两次二分查找函数，每次耗时 O(log N)，因此总时间复杂度为 O(2 * log N)，即 O(log N)。
*   **空间复杂度**: **O(1)**。我们只使用了常数级别的额外空间。

### 总结

本题通过将“寻找起始和结束位置”巧妙地转化为两次“寻找下界”的操作，统一并简化了问题。同时，我们展示了解决同一问题的三种不同二分查找区间写法：闭区间 `[l, r]`，左闭右开区间 `[l, r)`，以及开区间 `(l, r)`。

*   **闭区间法**：最传统，容易理解，但边界更新时 `+1`/`-1` 需要格外小心。
*   **左闭右开法**：在处理循环不变量和边界时非常优雅，是现代编程实践中较为推崇的写法。
*   **开区间法**：将 `l` 和 `r` 作为哨兵，逻辑也十分清晰。

