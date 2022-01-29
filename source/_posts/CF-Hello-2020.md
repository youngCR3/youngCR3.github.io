---
title: CF Hello 2020
date: 2022-01-03 18:30:48
tags: Codeforces
categories: Algorithm
mathjax: true
---

# 总结

- 常规时间内做出了前三题，其中第三题由于开始时思路出错卡了很久，最终发现正确思路后很顺利地就做出来了。第四题有一些思路，并在规定时间内实现并提交了代码，但该思路有缺陷，看了题解后也并未能够真正掌握该题目的数学原理，只是对着题解实现了一遍算法
- 总的来说，Hello 2020的难度似乎略高于一般的Div2竞赛，第三题需要根据规律推导数学公式，第四题的数学原理较为复杂

# [A. New Year and Naming](https://codeforces.com/contest/1284/problem/A)

- 标签：字符串
- 分析：根据取余数的规则分别求得两个字符串，然后拼接即可
- 代码

```c++
#include <iostream>
#include <vector>
#include <string>
using namespace std;

void solve(void) {
    int n, m;
    cin >> n >> m;
    vector<string> s(n, " ");
    vector<string> t(m, " ");
    for (int i = 0; i < n; i++)
        cin >> s[i];
    for (int i = 0; i < m; i++)
        cin >> t[i];
    int q;
    cin >> q;
    while (q--) {
        int year;
        cin >> year;
        cout << s[(year + (n - 1)) % n] << t[(year + (m - 1)) % m] << endl;
    }
    
}

int main(void) {
    solve();
}
```

<!--more-->

# [B. New Year and Ascent Sequence](https://codeforces.com/contest/1284/problem/B)

- 标签：排序、双指针、前缀和
- 分析
  - 假如一个序列本身就存在ascent，那么它和任意序列拼接后都是有效答案
  - 假如一个序列的最小值，小于另一个序列的最大值，那么将它们按顺序拼接后是有效答案
- 根据以上两点规则，可先求得各个序列的最小值、最大值，分别存储在两个数组中，并对其排序。然后采用双指针`i`、`j`从小到大遍历，对于每个最小值数组的元素，
  - 如果它代表的序列本身就有ascent，则可以直接加上`n`
  - 找到大于它的最小最大值元素，则该元素之后的所有元素代表的序列都可以与当前的最小值元素代表的序列构成有效答案，即加上`n-j`
  - 还需要考虑一种特殊情况，即虽然最大值元素小于等于当前最小值元素，但是最大值元素代表的序列本身存在ascent，那么也是可以构成有效答案的，因此采用一个遍历`prefix`代表小于等于当前最小值元素的最大值所代表序列中拥有ascent的序列数量，每次遍历时都加上`prefix`

- 代码

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
using namespace std;
typedef long long ll;


void solve(void) {
    int n;
    cin >> n;
    vector<vector<int>> minVal(n, vector<int>(2, 0));
    vector<vector<int>> maxVal(n, vector<int>(2, 0));
    vector<int> valid(n, 0);
    for (int i = 0; i < n; i++) {
        int m;
        cin >> m;
        int min = -1, max = -1;
        for (int j = 0; j < m; j++) {
            int tmp;
            cin >> tmp;
            if (min == -1 || tmp < min)
                min = tmp;
            if (max == -1 || tmp > max)
                max = tmp;
            if (min != -1 && tmp > min) {
                valid[i] = 1;
            }
        }
        minVal[i][0] = min;
        maxVal[i][0] = max;
        minVal[i][1] = i;
        maxVal[i][1] = i;

    }
    sort(minVal.begin(), minVal.end());
    sort(maxVal.begin(), maxVal.end());
    vector<int> prefix(n, 0);
    for (int i = 0; i < n; i++) {
        if (valid[maxVal[i][1]]) {
            if (i == 0)
                prefix[i] = 1;
            else
                prefix[i] = prefix[i - 1] + 1;
        } else {
            if (i > 0)
                prefix[i] = prefix[i - 1];
        }
    }
    
    ll res = 0;
    int j = 0;
    for (int i = 0; i < n; i++) {
        if (valid[minVal[i][1]]) {
            res += n;
        } else {
            while (j < n && maxVal[j][0] <= minVal[i][0])
                j++;
            res += n - j;
            if (j > 0)
                res += prefix[j - 1];
        }
    }
    cout << res << endl;
    
}

