---
title: LeetCode 2904 | SW求解最短且字典序最小的美丽子串
date: 2025-10-11 11:00:00
updated: 2025-10-11 11:00:00
comments: true
tags:
  - 滑动窗口
  - 字符串
  - 第367场周赛
categories:
  - 算法题解
  - 滑动窗口与双指针
  - 不定长滑动窗口
permalink: posts/leetcode-2904/
---
{% centerquote %}
LeetCode 第 2904 题：[最短且字典序最小的美丽子字符串](https://leetcode.cn/problems/shortest-and-lexicographically-smallest-beautiful-string/description/)。
该题要求在一个二进制字符串中，寻找一个包含 `k` 个 '1' 的子串，这个子串需要满足两个条件：首先长度最短，其次在所有最短的子串中，它的字典序最小。
{% endcenterquote %}
<!--more-->
### 问题解读

题目给了我们一个二进制字符串 `s` 和一个正整数 `k`。我们需要找到 `s` 的一个“美丽子字符串”。

一个子字符串被称为“美丽”的，当且仅当它包含的 '1' 的数量恰好为 `k`。

我们的目标是：
1.  在所有“美丽子字符串”中，找到那些长度最短的。
2.  在所有这些最短的“美丽子字符串”中，找到字典序最小的那一个。

如果不存在这样的子字符串，我们应该返回一个空字符串。

举个例子，对于 `s = "100011001"`, `k = 3`：
*   子字符串 `"100011"` 是美丽的（3 个 '1'），长度为 6。
*   子字符串 `"011001"` 也是美丽的（3 个 '1'），长度为 6。
*   子字符串 `"11001"` 还是美丽的（3 个 '1'），长度为 5。
*   最短的美丽子字符串长度是 5。
*   在所有长度为 5 的美丽子字符串中（这里只有一个 `"11001"`），字典序最小的就是 `"11001"`，因此它就是答案。

### 核心思路：不定长滑动窗口

这类寻找满足特定条件的“最优”子字符串的问题，是滑动窗口算法的绝佳应用场景。我们可以维护一个窗口 `[left, right]`，它代表了我们正在考察的子字符串。

我们的策略如下：
1.  **扩展窗口**：我们不断地将 `right` 指针向右移动，以扩大窗口。每当一个新字符进入窗口时，我们更新窗口内 '1' 的数量。

2.  **收缩窗口**：一旦窗口内 '1' 的数量达到了 `k`，我们就找到了一个“美丽子字符串”的候选者。这时，我们不能停下，因为这个候选者不一定是最短的。
    *   **评估候选者**：我们将当前窗口代表的子字符串与已找到的最佳答案进行比较。如果当前子串更短，或者长度相同但字典序更小，我们就更新答案。
    *   **启动收缩**：为了寻找可能存在的、更短的美丽子字符串，我们必须从左侧收缩窗口。我们将 `left` 指针向右移动，并相应地更新 '1' 的数量。这个收缩过程会一直持续，直到窗口不再满足 '1' 的数量为 `k` 的条件，之后我们再继续扩展 `right` 指针。

通过这种“扩展-评估-收缩”的循环，我们能确保遍历所有可能的美丽子字符串，并最终找到最优解。

### 算法详解

1.  **初始化**：
    *   `min_len`：用于记录当前找到的最短美丽子串的长度，初始为无穷大。
    *   `result`：用于存储最终答案，初始为空字符串 `""`。
    *   `left = 0`：滑动窗口的左边界。
    *   `ones_count = 0`：当前滑动窗口内 '1' 的数量。

2.  **遍历与扩展**：
    *   使用 `right` 指针从头到尾遍历字符串 `s`。
    *   当 `s[right]` 为 '1' 时，`ones_count` 加 1。

3.  **触发收缩与评估**：
    *   当 `ones_count` 恰好等于 `k` 时，我们找到了一个美丽子字符串。此时，启动一个 `while` 循环，因为 `[left, right]` 是一个候选项，`[left+1, right]` 也可能是（如果 `s[left]` 是 '0'）。
    *   **在 `while (ones_count == k)` 循环内**：
        1.  获取当前窗口的长度 `current_len = right - left + 1` 和子字符串 `current_substring = s[left:right+1]`。
        2.  **比较与更新**：
            *   如果 `current_len < min_len`，说明我们找到了一个更短的，直接更新 `min_len = current_len` 和 `result = current_substring`。
            *   如果 `current_len == min_len`，我们需要比较字典序。如果 `current_substring < result`，则更新 `result = current_substring`。
        3.  **收缩**：将 `left` 指针右移一位。如果被移出窗口的字符 `s[left]` 是 '1'，则将 `ones_count` 减 1。这一步可能会导致 `ones_count` 变为 `k-1`，从而结束 `while` 循环。

4.  **返回结果**：
    *   主循环结束后，`result` 中存储的就是最终答案。如果从未找到过美丽子字符串，`result` 将保持其初始值 `""`。

### Python 代码实现

```python
class Solution:
    def shortestBeautifulSubstring(self, s: str, k: int) -> str:
        # 如果 s 中 '1' 的总数都小于 k，则不可能有美丽子字符串
        if s.count('1') < k:
            return ""

        n = len(s)
        min_len = float('inf')
        result = ""
        
        left = 0
        ones_count = 0
        
        for right in range(n):
            # 扩展窗口
            if s[right] == '1':
                ones_count += 1
            
            # 当窗口内 '1' 的数量达到 k 时，开始评估和收缩
            while ones_count == k:
                current_len = right - left + 1
                current_substring = s[left : right + 1]
                
                # 评估：如果找到了更短的，或同样短但字典序更小的
                if current_len < min_len:
                    min_len = current_len
                    result = current_substring
                elif current_len == min_len and current_substring < result:
                    result = current_substring
                
                # 收缩窗口
                if s[left] == '1':
                    ones_count -= 1
                left += 1

        return result

```

### 复杂度分析

*   **时间复杂度**: O(N)，其中 N 是字符串 `s` 的长度。虽然代码里有嵌套的 `while` 循环，但 `left` 和 `right` 两个指针都只从左到右单向移动一次，每个字符最多被访问两次（一次被 `right` 指针纳入，一次被 `left` 指针排除），因此总的时间复杂度是线性的。
*   **空间复杂度**: O(1)。我们只使用了常数个额外变量来存储窗口状态和结果。返回的字符串 `result` 的空间不算在额外空间复杂度内。

### 总结

本题是滑动窗口思想的直接应用。解决此题的关键在于清晰地定义窗口的“合法”状态（`ones_count == k`），并在进入此状态时，不仅要与全局最优解进行比较，还要尝试通过收缩窗口来寻找一个可能更优的解。双重优化目标（长度优先，字典序其次）的逻辑判断是本题的核心细节。
