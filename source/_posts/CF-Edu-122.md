---
title: CF Edu 122
date: 2022-02-09 22:43:53
tags: Codeforces
categories: Algorithm
mathjax: true
---



# 总结

- 完成A-D题，但D题由于动态规划转移时的一个愚蠢错误，未能在比赛时AC



# [D. Make Them Equal](https://codeforces.com/contest/1633/problem/D)

- 标签：动态规划
- 分析
  - 由于$b_i\leq1000$，因此可以通过dp直接求得转换得到某个值$v$的最小操作次数$data(v)$
  - 可以发现，对于1000以内的数，最多的转换次数为12次，因此总的转换次数不会超过12000次，则$k\leq12000$，由此本题可以通过$O(nk)$的动态规划求解
  - 本题关键便在于通过分析求解操作次数，发现$k$的取值范围可以缩小，否则会被题目$k\leq 10^6$的数据范围所迷惑，误以为动态规划无法求解
- 代码

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <set>
#include <unordered_set>
#include <utility>
#include <unordered_map>
#include <queue>
#include <math.h>
using namespace std;
typedef long long ll;

vector<int> d;

void solve(void) {
	int n, k;
	cin >> n >> k;
	vector<int> b(n);
	vector<int> c(n);
	vector<int> a(n);
	for (int i = 0; i < n; i++) {
		cin >> b[i];
		a[i] = d[b[i]];
	}
	for (int i = 0; i < n; i++)
		cin >> c[i];
	if (k >= 12000)
		k = 12000;
	vector<vector<int>> dp(n + 1, vector<int>(k + 1, -1));
	dp[0][k] = 0;
	int r = 0;
	for (int i = 0; i <= n; i++) {
		for (int j = 0; j <= k; j++) {
			if (i + 1 <= n && dp[i][j] >= 0 && j - a[i] >= 0) {
				dp[i + 1][j - a[i]] = max(dp[i + 1][j - a[i]], dp[i][j] + c[i]);
			}
			if (i + 1 <= n)
				dp[i + 1][j] = max(dp[i + 1][j], dp[i][j]);
			r = max(r, dp[i][j]);
		}
	}
	cout << r << '\n';

}

int main(void) {
	ios::sync_with_stdio(false);
  	cin.tie(nullptr);
	int t;
	cin >> t;
	d = vector<int>(1001, -1);
	d[1] = 0;
	for (int i = 1; i < 1001; i++) {
		if (d[i] >= 0) {
			for (int j = 1; j <= i; j++) {
				int v = i + i / j;
				if (v < 1001 && (d[v] == -1 || d[v] > d[i] + 1))
					d[v] = d[i] + 1;
			}
		}
	}
	while(t--) {
		solve();
	}
	return 0;
}
```

<!--more-->

# [C. Kill the Monster](https://codeforces.com/contest/1633/problem/C)

- 标签：暴力
- 分析
  - 对于每个硬币，要么用于升级防具，要么用于升级武器
  - 因此，我们可以考虑将$i$个硬币升级防具，剩下的$k-i$个用于升级武器，遍历$i$即可求得
- 代码

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <set>
#include <unordered_set>
#include <utility>
#include <unordered_map>
#include <queue>
using namespace std;
typedef long long ll;

void solve(void) {
	ll h1, d1, h2, d2;
	cin >> h1 >> d1 >> h2 >> d2;
	ll k, w, a;
	cin >> k >> w >> a;
	for (ll i = 0; i <= k; i++) {
		ll h3 = h1 + i * a;
		ll d3 = d1 + (k - i) * w;
		ll x = h2 / d3;
		if (h2 % d3)
			++x;
		ll y = h3 / d2;
		if (h3 % d2)
			++y;
		if (x <= y) {
			cout << "YES" << '\n';
			return;
		}

	}
	cout << "NO" << '\n';
}

int main(void) {
	ios::sync_with_stdio(false);
  	cin.tie(nullptr);
	int t;
	cin >> t;
	while(t--) {
		solve();
	}
	return 0;
}
```



# [B. Minority](https://codeforces.com/contest/1633/problem/B)

- 标签：模拟
- 分析
  - 最多的情况必定是考虑整个字符串，假如0和1的数量刚好相等且为$c$，则不考虑首字符的情况下可以移除$c-1$个字符
  - 若不相等，且可以移除较小的一方，即$min(c_0,c_1)$个字符
- 代码

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <set>
#include <unordered_set>
#include <utility>
#include <unordered_map>
#include <queue>
using namespace std;
typedef long long ll;

void solve(void) {
	// int n;
	string s;
	cin >> s;
	int zero = 0, one = 0;
	for (auto& c: s)
		if (c == '0')
			zero++;
		else
			one++;
	if (zero == one)
		cout << zero - 1 << '\n';
	else
		cout << min(zero, one) << '\n';
}

int main(void) {
	ios::sync_with_stdio(false);
  	cin.tie(nullptr);
	int t;
	cin >> t;
	while(t--) {
		solve();
	}
	return 0;
}
```



# [A. Div. 7](https://codeforces.com/contest/1633/problem/A)

- 标签：模拟
- 分析
  - 我们只需要加减7以内的数，必定可以让其能够被7整除
  - 最小变动的位数只可能是0、1、2
    - 如果数字本身就能被7整除则答案为0
    - 若数字加上/减去某个数的情况下能只改变个位，且被7整除，则答案为1
    - 否则，答案为2，这里应以只改变个位、十位的原则选择加或者减
- 代码

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <set>
#include <unordered_set>
#include <utility>
#include <unordered_map>
#include <queue>
using namespace std;
typedef long long ll;

void solve(void) {
	int n;
	cin >> n;
	if (n % 7 == 0) {
		cout << n << '\n';
		return;
	}
	int x = n % 7;
	if (n % 10 >= n % 7) {
		cout << n - x << '\n';
		return;
	}
	if (n % 100 + 7 - x < 100) {
		cout << n + 7 - x << '\n';
		return;
	} else {
		cout << n - x << '\n';
		return;
	}
}

int main(void) {
	ios::sync_with_stdio(false);
  	cin.tie(nullptr);
	int t;
	cin >> t;
	while(t--) {
		solve();
	}
	return 0;
}
```

