title: LeetCode 2258 | 逃离火灾
date: 2025-11-05 22:22:00
updated: 2025-11-05 22:22:00
comments: true
tags:
  - 数组
  - 二分查找
  - 广度优先搜索
  - BFS
  - 矩阵
  - 第77场双周赛
categories:
  - 算法题解
  - 二分算法
  - 二分答案
  - 求最大
permalink: posts/leetcode-2258/
---
{% centerquote %}
LeetCode 第 2258 题：[逃离火灾](https://leetcode.cn/problems/escape-the-spreading-fire/description/)。
本题是一道结合了广度优先搜索（BFS）和二分查找的经典问题。核心在于将求解“最大等待时间”的优化问题，转化为一个“给定等待时间，能否逃脱”的判定性问题，从而利用答案的单调性进行高效求解。
{% endcenterquote %}
<!--more-->

### 问题解读

我们身处一个 `m x n` 的网格中，网格包含草地(0)、火(1)和墙(2)。我们的任务是从左上角 `(0, 0)` 出发，到达右下角的安全屋 `(m - 1, n - 1)`。

游戏规则如下：
1.  我们每分钟可以移动到相邻的草地格子。
2.  在我们每次移动**之后**，火会向所有相邻的非墙格子蔓延。
3.  我们可以在起点 `(0, 0)` 等待任意分钟数再出发。

目标是找到我们可以在起点停留的**最多**分钟数，并且之后仍然能够安全到达安全屋。

这里有几个特殊情况需要注意：
*   如果无论如何都无法到达，返回 `-1`。
*   如果我们总能安全到达（例如火被墙完全困住），无论等待多久都可以，返回 `10^9`。
*   如果在我们到达安全屋的同一分钟，火也到达了，这算作安全到达。

### 核心思路：将“求解问题”转化为“判定问题”与二分查找

直接求解“最多等待多少分钟”非常棘手。但我们可以换一个角度思考：如果我们**假定**一个等待时间 `k`，我们能否判断出在这个条件下，是否可以安全到达安全屋？

这个问题（我们称之为 `check(k)`）就清晰多了。而这个判定结果具有非常重要的**单调性**：
*   如果等待 `k` 分钟后我们**可以**安全逃脱，那么等待一个更短的时间 `k-1` 分钟，我们给了自己一个时间优势，**必然**也能安全逃脱。
*   反之，如果等待 `k` 分钟后**无法**逃脱，那么等待更长的时间 `k+1` 分钟，火势会更猛烈，我们就**更不可能**逃脱。

这种单调性是应用**二分查找**的完美信号。我们可以对“等待时间”这个值进行二分查找，来快速定位到那个“最长的可行等待时间”的临界点。

这个二分查找的搜索范围是什么呢？
*   **下界 (left)**：最少可以等待 0 分钟。
*   **上界 (right)**：一个足够大的数。考虑到网格大小最多为 `m*n`，如果等待 `m*n` 分钟后人还能走，火此时基本已经蔓延到所有能去的地方了。如果此时人还能到达终点，那么等再久也一样。所以 `m*n` 是一个合理的上界。

因此，我们就在 `[0, m*n]` 这个区间内，对等待时间 `k` 进行二分查找。

### 算法详解

整个算法可以分为三个主要步骤：

#### 1. 预计算火焰传播时间

为了判断人的每一步是否安全，我们首先需要知道火在什么时候会到达地图上的每一个格子。这个问题可以通过一个**多源广度优先搜索（Multi-Source BFS）**来解决。
*   创建一个 `fire_time[m][n]` 数组，用于存储火到达每个格子的最短时间，全部初始化为一个特殊值（如 -1 或无穷大）。
*   将所有初始的火源格子 `(r, c)` 加入一个队列中，并将它们的 `fire_time` 设为 0。
*   运行 BFS，逐层扩展火势，计算并填充 `fire_time` 数组。

#### 2. 实现 `check(k)` 判定函数

这个函数是二分查找的核心，它用来判定“等待 `k` 分钟后出发，能否安全到达”。这同样可以用一次**BFS**来模拟人的移动过程。
*   创建一个队列，并将人的初始状态 `(row=0, col=0, time=k)` 加入队列。`time` 表示当前时刻。
*   运行 BFS 模拟人的移动。从当前格子 `(r, c)`（在 `t` 时刻到达）移动到相邻的草地格子 `(nr, nc)` 时，必须满足安全条件：
    *   人到达 `(nr, nc)` 的时间是 `t + 1`。
    *   这个格子必须还没被火烧到。即 `t + 1` 必须**严格小于** `fire_time[nr][nc]`。
    *   **特殊情况**：对于终点安全屋，题目允许同时到达，所以条件放宽为 `t + 1 <= fire_time[nr][nc]`。
*   如果在 BFS 过程中，人能够成功到达安全屋，则 `check(k)` 返回 `True`。如果队列为空还没到达，说明此路不通，返回 `False`。

#### 3. 执行二分查找

现在我们有了判定函数 `check(k)`，可以进行标准的二分查找了。
*   设置 `left = 0`, `right = m * n`，并用一个变量 `ans = -1` 记录最终答案。
*   在 `while left <= right` 循环中：
    *   取中间值 `mid = left + (right - left) // 2`。
    *   调用 `check(mid)`：
        *   如果返回 `True`，说明等待 `mid` 分钟是可行的。我们记录下这个可行解 `ans = mid`，然后尝试等待更长的时间，即 `left = mid + 1`。
        *   如果返回 `False`，说明等待 `mid` 分钟太久了，必须缩短等待时间，即 `right = mid - 1`。
*   循环结束后，`ans` 中存储的就是最大的可行等待时间。最后根据 `ans` 的值处理 `10^9` 和 `-1` 的特殊情况即可。

### Python 代码实现

```python
import collections
from typing import List

class Solution:
    def maximumMinutes(self, grid: List[List[int]]) -> int:
        m, n = len(grid), len(grid[0])
        dirs = [(-1, 0), (1, 0), (0, -1), (0, 1)]

        # 步骤1: 使用多源BFS计算火到达每个格子(非墙)的时间
        fire_time = [[-1] * n for _ in range(m)]
        q = collections.deque()
        for r in range(m):
            for c in range(n):
                if grid[r][c] == 1:
                    q.append((r, c, 0))
                    fire_time[r][c] = 0

        while q:
            r, c, t = q.popleft()
            for dr, dc in dirs:
                nr, nc = r + dr, c + dc
                if 0 <= nr < m and 0 <= nc < n and grid[nr][nc] != 2 and fire_time[nr][nc] == -1:
                    fire_time[nr][nc] = t + 1
                    q.append((nr, nc, t + 1))

        # 步骤2: 定义check函数, 判定等待wait_time分钟后是否能逃脱
        def check(wait_time: int) -> bool:
            person_q = collections.deque()
            if fire_time[0][0] != -1 and wait_time >= fire_time[0][0]:
                return False
            person_q.append((0, 0, wait_time))
            visited = set([(0, 0)])

            while person_q:
                r, c, t = person_q.popleft()
                if r == m - 1 and c == n - 1:
                    return True

                for dr, dc in dirs:
                    nr, nc = r + dr, c + dc
                    if 0 <= nr < m and 0 <= nc < n and grid[nr][nc] != 2 and (nr, nc) not in visited:
                        arrival_time = t + 1
                        fire_arrival_time = fire_time[nr][nc]

                        if fire_arrival_time == -1 or arrival_time < fire_arrival_time or \
                           (nr == m - 1 and nc == n - 1 and arrival_time <= fire_arrival_time):
                            visited.add((nr, nc))
                            person_q.append((nr, nc, arrival_time))
            return False

        # 步骤3: 二分查找最大等待时间
        left, right = 0, m * n
        ans = -1
        while left <= right:
            mid = left + (right - left) // 2
            if check(mid):
                ans = mid
                left = mid + 1
            else:
                right = mid - 1

        # 步骤4: 处理特殊返回值
        if ans == m * n:
            return 10**9
        else:
            return ans
```

### 复杂度分析

*   **时间复杂度**: O(M * N * log(M*N))。
    *   预计算火势蔓延时间的 BFS 复杂度为 O(M * N)。
    *   二分查找的范围是 `m*n`，所以需要进行 `log(M*N)` 次迭代。
    *   在每次迭代中，我们都需要调用 `check` 函数，该函数内部是一次人的 BFS，复杂度也为 O(M * N)。
    *   因此，总时间复杂度由二分查找部分主导，为 O(M * N * log(M*N))。
*   **空间复杂度**: O(M * N)。
    *   我们需要一个 `fire_time` 数组来存储火的到达时间，以及在 `check` 函数中需要一个 `visited` 集合或数组，空间都是 O(M * N)。

### 总结

本题是“二分答案”思想的绝佳体现。它将一个看似复杂的“求最优值”问题，通过挖掘其内在的单调性，巧妙地转化成了一个易于解决的“判定性”问题。当我们遇到求解“最大值最小化”或“最小值最大化”这类问题时，都应该优先思考是否能套用“二分答案 + 验证函数”这一强大的算法模型。而本题中的验证函数又与图论中的最短路算法 BFS 紧密结合，是一道综合性很强的好题。
