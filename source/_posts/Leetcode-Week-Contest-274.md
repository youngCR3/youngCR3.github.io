---
title: Leetcode Week Contest 274
date: 2022-01-02 14:48:54
tags: Leetcode
categories: Algorithm
mathjax: true
---

# [5970. 参加会议的最多员工数](https://leetcode-cn.com/problems/maximum-employees-to-be-invited-to-a-meeting/)

- 标签：图论，环
- 分析：本题的分析较为复杂
  - 首先能想到的是图中必定存在环，因为图中存在n个节点与n条边，因此任意环中的节点必定能形成会议
  - 然后考虑一种特殊的环，**长度为2的环**，这意味着两名员工相互喜欢。这种环的特殊之处在于**其两个节点再接一条“长链”后还可以形成会议**（这种情况的特殊之处是根据给的测试用例，**画图后观察**想到的）
  - 进一步考虑，将所有长度为2的环放在圆桌上，只要保证环中节点是相邻的，也可以形成会议
  - 再进一步考虑，**将长度为2的环以及其连着的两条长链视为一个单位**，那么多个单位放在圆桌上，也可以形成会议（这一点开始是并没有想到，WA后根据测试用例**画图后观察**想到的）
  - 由此，解题思路产生，答案可能是以下两种情况的一种
    - 最长的环的节点数量
    - 所有“单位”的节点数量和
- 实现
  - 对于最长的环的确定，可以直接DFS求得所有环后得到，通过`vis`数组记录访问过的节点即可
  - “单位”的求解稍微复杂，关键在于”长链“的长度的确定。容易想到的是可以先从入度为0的节点开始，它必定是某条“链”的起点，但是不同的链可能在某一个节点汇合，为了避免重复计算，可以引入两个数组，`target[i]`代表节点i所在的链最终通往的节点，该节点处于一个长度为2的环中，`dist[i]`代表节点i离它最终通往的节点的距离，将`dist[i]`和`target[i]`的初始值设为-1
  - 求解时，需要记录当前链上的各个节点，如果遇到`dist[i]`非0，则说明该节点之前已经遍历过，由此可以反向确定当前链上各个节点的`target[j]=target[i]`，`dist[j]=dist[i]+d`，其中`j`表示当前链上的各个节点，`d`表示链上节点与`i`之间的距离，`dist`需要倒序更新。如路径为`[3,1,2]`，且遍历到节点4时发现`target[4]=0`，`dist[4]=1`，则可以推得`dist[2]=2`，`dist[1]=3`，`dist[3]=4`，且`target[2]=target[1]=target[3]=0`
- 代码

```python
class Solution:
    def maximumInvitations(self, favorite: List[int]) -> int:
        vis = [0] * len(favorite)
        pairs = {}
        maxLen = {}
        dist = [-1] * len(favorite)
        target = [-1] * len(favorite)
        for i in range(len(favorite)):
            if favorite[favorite[i]] == i:
                vis[i] = vis[favorite[i]] = 1
                pairs[i] = favorite[i]
                pairs[favorite[i]] = i
                maxLen[i] = 0
                maxLen[favorite[i]] = 0
                dist[i] = dist[favorite[i]] = 0
                target[i] = i
                target[favorite[i]] = favorite[i]
        
        res = 0
        for i in range(len(favorite)):
            if vis[i] == 0:
                j = i
                path = []
                idx = {}
                while vis[j] == 0:
                    idx[j] = len(path)
                    path.append(j)
                    vis[j] = 1
                    j = favorite[j]
                    
                else:
                    if dist[j] >= 0:
                        # find a visited node leading to a two-node-loop
                        for k in range(len(path) - 1, -1, -1):
                            dist[path[k]] = len(path) - k + dist[j]
                            target[path[k]] = target[j]
                        maxLen[target[j]] = max(maxLen[target[j]], len(path) + dist[j])
                    elif j in idx:
                        # find a loop
                        res = max(res, len(path) - idx[j])
        L = 0
        for i in maxLen:
            L += maxLen[i] + 1
        res = max(res, L)
        return res
```

<!--more-->

# [5967. 检查是否所有 A 都在 B 之前](https://leetcode-cn.com/problems/check-if-all-as-appears-before-all-bs/)

- 标签：字符串

- 代码

```python
class Solution:
    def checkString(self, s: str) -> bool:
        a = b = -1
        for i in range(len(s)):
            if s[i] == 'a':
                a = i
            elif b == -1:
                b = i
        return b == -1 or a < b
```

# [5968. 银行中的激光束数量](https://leetcode-cn.com/problems/number-of-laser-beams-in-a-bank/)

- 标签：数组
- 分析：假设上一个有装置的行有x台装置，本行有y台装置，则会形成$x \cdot y$束激光
- 代码

```python
class Solution:
    def numberOfBeams(self, bank: List[str]) -> int:
        cnt = [0] * len(bank)
        for i in range(len(bank)):
            for j in range(len(bank[i])):
                if bank[i][j] == '1':
                    cnt[i] += 1
        res = last = 0
        for i in range(len(cnt)):
            if cnt[i]:
                if last:
                    res += last * cnt[i]
                last = cnt[i]
        return res
```

# [5969. 摧毁小行星](https://leetcode-cn.com/problems/destroying-asteroids/)

- 标签：排序
- 代码

```python 
class Solution:
    def asteroidsDestroyed(self, mass: int, asteroids: List[int]) -> bool:
        asteroids.sort()
        for i in asteroids:
            if mass >= i:
                mass += i
            else:
                return False
        return True
```

## O(N)做法

- 引入：上述排序后遍历一次的$O(NlogN)$非常简单，赛后题解区指出，本题可基于桶对数据进行划分，从而实现$O(N)$时间复杂度的解法

- 标签：桶

- 分析：将数据按$[1, 2), [2, 4), ..., [2^i, 2^{i+1})$的规律划分为多组，如果当前行星的质量大于等于组内最小值，则其加上最小值的行星后必定大于组内所有其他行星，否则返回false，这是因为
  $$
  mass+min \geq 2 \cdot 2^{i}=2^{i+1}
  $$

- 代码

```python
class Solution:
    def asteroidsDestroyed(self, mass: int, asteroids: List[int]) -> bool:
        buckets = [0] * 17
        minVal = [-1] * 17
        for i in asteroids:
            j = int(log(i, 2))
            buckets[j] += i
            if minVal[j] == -1 or i < minVal[j]:
                minVal[j] = i

        for i in range(17):
            if minVal[i] >= 1:
                if mass >= minVal[i]:
                    mass += buckets[i]
                else:
                    return False
        return True
```

# 排名情况

- 国服133，全球299
