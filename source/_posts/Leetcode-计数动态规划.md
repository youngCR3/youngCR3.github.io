---
title: Leetcode 计数动态规划
date: 2022-01-09 13:20:53
tags: Leetcode
mathjax: true
categories: Algorithm
---

# 斯特林数

## [1692. 计算分配糖果的不同方式](https://leetcode-cn.com/problems/count-ways-to-distribute-candies/)

- 分析：$dp(n,k)$表示将n个数划分为k个集合的分配方案，它可视作两个部分组成：
  - 第一部分是前面的$n-1$个数分为$k$个集合，第$n$个数加入$k$个集合中的一个
  - 第二部分是前面的$n-1$个数分为$k-1$个集合，第$n$个数单独组成第$k$个集合
- 状态转移方程

$$
dp(n,k)=k \cdot dp(n-1,k)+dp(n-1,k-1)
$$

- 代码

```c++
#include <map>
#include <utility>
typedef long long ll;

class Solution {
public:
    int waysToDistribute(int n, int k) {
        vector<vector<ll>> dp(n + 1, vector<ll>(k + 1, 0));
        const int mod = 1e9 + 7;
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= k; j++) {
                if (j == 1)
                    dp[i][j] = 1;
                else {
                    dp[i][j] = j * dp[i - 1][j] + dp[i - 1][j - 1];
                    dp[i][j] %= mod;
                }
            }
        }
        return dp[n][k];
    }
};
```

