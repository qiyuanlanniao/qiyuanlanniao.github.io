---
title: LeetCode 2781 | 滑动窗口与后缀校验求解“最长合法子字符串”
date: 2025-10-04 08:08:00
updated: 2025-10-04 08:08:00
comments: true
tags:
  - 字符串
  - 数组
  - 哈希表
  - 滑动窗口
  - 第 354 场周赛
categories:
  - 算法题解
  - 滑动窗口与双指针
  - 不定长滑动窗口
permalink: posts/leetcode-2781/
---
{% centerquote %}
LeetCode 第 2781 题：[最长合法子字符串的长度](https://leetcode.cn/problems/length-of-the-longest-valid-substring/)。
这道题要求我们寻找一个不含任何“禁用词”的最长子串。它是一个典型的滑动窗口问题，但其巧妙之处在于如何高效地判断窗口的“合法性”，避免在每次窗口移动时都进行暴力检查。
{% endcenterquote %}
<!--more-->
### 问题解读

题目给了我们一个字符串 `word` 和一个禁用词列表 `forbidden`。如果一个子字符串不包含 `forbidden` 中的任何字符串，我们就称它是“合法的”。我们的任务是返回 `word` 中最长合法子字符串的长度。

举个例子，对于 `word = "cbaaaabc"`, `forbidden = ["aaa","cb"]`：
*   子字符串 `"cbaaa"` 是非法的，因为它包含了禁用词 `"aaa"`。
*   子字符串 `"aabc"` 是合法的，因为它不包含 `"aaa"` 也不包含 `"cb"`。
*   在所有合法的子字符串中，`"aabc"` 的长度是 4，也是最长的。因此答案是 4。

一个简单的思路是找出 `word` 的所有子字符串，然后逐一检查它们是否包含禁用词。但这种方法的时间复杂度非常高（大约是 O(N^2 * M * L)），在 `word` 长度达到 10^5 时会严重超时。我们需要一个更高效的算法。

### 核心思路：滑动窗口与高效校验

这个问题非常适合使用滑动窗口来解决。“寻找满足条件的最长子串”是滑动窗口的经典应用场景。我们用 `left` 和 `right` 两个指针来维护一个窗口 `word[left:right+1]`。

关键的挑战在于：当 `right` 指针向右移动一位，我们如何快速判断新窗口是否仍然合法？

常规的滑动窗口问题，通常是在窗口扩张后，通过一个 `while` 循环收缩左边界，直到窗口再次满足条件。但在这里，“不包含任何禁用词”这个条件很难通过简单地增减字符计数来维护。

这里的突破口在于：**当我们将 `word[right]` 加入窗口时，所有新产生的、可能导致窗口非法的子字符串，都必然是以 `word[right]` 结尾的。**

  <!-- 这是一个示意图的占位符 -->

换句话说，我们不需要检查整个 `word[left:right+1]`。我们只需要检查以 `word[right]` 结尾的各个后缀，例如 `word[right:right+1]`, `word[right-1:right+1]`, `word[right-2:right+1]` 等，看它们是否是禁用词。

题目中还有一个至关重要的提示：`forbidden[i].length <= 10`。这意味着我们最多只需要检查长度为 10 的后缀。

这样，我们的策略就清晰了：
1.  用 `right` 指针扩张窗口。
2.  每扩张一步，就检查以 `word[right]` 结尾的、长度不超过 10 的所有后缀。
3.  如果发现一个后缀 `word[j:right+1]` 是禁用词，那么就说明整个窗口 `word[left:right+1]` 是非法的。不仅如此，任何以 `j` 或 `j` 之前的字符为起点的、延伸到 `right` 的子串都是非法的。
4.  因此，为了让窗口重新合法，我们必须将左边界 `left` 移动到 `j + 1` 的位置。

### 算法详解

1.  **预处理**：
    *   将 `forbidden` 列表转换成一个哈希集合（Set），这样我们就能以 O(1) 的平均时间复杂度查询一个字符串是否为禁用词。

2.  **滑动窗口初始化**：
    *   左指针 `left = 0`。
    *   记录最长合法子串的长度 `max_len = 0`。

3.  **遍历与窗口扩张**：
    *   用右指针 `right` 从 `0` 遍历到 `n-1`，其中 `n` 是 `word` 的长度。

4.  **合法性校验与窗口调整**：
    *   在 `right` 每移动一步后，我们启动一个内部循环，从 `j = right` 开始，向左回溯。
    *   这个内部循环检查子字符串 `word[j:right+1]`。由于禁用词长度不超过 10，`j` 最多回溯到 `right - 9` 即可。同时，`j` 不能小于当前的左边界 `left`。所以，`j` 的范围是 `[max(left, right - 9), right]`。
    *   如果在回溯检查中，发现 `word[j:right+1]` 存在于禁用词集合中，我们就找到了一个破坏合法性的源头。此时，我们将左指针 `left` 直接更新为 `j + 1`，并立即跳出内部的回溯检查。

5.  **更新结果**：
    *   在（可能发生的）窗口调整之后，当前的窗口 `word[left:right+1]` 一定是合法的。
    *   我们计算其长度 `right - left + 1`，并用它来更新 `max_len`。

6.  **返回最终答案**：
    *   遍历结束后，`max_len` 就是题目的答案。

### Python 代码实现

```python
from typing import List

class Solution:
    def longestValidSubstring(self, word: str, forbidden: List[str]) -> int:
        # 步骤 1: 预处理，转换为集合以快速查找
        forbidden_set = set(forbidden)
        n = len(word)
        
        # 步骤 2: 初始化左指针和最大长度
        left = 0
        max_len = 0
        
        # 步骤 3: 遍历字符串，扩张右边界
        for right in range(n):
            # 步骤 4: 回溯检查以 right 结尾的后缀
            # 最多回溯 10 个字符，且不能越过左边界 left
            for j in range(right, max(left, right - 10) - 1, -1):
                substring = word[j : right + 1]
                
                if substring in forbidden_set:
                    # 发现禁用词，窗口不合法
                    # 新的合法起点必须在禁用词之后，所以移动 left
                    left = j + 1
                    # 找到一个即可，无需继续检查更长的后缀
                    break
            
            # 步骤 5: 当前窗口 [left, right] 合法，更新最大长度
            current_length = right - left + 1
            max_len = max(max_len, current_length)
            
        # 步骤 6: 返回结果
        return max_len

```

### 复杂度分析

*   **时间复杂度**: O(N * C)。其中 N 是字符串 `word` 的长度，C 是 `forbidden` 中字符串的最大长度（本题中 C <= 10）。外层循环遍历 `word` 一次，内层循环最多执行 C 次。由于 C 是一个很小的常数，所以时间复杂度接近线性 O(N)。
*   **空间复杂度**: O(M * C)。其中 M 是 `forbidden` 列表的长度。这部分空间主要用于存储 `forbidden_set`。

### 总结

本题通过一个精妙的优化，将滑动窗口算法应用到了看似复杂的字符串匹配问题上。其核心思想是：**在窗口扩张时，只检查新产生的、以新字符结尾的后缀**。这个方法避免了对整个窗口的重复扫描，而题目中关于禁用词长度的限制则是实现这一优化的关键。这个技巧告诉我们，在处理滑动窗口问题时，要仔细分析窗口状态变化的本质，寻找最高效的合法性校验方式。
