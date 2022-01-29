---
title: CF 766 (Div.2)
date: 2022-01-16 00:57:09
tags: Codeforces
categories: Algorithm
mathjax: true
---

# 总结

- 完成了A-D题，且有丰富的时间思考E题，有很大的进步
- D题利用gcd的性质进行dp，该知识点在CF757中考察过，且几小时前才练习过，因此较快做出了D题
- E题是一道dp，由于读题不仔细，忽略了数据范围上的特殊之处



# [D. Not Adding](https://codeforces.com/contest/1627/problem/D)

- 标签：数论、动态规划

- 分析：由于$gcd(a_i,a_j) \leq min(a_i,a_j)$，因此我们只需要考虑$[1,10^6]$中的数是否能够添加到数组中，维护动态规划数组，$dp(i)=1$代表$i$可以加入数组中

  - 从大到小遍历每个数$i$，我们考虑其每个倍数$2i$，$3i$，...是否在数组中，同时维护一个最大公因数$g$，假如找到某个倍数$ki$在数组中，则更新$g=gcd(g,k)$，若最终$g$变为1则代表数$i$可以通过操作假如数组中，并将$dp(i)$置为1

- 时间复杂度：$O(ClogC)$，$C$是数组元素的最大值$10^6$
  $$
  \frac{C}{1}+\frac{C}{2}+...+\frac{C}{C} \sim O(ClogC)
  $$

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
const int MAX_A = 1e6 + 1;

int gcd(int x, int y) {
	if (x < y) {
		int t = x;
		x = y;
		y = t;
	}
	while (y) {
		int t = x % y;
		x = y;
		y = t;
	}
	return x;
}

void solve(void) {
	int n;
	cin >> n;
	vector<int> a(n);
	vector<int> dp(MAX_A, 0);
	for (int i = 0; i < n; i++) {
		cin >> a[i];
		dp[a[i]] = 1;
	}
	for (int i = MAX_A; i >= 1; i--) {
		if (dp[i])
			continue;
		int r = -1;
		for (int j = 2 * i; j < MAX_A; j += i) {
			if (dp[j]) {
				if (r == -1)
					r = j / i;
				else
					r = gcd(r, j / i);
			}
			if (r == 1) {
				dp[i] = 1;
				break;
			}
		}
	}
	int r = 0;
	for (int i = 1; i < MAX_A; i++) {
		if (dp[i])
			r += 1;
	}
	cout << r - n << endl;
}

int main(void) {
	ios::sync_with_stdio(false);
  	cin.tie(nullptr);
	int t = 1;
	while(t--) {
		solve();
	}
	return 0;
}
```

<!--more-->

# [C. Not Assigning](https://codeforces.com/contest/1627/problem/C)

- 标签：数学、奇偶性
- 分析：加入某个节点有三条边连接，那么必定不可能。反证法，假设三条边权重分别为$a$、$b$、$c$，且都为质数，根据题意则$a+b$、$a+c$、$b+c$都为质数，则它们必定都为奇数，矛盾。
  - 假设所有节点都只连接两条或两条以下的边，则可以，我们分别将两条边赋值为2、3即可
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
	vector<vector<pair<int, int>>> data(n + 1);
	vector<vector<int>> edge{};
	bool flag = true;
	for (int i = 0; i < n - 1; i++) {
		int u, v;
		cin >> u >> v;

		data[u].push_back(make_pair(v, i));
		data[v].push_back(make_pair(u, i));
		if (data[u].size() == 3 || data[v].size() == 3)
			flag = false;
	}
	if (!flag) {
		cout << -1 << endl;
		return;
	}
	vector<int> r(n - 1, 0);
	deque<int> q;
	q.push_back(1);
	vector<int> val(n + 1, 5);
	vector<int> vis(n + 1, 0);
	vis[1] = 1;
	while (!q.empty()) {
		int node = q.front();
		q.pop_front();
		for (auto& p: data[node]) {
			if (r[p.second] == 0) {
				if ((val[node] == 5 || val[node] == 2) && (val[p.first] == 5 || val[p.first] == 2)) {
					r[p.second] = 2;
					val[node] -= 2;
					val[p.first] -= 2;
				} else {
					r[p.second] = 3;
					val[node] -= 3;
					val[p.first] -= 3;
				}
			}
			if (vis[p.first] == 0) {
				vis[p.first] = 1;
				q.push_back(p.first);
			}
		}
	}
	for (int i = 0; i < n - 1; i++)
		cout << r[i] << " ";
	cout << endl;
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



# [B. Not Sitting](https://codeforces.com/contest/1627/problem/B)

- 标签：模拟
- 分析：Rahul必定会尽量坐在靠近中间的位置，而Tina必定会将靠近中间的格子优先涂色
  - 首先假设只有一维，若为奇数，则Tina会将最中间的格子涂色，而Rahul只能坐在中间格子的往左或往右一格；若为偶数，则Tina会将最中间的两个格子依次涂色，Rahul第一轮会坐在未被涂色的一个，第二轮只能坐在最中间两个格子的往左或往右一格
  - 推广到二维，本质是一样的，这两维可以分开考虑，最终将结果按从小到大排序即可。因为结果满足单调性，即涂的格子越多，Tina就可以让Rahul离自己越远
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
// const int mod = 1e9+ 7;

void solve(void) {
	int n, m;
	cin >> n >> m;
	vector<int> r{};
	for (int i = 0; i < (n + 1) / 2; i++) {
		for (int j = 0; j < (m + 1) / 2; j++) {
			int x, y;
			if (n % 2) {
				x = n / 2 + 1 + i;
			} else {
				x = n / 2 + 1 + i;
			}
			if (m % 2) {
				y = m / 2 + 1 + j;
			} else {
				y = m / 2 + 1 + j;
			}
			if (i == 0 && j == 0 && n % 2 && m % 2) {
				r.push_back(x - 1 + y - 1);
			} else if ((i == 0 && n % 2) || (j == 0 && m % 2)) {
				for (int k = 0; k < 2; k++)
					r.push_back(x - 1 + y - 1);
			} else {
				for (int k = 0; k < 4; k++)
					r.push_back(x - 1 + y - 1);
			}
		}
	}
	sort(r.begin(), r.end());
	for (auto& v: r)
		cout << v << " ";
	cout << endl;

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



# [A. Not Shading](https://codeforces.com/contest/1627/problem/A)

- 标签：模拟
- 分析：若该格子本身为黑色，则返回0；若该格子所在行/列有黑色格子，则返回1；若任意格子为黑色，则返回2；否则，返回-1

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
// const int mod = 1e9+ 7;

void solve(void) {
	int n, m, r, c;
	cin >> n >> m >> r >> c;
	vector<string> grid;
	for (int i = 0; i < n; i++) {
		string s;
		cin >> s;
		grid.push_back(s);
	}
	if (grid[r - 1][c - 1] == 'B') {
		cout << 0 << endl;
		return;
	}
	for (int i = 0; i < n; i++) {
		if (grid[i][c - 1] == 'B') {
			cout << 1 << endl;
			return;
		} 
	}
	for (int j = 0; j < m; j++) {
		if (grid[r - 1][j] == 'B') {
			cout << 1 << endl;
			return;
		}
	}
	for (int i = 0; i < n; i++) {
		for (int j = 0; j < m; j++) {
			if (grid[i][j] == 'B') {
				cout << 2<< endl;
				return;
			}

		}
	}
	cout << -1 << endl;

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

