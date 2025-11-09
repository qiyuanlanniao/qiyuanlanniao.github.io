---
title: LeetCode 2156 | 滚动哈希巧解给定哈希值的子串
date: 2025-09-20 12:49:54
updated: 2025-09-20 12:49:54
comments: true
tags:
  - 字符串
  - 滑动窗口
  - 哈希函数
  - 滚动哈希
  - 第 278 场周赛
categories:
  - 算法题解
  - 滑动窗口与双指针
  - 定长滑动窗口
permalink: posts/leetcode-2156/
---
{% centerquote %}
LeetCode 第 2156 题：[查找给定哈希值的子串](https://leetcode.cn/problems/find-substring-with-given-hash-value/description/)。
这道题将我们带入了“滚动哈希”（Rolling Hash）的精妙世界。它不仅仅是一个简单的滑动窗口，更是一次关于如何在模运算下高效更新哈希值的深刻实践。
{% endcenterquote %}
<!--more-->
### 问题解读

题目要求我们根据一个自定义的哈希函数，在一个长字符串 `s` 中找到**第一个**长度为 `k` 且哈希值等于 `hashValue` 的子串。

这个哈希函数定义如下：
`hash(sub, p, m) = (val(sub[0])*p^0 + val(sub[1])*p^1 + ... + val(sub[k-1])*p^(k-1)) mod m`

其中 `val(c)` 是字符 `c` 的字母表顺序值（`'a'`=1, `'b'`=2, ...）。

*   **关键点**：我们需要在一个大字符串中，对所有长度为 `k` 的子串计算哈希值。
*   **目标**：找到满足 `hash(sub) == hashValue` 的**最靠前**的那个子串。

例如，对于 `s = "fbxzaad"`, `p = 31`, `m = 100`, `k = 3`, `hashValue = 32`，子串 `"fbx"` 和 `"bxz"` 的哈希值都是 32。但因为 `"fbx"` 在字符串中出现得更早，所以它是正确答案。

### 核心思路：逆向的滚动哈希

#### 暴力法的瓶颈

最直观的方法是遍历 `s` 中每一个长度为 `k` 的子串，然后对每个子串都独立计算一次哈希值。这种方法的时间复杂度是 O(N*K)，其中 N 是 `s` 的长度。对于题目给定的数据范围 (`N` 可达 2*10^4)，这种方法会“超出时间限制”。

#### 从左向右滑动的困境

为了优化，我们自然会想到滑动窗口：当窗口从 `s[i:i+k]` 滑动到 `s[i+1:i+k+1]` 时，能否在 O(1) 的时间内更新哈希值？

*   **移出**：我们需要减去 `val(s[i]) * p^0` 的贡献。
*   **移入**：我们需要加入 `val(s[i+k]) * p^k`... 等等，这里就出现了问题。

为了让原有的 `val(s[i+1])*p^1` 变为 `val(s[i+1])*p^0`，我们需要将整个哈希值（减去 `val(s[i])` 之后）除以 `power`。在模运算中，除以一个数等于乘以它的**模逆元**。计算模逆元本身就很复杂，而且当 `power` 和 `modulo` 不互质时，模逆元甚至不存在。这条路走不通。

#### 柳暗花明：从右向左滑动

既然从左向右有困难，我们何不逆向思考？**从右向左滑动窗口**！

假设我们已知窗口 `s[i+1 : i+k+1]` 的哈希值 `h_old`，现在要计算它左边的窗口 `s[i : i+k]` 的哈希值 `h_new`。

*   `h_old = (val(s[i+1])*p^0 + ... + val(s[i+k])*p^(k-1)) mod m`

观察 `h_new` 的构成：
*   `h_new = (val(s[i])*p^0 + val(s[i+1])*p^1 + ... + val(s[i+k-1])*p^(k-1)) mod m`

我们可以找到一个递推关系：
1.  从 `h_old` 中减去最右侧字符 `s[i+k]` 的贡献，即 `val(s[i+k]) * p^(k-1)`。
2.  将剩余的部分整体乘以 `p`，这使得每一项的幂次都增加了 1 (`p^0` -> `p^1`, `p^1` -> `p^2`...)。
3.  最后，加上新进入窗口的字符 `s[i]` 的贡献，即 `val(s[i]) * p^0`。

于是，我们得到了一个完美的 O(1) 更新公式：
`h_new = ((h_old - val(s[i+k]) * p^(k-1)) * p + val(s[i])) mod m`
这个公式只涉及加、减、乘，在模运算下非常简单！

### 滑动窗口算法详解

1.  **准备工作**:
    *   预先计算 `power^(k-1) % modulo`，记为 `pk_1`。这个值在每次移除窗口最右侧字符时都会用到。

2.  **初始化窗口**:
    *   我们从字符串的**最右端**开始。首先完整计算最后一个窗口 `s[n-k : n]` 的哈希值，作为我们的初始 `current_hash`。
    *   检查这个初始哈希值是否与 `hashValue` 匹配。如果匹配，记录下当前的起始索引 `n-k` 作为候选答案。

3.  **逆向滑动窗口**:
    *   从 `i = n-k-1` 开始，循环递减至 `0`。`i` 代表新窗口的起始位置。
    *   在每次循环中，使用上面推导出的滚动哈希公式来更新 `current_hash`。
        *   **移出**：减去旧窗口最右侧字符 `s[i+k]` 的贡献。
        *   **乘幂**：将结果乘以 `power`。
        *   **移入**：加上新窗口最左侧字符 `s[i]` 的贡献。
        *   **注意**：在进行模减法时，为防止出现负数，应写成 `(a - b + modulo) % modulo`。
    *   每次更新哈希值后，都与 `hashValue` 进行比较。
    *   由于我们是从右向左遍历，任何一次匹配都意味着我们找到了一个比之前更靠前的答案。因此，只要匹配成功，就更新结果索引为当前的 `i`。

4.  **返回结果**:
    *   循环结束后，记录的最终索引就是题目所求的第一个满足条件的子串的起始位置。返回该子串即可。

### Python 代码实现

```python
class Solution:
    def subStrHash(self, s: str, power: int, modulo: int, k: int, hashValue: int) -> str:
        n = len(s)

        # 辅助函数，将 'a'-'z' 映射到 1-26
        def val(c):
            return ord(c) - ord('a') + 1

        # 步骤 1：预计算 p^(k-1) % m，用于移除窗口最右侧字符
        pk_1 = pow(power, k - 1, modulo)

        # 步骤 2：计算最右侧第一个窗口 s[n-k : n] 的哈希值
        current_h = 0
        for j in range(k):
            i = n - k + j # 窗口内字符的索引
            current_h = (current_h + val(s[i]) * pow(power, j, modulo)) % modulo
        
        res_index = -1
        if current_h == hashValue:
            res_index = n - k

        # 步骤 3：从右向左滑动窗口
        # i 是新窗口的起始索引
        for i in range(n - k - 1, -1, -1):
            # old_hash (current_h) 对应 s[i+1 : i+k+1]
            # new_hash (将要计算的) 对应 s[i : i+k]
            
            old_right_val = val(s[i + k])
            new_left_val = val(s[i])

            # 应用滚动哈希公式
            # a) 移除旧窗口最右侧字符的贡献
            term_to_remove = (old_right_val * pk_1) % modulo
            current_h = (current_h - term_to_remove + modulo) % modulo
            
            # b) 所有项幂次加一
            current_h = (current_h * power) % modulo
            
            # c) 加上新窗口最左侧字符的贡献
            current_h = (current_h + new_left_val) % modulo

            # 检查哈希值是否匹配
            if current_h == hashValue:
                res_index = i # 更新为更靠前的索引

        # 步骤 4：返回结果
        return s[res_index : res_index + k]
```

### 复杂度分析

*   **时间复杂度**: O(N)。其中 N 是字符串 `s` 的长度。初始哈希计算耗时 O(K)，逆向滑动窗口循环 N-K 次，每次更新是 O(1)。总复杂度为 O(K + N - K) = O(N)，实现了线性时间求解。
*   **空间复杂度**: O(1)。我们只使用了常数个变量来存储哈希值和中间计算结果。

### 总结

本题是滚动哈希算法的绝佳应用案例。它告诉我们，当标准的从左到右的滑动窗口遇到数学难题（如模逆元）时，不妨换个方向思考。**逆向滑动**在本题中优雅地绕开了复杂的模除法，将哈希值的更新简化为基础的模运算，充分展现了算法设计的灵活性和数学之美。
