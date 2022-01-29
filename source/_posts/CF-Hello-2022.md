---
title: CF Hello 2022
date: 2022-01-04 01:02:38
tags: Codeforces
categories: Algorithm
mathjax: true 
---

# 总结

- 本次题目难度较高，B题就卡了许久，C题有思路但是也没做出来，属于是信心打击器了属于是

# [A. Stable Arrangement of Rooks](https://codeforces.com/contest/1621/problem/A)

- 标签：字符串
- 分析：可知棋盘中最多容纳$(n+1)/2$个棋子
- 代码

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <set>
using namespace std;


void solve(void) {
    int n, k;
    cin >> n >> k;
    if (k > (n + 1) / 2) {
        cout << -1 << endl;
        return;
    } 
    int j = 0;
    for (int i = 0; i < n; i++) {
        if (j < k && i % 2 == 0) {
            for (int x = 0; x < i; x++)
                cout << ".";
            cout << "R";
            for (int x = 0; x < n - 1 - i; x++)
                cout << ".";
            cout << endl;
            j++;
        } else {
            for (int x = 0; x < n; x++)
                cout << ".";
            cout << endl;
        }
    }
}

int main(void) {
    int t;
    cin >> t;
    while (t--) {
        solve();
    }
    
}
```

<!--more-->

# [B. Integers Shop](https://codeforces.com/contest/1621/problem/B)

- 标签：贪心
- 分析：按顺序遍历，记录当前最大值$maxVal$、最小值$minVal$，以及最大值对应的花费$maxC$、最小值对应的花费$minC$，并考虑一种特殊情况，加入某个区间兼有最大值和最小值，则其花费为

$$
C=maxC=minC
$$

- 一般情况下花费如下，需要对比两者中取更小的即可

$$
C=maxC+minC
$$

- 代码

```c++
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;
typedef long long ll;


void solve(void) {
    int n;
    cin >> n;
    int minVal = -1, maxVal = -1;
    ll minc = 0, maxc = 0;
    ll val = -1;
    for (int i = 0; i < n; i++) {
        int l, r;
        ll c;
        cin >> l >> r >> c;
        if (minVal == -1 || l < minVal) {
            minVal = l;
            minc = c;
            val = -1;
        }
        if (maxVal == -1 || r > maxVal) {
            maxVal = r;
            maxc = c;
            val = -1;
        }
        if (minVal == l) {
            minc = min(minc, c);
        }
        if (maxVal == r) {
            maxc = min(maxc, c);
        }
        if (minVal == l && maxVal == r) {
            if (val == -1 || c < val)
                val = c;
        }
        if (val != -1 && val < minc + maxc)
            cout << val << endl;
        else
            cout << minc + maxc << endl;
    }
}

int main(void) {
    int t;
    cin >> t;
    while (t--) {
        solve();
    }
    
}
```

# [C. Hidden Permutations](https://codeforces.com/contest/1621/problem/C)

- 标签：环、数学
- 分析
  - 该排列必定由1个或多个环组成，不断获取同一位置的元素，即可得到环中的所有元素，当获取到重复元素时候可以停止，此时代表我们已经遍历了整个环
  - 然后需要还原排列，假设得到的元素为`arr=[5,4,3]`，则当`0<=i<len(arr)-1`时有`p[arr[i]]=arr[i+1]`，当`i==len(arr)-1`时有`p[arr[i]]=arr[0]`，由此可还原原排列
- 代码

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <set>
using namespace std;
typedef long long ll;


void solve(void) {
    int n;
    cin >> n;
    vector<int> p(n, 0);
    set<int> a{};
    for (int i = 1; i <= n; i++) {
        a.insert(i);
    }
    int k = 0;
    while (a.size()) {
        vector<int> arr{};
        set<int> vis{};
        int j = *a.begin();
        int last_k = k;
        while (true) {
            cout << "?" << " " << j << endl;
            k++;
            int t;
            cin >> t;
            if (vis.find(t) == vis.end()) {
                arr.push_back(t);
                vis.insert(t);
            } else {
                break;
            }
        }
        for (int i = 0; i < arr.size(); i++) {
            a.erase(arr[i]);
            if (i + 1 < arr.size())
                p[arr[i] - 1] = arr[i + 1];
            else
                p[arr[i] - 1] = arr[0];
        }
        
    }
    cout << "! ";
    for (int i = 0; i < n; i++) 
        cout << p[i] << " ";
    cout << endl;
}

int main(void) {
    int t;
    cin >> t;
    while (t--) {
        solve();
    }
    
}
```

