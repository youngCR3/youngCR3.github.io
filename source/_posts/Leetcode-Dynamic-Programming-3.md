---
title: Leetcode Dynamic Programming 3
date: 2022-01-05 13:34:09
tags: Leetcode
mathjax: true
categories: Algorithm
---

# 鸡蛋掉落问题

## [1884. 鸡蛋掉落-两枚鸡蛋](https://leetcode-cn.com/problems/egg-drop-with-2-eggs-and-n-floors/)

- 分析：令$dp(i,j)$代表拥有$i$枚鸡蛋，且楼层数为$j$时所需的最小操作次数，则可以找到以下规律

  - 当$j=0$时，$dp(i,0)=0$

  - 当$i=1$时，由于只有一枚鸡蛋，必须一层一层地试，因此$dp(1,j)=j$
  - 对于其他情况，我们假设从第$k$层往下投一枚鸡蛋
    - 如果没碎，则我们需要考虑比$k$更高的楼层，操作数为$dp(i,j-k)$
    - 如果碎了，则我们需要考虑比$k$更低的楼层，注意此时我们的鸡蛋数目变为$i-1$，操作数为$dp(i-1,k-1)$
  - 由此可得状态转移方程为

  $$
  dp(i,j)=min\{1+max(dp(i-1,k-1),dp(i,j-k))\} \quad k=1,2,...,j
  $$

- 简化：由于只有两枚鸡蛋，且当$i=1$时，$dp(i,j)=j$，因此可以简化为一维dp（时间复杂度仍然是$O(n^2)$）

- 代码

```python
class Solution:
    def twoEggDrop(self, n: int) -> int:
        dp = [i for i in range(n + 1)]
        for i in range(1, n + 1):
            for j in range(1, i + 1):
                dp[i] = min(dp[i], 1 + max(j - 1, dp[i - j]))
        return dp[-1]
```



<!--more-->

## [887. 鸡蛋掉落](https://leetcode-cn.com/problems/super-egg-drop/)

### 时间复杂度$O(n^2k)$

- 分析：动态规划，基本原理和上题一致
- 数据范围：$1 \leq k \leq 100$，$1 \leq n \leq 10^4$
- 时间复杂度：$O(n^2k)$，无法通过
- 代码

```python 
class Solution:
    def superEggDrop(self, k: int, n: int) -> int:
        dp = [[i for i in range(n + 1)] for _ in range(k + 1)]
        for i in range(1, k + 1):
            for j in range(1, n + 1):
                if i == 1:
                    dp[i][j] = j
                else:
                    for x in range(1, j + 1):
                        dp[i][j] = min(dp[i][j], 1 + max(dp[i - 1][x - 1], dp[i][j - x]))
        return dp[-1][-1]
```

### 二分搜索优化，时间复杂度$O(k \cdot nlogn )$

- 分析：由于上述方法时间复杂度过高，需要考虑优化的方法，再次对动态转移方程进行分析

$$
dp(i,j)=min\{1+max(dp(i-1,k-1),dp(i,j-k))\} \quad k=1,2,...,j
$$

- $max$函数中的两项，第一项$dp(i-1,k-1)$是关于$k$的单调递增函数，第二项$dp(i,j-k)$是关于$k$的单调递减函数，两者最大值的最小值**在函数交点处**取得。但由于本题的$dp$是离散函数，仅在整数处有定义，如果交点$k_{opt}$并非整数，则需要分别考虑$k_1=ceil(k_{opt})$和$k_2=floor(k_{opt})$，并取两点中函数值较小的一点
- 时间复杂度：$O(k \cdot nlogn)$
- 图示分析如下，图源自力扣官方题解

![](function.png)

- 代码

```python
class Solution:
    def superEggDrop(self, k: int, n: int) -> int:
        dp = [[i for i in range(n + 1)] for _ in range(k + 1)]
        for i in range(1, k + 1):
            for j in range(1, n + 1):
                if i == 1:
                    dp[i][j] = j
                else:
                    L = 1
                    R = j
                    while L <= R:
                        mid = (L + R) // 2
                        if mid == 1:
                            L = mid + 1
                        elif mid == j:
                            R = mid - 1
                        else:
                            if dp[i - 1][mid - 1] <= dp[i][j - mid]:
                                L = mid + 1
                            else:
                                R = mid - 1
                    if R < 1:
                        dp[i][j] = 1 + dp[i - 1][(R + 1) - 1]
                    elif R + 1 > j:
                        dp[i][j] = 1 + dp[i][j - R]
                    else:
                        dp[i][j] = 1 + min(dp[i][j - R], dp[i - 1][(R + 1) - 1])
        return dp[-1][-1]
```

