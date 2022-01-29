---
title: CF Edu 119
date: 2022-01-06 19:41:39
tags: Codeforces
categories: Algorithm
mathjax: true
---

# 总结

- 这次竞赛的题目质量较高，有几道题目很容易理解，但思路比较巧妙，或者存在难以发现的Corner case的题目。很多人都能对题目有所思考，但想真正做对又很体现水平
- 其中D题的corner case较难想到，是否能够用2枚面值2替代1枚面值1和1枚面值3？约束条件是什么？
- E题陷入了一种错误思路（并查集），查询之间的相互影响存在单向性，即某一时刻某个数的值只与其后发生的查询有关，与之前发生的查询无关
- C题是一道数学味道较强的题目，容易想到基本原理，但直接求解存在数据溢出问题，需要联想到进制，并通过不断取余和整除避免溢出问题

# [A. Equal or Not Equal](https://codeforces.com/contest/1620/problem/A)

- 分析：只有一种情况不符合，就是有且只有1个N时，此时某两个相邻的数不等，但其他相邻的数都相等，这是矛盾的

- 代码

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <set>
#include <deque>
using namespace std;
typedef long long ll;

void solve(void) {
    string s;
    cin >> s;
    int r = 0;
    for (auto &c: s) {
        if (c == 'N')
            r++;
    }
    if (r == 1)
        cout << "NO" << endl;
    else
        cout << "YES" << endl;
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

# [B. Triangles on a Rectangle](https://codeforces.com/contest/1620/problem/B)

- 分析：对于每条边，取其最小的点和最大的点之间的距离作为边，乘以宽或高即可得到面积，四条边分别计算面积并取最大值即可

- 代码

```python
#include <iostream>
#include <vector>
#include <algorithm>
#include <set>
#include <deque>
using namespace std;
typedef long long ll;

void solve(void) {
    ll w, h;
    cin >> w >> h;
    int k;
    ll r = 0;
    for (int i = 0; i < 4; i++) {
        cin >> k;
        ll left, right, t;
        for (int j = 0; j < k; j++) {
            if (j == 0) {
                cin >> left;
            } else if (j == k - 1) {
                cin >> right;
            } else {
                cin >> t;
            }
        }
        if (i < 2)
            r = max(r, (right - left) * h);
        else
            r = max(r, (right - left) * w);
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

# [C. BA-String](https://codeforces.com/contest/1620/problem/C)

- 标签：数学、进制
- 分析：对于连续的$x$个\*，最多可以添加$kx$个b，最少可以添加0个b，这相当于形成了$kx+1$个不同的字符串。假设原字符串一共有$n$段\*，且第$i$段一共有$c_i$个\*，则字符串数量为

$$
S=\prod_{i=1}^{n}(kc_i+1)
$$

- 我们逆序考虑并重新编号，假设从右到左的第一段是$c_0$，最后一段是$c_{n-1}$，则在考虑$i$时，其取值相当于是在$\prod_{j=0}^{i-1}(kc_j+1)$的基础上进行考虑的。若直接使用该公式求解，由于乘积结果过大可能导致溢出（超出long long的数据范围）。

## 防止溢出：类比数制

- 将该公式与数制对比，发现两者的相似之处，当我们确定第$i$段取多少个b时，只需要将$x$对$kc_i+1$取余即可，然后遍历到下一位前，再将$x$与$kc_i+1$做整数除法，这样避免了连乘导致的溢出

- 代码

```python
#include <iostream>
#include <vector>
#include <algorithm>
#include <set>
#include <deque>
using namespace std;
typedef long long ll;

void solve(void) {
    int n, k;
    ll x;
    cin >> n >> k >> x;
    x -= 1;
    string s;
    cin >> s;
    string r;
    int c = 0;
    for (int i = s.size() - 1; i >= 0; i--) {
        if (s[i] == 'a') {
            if (c) {
                int cur = c * k + 1;
                for (int j = 0; j < x % cur; j++) {
                    r += 'b';
                }
                x /= cur;
                c = 0;
            }
            r += s[i];
        } else {
            c++;
        }
    } 
    if (c) {
        int cur = c * k + 1;
        for (int j = 0; j < x % cur; j++) {
            r += 'b';
        }
        x /= cur;
        c = 0;
    }
    reverse(r.begin(), r.end());
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



# [D. Exact Change](https://codeforces.com/contest/1620/problem/D)

- 标签：贪心

- 分析：本题思路较容易想到，关键在于corner case是否能想到。首先总体思路是尽量先用面值为3的硬币，再用面值为1和2的硬币。我们需要用到面值为3的硬币的数量最大值`three`，以及它的余数`r`，若有多个商品使用同样多的面值3硬币，则余数需要都记录下来。此外，需要记录各个商品对3的余数，若有余数为1则设置`one`，若有余数为2则设置`two`

  - 假设所有商品都能被3整除，直接返回`three`即可

  - 假设所有商品的余数都为0或1，则只需要在`three`个面值3的基础上，增加一个面值1，返回`three+1`

  - 假设所有商品的余数都为0或2，则只需要在`three`个面值3的基础上，增加一个面值2，返回`three+1`

  - 假设余数包含0、1、2，则需要考虑使用最多面值3的商品的余数`r`

    - 余数为2，此时我们必须使用`three`个面值3，1个面值2、1个面值1，返回`three+2`

    - 余数为1，此时我们可能将“1个面值1和1个面值3”用“2个面值2”代替，从而只使用`three-1`个面值3和2个面值2，不使用面值1，从而减小硬币数量，但需要满足以下两个条件

      1）**商品中不包含1**，若包含，则我们必须携带面值1

      2）余数`r`中不包含0，若包含，则我们必须携带`three`个面值3

    - 只有满足上述两个条件，我们才能只带`three+1`个硬币，否则还是要带`three+2`个硬币（three个面值3，1个面值1，1个面值2）

- 代码

```python
#include <iostream>
#include <vector>
#include <algorithm>
#include <set>
#include <unordered_set>
#include <deque>
using namespace std;
typedef long long ll;


void solve(void) {
    int n;
    cin >> n;
    vector<int> a(n, 0);
    bool flag = false;
    for (int i = 0; i < n; i++) { 
        cin >> a[i];
        if (a[i] == 1)
            flag = true;
    }
    
    int one = 0, two = 0, three = 0;
    unordered_set<int> r{};
    for (int i = 0; i < n; i++) {
        if (a[i] % 3 == 1)
            one = 1;
        else if (a[i] % 3 == 2)
            two = 1;
        if (a[i] / 3 > three) {
            three = a[i] / 3;
            r.clear();
            r.insert(a[i] % 3);
        } else if (a[i] / 3 == three)
            r.insert(a[i] % 3);
    }
    int res = 0;
    if (one == 0 && two == 0) {
        res = three;
    } else if (one == 0 || two == 0) {
        res = three + 1;
    } else {
        res = three + 1;
        if (r.find(2) != r.end())
            res++;
        else if (r.find(1) != r.end()) {
            // try to replace 1, 3 with 2, 2
            if (flag || r.find(0) != r.end())
                res++;
        }
        
    }
    cout << res << endl;
}

int main(void) {
    int t;
    cin >> t;
    while (t--) {
        solve();
    }
}
```



# [E. Replace the Numbers](https://codeforces.com/contest/1620/problem/E)

- 分析
  - 某个数最终的数值只与其后面发生的赋值操作有关，因此倒序处理所有查询
  - 使用`next`数组记录某个数最终会变为的值，对于操作2，只需将`next[y]`赋值给`next[x]`即可，`next[x]=next[y]`
  - 若查询时发生某个数`x`不在`next`中，则初始化`next[x]=x`

- 代码

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <set>
#include <unordered_set>
#include <unordered_map>
#include <utility>
using namespace std;
typedef long long ll;

void solve(void) {
    vector<int> arr{};
    unordered_map<int, int> next;
    vector<pair<int, int>> quries{};
    int q;
    cin >> q;
    int r = 0;
    while (q--) {
        int flag;
        cin >> flag;
        if (flag == 1) {
            int x;
            cin >> x;
            quries.push_back(make_pair(x, -1));
        } else {
            int x, y;
            cin >> x >> y;
            quries.push_back(make_pair(x, y));
        }
    }
    for (int i = quries.size() - 1; i >= 0; i--) {
        if (quries[i].second == -1) {
            int x = quries[i].first;
            if (next.find(x) == next.end())
                next[x] = x;
            arr.push_back(next[x]);
        } else {
            int x = quries[i].first;
            int y = quries[i].second;
            if (next.find(y) == next.end())
                next[y] = y;
            next[x] = next[y];
        }
    }
    for (int i = arr.size() - 1; i >= 0; i--) 
        cout << arr[i] << " ";
    cout << endl;
    
}

int main(void) {
    solve();
}
```

## 错误思路：并查集

- 本题可能会错误地认为应该使用并查集，考虑以下情况

  ```
  2 1 3
  2 2 1
  ```

  - 1的最终值为3，2的最终值为1，若使用并查集处理则会导致2的最终值为3
  - 并查集的错误之处，在于它没有遵循“**所有数的值只会受到其之后进行的查询的影响**”这一规则，上述例子中2的值受到了其之前发生的查询`2 1 3`的影响
