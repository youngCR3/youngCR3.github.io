---
title: CF 648 (Div 2)
date: 2022-01-21 23:15:44
tags: Codeforces
categories: Algorithm
mathjax: true
---

# 总结

- 完成了A-E题



# [E. Maximum Subsequence Value](https://codeforces.com/contest/1365/problem/E)

- 标签：贪心
- 分析
  - 需要想到的是，由于要求必须有$max(1,k-2)$个元素在该位上是1，则我们最优的策略是选择3个元素为一组
  - 假如数组中总共只有1或2个元素，则返回其或值
  - 假如数组中有3个或以上元素，则遍历每3个元素，并取其或值的最大值即可
- 时间复杂度：$O(N^3)$
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
	vector<ll> a(n);
	for (int i = 0; i < n; i++)
		cin >> a[i];
	if (n == 1) {
		cout << a[0] << endl;
	} else if (n == 2) {
		cout << (a[0] | a[1]) << endl;
	} else {
		ll r = 0;
		for (int i = 0; i < n; i++) {
			for (int j = i + 1; j < n; j++) {
				for (int k = j + 1; k < n; k++) {
					r = max(r, a[i] | a[j] | a[k]);
				}
			}
		}
		cout << r << endl;
	}
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

# [D. Solve The Maze](https://codeforces.com/contest/1365/problem/D)

- 标签：BFS、图
- 分析：这是一道有趣的分析题目
  - 假如迷宫中不存在G，则我们直接堵住出口即可，返回Yes
  - 否则，假如我们想要堵住所有坏人，则必须堵住其相邻的空格。假如有任意坏人与好人相邻或与出口相邻，则我们无法阻止坏人，返回No
  - 否则，当我们堵住所有坏人后，从出口进行BFS，验证是否能到达所有好人所在地，若能则返回Yes，否则返回No
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
	int n, m;
	cin >> n >> m;
	int good = 0;
	vector<vector<char>> grid(n, vector<char>(m));
	for (int i = 0; i < n; i++) {
		string s;
		cin >> s;
		for (int j = 0; j < m; j++) {
			grid[i][j] = s[j];
			if (s[j] == 'G')
				good++;
		}
	}
	if (good == 0) {
		cout << "YES" << endl;
		return;
	}
	vector<vector<int>> delta{{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
	for (int i = 0; i < n; i++) {
		for (int j = 0; j < m; j++) {
			if (grid[i][j] == 'B') {
				for (auto& d: delta) {
					if (i + d[0] >= 0 && i + d[0] < n && j + d[1] >= 0 && j + d[1] < m) {
						if (grid[i + d[0]][j + d[1]] == 'G' || (i + d[0] == n - 1 && j + d[1] == m - 1)) {
							cout << "NO" << endl;
							return;
						} else if (grid[i + d[0]][j + d[1]] == '.') {
							grid[i + d[0]][j + d[1]] = '#';
						}
					}
				}
			} 
		}
	}

	queue<int> q;
	q.push(n * m - 1);
	int g = 0;
	vector<vector<int>> vis(n, vector<int>(m));
	vis[n - 1][m - 1] = 1;
	while (!q.empty()) {
		int x = q.front() / m;
		int y = q.front() % m;
		q.pop();
		for (auto& d: delta) {
			int i = x + d[0], j = y + d[1];
			if (i >= 0 && i < n && j >= 0 && j < m && vis[i][j] == 0) {
				if (grid[i][j] == '.') {
					q.push(i * m + j);
					vis[i][j] = 1;
				}
				else if (grid[i][j] == 'G') {
					q.push(i * m + j);
					g++;
					vis[i][j] = 1;
				}
			}
		}
	}
	if (g == good)
		cout << "YES" << endl;
	else
		cout << "NO" << endl;
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



# [C. Rotation Matching](https://codeforces.com/contest/1365/problem/C)

- 标签：哈希表
- 分析
  - 我们首先计算$a$、$b$中对应元素的偏差量，假设$a_i$和$b_j$对应，则偏差量为$d=(i-j+n) \ mod \ n$，偏差量的取值范围是$[0,n)$
  - 然后遍历所有元素，找到拥有相同偏差量的最多元素个数$r$即为答案
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
	vector<int> a(n);
	vector<int> b(n);
	vector<int> p(n + 1);
	for (int i = 0; i < n; i++)
		cin >> a[i];
	for (int i = 0; i < n; i++) {
		cin >> b[i];
		p[b[i]] = i;
	}
	int r = 0;
	unordered_map<int, int> data;
	for (int i = 0; i < n; i++) {
		int delta = (i - p[a[i]] + n) % n;
		data[delta]++;
		r = max(r, data[delta]);
	}
	cout << r << endl;
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



# [B. Trouble Sort](https://codeforces.com/contest/1365/problem/B)

- 标签：模拟
- 分析：只要数组中存在0、1两种类型，则元素可以排列成任意顺序，返回True；否则，元素只能按原顺序排列，此时需要对比原顺序是否已经为升序
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
	vector<int> a(n);
	vector<int> b(n);
	int s = 0;
	for (int i = 0; i < n; i++)
		cin >> a[i];
	for (int i = 0; i < n; i++) {
		cin >> b[i];
		s += b[i];
	}
	if (s != 0 && s != n) {
		cout << "YES" << endl;
	} else {
		for (int i = 1; i < n; i++) {
			if (a[i] < a[i - 1]) {
				cout << "NO" << endl;
				return;
			}
		}
		cout << "YES" << endl;
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



# [A. Matrix Game](https://codeforces.com/contest/1365/problem/A)

- 标签：数学、模拟
- 分析：因为每次操作必定会占用一行、一列，因此我们只需要统计空闲的行数$r$、列数$c$，并取最小值$min(r,c)$，若最小值为奇数则先手者获胜，反之则后手者获胜
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
	int n, m;
	cin >> n >> m;
	vector<int> rows(n);
	vector<int> cols(m);
	for (int i = 0; i < n; i++) {
		for (int j = 0; j < m; j++) {
			int s = 0;
			cin >> s;
			if (s == 1) {
				rows[i] = 1;
				cols[j] = 1;
			}
		}
	}
	int r1 = 0, r2 = 0;
	for (int i = 0; i < n; i++)
		if (rows[i] == 0)
			r1 += 1;
	for (int j = 0; j < m; j++)
		if (cols[j] == 0)
			r2 += 1;
	r1 = min(r1, r2);
	if (r1 % 2)
		cout << "Ashish" << endl;
	else
		cout << "Vivek" << endl;
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



