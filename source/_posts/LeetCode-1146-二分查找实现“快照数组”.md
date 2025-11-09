---
title: LeetCode 1146 | 二分查找实现“快照数组”
date: 2025-10-21 11:20:00
updated: 2025-10-21 11:20:00
comments: true
tags:
  - 数组
  - 设计
  - 二分查找
  - 哈希表
  - 第148场周赛
categories:
  - 算法题解
  - 二分算法
  - 二分查找
permalink: posts/leetcode-1146/
---
{% centerquote %}
LeetCode 第 1146 题：[快照数组](https://leetcode.cn/problems/snapshot-array/)。
通过为每个数组索引维护一个历史记录列表，并利用二分查找，我们可以在 O(log K) 的时间内高效地获取任意历史快照中的值。
{% endcenterquote %}
<!--more-->
### 问题解读

这道题要求我们设计一个名为 `SnapshotArray` 的类，它模拟一个数组，但增加了“快照”功能。具体来说，需要实现以下几个接口：

*   `SnapshotArray(int length)`：构造函数，初始化一个长度为 `length` 的数组，所有元素的初始值都为 0。
*   `void set(index, val)`：将当前数组在 `index` 位置的元素值修改为 `val`。
*   `int snap()`：对当前数组的状态进行一次“快照”，并返回这次快照的唯一标识 `snap_id`。这个 ID 是从 0 开始递增的。
*   `int get(index, snap_id)`：获取在 `snap_id` 这次快照时，`index` 位置上元素的值。

举个例子来理解：
```
SnapshotArray snapshotArr = new SnapshotArray(3); // 数组为 [0, 0, 0]
snapshotArr.set(0, 5);  // 当前数组为 [5, 0, 0]
snapshotArr.snap();     // 拍了一张快照，snap_id = 0。这张快照记录了 [5, 0, 0] 的状态
snapshotArr.set(0, 6);  // 当前数组为 [6, 0, 0]
snapshotArr.get(0, 0);  // 获取 snap_id=0 的快照中，索引0的值。当时的值是 5，所以返回 5。
```

### 核心思路：空间换时间 + 二分查找

一个最直观但效率低下的想法是，每次调用 `snap()` 时，都完整地复制一份当前数组的全部内容并存储起来。如果数组长度为 `L`，调用 `snap()` 的次数为 `S`，这种方法的空间复杂度将高达 O(L * S)，在 `L` 和 `S` 都很大时会消耗巨量内存。同时，`snap()` 操作本身的时间复杂度也是 O(L)，效率堪忧。

我们可以进行优化。实际上，每次调用 `set` 时，只有单个元素的值发生了改变。我们没有必要为整个数组创建副本，而只需要记录**哪个元素，在哪个时刻（snap_id），变成了什么值**。

基于这个想法，我们可以为数组的**每一个索引**都维护一个独立的修改历史列表。这个列表里存储的是一个个的 `(snap_id, value)` 对。

例如，`history[0] = [(0, 5), (2, 8)]` 就表示：
*   在 `snap_id = 0` 时，索引 `0` 的值被设置为了 `5`。
*   在 `snap_id = 2` 时，索引 `0` 的值被更新为了 `8`。

当我们要调用 `get(index, snap_id)` 时，问题就转变成了：**在 `index` 的历史记录中，找到在小于等于 `snap_id` 的所有记录里，离 `snap_id` 最近的那一条记录的值。**

由于我们是按时间顺序调用 `set` 和 `snap` 的，所以每个索引的历史记录中，`snap_id` 自然是单调递增的。这就为我们使用**二分查找**创造了完美的条件，从而可以快速地定位到目标记录。

### 算法详解

1.  **数据结构设计**
    *   我们需要一个变量 `csi` (current snap id) 来记录当前的快照编号，初始为 0。
    *   我们需要一个核心数据结构来存储每个索引的历史变更。这里使用一个哈希表（在 Python 中是 `defaultdict`），键是数组的 `index`，值是一个列表，列表里存放 `(snap_id, val)` 的元组。例如：`self.history = {0: [(0, 5), (2, 8)], 1: [(0, 4)]}`。

2.  **`__init__(self, length)`**
    *   初始化 `csi = 0`。
    *   初始化 `self.history = defaultdict(list)`。

3.  **`set(self, index, val)`**
    *   向 `index` 对应的历史记录列表中追加一个新的元组 `(self.csi, val)`。这表示在当前的快照时间点，`index` 的值被设置为了 `val`。
    *   *优化提示*: 如果 `index` 对应的历史列表不为空，且最后一条记录的 `snap_id` 与当前的 `csi` 相同，说明在同一次快照内对同一位置进行了多次 `set`。此时可以直接覆盖最后一条记录的值，而不是追加，可以节省一点空间。但直接追加在逻辑上也是完全正确的。

4.  **`snap(self)`**
    *   这个操作非常简单，只需将 `csi` 加一，并返回加一之前的旧值即可，即 `self.csi - 1`。

5.  **`get(self, index, snap_id)`**
    *   这是算法的核心。首先获取 `index` 对应的历史记录列表 `history[index]`。
    *   我们需要在这个列表中找到满足 `记录的snap_id <= 给定的snap_id` 的最右侧（即 `snap_id` 最大）的那条记录。
    *   我们可以使用二分查找来高效地完成这个任务。具体来说，我们查找第一个 `snap_id > 给定的snap_id` 的位置。这个位置的前一个元素，就是我们想要的答案。
    *   Python 的 `bisect_left` 或 `bisect_right` 模块非常适合这个场景。`bisect_left(history[index], (snap_id + 1, ))` 会找到 `(snap_id + 1, )` 这个元组应该插入的位置，这个位置索引 `j` 正是第一条 `snap_id` 大于 `snap_id` 的记录。
    *   因此，`j - 1` 就是我们目标记录的索引。
    *   需要处理边界情况：
        *   如果 `j - 1` 为负数，说明 `history[index]` 中所有的记录 `snap_id` 都大于给定的 `snap_id`，或者这个索引从未被 `set` 过。根据题意，此时应返回初始值 0。
        *   否则，返回 `history[index][j-1]` 中存储的值即可。

### Python 代码实现

```python
from collections import defaultdict
from bisect import bisect_left
from typing import List

class SnapshotArray:
    def __init__(self, length: int):
        # csi: current snap id, 当前快照编号
        self.csi = 0
        # history: 使用哈希表存储每个索引的历史记录
        # key: 数组索引, value: [(snap_id, val), ...] 列表
        self.history = defaultdict(list)

    def set(self, index: int, val: int) -> None:
        # 在当前快照时间点，记录 index 处的值
        # 如果历史记录的最后一次快照ID和当前ID相同，可以优化为覆盖，但追加更简单且正确
        if self.history[index] and self.history[index][-1][0] == self.csi:
            self.history[index][-1] = (self.csi, val)
        else:
            self.history[index].append((self.csi, val))

    def snap(self) -> int:
        # 快照数加一，并返回旧的 snap_id
        self.csi += 1
        return self.csi - 1

    def get(self, index: int, snap_id: int) -> int:
        # 在 index 的历史记录中，二分查找 snap_id
        # 目标是找到 <= snap_id 的最大快照记录
        records = self.history[index]
        
        # bisect_left 寻找插入 (snap_id + 1, ) 的位置
        # 这个位置的左边所有元素的 snap_id 都 <= snap_id
        # 因为元组比较时会先比较第一个元素
        j = bisect_left(records, (snap_id + 1,)) - 1
        
        # 如果 j >= 0，说明找到了一个有效的历史记录
        if j >= 0:
            return records[j][1]
        
        # 否则，说明在该快照或之前，该索引从未被 set 过，返回初始值 0
        return 0

```

### 复杂度分析

*   `N` 是数组的长度, `C` 是 `set` 操作的总调用次数。
*   **时间复杂度**:
    *   `__init__`: O(1)。`defaultdict` 的初始化是常数时间。
    *   `set`: O(1)。向列表末尾追加元素是摊还 O(1) 的时间复杂度。
    *   `snap`: O(1)。仅涉及一次整数自增操作。
    *   `get`: O(log K)。其中 `K` 是对 `index` 调用 `set` 的次数。在最坏的情况下，所有的 `set` 操作都作用于同一个 `index`，此时 `K` 等于 `C`，所以最坏时间复杂度为 O(log C)。
*   **空间复杂度**: O(C)。我们存储了每一次 `set` 操作的数据。每次 `set` 都会创建一个 `(snap_id, val)` 元组，所以总的空间消耗与 `set` 的调用次数成正比。

### 总结

本题是“预处理 + 二分查找”模式的一个经典应用，同时也考察了数据结构设计的能力。面对可能导致巨大时空开销的朴素解法，我们应该思考**变化的是什么，不变的是什么**。在这里，数组的大部分内容在快照之间是不变的，只有被 `set` 的部分才变化。抓住这一关键点，通过只记录增量变化（即历史记录）的方式，成功地用可控的空间换取了极高的查询效率。而二分查找，正是利用历史记录中 `snap_id` 的有序性来实现高效查询的利器。
