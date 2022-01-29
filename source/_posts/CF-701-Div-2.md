---
title: CF 701 (Div 2)
date: 2022-01-27 15:42:52
tags: Codeforces
categories: Algorithm
mathjax: true
---

# 总结

- 完成了A-C题
- 这次竞赛的题目都需要经过一定的思维转换才能求解，没有明显的套路



# [C. Floor and Mod](https://codeforces.com/contest/1485/problem/C)

- 标签：数学
- 分析：https://codeforces.com/blog/entry/87470
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
	int x, y;
	cin >> x >> y;
	int b = y;
	ll r = 0;
	while (b >= 1) {
		int c = x / (b + 1);
		if (c <= b - 2) {
			int b1 = x / (c + 1) - 1;
			if (b1 == 0)
				r += c * (b - 1);
			else
				r += c * (b - b1);
			b = b1; 
		} else {
			r += b - 1;
			b--;
		}

	} 
	
	cout << r << '\n';
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



<!--more-->

# [B. Replace and Keep Sorted](https://codeforces.com/contest/1485/problem/B)

- 标签：前缀和
- 分析
  - 一个数的可能取值取决于其前一个数和后一个数，假如前面的值为$a$，后面的值为$b$，则其取值范围是$[a+1,b-1]$，数量为$b-a-1$
  - 特别地，对于第一个数，若其后一个数为$b$，其取值范围是$[1,b-1]$，数量为$b-1$
  - 对于最后一个数，若其前一个数为$a$，则其取值范围是$[a+1,k]$，数量为$k-a$
  - 通过预处理前缀和数组，我们能够在$O(1)$时间内处理每个查询
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
	int n, q, k;
	cin >> n >> q >> k;
	vector<int> a(n);
	for (int i = 0; i < n; i++)
		cin >> a[i];
	vector<int> v(n);
	for (int i = 0; i < n; i++) {
		if (i == 0) {
			if (i + 1 < n) {
				v[i] = a[i + 1] - 1;
			} else {
				v[i] = k;
			}
		} else if (i == n - 1) {
			v[i] = k - a[i - 1];
		} else {
			v[i] = a[i + 1] - a[i - 1] - 1;
		}
	}
	vector<ll> prefix(n, 1);
	ll s = 0;
	for (int i = 0; i < n; i++) {
		s += v[i] - 1;
		prefix[i] = s;
	}
	for (int i = 0; i < q; i++) {
		int l, r;
		cin >> l >> r;
		l--;
		r--;
		if (l == r) {
			cout << k - 1 << '\n';
		} else {
			ll res = prefix[r - 1] - prefix[l];
			res += k - a[r - 1] - 1;
			res += a[l + 1] - 2;
			cout << res << '\n';
		}
	}
}

int main(void) {
	ios::sync_with_stdio(false);
  	cin.tie(nullptr);
	int t = 1;
	// cin >> t;
	while(t--) {
		solve();
	}
	return 0;
}
```



# [A. Add and Divide](https://codeforces.com/contest/1485/problem/A)

- 标签：暴力
- 分析
  - 最多只需要经过31次操作，$a$必定变为0。这是因为考虑最坏情况，$b$初始为1，$a=10^9$，则需要将$b$增大为2，然后不断除以2即可
  - $b$增大到什么值再进行除法能使次数最少呢？这一问题并非显而易见，但由于操作次数最多为31，因此我们可以考虑$b$增大0次到30次的每一种可能，然后计算$a$需要除以多少次才能变为0，最后将次数相加求得总操作次数，并取最小值即可
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
	int a, b;
	cin >> a >> b;
	if (a < b) {
		cout << 1 << '\n';
		return;
	} else if (a == b) {
		cout << 2 << '\n';
		return;
	}
	int r = -1;
	for (int i = 0; i < 32; i++) {
		if (i == 0 && b == 1) {
			continue;
		}
		int c = i;
		int a1= a;
		int b1 = b + i;
		while (a1 > 0) {
			a1 /= b1;
			c++;
		}
		if (r == -1 || r > c)
			r = c;
	}
	cout << r << '\n';
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

