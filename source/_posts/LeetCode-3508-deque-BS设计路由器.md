---
title: LeetCode 3508 | 设计路由器：双端队列与二分查找的高效结合
date: 2025-10-22 10:20:00
updated: 2025-10-22 10:20:00
comments: true
tags:
  - 有序集合
  - 设计
  - 哈希表
  - 队列
  - 二分查找
  - 第444场周赛
categories:
  - 算法题解
  - 二分算法
  - 二分查找
permalink: posts/leetcode-3508/
---
{% centerquote %}
LeetCode 第 3508 题：[设计路由器](https://leetcode.cn/problems/implement-router/solutions/3641772/mo-ni-ha-xi-biao-dui-lie-er-fen-cha-zhao-y7l7/)。
本题要求设计一个数据结构，高效地模拟路由器的行为。解法的关键在于为不同的操作需求选择最合适的数据结构，特别是利用题目中时间戳的有序性，通过二分查找优化范围查询的性能。
{% endcenterquote %}
<!--more-->
### 问题解读

我们需要设计并实现一个 `Router` 类，它能模拟网络路由器处理数据包的基本功能。这个类需要支持三个核心操作：

1.  `Router(int memoryLimit)`: 构造函数，初始化一个有固定内存上限 `memoryLimit` 的路由器。路由器中存储的数据包数量不能超过这个限制。

2.  `bool addPacket(int source, int destination, int timestamp)`: 添加一个数据包。
    *   每个数据包由来源、目的地和时间戳唯一标识。如果一个完全相同的数据包已经存在，则视为重复，添加失败。
    *   如果内存已满（达到 `memoryLimit`），在添加新数据包前，必须先将**最旧的**一个数据包转发出去，以腾出空间。
    *   一个关键信息是：`addPacket` 的调用会按照 `timestamp` 的**非递减顺序**进行。

3.  `int[] forwardPacket()`: 转发数据包。
    *   遵循**先进先出 (FIFO)** 原则，移除并返回存储在路由器中**最旧**的数据包。
    *   如果没有数据包，返回空数组。

4.  `int getCount(int destination, int startTime, int endTime)`: 计数查询。
    *   返回当前路由器中，目的地为 `destination` 且时间戳在 `[startTime, endTime]` 闭区间内的数据包数量。

### 核心思路：组合数据结构 + 二分查找

这是一个典型的系统设计题，需要我们为不同的需求选择最优的数据结构。

1.  **处理 FIFO 和内存限制**：`addPacket` 和 `forwardPacket` 方法共同描述了一个先进先出 (FIFO) 的队列行为。当队列满了之后，需要从队头移除元素。Python 中的 `collections.deque` (双端队列) 是实现这种操作的完美选择，它支持 O(1) 时间复杂度的队头移除和队尾添加。我们将用一个 `deque` 来存储所有的数据包，记为 `pq`。

2.  **处理重复数据包**：`addPacket` 方法要求我们能快速判断一个数据包 `(source, destination, timestamp)` 是否已经存在。为了实现 O(1) 级别的平均查找效率，哈希集合 `set` 是不二之选。我们将用一个 `set` 来存储所有当前存在的数据包，记为 `ps`。

3.  **处理计数查询**：`getCount` 是本题的性能关键点。对于一个给定的 `destination`，我们需要快速统计出在 `[startTime, endTime]` 时间范围内的数据包数量。
    *   一个朴素的想法是遍历整个 `pq` 队列，检查每个数据包的目的地和时间戳，但这样做的时间复杂度是 O(L)（L 为 `memoryLimit`），在大量调用下会超时。
    *   更好的方法是将数据包按目的地进行分组。我们可以使用一个哈希表（`defaultdict`），键是 `destination`，值是一个列表，存储该目的地的所有数据包的时间戳。
    *   注意到题目给出的关键线索：“`addPacket` 的查询，`timestamp` 按非递减顺序给出”。这意味着，当我们向某个目的地的列表中添加时间戳时，这个列表天然就是**有序的**！
    *   对于一个有序的列表，查找某个范围内元素的个数，正是**二分查找**的经典应用场景。我们可以通过 `bisect_left` 找到范围的左边界，通过 `bisect_right` 找到范围的右边界，两者相减即可得到结果，时间复杂度为 O(log K)，其中 K 是该目的地的数据包数量。

综上，我们的整体方案是：
*   使用 `deque` (`pq`) 维护数据包的 FIFO 顺序。
*   使用 `set` (`ps`) 快速检查数据包是否重复。
*   使用 `defaultdict(deque)` (`dt`) 将时间戳按目的地分组并保持有序，以供二分查找。

### 算法详解

1.  **初始化 `__init__`**
    *   `self.limit`: 存储内存限制。
    *   `self.pq = deque()`: 主队列，按到达顺序存储 `(source, destination, timestamp)` 元组，用于 FIFO 转发。
    *   `self.ps = set()`: 集合，同样存储元组，用于 O(1) 去重。
    *   `self.dt = defaultdict(deque)`: 字典，`key` 为 `destination`，`value` 为一个 `deque`，按顺序存储该目的地的所有时间戳。

2.  **`addPacket` 方法**
    *   将传入的参数构造成 `packet` 元组。
    *   检查 `packet` 是否在 `self.ps` 中，若存在则为重复，返回 `False`。
    *   如果不是重复包，将其加入 `self.ps`。
    *   检查 `len(self.pq)` 是否等于 `self.limit`。如果是，则调用 `self.forwardPacket()` 移除最旧的数据包。
    *   将新 `packet` 添加到 `self.pq` 的队尾。
    *   将 `packet` 的 `timestamp` 添加到 `self.dt[destination]` 的队尾。
    *   返回 `True`。

3.  **`forwardPacket` 方法**
    *   检查 `self.pq` 是否为空，若为空则返回 `[]`。
    *   从 `self.pq` 的队首弹出一个最旧的 `packet`。
    *   将该 `packet` 从 `self.ps` 中移除。
    *   从 `self.dt[packet[1]]` (即该包的目的地对应的时间戳队列) 的队首弹出一个时间戳。
    *   返回这个 `packet`。

4.  **`getCount` 方法**
    *   通过 `self.dt[destination]` 获取目标地址所有的时间戳队列 `timestamp`。
    *   使用 `bisect_left(timestamp, startTime)` 查找第一个大于等于 `startTime` 的元素索引 `l`。
    *   使用 `bisect_right(timestamp, endTime)` 查找第一个严格大于 `endTime` 的元素索引 `r`。
    *   `r - l` 的值即为时间戳在 `[startTime, endTime]` 区间内的元素数量。返回这个差值。

### Python 代码实现

```python
from collections import deque, defaultdict
from bisect import bisect_left, bisect_right
from typing import List

class Router:

    def __init__(self, memoryLimit: int):
        # 内存限制
        self.limit = memoryLimit
        # 主队列，维护所有数据包的FIFO顺序
        self.pq = deque()
        # 集合，用于快速去重
        self.ps = set()
        # 按目的地分组的时间戳队列，用于高效范围查询
        self.dt = defaultdict(deque)

    def addPacket(self, source: int, destination: int, timestamp: int) -> bool:
        packet = (source, destination, timestamp)
        # 检查是否重复
        if packet in self.ps:
            return False
        
        self.ps.add(packet)
        # 如果内存已满，转发最旧的数据包
        if len(self.pq) == self.limit:
            self.forwardPacket()
        
        # 添加新数据包到各个数据结构
        self.pq.append(packet)
        self.dt[destination].append(timestamp)
        return True

    def forwardPacket(self) -> List[int]:
        if not self.pq:
            return []
        
        # 从队首取出最旧的数据包
        packet = self.pq.popleft()
        
        # 从所有数据结构中移除该包的信息
        self.ps.remove(packet)
        self.dt[packet[1]].popleft()
        return list(packet)

    def getCount(self, destination: int, startTime: int, endTime: int) -> int:
        # 获取该目的地对应的有序时间戳列表
        timestamp_q = self.dt[destination]
        
        # 使用二分查找定位区间的左右边界
        # l 是第一个 >= startTime 的位置
        l = bisect_left(timestamp_q, startTime)
        # r 是第一个 > endTime 的位置
        r = bisect_right(timestamp_q, endTime)
        
        # r - l 即为区间内元素个数
        return r - l
```

### 复杂度分析

*   **时间复杂度**:`O(log(min(q, memoryLimit)))`，其中 q 是 `addPacket` 的总调用次数。

*   **空间复杂度**: `O(min(q, memoryLimit))`

### 总结

本题是一个优秀的数据结构设计问题，它考验了我们根据不同操作需求选择和组合多种数据结构的能力。通过使用 `deque` 满足 FIFO 要求，使用 `set` 实现快速去重，并巧妙地利用题目中“时间戳非递减”这一隐藏条件，将目的地的时间戳维护成一个有序序列，最终通过二分查找将最复杂的范围查询操作优化到了对数时间复杂度。这个思路清晰地展示了如何通过分析问题特性来设计高效的算法。
