---
title: Leetcode Weekly Contest 276
date: 2022-01-18 23:13:32
tags: Leetcode
categories: Algorithm
mathjax: true
---

# 总结

- AK，但是T4用了太长时间，又掉一波分
- T4一开始就想到了二分查找思路，但是具体实现的确不易想到，花了较长时间
- 排名：国服155，全服371



# [2141. 同时运行 N 台电脑的最长时间](https://leetcode-cn.com/problems/maximum-running-time-of-n-computers/)

- 标签：二分查找
- 分析：对能够运行的最长时间$t$进行二分查找
  - check函数中，对每个电池进行遍历，假如它的电量大于$x$则最多也只可提供$x$的电量。最终我们计算得到总电量，并与$n \times x$进行对比
- 代码

```python
class Solution:
    def maxRunTime(self, n: int, batteries: List[int]) -> int:
        def check(x):
            r = 0
            for i in batteries:
                r += min(i, x)
            return r >= n * x
        
        batteries.sort()
        L = batteries[0]
        R = sum(batteries) // n
        while L <= R:
            mid = (L + R) // 2
            if check(mid):
                L = mid + 1
            else:
                R = mid - 1
        return R
```

<!--more-->

# [2140. 解决智力问题](https://leetcode-cn.com/problems/solving-questions-with-brainpower/)

- 标签：动态规划
- 分析：$dp[i]$表示遍历到第$i$个问题时获得的最大分数，此时可以选择
  - 解决问题$i$并跳转到第$i+questions[i][1]+1$个问题
  - 不解决问题$i$并跳转到第$i+1$个问题
- 代码

```python
class Solution:
    def mostPoints(self, questions: List[List[int]]) -> int:
        dp = [-1] * (len(questions) + 1)
        dp[0] = 0
        r = 0
        for i in range(len(questions) + 1):
            if dp[i] >= 0:
                r = max(r, dp[i])
                if i + 1 < len(dp):
                    if dp[i + 1] == -1 or dp[i + 1] < dp[i]:
                        dp[i + 1] = dp[i]
                    if i + questions[i][1] + 1 < len(dp):
                        if dp[i + questions[i][1] + 1] == -1 or dp[i + questions[i][1] + 1] < dp[i] + questions[i][0]:
                            dp[i + questions[i][1] + 1] = dp[i] + questions[i][0]
                    else:
                        r = max(r, dp[i] + questions[i][0])
        # print(dp)
        return r
```



# [2139. 得到目标值的最少行动次数](https://leetcode-cn.com/problems/minimum-moves-to-reach-target-score/)

- 标签：贪心

- 分析：由于数越大的时候，乘以二的收益越大。因此，将此过程逆向思考，我们可以从$x=target$开始，若$x$是偶数且还有乘以的次数，则令$x = x/2$；我们令$x=x-1$
- 由于$target$较大，而乘以次数较少，当乘法的次数用完后，我们只能每次减一，因此此时可直接根据$x$的值求得行动次数
- 代码

```python
class Solution:

    def minMoves(self, target: int, maxDoubles: int) -> int:
        r = 0
        while target > 1:
            if target % 2 == 0 and maxDoubles:
                target //= 2
                maxDoubles -= 1
            else:
                target -= 1
            r += 1
            if maxDoubles == 0:
                return r + target - 1
            
        return r 
```



# [2138. 将字符串拆分为若干长度为 k 的组](https://leetcode-cn.com/problems/divide-a-string-into-groups-of-size-k/)

- 标签：模拟
- 代码

```python
class Solution:
    def divideString(self, s: str, k: int, fill: str) -> List[str]:
        r = []
        for i in range(0, len(s), k):
            if i + k <= len(s):
                r.append(s[i: i + k])
            else:
                t = s[i: ]
                t += fill * (k - len(t))
                r.append(t)
        return r
```

