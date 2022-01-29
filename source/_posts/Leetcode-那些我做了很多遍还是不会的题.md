---
title: Leetcode 那些我做了很多遍还是不会的题
date: 2022-01-12 16:07:24
tags: Leetcode
mathjax: true
categories: Algorithm
---

# [1049. 最后一块石头的重量 II](https://leetcode-cn.com/problems/last-stone-weight-ii/)

- 总结：这道题的难点在于想到把问题转化为0-1背包。而对于第一次看题/第n次看题但却不记得这道题怎么做的人，能想到这个几乎是不可能的。
- 用归纳法可以证明，无论按照何种顺序粉碎石头，最后一块石头的重量总是可以表示成

$$
\sum_{i=0}^{n-1}k_i \cdot stones_i \quad k_i \in \{-1,1\}
$$

- 代码

```c++
class Solution {
public:
    int lastStoneWeightII(vector<int>& stones) {
        unordered_set<int> data;
        data.insert(0);
        for (auto &v: stones) {
            unordered_set<int> tmp;
            for (auto &u: data) {
                tmp.insert(v + u);
                tmp.insert(abs(v - u));
            }
            data = tmp;
        }
        int r = -1;
        for (auto &v: data) {
            if (r == -1 || v < r)
            r = v;
        }
        return r;

    }
};
```



# 栈上动态规划

## [32. Longest Valid Parentheses](https://leetcode-cn.com/problems/longest-valid-parentheses/)

- 标签：栈
- 分析
  - 从左到右遍历，并用栈维护当前的字符串，若为`(`则入栈，否则出栈
  - 若当前`s[i]`为`)`，则其可构成的最长有效字符串为`i-stack[-1]`
  - 为了方便计算，在遍历前将`-1`入栈，这样本来空栈代表我是从最左边开始，现在栈的长度为1则代表同样意思，但计算时仍可用`i-stack[-1]`表示
  - 特别地，假如出栈后栈为空，说明此时`)`导致了非法的字符串，因为在处理后栈大小为1就代表着本来的空栈，因此必须维护栈非空，此时我们需要将当前位置$i$从新入栈

- 代码

```c++
class Solution {
public:
    int longestValidParentheses(string s) {
        stack<int> stk;
        stk.push(-1);
        int r = 0;
        int n = (int) s.size();
        for (int i = 0; i < n; i++) {
            if (s[i] == '(')
                stk.push(i);
            else {
                if (stk.size() == 1) {
                    stk.pop();
                    stk.push(i);
                } else {
                    stk.pop();
                    r = max(r, i - stk.top());
                }

            }
        }
        return r;
    }
};
```



# [940. 不同的子序列 II](https://leetcode-cn.com/problems/distinct-subsequences-ii/)

- 状态转移方程：$dp(i)=2\times dp(i-1)-dp(prev(i)-1)$，其中$prev(i)$为与`s[i]`为同一字符的上一索引位置

- 代码

```c++
typedef long long ll;

class Solution {
public:
    const int mod = 1e9 + 7;
    int distinctSubseqII(string s) {
        int n = (int)s.size();
        vector<ll> dp(n);
        vector<int> prev(n, -1);
        vector<int> c(26, -1);
        for (int i = 0; i < n; i++) {
            prev[i] = c[s[i] - 'a'];
            c[s[i] - 'a'] = i;
        }
        for (int i = 0; i < n; i++) {
            if (i == 0)
                dp[i] = 2;
            else {
                dp[i] = 2 * dp[i - 1] % mod;
                if (prev[i] > 0) {
                    dp[i] -= dp[prev[i] - 1];
                    dp[i] = (dp[i] + mod) % mod;
                } else if (prev[i] == 0) {
                    dp[i] -= 1;
                    dp[i] = (dp[i] + mod) % mod;
                }
            }
        }
        return dp[n - 1] - 1;
    }
};
```



# [1866. 恰有 K 根木棍可以看到的排列数目](https://leetcode-cn.com/problems/number-of-ways-to-rearrange-sticks-with-k-sticks-visible/)

- 分析
  - $dp(i,j)$表示排列$i$根木棍使得恰有$j$根木棍可见的排列数目
  - 考虑最后一根木棍，若其可见则必为最大的木棍，此时前$i-1$根木棍只需$j-1$根可见；若其不可见则并非最大的木棍，长度有$i-1$种选择
  - 状态转移方程：$dp(i,j)=dp(i-1,j-1)+(i-1) \cdot dp(i-1,j)$
- 代码

```c++
typedef long long ll;
const int mod = 1e9 + 7;
class Solution {
public:

    int rearrangeSticks(int n, int k) {
        vector<vector<ll>> dp(n + 1, vector<ll> (k + 1));
        dp[0][0] = 1;
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= k; j++) {
                dp[i][j] = dp[i - 1][j - 1];
                ll tmp = (ll)(i - 1) * dp[i - 1][j] % mod;
                dp[i][j] += tmp;
                dp[i][j] %= mod;
            }
        }
        return dp[n][k];
    }
};
```



# 动态规划 + 差分

## [837. 新 21 点](https://leetcode-cn.com/problems/new-21-game/)

```c++
class Solution {
public:
    double new21Game(int n, int k, int maxPts) {
        if (k == 0)
            return 1;
        vector<double> dp(k + maxPts);
        dp[0] = 1;
        vector<double> add(k + maxPts);
        vector<double> minus(k + maxPts);
        double p = 0;
        for (int i = 0; i < k + maxPts; i++) {
            p += add[i] - minus[i];
            if (i > 0)
                dp[i] = p;
            if (i < k) {
                if (i + 1 < k + maxPts)
                    add[i + 1] += dp[i] / maxPts;
                if (i + maxPts + 1 < k + maxPts)
                    minus[i + maxPts + 1] += dp[i] / maxPts;
            }
        }
        double r = 0;
        for (int i = k; i < min(n + 1, k + maxPts); i++) {
            r += dp[i];
        }
        return r;

    }
};
```



#  难以想到的状态及转移方程

## [546. 移除盒子](https://leetcode-cn.com/problems/remove-boxes/)

- $dp(l,r,k)$表示当前区间为$[l,r]$且右侧还有$k$个元素与`boxes[r]`颜色相同时，能获得的最大分数

```c++
class Solution {
public:
    int memo[101][101][101];

    int dp(vector<int>& boxes, int l, int r, int k) {
        if (l < 0 || r < 0)
            return 0;
        if (memo[l][r][k] >= 0)
            return memo[l][r][k];
        if (l > r)
            return 0;
        int ans = 0;
        ans = dp(boxes, l, r - 1, 0) + (k + 1) * (k + 1);
        int tmp = 0;
        int right = r;
        for (int i = r - 1; i >= l; i--) {
            if (boxes[i] == boxes[r]) {
                ans = max(ans, dp(boxes, l, i, k + 1) + dp(boxes, i + 1, r - 1, 0));
            }
        }
        memo[l][r][k] = ans;
        return ans;
    }

    int removeBoxes(vector<int>& boxes) {
        memset(memo, -1, sizeof(memo));
        int ans = dp(boxes, 0, (int)boxes.size() - 1, 0);
        return ans;
    }
};
```

