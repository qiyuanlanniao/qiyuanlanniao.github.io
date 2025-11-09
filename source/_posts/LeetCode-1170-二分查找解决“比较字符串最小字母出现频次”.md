---
title: LeetCode 1170 | 二分查找解决“比较字符串最小字母出现频次”
date: 2025-10-19 11:20:00
updated: 2025-10-19 11:20:00
comments: true
tags:
  - 数组
  - 哈希表
  - 字符串
  - 二分查找
  - 排序
  - 第151场周赛
categories:
  - 算法题解
  - 二分算法
  - 二分查找
permalink: posts/leetcode-1170/
---
{% centerquote %}
LeetCode 第 1170 题：[比较字符串最小字母出现频次](https://leetcode.cn/problems/compare-strings-by-frequency-of-the-smallest-character/)。
通过对词汇表进行预处理和排序，我们可以利用二分查找将查询效率从线性级别优化到对数级别。
{% endcenterquote %}
<!--more-->
### 问题解读

题目首先定义了一个函数 `f(s)`，它的功能是计算一个非空字符串 `s` 中，按字典序最小的那个字母出现的次数。例如，对于 `s = "dcce"`，最小的字母是 'c'，它出现了 2 次，所以 `f("dcce") = 2`。

接着，题目给了我们两个字符串数组：一个查询数组 `queries` 和一个词汇表数组 `words`。我们的任务是，对于 `queries` 中的每一个查询字符串 `queries[i]`，都要计算出在 `words` 数组中有多少个单词 `W` 满足 `f(queries[i]) < f(W)`。最后，返回一个整数数组 `answer`，其中 `answer[i]` 就是第 `i` 次查询的结果。

举个例子，`queries = ["bbb","cc"]`, `words = ["a","aa","aaa","aaaa"]`：
*   对于第一个查询 "bbb"：
    *   最小字母是 'b'，出现 3 次，所以 `f("bbb") = 3`。
    *   我们需要在 `words` 中找到 `f(W) > 3` 的单词。
    *   `f("a")=1`, `f("aa")=2`, `f("aaa")=3`, `f("aaaa")=4`。只有一个 "aaaa" 满足条件。所以结果是 1。
*   对于第二个查询 "cc"：
    *   最小字母是 'c'，出现 2 次，所以 `f("cc") = 2`。
    *   我们需要在 `words` 中找到 `f(W) > 2` 的单词。
    *   "aaa" 和 "aaaa" 满足条件。所以结果是 2。
*   因此，最终答案是 `[1, 2]`。

### 核心思路：预计算 + 二分查找

一个直观的想法是，对每个 `query`，我们都遍历一遍 `words` 数组，计算每个 `word` 的 `f` 值并进行比较。如果 `queries` 的长度是 N，`words` 的长度是 M，那么这种暴力解法的时间复杂度大约是 O(N * M)，在 N 和 M 都达到 2000 的情况下，可能会超时。

注意到，`words` 数组是固定不变的。我们可以对其进行**预处理**，来加速后续的查询过程。

整个优化的思路分为两步：

1.  **预计算与排序**：我们先遍历一次 `words` 数组，计算出其中每个单词 `W` 的 `f(W)` 值，并将这些频率值存入一个新数组 `word_freqs` 中。为了能够快速查找，我们对 `word_freqs` 进行升序排序。

2.  **高效查询**：完成预处理后，对于每个 `query`，我们先计算出它的频率 `q_freq`。然后，问题就转化成了：**在一个排好序的数组 `word_freqs` 中，有多少个元素大于 `q_freq`？** 这是一个典型的二分查找应用场景。我们可以通过二分查找，快速定位到第一个大于 `q_freq` 的元素的位置。一旦找到了这个位置，那么它后面的所有元素都满足条件，我们就能立刻得到答案。

通过这种方式，每次查询的时间复杂度从 O(M) 降到了 O(log M)，从而显著提高了算法的整体效率。

### 算法详解

1.  **定义辅助函数 `f(s)`**
    *   这个函数接收一个字符串 `s`。
    *   找到 `s` 中的最小字符 `min_char`。
    *   返回 `min_char` 在 `s` 中出现的次数。

2.  **预处理 `words` 数组**
    *   创建一个整数数组 `word_freqs`。
    *   遍历 `words` 中的每一个 `word`，调用 `f(word)` 计算其频率，并将结果添加到 `word_freqs` 中。
    *   对 `word_freqs` 数组进行升序排序。

3.  **处理 `queries` 数组**
    *   创建一个空的答案数组 `ans`。
    *   遍历 `queries` 中的每一个 `query`：
        *   计算其频率 `q_freq = f(query)`。
        *   在排好序的 `word_freqs` 数组上执行二分查找，目标是找到第一个**严格大于** `q_freq` 的元素的索引。
        *   在 Python 中，`bisect.bisect_right(array, value)` 函数可以完美地实现这个功能。它会返回 `value` 在 `array` 中的插入点索引，这个索引恰好是数组中大于 `value` 的第一个元素的位置。
        *   假设 `word_freqs` 的长度为 `n`，找到的索引为 `index`，那么 `word_freqs` 中大于 `q_freq` 的元素个数就是 `n - index`。
        *   将这个计数结果添加到 `ans` 数组中。

4.  **返回结果**
    *   遍历结束后，返回 `ans` 数组。

### Python 代码实现

```python
import bisect
from typing import List

class Solution:
    def numSmallerByFrequency(self, queries: List[str], words: List[str]) -> List[int]:
        
        # 步骤 1: 定义辅助函数 f(s)
        def f(s: str) -> int:
            if not s:
                return 0
            min_char = min(s)
            return s.count(min_char)

        # 步骤 2: 预处理 words 数组
        # 计算 words 中每个单词的频率并排序
        word_freqs = sorted([f(word) for word in words])
        
        # 步骤 3: 处理 queries 数组
        ans = []
        n = len(word_freqs)
        for query in queries:
            q_freq = f(query)
            
            # 使用二分查找找到第一个 > q_freq 的元素的位置
            # n - (该位置的索引) 即为满足条件的单词数量
            index = bisect.bisect_right(word_freqs, q_freq)
            count = n - index
            ans.append(count)
            
        # 步骤 4: 返回结果
        return ans

```

### 复杂度分析

*   **时间复杂度**: O(M * Lw + M log M + N * (Lq + log M))。
    *   `N` 是 `queries` 的长度, `M` 是 `words` 的长度。
    *   `Lq` 和 `Lw` 分别是 `queries` 和 `words` 中字符串的最大长度。
    *   计算 `word_freqs` 需要 O(M * Lw)。
    *   排序 `word_freqs` 需要 O(M log M)。
    *   对于每个查询，计算 `f(query)` 需要 O(Lq)，二分查找需要 O(log M)。总共是 O(N * (Lq + log M))。
*   **空间复杂度**: O(M)。我们需要一个额外的数组 `word_freqs` 来存储 `words` 数组中每个单词的频率。

### 总结

本题是“预处理 + 二分查找”模式的一个经典应用。当题目要求对一个固定的数据集进行多次查询时，我们应该优先考虑是否能通过对数据集进行一次性的预处理（例如计算、排序等），来为后续的查询操作提供便利，从而将整体复杂度降低一个量级。二分查找正是利用了数据的有序性来实现高效查询的强大工具。