### 记忆化搜索代替dp，常数优化

- 分析：上述代码仍然无法通过，总时间超时，推测是力扣卡常，可以使用记忆化搜索代替dp，减少对无效状态的求解，从而优化常数
- 时间复杂度仍为$O(k \cdot nlogn)$
- 代码

```python
class Solution:
    def superEggDrop(self, k: int, n: int) -> int:
        @lru_cache(None)
        def dp(k, n):
            if n == 0:
                return 0
            elif k == 1:
                return n
            L = 1
            R = n
            while L <= R:
                m = (L + R) // 2
                if dp(k - 1, m - 1) >= dp(k, n - m):
                    R = m - 1
                else:
                    L = m + 1
            if R < 1:
                return 1 + dp(k - 1, R + 1 - 1)
            elif R + 1 > n:
                return 1 + dp(k, n - R)
            else:
                return 1 + min(dp(k - 1, R + 1 - 1), dp(k, n - R))

        return dp(k, n)
```

### 其他解法

- 本题还有时间复杂度更低的解法，涉及竞赛知识，可参考力扣 [官方题解](https://leetcode-cn.com/problems/super-egg-drop/solution/ji-dan-diao-luo-by-leetcode-solution-2/)



# 股票问题

## [123. 买卖股票的最佳时机 III](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii/)

- 分析：考虑当前的状态可以是以下五种
  - 0：未进行任何交易
  - 1：购买了第一笔股票，持有中
  - 2：购买并售出了第一笔股票
  - 3：购买了第二笔股票，持有中
  - 4：购买并售出了第二笔股票
- 令$dp(i,j)$表示第$i$天位于状态$j$时所能获得的最大利润，状态转移方程可简写为下式

$$
dp(i,j) =\left\{
\begin{aligned}
&0 &\quad j=0\\
&max\{dp(i-1,j),dp(i-1,j-1)+sign \cdot prices[i]\} &\quad 1 \leq j \leq 4
\end{aligned}
\right.
$$

- 其中$sign$是正负变量，由下式求得

$$
sign=\left\{
\begin{aligned}
&1 \quad &j \ mod \ 2 = 0 \\
&-1 \quad &j \ mod \ 2 = 1 \\
\end{aligned}
\right.
$$

- 代码

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        dp = [[float('-inf')] * 5 for _ in range(len(prices))]
        for i in range(len(prices)):
            dp[i][0] = 0
            if i == 0:
                dp[i][1] = -prices[i]
            else:
                for j in range(1, 5):
                    val1 = dp[i - 1][j]
                    val2 = dp[i - 1][j - 1] + (prices[i] if j % 2 == 0 else -prices[i])
                    dp[i][j] = max(val1, val2)
        return max(dp[-1][0], dp[-1][2], dp[-1][4])
```

## [188. 买卖股票的最佳时机 IV](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iv/)

- 分析：基本原理与上题类似，$dp(i,j,k)$表示第$i$天，完成了$j$笔交易，且当前持有股票状态为$k$时的最大利润，$k=1$代表当前已经购买第$j+1$笔交易但未卖出，$k=0$代表完成$j$笔交易后没有进行其他交易，状态转移方程为

$$
\left\{
\begin{aligned}
dp(i,j,0)&=max\{dp(i-1,j,0),\ dp(i-1,j-1,1)+prices[i]\} \\
dp(i,j,1)&=max\{dp(i-1,j,1),\ dp(i-1,j,0)-prices[i]\}
\end{aligned}
\right.
$$

- 代码

```python
class Solution:
    def maxProfit(self, m: int, prices: List[int]) -> int:
        n = len(prices)
        if n == 0 or m == 0:
            return 0
        dp = [[[float('-inf')] * 2 for _ in range(m + 1)] for _ in range(n)]
        res = float('-inf')
        for i in range(n):
            for j in range(m + 1):
                if j == 0:
                    dp[i][j][0] = 0
                    if i > 0:
                        dp[i][j][1] = max(-prices[i], dp[i - 1][j][1])
                    else:
                        dp[i][j][1] = -prices[i]
                else:
                    dp[i][j][0] = max(dp[i - 1][j][0], dp[i - 1][j - 1][1] + prices[i])
                    if j <= m:
                        dp[i][j][1] = max(dp[i - 1][j][1], dp[i - 1][j][0] - prices[i])
                res = max(res, dp[i][j][0])
        return res
```

# 二分查找 + 动态规划

## [1751. 最多可以参加的会议数目 II](https://leetcode-cn.com/problems/maximum-number-of-events-that-can-be-attended-ii/)

- 标签：二分查找 + 动态规划

- 分析：首先将所有会议按起始时间排序，令$dp(i,j)$为遍历到第$i$个会议且已经开了$j$个会议（此时可选择开或不开第$i$个会议）时能获得的最大价值，接下来

  - 如果不开第$i$个会议，则可以转移到第$i+1$个会议，即

  $$
  dp(i+1,j)=max\{dp(i+1,j),dp(i,j)\}
  $$

  

  - 如果开第$i$个会议，则可以转移到下一个起始时间大于当前结束时间的工作，通过二分查找可以找到其下标$k$，则有

  $$
  dp(k,j+1)=max\{dp(k,j+1), \ dp(i,j)+value[i]\}
  $$

- 代码

```python
class Solution:
    def maxValue(self, events: List[List[int]], k: int) -> int:
        events.sort()
        dp = [[-1] * len(events) for _ in range(k + 1)]
        for j in range(len(events)):
            dp[0][j] = 0
        res = 0
        for i in range(k + 1):
            for j in range(len(events)):
                if dp[i][j] >= 0:
                    res = max(res, dp[i][j])
                    # 不完成事件j
                    if j + 1 < len(events):
                        dp[i][j + 1] = max(dp[i][j + 1], dp[i][j])
                    # 选择完成事件j
                    if i + 1 < k + 1:
                        L = j + 1
                        R = len(events) - 1
                        while L <= R:
                            mid = (L + R) // 2
                            if events[mid][0] <= events[j][1]:
                                L = mid + 1
                            else:
                                R = mid - 1
                        if R + 1 < len(events):
                            dp[i + 1][R + 1] = max(dp[i + 1][R + 1], dp[i][j] + events[j][2]) 
                        else:
                            res = max(res, dp[i][j] + events[j][2])
    
        return res
```

### 一种错误思路

- 开始时误以为当$i$确定时，$dp(i,j)$是关于$j$的单调递增函数，于是采用单调栈找到每个会议的上一个能参加的会议，设为$last1$，则状态转移方程为$dp(i,j)=max\{dp(k,j-1)+value[i],dp(i-1,j)\}$。这种思路的错误之处在于**当$i$确定时，$dp(i,j)$并非关于$j$的单调递增函数**

## [1235. 规划兼职工作](https://leetcode-cn.com/problems/maximum-profit-in-job-scheduling/)

- 标签：二分查找 + 动态规划

- 分析：首先将所有工作按起始时间`startTime`排序，令$dp(i)$为遍历到第$i$个工作（此时可选择做或不做第$i$个工作）时能获得的最大报酬，接下来

  - 如果不做第$i$个工作，则可以转移到第$i+1$个工作，即

  $$
  dp(i+1)=max\{dp(i+1),dp(i)\}
  $$

  - 如果做第$i$个工作，则可以转移到下一个`startTime`大于`endTime[i]`的工作，通过二分查找可以找到其下标$j$，则有

  $$
  dp(j)=max\{dp(j), dp(i)+profit[i]\}
  $$

- 代码

```python
class Solution:
    def jobScheduling(self, startTime: List[int], endTime: List[int], profit: List[int]) -> int:
        arr = []
        for i in range(len(startTime)):
            arr.append([startTime[i], endTime[i], profit[i]])
        arr.sort()
        dp = [0] * len(arr)
        res = 0
        for i in range(len(arr)):
            if i + 1 < len(arr):
                dp[i + 1] = max(dp[i + 1], dp[i])
            
            L = i + 1
            R = len(arr) - 1
            while L <= R:
                mid = (L + R) // 2
                if arr[mid][0] < arr[i][1]:
                    L = mid + 1
                else:
                    R = mid - 1
            if R + 1 < len(arr):
                dp[R + 1] = max(dp[R + 1], dp[i] + arr[i][2])
            else:
                res = max(res, dp[i] + arr[i][2])
        return res
```



# 数学推导 + 动态规划

## [1259. 不相交的握手](https://leetcode-cn.com/problems/handshakes-that-dont-cross/)

- 标签：数学，有Codeforces的味道了
- 分析：可以从第一对握手人入手，假设$1$与$i$握手，则其左侧有$n-i$个人，右侧有$i-2$个人，假设$dp(i)$表示$i$个人相互握手不相交的方案数，则这种情况共有$dp(n-i) \times dp(i-2)$种方案，而事实上$1$可以和所有编号为奇数的人握手
- 状态转移方程

$$
dp(i)=\sum_{j=1}^{i/2} dp(2j-2) \cdot dp(i-2j)
$$

- 代码

```python
class Solution:
    def numberOfWays(self, n: int) -> int:
        dp = [0] * (n + 1)
        MOD = 10 ** 9 + 7
        for i in range(0, n + 1, 2):
            if i == 0:
                dp[i] = 1
            else:
                for j in range(0, i - 2 + 1, 2):
                    dp[i] += dp[j] * dp[i - j - 2]
                    dp[i] %= MOD
        return dp[n]
```



## [1478. 安排邮筒](https://leetcode-cn.com/problems/allocate-mailboxes/)

- 分析：$dp(i,j)$代表前$i$个房子可以安装$j$个邮筒时的最小距离和。状态转移时，考虑前$x$个房子由$j-1$个邮筒覆盖，最右边的$i-x$个房子 由新装设的邮筒覆盖，则状态转移方程为

$$
dp(i,j)=min_{x}\{dp(x,j-1)+get(x+1,i)\}
$$

- 其中$get(l,r)$表示当第$l$到第$r$个房子由一个邮筒覆盖时的最小距离和，此时邮筒必定安装在**中心房子所在的位置**，若房子总数为偶数，则安装在两间中心房子的任意一个都可以 

- 代码

```python
class Solution:
    def minDistance(self, houses: List[int], k: int) -> int:
        houses.sort()

        @lru_cache(None)
        def get(left, right):
            # find median
            pos = houses[(left + right) // 2]
            r = 0
            for i in range(left, right + 1):
                r += abs(houses[i] - pos)
            return r


        n = len(houses)
        dp = [[-1] * (k + 1) for _ in range(n)]
        for i in range(n):
            for j in range(1, k + 1):
                if j == 1:
                    dp[i][j] = get(0, i)
                else:
                    for x in range(i):
                        val = dp[x][j - 1] + get(x + 1, i)
                        if dp[i][j] == -1 or val < dp[i][j]:
                            dp[i][j] = val
        return dp[-1][-1]
```

# 最长上升子序列

## [435. 无重叠区间](https://leetcode-cn.com/problems/non-overlapping-intervals/)

- 标签：最长上升子序列
- 分析：本题可视为最长递增子序列的变种, dp[i]表示包含i个区间的最小右端点值
- 代码

```python
class Solution:
    def eraseOverlapIntervals(self, intervals: List[List[int]]) -> int:
        dp = [float('inf')] * (len(intervals) + 1)
        intervals.sort()
        res = 0
        for i in range(len(intervals)):
            L = 0
            R = len(intervals) - 1
            while L <= R:
                mid = (L + R) // 2
                if dp[mid] > intervals[i][0]:
                    R = mid - 1
                else:
                    L = mid + 1
            if dp[R + 1] > intervals[i][1]:
                dp[R + 1] = intervals[i][1]
            res = max(res, R + 2)
        return len(intervals) - res
```



# 最长回文子序列及其变式

## [1682. 最长回文子序列 II](https://leetcode-cn.com/problems/longest-palindromic-subsequence-ii/)

- 分析：$dp(i,j,k)$表示`s[i-j]`中两端字符为k的最长回文子序列
- 时间复杂度：$O(n^2k^2)$，其中$k=26$代表字母数量

- 代码

```python
class Solution:
    def longestPalindromeSubseq(self, s: str) -> int:
        n = len(s)
        dp = [[[0] * 26 for _ in range(n)] for _ in range(n)]

        for l in range(n - 1, -1, -1):
            for r in range(l + 1, n):
                for k in range(26):
                    if s[l] == s[r] and l != r and ord(s[l]) - ord('a') == k:
                        if l + 1 == r:
                            dp[l][r][k] = 2
                        else:
                            for x in range(26):
                                if x != k:
                                    dp[l][r][k] = max(dp[l][r][k], 2 + dp[l + 1][r - 1][x])
                    else:
                        dp[l][r][k] = max(dp[l + 1][r][k], dp[l][r - 1][k])
        return max(dp[0][-1])
```



## [1216. 验证回文字符串 III](https://leetcode-cn.com/problems/valid-palindrome-iii/)

- 标签：最长回文子序列
- 分析：求出最长回文子序列，若其长度大于$n-k$则返回true
- 代码

```python
class Solution:
    def isValidPalindrome(self, s: str, k: int) -> bool:
        n = len(s)
        dp = [[0] * n for _ in range(n)]
        for l in range(n - 1, -1, -1):
            for r in range(l, n):
                if s[l] == s[r]:
                    if l == r:
                        dp[l][r] = 1
                    elif l + 1 == r:
                        dp[l][r] = 2
                    else:
                        dp[l][r] = dp[l + 1][r - 1] + 2
                else:
                    dp[l][r] = max(dp[l][r - 1], dp[l + 1][r])
        return n - dp[0][-1] <= k
```



# 最长公共子序列及其变式

## [1092. 最短公共超序列](https://leetcode-cn.com/problems/shortest-common-supersequence/)

- 标签：最长公共子序列
- 分析：找到最长公共子序列，然后双指针遍历两个字符串，若当前字符不等于公共子序列当前字符，则将其加入$r$并将相应指针前移，直至两个字符串的当前字符都等于公共子序列的当前字符，此时将公共子序列当前字符加入$r$，双指针都前移

- 代码

```python
class Solution:
    def shortestCommonSupersequence(self, str1: str, str2: str) -> str:
        
        m = len(str1)
        n = len(str2)
        dp = [[''] * n for _ in range(m)]
        for i in range(m):
            for j in range(n):
                if str1[i] == str2[j]:
                    if i > 0 and j > 0:
                        dp[i][j] = dp[i - 1][j - 1] + str1[i]
                    else:
                        dp[i][j] = str1[i]
                else:
                    if i == 0 and j == 0:
                        dp[i][j] = ""
                    elif i == 0:
                        dp[i][j] = dp[i][j - 1]
                    elif j == 0:
                        dp[i][j] = dp[i - 1][j]
                    else:
                        if len(dp[i][j - 1]) >= len(dp[i - 1][j]):
                            dp[i][j] = dp[i][j - 1]
                        else:
                            dp[i][j] = dp[i - 1][j]

        i = j = 0
        r = ''
        s = dp[-1][-1]
        for k in range(len(s)):
            while i < len(str1) and str1[i] != s[k]:
                r += str1[i]
                i += 1
            while j < len(str2) and str2[j] != s[k]:
                r += str2[j]
                j += 1
            r += s[k]
            i += 1
            j += 1
        while i < len(str1):
            r += str1[i]
            i += 1
        while j < len(str2):
            r += str2[j]
            j += 1
        return r
```



# Rabin-Karp

## [1062. 最长重复子串](https://leetcode-cn.com/problems/longest-repeating-substring/)

- 标签：二分查找 + 滚动哈希
- 时间复杂度：$O(NlogN)$
- 分析：最长重复子串的长度满足**单调性**规律，假设存在长度为$L$的重复子串，则必定存在长度为$0 \leq l \leq L$的重复子串，因此可以使用二分查找
- 代码

```python
class Solution:
    def longestRepeatingSubstring(self, s: str) -> int:
        def check(L):
            data = {}
            base = 31
            mod = 10 ** 9 + 7
            H = pow(base, L, mod)
            val = 0
            for i in range(len(s)):
                val = val * base + ord(s[i]) - ord('a')
                if i >= L:
                    val -= H * (ord(s[i - L]) - ord('a'))
                    val %= mod
                if val in data:
                    for j in data[val]:
                        if s[j - L + 1: j + 1] == s[i - L + 1: i + 1]:
                            return True
                if i - L + 1 >= 0:
                    if val in data:
                        data[val].add(i)
                    else:
                        data[val] = {i}
            return False

        L = 0
        R = len(s) - 1
        while L <= R:
            mid = (L + R) // 2
            if check(mid):
                L = mid + 1
            else:
                R = mid - 1
        return R 
```



# 字符串的Next数组（有限确定状态机）

## [727. 最小窗口子序列](https://leetcode-cn.com/problems/minimum-window-subsequence/)

- 时间复杂度：$O(nk + nc)$，其中$c=26$是小写字母的数量
- 分析
  - Next数组的构建
  - 基于Next数组的状态转移
- 代码

```python
class Solution:
    def minWindow(self, s1: str, s2: str) -> str:        
        n = len(s1)
        dp = [[-1] * 26 for _ in range(n)]
        for i in range(n - 1, -1, -1):
            if i == n - 1:
                dp[i][ord(s1[i]) - ord('a')] = i
            else:
                for j in range(26):
                    if ord(s1[i]) - ord('a') == j:
                        dp[i][j] = i
                    else:
                        dp[i][j] = dp[i + 1][j]

        start = i = 0
        j = 0
        r = ""
        while i < len(s1):
            i = dp[i][ord(s2[j]) - ord('a')]
            if i == -1:
                break
            if j == 0:
                start = i
            j += 1
            if j == len(s2):
                if r == "" or i - start + 1 < len(r):
                    r = s1[start: i + 1]
                i = start + 1
                j = 0
            else:
                i += 1
        return r
```



# 字典树

## [140. 单词拆分 II](https://leetcode-cn.com/problems/word-break-ii/)

- 标签：字典树

- 代码

```python
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> List[str]:
        # build up the dict tree with words in wordDict
        tree = {}
        for w in wordDict:
            if not w:
                continue
            t = tree
            for c in w:
                if c not in t:
                    t[c] = {}
                t = t[c]
            else:
                t['.'] = '.'
        
        # develop DFS on the dict tree
        def dfs(i, t, path):
            if i == len(s):
                if '.' in t:
                    self.res.append(path)
                return

            if '.' in t:    
                dfs(i, tree, path + " ")
            
            if s[i] in t:
                dfs(i + 1, t[s[i]], path + s[i])
            
        
        self.res = []
        dfs(0, tree, "")    
        return self.res
```



# DP中状态矩阵的定义

## [1463. 摘樱桃 II](https://leetcode-cn.com/problems/cherry-pickup-ii/)

- 分析：由于两个机器人所处的行始终是相同的，因此$dp(i,j,k)$定义为机器人均位于第$i$行，且机器人1位于第$j$列、机器人2位于第$k$列时所能摘得的最多樱桃数量

- 时间复杂度：$O(rc^2)$

- 代码

```python
class Solution:
    def cherryPickup(self, grid: List[List[int]]) -> int:
        r = len(grid)
        c = len(grid[0])
        dp = [[[-1] * c for _ in range(c)] for _ in range(r)]
        dp[0][0][c - 1] = grid[0][0] + grid[0][c - 1]
        res = 0
        for i in range(r):
            for j in range(c):
                for k in range(c):
                    if dp[i][j][k] >= 0:
                        if i + 1 < r:
                            for j1 in [j-1, j, j+1]:
                                # 提早退出, 可以大幅减少运行时间
                                if j1 < 0 or j1 >= c:
                                    continue
                                for k1 in [k-1, k, k+1]:
                                    if k1 < 0 or k1 >= c:
                                        continue
                                    if j1 == k1:
                                        dp[i + 1][j1][k1] = max(dp[i + 1][j1][k1], dp[i][j][k] + grid[i + 1][k1])
                                    else:
                                        dp[i + 1][j1][k1] = max(dp[i + 1][j1][k1], dp[i][j][k] + grid[i + 1][j1] + grid[i + 1][k1])
                        else:
                            res = max(res, dp[i][j][k])
        return res
```

## [741. 摘樱桃](https://leetcode-cn.com/problems/cherry-pickup/)

- 分析：这道题能否做出取决于能否想到这个巧妙的状态定义方法。先完成`摘樱桃II`能够帮助思考
  - 将**一来一回**，视作**两个人**同时从$(0,0)$出发并走向$(N-1,N-1)$，两个人每次同时行动，其与$(0,0)$的哈明顿距离必定相等
  - $dp(i,r_1,r_2)$视作第$i$次行动时，第一个人处于第$r_1$行、第二个人处于第$r_2$行时能摘得的最多樱桃数量，由于此时两人的行动位移都为$i$，因此第一人所处列为$i-r_1$，第二人所处列为$i-r_2$
- 代码

```python
class Solution:
    def cherryPickup(self, grid: List[List[int]]) -> int:
        n = len(grid)
        dp = [[[-1] * n for _ in range(n)] for _ in range(2 * n - 1)]
        dp[0][0][0] = grid[0][0]
        for i in range(2 * n - 2): 
            for r1 in range(n):
                for r2 in range(n):
                    if dp[i][r1][r2] >= 0:
                        for r1_ in [r1, r1 + 1]:
                            c1_ = i + 1 - r1_
                            if not (0 <= r1_ < n and 0 <= c1_ < n) or grid[r1_][c1_] < 0:
                                continue
                            for r2_ in [r2, r2 + 1]:
                                c2_ = i + 1 - r2_
                                if not (0 <= r2_ < n and 0 <= c2_ < n) or grid[r2_][c2_] < 0:
                                    continue
                                if r1_ == r2_:
                                    dp[i + 1][r1_][r2_] = max(dp[i + 1][r1_][r2_], dp[i][r1][r2] + grid[r1_][c1_])
                                else:
                                    dp[i + 1][r1_][r2_] = max(dp[i + 1][r1_][r2_], dp[i][r1][r2] + grid[r1_][c1_] + grid[r2_][c2_])
        return max(0, dp[-1][-1][-1])
```



# 字符串匹配

## [10. 正则表达式匹配](https://leetcode-cn.com/problems/regular-expression-matching/)

- 分析：关键在于特殊字符（`.`和`*`）的处理

```python
class Solution:
    def isMatch(self, s: str, p: str) -> bool:
        dp = [[0] * (len(s) + 1) for _ in range(len(p) + 1)]
        dp[0][0] = 1
        for i in range(len(p)):
            if p[i] == '*':
                continue
            elif p[i] == '.':
                for j in range(len(s)):
                    if dp[i][j]:
                        dp[i + 1][j + 1] = 1
                if i + 1 < len(p) and p[i + 1] == '*':
                    for j in range(len(s) + 1):
                        if dp[i][j]:
                            for k in range(j, len(s) + 1):
                                dp[i + 2][k] = 1
                            break
            elif 'a' <= p[i] <= 'z':
                for j in range(len(s)):
                    if dp[i][j]:
                        if p[i] == s[j]:
                            dp[i + 1][j + 1] = 1
                if i + 1 < len(p) and p[i + 1] == '*':
                    for j in range(len(s) + 1):
                        if dp[i][j]:
                            dp[i + 2][j] = 1
                            for k in range(j, len(s)):
                                if p[i] == s[k]:
                                    dp[i + 2][k + 1] = 1
                                else:
                                    break
        return dp[-1][-1] == 1
```

## [44. 通配符匹配](https://leetcode-cn.com/problems/wildcard-matching/)

- 分析：关键在于特殊字符（`?`和`*`）的处理

```python
class Solution:
    def isMatch(self, s: str, p: str) -> bool:
        dp = [[0] * (len(s) + 1) for _ in range(len(p) + 1)]
        dp[0][0] = 1
        for i in range(len(p)):
            for j in range(len(s) + 1):
                if dp[i][j]:
                    if p[i] == '?':
                        if j + 1 < len(s) + 1:
                            dp[i + 1][j + 1] = 1
                    elif p[i] == '*':
                        for k in range(j, len(s) + 1):
                            dp[i + 1][k] = 1
                    else:
                        if j < len(s) and p[i] == s[j]:
                            dp[i + 1][j + 1] = 1
        return dp[-1][-1] == 1
```



# 其他问题

## [115. 不同的子序列](https://leetcode-cn.com/problems/distinct-subsequences/)

- 分析：$dp(i,j)$表示`s[: i]`中`t[: j]`出现的次数，状态转移时，$dp(i-1,j)$代表`s[: i-1]`中`t[: j]`出现的次数，必定也包含在$dp(i,j)$中。此外，当`s[i]==t[j]`时，则$dp(i-1,j-1)$也包含在其中，因此

$$
dp(i,j)=\left\{
\begin{aligned}
&dp(i-1,j-1)+dp(i-1,j) \quad &s[i]==t[j] \\
&dp(i-1,j) \quad &s[i]!= t[j]
\end{aligned}
\right.
$$

- 代码

```python
class Solution:
    def numDistinct(self, s: str, t: str) -> int:
        dp = [[0] * (len(t) + 1) for _ in range(len(s) + 1)]
        for i in range(len(s) + 1):
            dp[i][0] = 1
        for i in range(len(s)):
            for j in range(len(t)):
                if s[i] == t[j]:
                    dp[i + 1][j + 1] += dp[i][j]
                dp[i + 1][j + 1] += dp[i][j + 1]
        return dp[-1][-1] 
```



## [452. 用最少数量的箭引爆气球](https://leetcode-cn.com/problems/minimum-number-of-arrows-to-burst-balloons/)

- 标签：贪心
- 分析：将气球按起点排序，从左到右遍历并记录当前的射箭位置$pos$
  - 若气球的起点$l_i$大于当前射箭位置$pos$，则需要增加一支箭，初始位置定在该气球的右端点$r_i$
  - 若气球的起点$l_j$小于等于射箭位置$pos$，但终点$r_j$小于射箭位置，则需要调整射箭位置至其终点$r_j$**（该调整不会影响之前的气球，即之前的气球仍然能被击中）**
- 代码

```python
class Solution:
    def findMinArrowShots(self, points: List[List[int]]) -> int:
        # greedy
        points.sort()
        res = 0
        right = float('-inf')
        for x, y in points:
            if x > right:
                res += 1
                right = y
            elif y < right:
                right = y
        return res
```



## [72. 编辑距离](https://leetcode-cn.com/problems/edit-distance/)

- 分析：$dp(i, j)$表示将`word1[: i]`变为与`word2[: j]`相等所需要的最小操作次数，当`word1[i] == word2[j]`时，可直接退化为$dp(i-1,j-1)$的情况；不相等时，则考虑以下三种情况

  - 修改使得`word1[i] == word2[j]`，对应$dp(i-1,j-1)+1$
  - 删除`word1[i]`，对应$dp(i-1,j)+1$
  - 删除`word2[j]`，对应$dp(i,j-1)+1$

  状态转移方程如下

$$
dp(i,j)=\left\{
\begin{aligned}
&dp(i-1,j-1) \quad &word1[i]=word2[j] \\
&min\{dp(i-1,j-1), \ dp(i-1,j), \ dp(i, j-1) \}+1 \quad &word1[i] \neq word2[j]
\end{aligned}
\right.
$$

- 代码

```python
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        dp = [[0] * (len(word2) + 1) for _ in range(len(word1) + 1)]
        for i in range(len(word1) + 1):
            dp[i][0] = i
        for j in range(len(word2) + 1):
            dp[0][j] = j
        for i in range(len(word1)):
            for j in range(len(word2)):
                if word1[i] == word2[j]:
                    dp[i + 1][j + 1] = dp[i][j]
                else:
                    dp[i + 1][j + 1] = min(dp[i + 1][j], dp[i][j + 1], dp[i][j]) + 1
        return dp[-1][-1]
```



# 较复杂的模拟问题

## [418. 屏幕可显示句子的数量](https://leetcode-cn.com/problems/sentence-screen-fitting/)

- 标签：模拟、预处理
- 分析：本题的$rows$和$cols$变量数据范围较大，若直接模拟会超时，可先预处理两个数组
  - $next(i)$表示当本行是从`sentence[i]`开始显示的，下一行显示的第一个单词的索引
  - $cnt(i)$表示当本行是从`sentence[i]`开始显示的，这一行显示的句子数量（这里的数量等同于**显示句子最后一个单词的次数**）
- 代码

```c++
class Solution {
public:
    int wordsTyping(vector<string>& sentence, int rows, int cols) {
        int n = (int)sentence.size();
        vector<int> next(n, 0);
        vector<int> cnt(n, 0);
        int l = 0;
        for (int i = 0; i < n; i++)
            l += (int)sentence[i].size();
        l += n - 1;
        int w = cols;
        cnt[0] += w / (l + 1);
        w %= l + 1;
        for (int i = 0; i < n; i++) {
            if (w >= (int)sentence[i].size())
                w -= (int)sentence[i].size() + 1;
            else {
                next[0] = i;
                break;
            }
            if (i == n - 1)
                cnt[0] += 1;
        }
        for (int i = 1; i < sentence.size(); i++) {
            cnt[i] = cnt[i - 1];
            w += (int)sentence[i - 1].size() + 1;
            int j = next[i - 1];
            while (w >= (int)sentence[j].size()) {
                w -= (int)sentence[j].size() + 1;
                j += 1;
                if (j == n) {
                    cnt[i] += 1;
                    j = 0;
                }
            }
            next[i] = j;
        }
        int right = 0;
        int r = 0;
        for (int i = 0; i < rows; i++) {
            r += cnt[right];
            right = next[right];
            
        }
        return r;
    }
};
```

