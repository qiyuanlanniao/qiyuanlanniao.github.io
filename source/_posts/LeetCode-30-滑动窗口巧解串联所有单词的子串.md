---
title: LeetCode 30 | 滑动窗口巧解串联所有单词的子串
date: 2025-09-19 14:30:22
updated: 2025-09-19 15:10:45
comments: true
tags:
  - 字符串
  - 滑动窗口
  - 哈希表
  - 双指针
categories:
  - 算法题解
  - 滑动窗口与双指针
  - 定长滑动窗口
permalink: posts/leetcode-30/
---
{% centerquote %}
LeetCode 第 30 题：[串联所有单词的子串](https://leetcode.cn/problems/substring-with-concatenation-of-all-words/)。
这道题是滑动窗口思想的一次精彩升级。它不再是处理单个字符，而是将“单词”作为窗口滑动的基本单位，同时引入了“分组”处理的巧妙思想，是理解滑动窗口灵活性的绝佳案例。
{% endcenterquote %}
<!--more-->
### 问题解读

我们来仔细分析一下题目：给定一个主字符串 `s` 和一个单词数组 `words`，要求找出 `s` 中所有“串联子串”的起始索引。

*   **串联子串**：这个子串由 `words` 中的**所有**单词**恰好一次**、以**任意顺序**拼接而成。
*   **关键约束**：`words` 中的所有单词长度都**相同**。这是一个非常重要的突破口。

例如，如果 `words = ["foo", "bar"]`，那么 `s` 中的 `"foobar"` 和 `"barfoo"` 都是串联子串。子串 `"barfoothefoobarman"` 中，从索引 `0` 开始的 `"barfoo"` 和从索引 `9` 开始的 `"foobar"` 都是有效的串联子串。

### 核心思路：从字符窗口到单词窗口

直接检查 `s` 中每一个可能的子串是否由 `words` 的排列构成，效率会很低。正确的思路依然是**滑动窗口**，但需要进行一些关键的调整。

#### 滑动单位的转变

该题的基本单位不再是单个字符，而是一个个**单词**。

*   目标 `words` 包含 `k` 个长度为 `len` 的单词。
*   那么，一个有效的“串联子串”的总长度是固定的：`total_len = k * len`。
*   我们的滑动窗口大小也应该是 `total_len`。
*   窗口每次滑动的步长，不再是 `1` 个字符，而应该是 `len` 个字符，即一个单词的长度。

#### 分组滑动的巧妙构思

一个最关键的问题是：滑动窗口的起点应该在哪里？

如果单词长度 `len = 3`，一个有效的串联子串可能从索引 `0, 3, 6, ...` 开始，也可能从 `1, 4, 7, ...` 或 `2, 5, 8, ...` 开始。这三种情况是**完全独立、互不干扰**的。

因此，我们可以把对 `s` 的一次遍历，拆分成 `len` 次独立的遍历。

*   第一次遍历，我们只考虑从 `s[0]` 开始，以 `len` 为步长构建的单词序列。
*   第二次遍历，我们只考虑从 `s[1]` 开始，以 `len` 为步长构建的单词序列。
*   ...
*   第 `len` 次遍历，我们只考虑从 `s[len-1]` 开始的序列。

通过这种方式，我们用一个外层循环（遍历 `0` 到 `len-1` 这 `len` 种起始偏移）和内层滑动窗口相结合，就能覆盖所有可能的情况。

### 滑动窗口算法详解

1.  **准备工作**:
    *   获取单词长度 `word_len`，单词数量 `num_words`，以及串联子串总长 `total_len`。
    *   处理边界情况，例如 `s` 或 `words` 为空，或 `s` 的长度小于 `total_len`。
    *   使用哈希表 `words_cnt` 统计 `words` 数组中每个单词的出现频率。

2.  **外层循环（分组）**:
    *   启动一个 `for` 循环，`i` 从 `0` 遍历到 `word_len - 1`。这个 `i` 代表每组滑动的起始偏移量。

3.  **内层循环（滑动窗口）**:
    *   在每组中，初始化一个左指针 `l = i` 和一个用于记录窗口内单词频率的哈希表 `window_cnt`。
    *   用右指针 `r` 从 `i` 开始，以 `word_len` 为步长向右遍历 `s`。
    *   **移入单词**：在每次迭代中，从 `s` 中截取 `s[r : r + word_len]` 作为一个新单词。
    *   **处理新单词**:
        *   如果这个新单词**不在** `words_cnt` 中，说明它是一个无效单词。当前窗口内的所有单词都无法构成串联，因此直接清空 `window_cnt`，并将左指针 `l` 移动到这个无效单词的后面，即 `l = r + word_len`。
        *   如果新单词是有效单词，则将其计入 `window_cnt`。
        *   **处理冗余**：检查 `window_cnt` 中该单词的数量是否超过了 `words_cnt` 中的需求。如果是，则需要从窗口左侧不断移出单词（`l` 右移 `word_len`），直到这个单词的数量恢复正常。
    *   **检查匹配**:
        *   在窗口不包含冗余单词后，检查当前窗口内的单词总数 `(r - l) // word_len + 1` 是否等于 `num_words`。
        *   如果相等，说明我们找到了一个完整的串联子串，其起始位置就是当前的左指针 `l`，将其加入结果列表。
        *   找到后，为了继续搜索，我们将窗口最左侧的单词移出（`l` 右移 `word_len`），以便寻找下一个可能的匹配。

4.  **返回结果**:
    *   所有分组遍历完成后，返回记录所有起始索引的结果列表。

### Python 代码实现

```python
from collections import Counter
from typing import List

class Solution:
    def findSubstring(self, s: str, words: List[str]) -> List[int]:
        # 步骤 1：准备工作
        if not s or not words:
            return []

        n = len(s)
        num_words = len(words)
        word_len = len(words[0])
        total_len = num_words * word_len

        if n < total_len:
            return []

        words_cnt = Counter(words)
        res = []

        # 步骤 2：外层循环，处理 len 种不同的分组
        for i in range(word_len):
            # 初始化滑动窗口的左右指针和当前窗口的单词频率表
            l = i
            window_cnt = Counter()
            
            # 步骤 3：内层循环，以 word_len 为步长滑动窗口
            for r in range(i, n - word_len + 1, word_len):
                # 从右侧移入一个新单词
                word = s[r : r + word_len]

                if word in words_cnt:
                    window_cnt[word] += 1
                    
                    # 如果窗口中此单词数量过多，从左侧收缩窗口
                    while window_cnt[word] > words_cnt[word]:
                        left_word = s[l : l + word_len]
                        window_cnt[left_word] -= 1
                        l += word_len
                    
                    # 步骤 3.检查匹配：窗口大小正好等于所需单词数
                    if (r - l) // word_len + 1 == num_words:
                        res.append(l)
                        # 匹配成功后，将窗口左侧单词移出，继续寻找下一个匹配
                        left_word = s[l : l + word_len]
                        window_cnt[left_word] -= 1
                        l += word_len

                else:
                    # 如果遇到无效单词，之前的所有努力都作废
                    # 重置窗口，并将左指针移到无效单词之后
                    window_cnt.clear()
                    l = r + word_len
                    
        return res
```

### 复杂度分析

*   **时间复杂度**: O(L * W)。其中 L 是字符串 `s` 的长度，W 是单个单词的长度 `word_len`。外层循环执行 W 次，内层循环对 `s` 的遍历步长为 W，每次截取子串的成本是 W。所以总的时间复杂度近似为 W * (L/W) * W = O(L * W)。
*   **空间复杂度**: O(K * W)，其中 K 是 `words` 中不同单词的个数，W 是单词的长度。这部分空间主要用于存储 `words_cnt` 和 `window_cnt` 这两个哈希表。

### 总结

本题是滑动窗口技巧应用的绝佳进阶范例。它告诉我们，滑动窗口的“单位”和“步长”都可以是灵活的。通过**将问题分解为 `word_len` 个独立的子问题**，我们成功地将一个看似复杂的问题，用清晰的滑动窗口逻辑进行了求解。这种“分组处理”的思想在解决某些具有特定步长或周期的字符串问题时非常有效。
