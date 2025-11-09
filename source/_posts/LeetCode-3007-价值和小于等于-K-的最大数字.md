---
title: LeetCode 3007 | 价值和小于等于 K 的最大数字
date: 2025-11-05 20:20:00
updated: 2025-11-05 20:20:00
comments: true
tags:
  - 位运算
  - 数学
  - 二分查找
  - 动态规划
  - 第380场周赛
categories:
  - 算法题解
  - 二分算法
  - 二分答案
  - 求最大
permalink: posts/leetcode-3007/
---
{% centerquote %}
LeetCode 第 3007 题：[价值和小于等于 K 的最大数字](https://leetcode.cn/problems/maximum-number-that-sum-of-the-prices-is-less-than-or-equal-to-k/description/)。
本题的核心在于识别出“累加价值”函数的单调性，从而将一个求解最大值的问题，转化为一个判定性问题，并利用二分查找高效求解。其中，判定函数的设计是本题的精髓，需要运用到位运算的规律。
{% endcenterquote %}
<!--more-->

### 问题解读

我们需要理解题目定义的两个核心概念：“价值”和“累加价值”。

1.  **整数的“价值”**: 对于一个数字 `num`，它的价值是其二进制表示中，所有位置索引是 `x` 的倍数（如 `x`, `2x`, `3x`, ...）的位上，值为 1 的数量总和。注意，位置是从 1 开始计数的最低有效位。

2.  **数字的“累加价值”**: 数字 `num` 的累加价值，是从 1 到 `num` 所有整数的“价值”的总和。

我们的目标是找到一个最大的数字 `num`，使得它的“累加价值”小于或等于给定的 `k`。

举个例子，`k = 7, x = 2`。
我们要找最大的 `num`，使得 `value(1) + value(2) + ... + value(num) <= 7`。
根据题目表格，`num=9` 时，累加价值是 6；`num=10` 时，累加价值是 8。所以，满足条件的最大数字是 9。

观察可以发现，“累加价值”会随着 `num` 的增大而增大（或保持不变）。这是一个**单调递增**的函数。这个特性是解决本题的关键信号。

### 核心思路：将“求解问题”转化为“判定问题”与二分查找

直接求解满足条件的“最大数字”很困难。但如果我们换一个角度思考：给定一个数字 `guess`，我们能否判断它的累加价值是否小于等于 `k`？

这个问题（我们称之为“判定问题”）如果能够被高效解决，我们就可以利用其单调性进行二分查找。
*   如果 `guess` 的累加价值小于等于 `k`，说明 `guess` 是一个“廉价”的数字，那么真正的答案可能就是 `guess` 或者比它更大的数。
*   如果 `guess` 的累加价值大于 `k`，说明 `guess` 太大了，真正的答案一定比它小。

这完美地契合了二分查找的框架。我们可以在一个很大的范围内（例如 `[1, 2*10^15]`）对答案进行二分搜索。

现在的核心挑战转移到了如何高效地实现这个判定函数：`count_accumulated_value(num)`。

### 算法详解：如何计算累加价值？

`count_accumulated_value(num)` 是计算从 1 到 `num` 所有整数的总价值。
`总价值 = Σ(value(i) for i in 1..num)`

根据 `value` 的定义，我们可以进一步展开：
`总价值 = Σ(Σ(bit_at(i, p*x)) for i in 1..num for p in 1,2,...)`

这里 `bit_at(i, j)` 表示数字 `i` 的二进制表示中第 `j` 位是否为 1。我们可以交换求和的顺序，这使得问题变得清晰：
`总价值 = Σ(Σ(bit_at(i, p*x)) for p in 1,2,... for i in 1..num)`

这个公式的内层 `Σ(bit_at(i, p*x) for i in 1..num)` 实际上是在问：**“对于一个固定的、需要计价的位置 `pos = p*x`，在从 1 到 `num` 的所有数字中，有多少个数字的二进制表示在第 `pos` 位上是 1？”**

我们可以设计一个函数 `countSetBitsAtPos(num, pos)` 来解决这个问题。
观察二进制数的规律可以发现，任何一个二进制位 `pos`（其代表的值为 `2^(pos-1)`）上的数字 `0` 和 `1` 都是周期性出现的。
*   第 `pos` 位的 `0` 和 `1` 交替出现，一个完整的周期（从 `0...0` 到 `1...1`）长度为 `2^pos`。
*   在这个周期中，前一半（`2^(pos-1)` 个数）在该位上是 `0`，后一半（`2^(pos-1)` 个数）在该位上是 `1`。

基于这个规律，我们可以计算出在 `1` 到 `num` 的范围内，有多少个数在第 `pos` 位是 1：
1.  令周期长度 `cycle_len = 2^pos`，半周期长度 `half_cycle = 2^(pos-1)`。
2.  为了方便计算，我们考虑 `[0, num]` 这个区间，共 `num + 1` 个数。
3.  完整周期的数量为 `num_cycles = (num + 1) // cycle_len`。
4.  每个完整周期都贡献了 `half_cycle` 个 1，所以这部分的贡献是 `num_cycles * half_cycle`。
5.  剩余部分的长度为 `remainder = (num + 1) % cycle_len`。
6.  这部分剩余的数从一个新周期的开头算起，开头是 `half_cycle` 个 0，所以这部分贡献的 1 的数量是 `max(0, remainder - half_cycle)`。
7.  将两部分贡献相加，就得到了 `countSetBitsAtPos(num, pos)` 的结果。

有了这个函数，我们就可以实现 `count_accumulated_value(num)`：我们遍历所有需要计价的位置 `pos = x, 2x, 3x, ...`（上限到 64 位足够），对每个 `pos` 调用 `countSetBitsAtPos(num, pos)` 并将结果累加即可。

### Python 代码实现

```python
class Solution:
    def findMaximumNumber(self, k: int, x: int) -> int:
        
        def count_accumulated_value(num: int) -> int:
            """
            高效计算从 1 到 num 的所有数字的累加价值。
            """
            total_value = 0
            # 遍历所有需要计价的二进制位：x, 2x, 3x, ...
            # 由于 k <= 10^15，数字不会超过 60 位，这里取到 64 作为安全上界。
            for pos in range(x, 64, x):
                # 计算在 1 到 num 中，第 pos 位为 1 的数字有多少个。
                
                # 第 pos 位的 0 和 1 的完整周期长度是 2^pos。
                cycle_len = 1 << pos
                # 每个完整周期中，有一半的数字在该位上是 1。
                half_cycle_len = cycle_len // 2
                
                # 为了计算方便，我们分析区间 [0, num]，共 num + 1 个数。
                num_plus_one = num + 1
                
                # 计算完整周期的数量和它们贡献的 set bits。
                num_cycles = num_plus_one // cycle_len
                set_bits = num_cycles * half_cycle_len
                
                # 计算剩余不足一个周期的部分贡献的 set bits。
                remainder = num_plus_one % cycle_len
                set_bits += max(0, remainder - half_cycle_len)
                
                total_value += set_bits
            return total_value

        # 对答案进行二分查找
        left, right = 1, 2 * 10**15  # 设置一个足够大的搜索上界
        ans = 0
        
        while left <= right:
            mid = (left + right) // 2
            if mid == 0:  # 避免 mid 成为 0
                break
            
            # 调用判定函数
            if count_accumulated_value(mid) <= k:
                # mid 是一个可行的答案，我们尝试寻找更大的解
                ans = mid
                left = mid + 1
            else:
                # mid 太大了，需要缩小搜索范围
                right = mid - 1
                
        return ans
```

### 复杂度分析

*   **时间复杂度**: O(log(K) * (64/x))。
    *   二分查找的搜索空间大小约为 `2*k`，因此迭代次数为 O(log K)。
    *   在每次迭代中，`count_accumulated_value` 函数需要循环 `64/x` 次来计算所有相关位的贡献。
    *   由于 `x` 是一个不大的常数（1 到 8），因此整体复杂度可以认为是 O(log K)。
*   **空间复杂度**: O(1)。
    *   算法只使用了有限的几个变量，没有使用额外的与输入规模相关的存储空间。

### 总结

本题是“二分答案”思想与“位运算”技巧结合的典范。解决这类问题的关键路径通常是：
1.  分析问题的目标函数（本题中的“累加价值”），判断其是否具有单调性。
2.  如果存在单调性，就可以将“求解最优值”问题转化为“判定一个值是否可行”的问题。
3.  设计一个高效的判定函数。在本题中，这涉及到深入理解二进制数的周期性规律，并通过数学方法快速统计特定位上 1 的数量。
4.  最后，在答案的可能范围内应用二分查找，通过判定函数不断缩小搜索区间，最终找到临界点，即为所求的最优解。
