---
title: LeetCode 739 | 每日温度
date: 2025-12-07 17:02:00
updated: 2025-12-07 17:02:00
comments: true
tags:
  - 栈
  - 单调栈
  - 数组
categories:
  - 算法题解
  - 单调栈
  - 基础
permalink: posts/leetcode-739/
---
{% centerquote %}
LeetCode 第 739 题：[每日温度](https://leetcode.cn/problems/daily-temperatures/)。
问题的核心是寻找数组中每个元素右侧第一个比它大的元素。
{% endcenterquote %}
<!--more-->

### 问题描述

给定一个整数数组 `temperatures` ，表示每天的温度，返回一个数组 `answer` ，其中 `answer[i]` 是指对于第 `i` 天，下一个更高温度出现在几天后。如果气温在这之后都不会升高，请在该位置用 `0` 来代替。

**示例 1:**
```text
输入: temperatures = [73,74,75,71,69,72,76,73]
输出: [1,1,4,2,1,1,0,0]
```

**示例 2:**
```text
输入: temperatures = [30,40,50,60]
输出: [1,1,1,0]
```

**示例 3:**
```text
输入: temperatures = [30,60,90]
输出: [1,1,0]
```

**提示：**
*   1 <= temperatures.length <= 10^5
*   30 <= temperatures[i] <= 100

### 解题思路

#### 1. 暴力法的局限性
对于每一天，如果我们都向后遍历去寻找第一个比它温度高的日子，最坏情况下的时间复杂度是 O(n^2)。当数组长度达到 10^5 时，这种做法会超时。我们需要一种 O(n) 的解法。

#### 2. 单调栈
这类“寻找下一个更大元素”的问题是单调栈的典型应用场景。

我们可以维护一个存储下标的栈。在本题的解法中，我们采用 **从后往前** 遍历数组的方式：

1.  **倒序遍历**：从最后一天向第一天遍历。因为对于第 `i` 天来说，我们需要知道的是它“未来”的信息。倒序遍历可以让我们先把未来的数据处理好放入栈中。
2.  **栈的作用**：栈中存储的是“未来”日期的下标。这些下标对应的温度，在栈中保持 **从栈顶到栈底递增**。
3.  **核心逻辑**：
    *   当我们遍历到第 `i` 天，温度为 `t` 时，我们查看栈顶的日期。
    *   如果栈顶日期的温度 **小于等于** 当前温度 `t`，说明栈顶的那个日子对于第 `i` 天来说没有任何意义（因为它不够热），而且对于 `i` 之前的日子也没意义（因为第 `i` 天距离更近且温度更高，完全遮挡了栈顶那个日子的作用）。所以，我们可以把栈顶元素 **弹出**。
    *   重复上述操作，直到栈为空或者栈顶日期的温度 **大于** `t`。
    *   此时，如果栈不为空，栈顶的下标就是第 `i` 天右侧第一个比它热的日子。计算距离 `st[-1] - i` 存入结果。
    *   最后，将当前第 `i` 天的下标入栈，因为它可能是更前面的日子的“下一个更高温度”。

通过这种方式，每个元素最多入栈一次、出栈一次，时间复杂度被优化到了 O(n)。

### 代码实现

{% tabs code_block %}
<!-- tab Python -->
```python
class Solution:
    def dailyTemperatures(self, temperatures: List[int]) -> List[int]:
        n = len(temperatures)
        ans = [0] * n
        st = []  # 栈，用于存储下标
        
        # 从后往前遍历 (n-1 到 0)
        # 这样栈里存的都是当前位置之后的一天
        for i in range(n - 1, -1, -1):
            t = temperatures[i]
            
            # 维护单调栈：如果栈顶元素的温度 <= 当前温度
            # 说明栈顶这天不可能是“下一个更高温度”，弹出
            while st and t >= temperatures[st[-1]]:
                st.pop()
            
            # 如果栈不为空，说明找到了右边第一个比当前高的温度
            if st:
                ans[i] = st[-1] - i
            
            # 将当前下标入栈，供前面的元素查找
            st.append(i)
            
        return ans
```
<!-- endtab -->
<!-- tab Go -->
```go
import "slices"

func dailyTemperatures(temperatures []int) []int {
    ans := make([]int, len(temperatures))
    st := []int{} // 栈，存储下标
    
    // 使用 slices.Backward 进行倒序遍历 (Go 1.23+)
    for i, t := range slices.Backward(temperatures) {
        // 当栈不为空，且当前温度 >= 栈顶下标对应的温度时
        // 栈顶元素对于前面的日子已经没有价值了（被当前更高的温度挡住了），弹出
        for len(st) > 0 && t >= temperatures[st[len(st)-1]] {
            st = st[:len(st)-1]
        }
        
        // 如果栈还有元素，栈顶就是右侧第一个比当前温度高的位置
        if len(st) > 0 {
            ans[i] = st[len(st)-1] - i
        }
        
        // 将当前位置推入栈中
        st = append(st, i)
    }
    return ans
}
```
<!-- endtab -->
{% endtabs %}

### 复杂度分析

*   **时间复杂度**: `O(n)`
    虽然代码中包含两层循环（外层 `for` 和内层 `while`），但观察栈的操作可以发现：数组中的每个元素的下标最多被压入栈一次，也最多被弹出栈一次。因此整体操作次数是线性的。
    
*   **空间复杂度**: `O(n)`
    我们需要一个栈来存储下标以及一个数组来存储结果。在最坏情况下（例如数组是单调递减的），栈的大小可以达到 `n`。