int main(void) {
    solve();
}
```

# [C. New Year and Permutation](https://codeforces.com/contest/1284/problem/C)

- 标签：数学
- 思路：本题我一开始的思考是利用动态规划，假如我已经知道$n$个数排列的happiness之和，能否求出$n+1$个数排列的happiness之和。在推导的过程中，以外发现可以直接写出$happiness(n)$的表达式，具体地

$$
happiness(n) = \sum_{i=1}^{n} i \cdot (n-i+1)! \cdot i!
$$

- 分析：该公式可以这样理解，考虑其中的某个片段的最大值是$r$，最小值是$l$，则该片段必定包含$r-l+1$个数，此外还剩下$n-(r-l+1)$个数，该片段本身的排列方式总共有$(r-l+1)!$种，而将该片段设为一个整体，与剩下的数的排列方式共有$(n-(r-l+1)+1)!$种，两者相乘 即为$(r-l+1)! \cdot (n-(r-l))!$种。根据以上规律，写出前几个数的$happiness$表达式，观察并总结即可得到上述公式
- 代码

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
using namespace std;
typedef long long ll;
ll Fac[250002];

ll fac(ll x, ll mod) {
    if (Fac[x] >= 0)
        return Fac[x];
    if (x == 0) {
        Fac[x] = 1;
    } else {
        Fac[x] = x * fac(x - 1, mod) % mod;
    }
    return Fac[x];
}

void solve(void) {
    ll n, m;
    cin >> n >> m;
    ll r = 0;
    for (ll i = 1; i <= n; i++) {
        r += i * fac(n - i + 1, m) % m * fac(i, m) % m;
        r %= m;
    }
    cout << r << endl;
}

int main(void) {
    for (int i = 0; i < 250002; i++)
        Fac[i] = -1;
    solve();
}
```

# [D. New Year and Conference](https://codeforces.com/contest/1284/problem/D)

- 标签：有序列表、扫描线法
- 分析：假如我们能够找到两节课，它们在两个场地的一个重叠、另一个不重叠，那么就说明存在venue-sensitive的课程集
- 实现：将场地a的各个课的开始、结束时间视为一个事件，对事件按时间顺序排列，当场地a的某个课开始时，需要将b中对应课程加入集合中，当场地b的某个课结束时，需要将b中对应课程移出集合。需要保证的是a中有重叠的课程，在b中也有重叠，该关系可由以下公式保证。此外，我们还需要确保在b中重叠的课程，在a中也有重叠，我们可直接调换`a`、`b`后再次调用`check`函数实现

$$
b_i \leq min(e) \quad and \quad e_i \geq max(b)
$$

- 代码

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <set>

using namespace std;

bool check(vector<vector<int>> &a1, vector<vector<int>> &b) {
    vector<vector<int>> a{};
    for (int i = 0; i < a1.size(); i++) {
        a.push_back(vector<int>{a1[i][0], 1, i});
        a.push_back(vector<int>{a1[i][1] + 1, 0, i});
    }
    sort(a.begin(), a.end());
    // sweep-line
    multiset<int> btime;
    multiset<int> etime;
    for (int i = 0; i < a.size(); i++) {
        if (a[i][1] == 1) {
            // add a lecture
            int min_e = -1, max_b = -1;
            if (btime.size()) {
                min_e = *etime.begin();
                max_b = *btime.rbegin();
            }
            if (min_e == -1 || (b[a[i][2]][0] <= min_e && b[a[i][2]][1] >= max_b)) {
                btime.insert(b[a[i][2]][0]);
                etime.insert(b[a[i][2]][1]);
            } else {
                return false;
            }
        } else {
            // delete a lecture
            btime.erase(btime.find(b[a[i][2]][0]));
            etime.erase(etime.find(b[a[i][2]][1]));
        }
    }
    return true;
}


void solve(void) {
    int n;
    cin >> n;
    vector<vector<int>> a(n, vector<int>(2, 0));
    vector<vector<int>> b(n, vector<int>(2, 0));
    for (int i = 0; i < n; i++) {
        int sa, ea, sb, eb;
        cin >> sa >> ea >> sb >> eb;
        a[i][0] = sa;
        a[i][1] = ea;
        b[i][0] = sb;
        b[i][1] = eb;
    }
    if (check(a, b) && check(b, a))
        cout << "YES" << endl;
    else
        cout << "NO" << endl;
}

int main(void) {
    solve();
}
```

