---
title: CF Edu 118
date: 2022-01-08 17:27:36
tags: Codeforces
mathjax: true
categories: Algorithm
---



# [D. MEX Sequences](https://codeforces.com/contest/1613/problem/D)

- 标签：**计数动态规划**
- 分析
- 代码

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <set>
#include <unordered_set>
#include <unordered_map>
#include <queue>
using namespace std;
typedef long long ll;
const int mod = 998244353;

void solve(void) {
    int n;
    cin >> n;
    vector<int> a(n, 0);
    for (int i = 0; i < n; i++)
        cin >> a[i];
    vector<ll> dp1(n + 1, 0);
    vector<ll> dp2(n + 1, 0);
    vector<ll> dp3(n + 1, 0);
    
    ll r = 0;
    for (int i = 0; i < n; i++) {
        int x = a[i];
        ll x1 = dp1[x];
        if (x == 0)
            x1 += 1;
        if (x - 1 >= 0)
            x1 += dp1[x - 1];
        
        ll x2 = dp2[x];
        if (x == 1)
            x2 += 1;
        if (x - 2 >= 0)
            x2 += dp1[x - 2] + dp3[x - 2];
        
        ll x3 = dp3[x];
        if (x + 2 <= n)
            x3 += dp2[x + 2];
        
        x1 %= mod;
        x2 %= mod;
        x3 %= mod;
        dp1[x] += x1;
        dp2[x] += x2;
        dp3[x] += x3;
        dp1[x] %= mod;
        dp2[x] %= mod;
        dp3[x] %= mod;
        r += x1 + x2 + x3;
        r %= mod;
    }
    cout << r << endl;
}

int main(void) {
    int t;
    cin >> t;
    while (t--) {
        solve();
    }
}
```

