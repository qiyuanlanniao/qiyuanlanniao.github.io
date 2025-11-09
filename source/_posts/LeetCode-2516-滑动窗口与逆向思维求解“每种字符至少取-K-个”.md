---
title: LeetCode 2516 | 滑动窗口与逆向思维：求解“每种字符至少取 K 个”
date: 2025-09-29 08:08:00
updated: 2025-09-29 08:08:00
comments: true
tags:
  - 字符串
  - 滑动窗口
  - 哈希表
  - 第 325 场周赛
categories:
  - 算法题解
  - 滑动窗口与双指针
  - 不定长滑动窗口
permalink: posts/leetcode-2516/
---
{% centerquote %}
LeetCode 第 2516 题：[每种字符至少取 K 个](https://leetcode.cn/problems/take-k-of-each-character-from-left-and-right/)。
这道题要求我们从字符串的两端拿走字符，乍看之下似乎需要复杂的双指针或动态规划。但如果我们转换一下思路，它就变成了一个非常巧妙的滑动窗口问题。
{% endcenterquote %}
<!--more-->
### 问题解读

题目给了我们一个仅包含 'a', 'b', 'c' 的字符串 `s` 和一个整数 `k`。我们需要从字符串的**最左侧**或**最右侧**取走字符，目标是达成“每种字符 ('a', 'b', 'c') 都至少取了 `k` 个”的条件。我们要找到满足这个条件所需的最少操作次数（即取走的最少字符总数）。

举个例子，对于 `s = "aabaaaacaabc"`, `k = 2`：
*   我们的目标是最终取走的字符集合中，至少有 2 个 'a'，2 个 'b'，和 2 个 'c'。
*   一种可行的方案是：从左边取 3 个字符 (`"aab"`)，从右边取 5 个字符 (`"aabc"`)。
*   总共取走的字符是 `"aab"` 和 `"aabc"` 的组合，包含了 3 个 'a'，2 个 'b'，2 个 'c'，满足条件。
*   总共取走的字符数是 3 + 5 = 8。题目要求我们找到这个最小的总数。

直接去模拟从两端取字符的过程，情况会非常复杂，因为左边取 `i` 个，右边就要相应地调整，很难找到最优解。

### 核心思路：逆向思维与滑动窗口

当“两端操作求最优解”变得困难时，我们不妨逆向思考：**从两端取走最少的字符，等价于在中间保留最长的连续子串。**

![Sliding Window Explanation](https://assets.leetcode.com/uploads/2022/11/29/example-1.png)

我们的问题就转化成了：在 `s` 中寻找一个**最长的连续子字符串**，使得这个子字符串**之外**的剩余部分，'a', 'b', 'c' 的数量都至少为 `k`。

假设整个字符串 `s` 中 'a', 'b', 'c' 的总数分别是 `total_a`, `total_b`, `total_c`。
如果我们中间保留的窗口（子字符串）中 'a', 'b', 'c' 的数量是 `window_a`, `window_b`, `window_c`。

那么，在窗口之外（也就是我们从两端取走的部分）的字符数量就是：
*   取走的 'a' 数量：`total_a - window_a`
*   取走的 'b' 数量：`total_b - window_b`
*   取走的 'c' 数量：`total_c - window_c`

根据题意，我们需要满足：
*   `total_a - window_a >= k`
*   `total_b - window_b >= k`
*   `total_c - window_c >= k`

将这些不等式变形，我们就可以得到对**中间保留窗口**的约束条件：
*   `window_a <= total_a - k`
*   `window_b <= total_b - k`
*   `window_c <= total_c - k`

现在，问题变得清晰了：**找到一个最长的连续子字符串，其中 'a', 'b', 'c' 的数量分别不超过 `total_a - k`、`total_b - k` 和 `total_c - k`**。

这正是滑动窗口算法擅长解决的“寻找满足特定条件的最长子串”问题！我们只需要用滑动窗口找到这个最长窗口的长度 `max_len`，最终答案就是 `s` 的总长度减去 `max_len`。

### 算法详解

1.  **预处理与边界检查**：
    *   如果 `k=0`，不需要取任何字符，直接返回 0。
    *   统计整个字符串 `s` 中 'a', 'b', 'c' 的总数 `total_counts`。
    *   如果任意一种字符的总数本身就小于 `k`，那无论如何也无法满足条件，直接返回 -1。

2.  **确定窗口约束**：
    *   计算出中间保留的窗口内，每种字符允许的最大数量 `max_allowed`。例如，`max_allowed['a'] = total_counts['a'] - k`。

3.  **滑动窗口初始化**：
    *   左指针 `left = 0`。
    *   记录最长有效窗口的长度 `max_len = 0`。
    *   用一个字典或数组 `window_counts` 实时记录当前窗口内 'a', 'b', 'c' 的数量。

4.  **遍历与窗口扩张**：
    *   用右指针 `right` 从 `0` 遍历到 `n-1`。
    *   在循环的每一步，将 `s[right]` 这个新字符纳入窗口，并更新 `window_counts`。

5.  **条件判断与窗口收缩**：
    *   扩张后，检查当前窗口是否依然满足约束（即 `window_counts` 中每种字符的数量是否都小于等于 `max_allowed` 中对应的上限）。
    *   如果不满足，说明窗口“太大了”，需要从左侧收缩。我们使用一个 `while` 循环来执行这个操作：
        *   只要窗口不满足约束，就将左侧字符 `s[left]` 移出窗口（更新 `window_counts`），然后将 `left` 指针右移。
        *   这个 `while` 循环会一直执行，直到窗口重新满足约束为止。

6.  **更新结果**：
    *   在每次 `right` 指针移动后，并且在（可能发生的）收缩操作完成后，当前的窗口 `s[left:right+1]` 一定是有效的。
    *   我们计算当前有效窗口的长度 `right - left + 1`，并用它来更新 `max_len`。

7.  **计算最终答案**：
    *   遍历结束后，`max_len` 就是我们找到的“可以保留的最长子串”的长度。
    *   那么，需要取走的最少字符数就是 `len(s) - max_len`。

### Python 代码实现

```python
import collections

class Solution:
    def takeCharacters(self, s: str, k: int) -> int:
        n = len(s)
        
        # 步骤 1: 预处理与边界检查
        if k == 0:
            return 0
        
        total_counts = collections.Counter(s)
        if total_counts['a'] < k or total_counts['b'] < k or total_counts['c'] < k:
            return -1
            
        # 步骤 2: 确定窗口内每种字符的数量上限
        max_allowed = {c: total_counts[c] - k for c in 'abc'}
        
        left = 0
        max_len = 0 # 存储满足条件的最长窗口长度
        window_counts = collections.defaultdict(int)
        
        # 步骤 4: 滑动窗口，扩张右边界
        for right in range(n):
            char_right = s[right]
            window_counts[char_right] += 1
            
            # 步骤 5: 如果窗口不满足条件，收缩左边界
            while (window_counts['a'] > max_allowed['a'] or 
                   window_counts['b'] > max_allowed['b'] or 
                   window_counts['c'] > max_allowed['c']):
                char_left = s[left]
                window_counts[char_left] -= 1
                left += 1
            
            # 步骤 6: 当前窗口有效，更新最大长度
            max_len = max(max_len, right - left + 1)
            
        # 步骤 7: 最终结果是总长度减去最长保留子串的长度
        return n - max_len
```

### 复杂度分析

*   **时间复杂度**: O(N)。其中 N 是字符串 `s` 的长度。滑动窗口的左指针 `left` 和右指针 `right` 都只会从头到尾各自扫描一遍字符串，每个元素最多被访问两次。
*   **空间复杂度**: O(1)。我们只使用了常数个额外变量来存储字符计数，因为字符集是固定的 ('a', 'b', 'c')。

### 总结

本题的核心在于**逆向思维**，它将一个看似复杂的“两端操作”问题，巧妙地转化为了一个经典的“寻找满足条件的最长子串”问题。一旦完成了这个思路转换，剩下的就是滑动窗口算法的模板化应用了。这个技巧在处理类似问题时非常有效，值得我们牢记。
