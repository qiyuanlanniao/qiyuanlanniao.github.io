---
title: LeetCode 1763 | 滑动窗口求解“最长的美好子字符串”
date: 2025-10-10 10:05:00
updated: 2025-10-10 10:05:00
comments: true
tags:
  - 滑动窗口
  - 字符串
  - 哈希表
  - 位运算
  - 分治
  - 第46场双周赛
categories:
  - 算法题解
  - 滑动窗口与双指针
  - 不定长滑动窗口
permalink: posts/leetcode-1763/
---
{% centerquote %}
LeetCode 第 1763 题：[最长的美好子字符串](https://leetcode.cn/problems/longest-nice-substring/description/)。
该题要求在字符串中寻找一个最长的子串且其中每个字母大小写必须同时存在。直接枚举所有子串的复杂度较高，我们可以通过枚举子串的“属性”，将问题转化为滑动窗口来高效求解。
{% endcenterquote %}
<!--more-->
### 问题解读

题目定义了一个“美好字符串”：对于字符串中出现的任意一种字母，它的大写和小写形式必须同时存在。例如 `"aAa"` 是美好的，因为 `'a'` 和 `'A'` 都出现了。而 `"abA"` 不是美好的，因为 `'b'` 出现了但 `'B'` 没有。

我们的任务是，给定一个字符串 `s`，找到它最长的一个“美好子字符串”。如果长度相同，返回最先出现的那一个。

直接用暴力法（枚举所有子串 `s[i:j]` 并逐一检查）的时间复杂度会达到 O(N³)，对于 `N=100` 的限制虽然可行，但不够优雅。我们可以构思一个更高效的算法。

### 核心思路：枚举字符类型的滑动窗口

一个常规的滑动窗口通常依赖于一个单调的性质（比如窗口内不同字符数不超过 `k`）。但“美好”这个性质并非单调的：给一个美好的子串 `"aA"` 添加一个字符 `'b'` 会使其变得不美好；给一个不美好的子串 `"aAb"` 添加 `'B'` 又可能使其变得美好。

这种性质的复杂性让我们很难用一个简单的规则来收缩窗口。此时，一个高级的技巧是**“枚举窗口属性”**。与其让窗口自己寻找满足复杂条件的子串，不如我们**指定窗口应该满足的某个简单属性**，然后用滑动窗口来寻找符合这个简单属性的最优解。

在这里，我们可以枚举美好子串中**包含的唯一字母类型的数量**，设为 `k`（例如，`"aAZ"` 包含 `'a'/'A'` 和 `'z'/'Z'` 两种类型，所以 `k=2`）。`k` 的取值范围是 1 到 26。

对于每一个固定的 `k`，问题就转化为：**寻找一个最长的子字符串，它恰好包含 `k` 种字母类型，并且这 `k` 种类型都是“美好”的（即大小写齐全）。**

这个问题就非常适合用滑动窗口来解决了。窗口需要维护两个核心信息：
1.  当前窗口内有多少种唯一字母类型。
2.  这其中，有多少种是“美好”的。

当两个计数器都等于我们正在枚举的 `k` 时，就意味着我们找到了一个候选的子字符串。

### 算法详解

1.  **外层循环**：我们用一个循环来枚举唯一字母类型的数量 `k`，从 1 遍历到 26。

2.  **内层滑动窗口**：对于每个 `k`，我们执行一次完整的滑动窗口遍历。
    *   **状态维护**：我们需要 `lower_count` 和 `upper_count` 两个数组来记录窗口内26个字母大小写的频率。此外，还需要两个计数器：`unique_types` 记录窗口内唯一字母类型的总数，`nice_types` 记录大小写都存在的字母类型数。
    *   **扩张窗口**：`right` 指针向右移动，将新字符 `s[right]` 纳入窗口。相应地更新 `lower/upper_count` 数组，并根据新字符的加入，判断 `unique_types` 和 `nice_types` 是否需要增加。
    *   **收缩窗口**：扩张后，如果 `unique_types > k`，说明当前窗口包含的字母类型太多了，不符合我们对 `k` 的设定。此时需要移动 `left` 指针向右，从窗口中移除 `s[left]`，并同步更新 `lower/upper_count` 和两个计数器。这个过程一直持续到 `unique_types <= k` 为止。
    *   **更新答案**：当窗口收缩完毕后（也可能无需收缩），我们检查是否满足 `unique_types == k` 并且 `nice_types == k`。如果满足，说明当前窗口 `s[left:right+1]` 就是一个恰好包含 `k` 种美好字母类型的子串。我们用它的长度来更新全局的最大长度和起始位置。

3.  **返回结果**：外层循环结束后，记录下的最长长度和起始位置对应的子串即为最终答案。

### Python 代码实现

```python
class Solution:
    def longestNiceSubstring(self, s: str) -> str:
        n = len(s)
        if n < 2:
            return ""

        max_len = 0
        result_start = 0

        # k: 枚举窗口中唯一字母类型的数量
        for k in range(1, 27):
            left = 0
            lower_count = [0] * 26
            upper_count = [0] * 26
            unique_types = 0
            nice_types = 0

            for right in range(n):
                # --- 1. 右指针扩张窗口 ---
                char_r = s[right]
                if 'a' <= char_r <= 'z':
                    idx = ord(char_r) - ord('a')
                    if lower_count[idx] == 0 and upper_count[idx] == 0:
                        unique_types += 1
                    if lower_count[idx] == 0 and upper_count[idx] > 0:
                        nice_types += 1
                    lower_count[idx] += 1
                else: # 大写字母
                    idx = ord(char_r) - ord('A')
                    if lower_count[idx] == 0 and upper_count[idx] == 0:
                        unique_types += 1
                    if upper_count[idx] == 0 and lower_count[idx] > 0:
                        nice_types += 1
                    upper_count[idx] += 1
                
                # --- 2. 左指针收缩窗口 ---
                while unique_types > k:
                    char_l = s[left]
                    if 'a' <= char_l <= 'z':
                        idx = ord(char_l) - ord('a')
                        lower_count[idx] -= 1
                        if lower_count[idx] == 0 and upper_count[idx] > 0:
                            nice_types -= 1
                        if lower_count[idx] == 0 and upper_count[idx] == 0:
                            unique_types -= 1
                    else: # 大写字母
                        idx = ord(char_l) - ord('A')
                        upper_count[idx] -= 1
                        if upper_count[idx] == 0 and lower_count[idx] > 0:
                            nice_types -= 1
                        if lower_count[idx] == 0 and upper_count[idx] == 0:
                            unique_types -= 1
                    left += 1

                # --- 3. 更新结果 ---
                if unique_types == k and nice_types == k:
                    current_len = right - left + 1
                    if current_len > max_len:
                        max_len = current_len
                        result_start = left
        
        if max_len == 0:
            return ""
        return s[result_start : result_start + max_len]
```

### 复杂度分析

*   **时间复杂度**: O(C * N)，其中 C 是字符集的大小（这里是 26），N 是字符串 `s` 的长度。外层循环固定为 26 次，内部的滑动窗口逻辑中，`left` 和 `right` 指针都只遍历一次字符串。因此，总时间复杂度为 O(N)。
*   **空间复杂度**: O(C)。我们使用了几个固定大小的数组（`lower_count`, `upper_count`）来存储状态，空间是常数级别的。

### 总结

本题是滑动窗口技巧的一个精彩应用。当窗口需要满足的条件比较复杂、不单调时，可以尝试**“升维”**的思路：通过枚举窗口的某个关键属性（本题中是唯一字母类型的数量），将一个复杂问题分解为一系列具有更清晰约束的简单子问题，然后逐一用滑动窗口求解。这种“分类讨论 + 滑动窗口”的模式是解决困难字符串问题的一个有力武器。
