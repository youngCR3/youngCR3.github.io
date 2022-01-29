---
title: CF 747 (Div 2)
date: 2022-01-22 16:11:54
tags: Codeforces
categories: Algorithm
mathjax: true
---



# 总结

- 完成了A-E1题，题目难度较低

- D题的分析比较有意思，需要观察得到两种comment意味着两人身份的关系



# [D. The Number of Imposters](https://codeforces.com/contest/1594/problem/D)

- 标签：逻辑学、BFS
- 分析：此题关键是理解comment背后的含义
  - 假如$i$称$j$为imposter，则他俩身份必定不同
  - 假如$i$称$j$为crewmate，则他俩身份必定相同
- 因此可以通过BFS检查是否存在矛盾，若不矛盾则取两种身份中数量较多者加入答案（这是因为初始时我们必须自己确定其中一人的身份，因此两种身份是可以互换的，我们取较多的作为imposter以满足题目要求）
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
	vector<int> r(n + 1);
	vector<vector<vector<int>>> edges(n + 1);
	for (int i = 0; i < m; i++) {
		int u, v;
		string s;
		cin >> u >> v;
		cin >> s;
		if (s[0] == 'i') {
			edges[u].push_back(vector<int>{v, 1});
			edges[v].push_back(vector<int>{u, 1});
		} else {
			edges[u].push_back(vector<int>{v, 2});
			edges[v].push_back(vector<int>{u, 2});
		}
	}
	queue<int> q;
	int res = 0;

	for (int i = 1; i <= n; i++) {
		if (r[i] == 0) {
			r[i] = 1;
			q.push(i);
		}
		int r1 = 0, r2 = 0;
		while (!q.empty()) {
			int x = q.front();
			if (r[x] == 1) 
				r1++;
			else
				r2++;
			q.pop();
			for (auto& v: edges[x]) {
				if (r[v[0]] == 0) {
					if (v[1] == 1)
						r[v[0]] = 3 - r[x];
					else
						r[v[0]] = r[x];
					q.push(v[0]);
				} else {
					if (v[1] == 1 && r[v[0]] == r[x]) {
						cout << -1 << endl;
						return;
					} 
					if (v[1] == 2 && r[v[0]] != r[x]) {
						cout << -1 << endl;
						return;
					}

				}
			}
		}
		res += max(r1, r2);
	}
	cout << res << endl;
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

# [E1. Rubik's Cube Coloring (easy version)](https://codeforces.com/contest/1594/problem/E1)

- 标签：数学、快速幂
- 分析：由于每个颜色都只可以与四种颜色相接，因此除了根节点可以选择6种颜色外，其他节点都只能选择4种颜色
  - 总节点数量为$2^{k}-1$
  - 答案为$6 \times 4 ^ {2^k-2}$
  - 由于答案较大，需要使用快速幂，注意取模运算不能用于幂运算的次数
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
const int mod = 1e9+7;

ll fastpow(ll x, ll y) {
	ll r = 1;
	while (y) {
		if (y & 1) {
			r *= x;
			r %= mod;
		}
		y >>= 1;
		x *= x;
		x %= mod;
	} 
	return r;
}

void solve(void) {
	int k;
	cin >> k;
	ll r = 1;
	for (int i = 1; i <= k; i++) {
		if (i == 1)
			r *= 6;
		else {
			r *= fastpow(4, pow(2, i - 1));
			r %= mod;
		}
	}
	cout << r << endl;

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



# [C. Make Them Equal](https://codeforces.com/contest/1594/problem/C)

- 标签：构造性问题
- 分析
  - 我们最多只需要两次操作，第一次我们取$x=n$，这样除了最后一个元素其他元素都变为该字符。第二次只需要取一个不是$n$的因数的数作为$x$即可
  - 假如原字符串所有字符相等，返回0
  - 否则，尝试能否只用一次操作。遍历整个字符，假如某个字符$a_i$不等于$ch$，则我们必须选取$x$使得该数并非$x$的倍数，因此$i$的因数都无法选取。我们将所有的“无法选取因数集合”求得。假如其大小为$n$则代表不存在只用一次操作的情况，否则我们只需要找到不在该集合的元素作为$x$即可
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
	char c;
	cin >> c;
	string s;
	cin >> s;
	vector<int> pos(n + 1);
	bool flag = true;
	for (int i = 0; i < n; i++) {
		if (s[i] != c) {
			flag = false;
			int x = 1;
			while (x * x <= i + 1) {
				if ((i + 1) % x == 0) {
					pos[x] = 1;
					pos[(i + 1) / x] = 1;
					// pos.insert(x / i);
				} 
				x++;
			}
		}
	}
	if (flag) {
		cout << 0 << endl;
		return;
	}
	for (int i = 1; i <= n; i++) {
		if (pos[i] == 0) {
		    cout << 1 << endl;
			cout << i << endl;
			return;
		}
	}
	int x = 1;
	while (n % x == 0) {
		x++;
	}
    cout << 2 << endl;
	cout << n << " " << x << endl;
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



# [B. Special Numbers](https://codeforces.com/contest/1594/problem/B)

- 标签：数学
- 分析：将$k$转化为二进制数，假如其第$i$位为1，则表示需要加上$n^i$
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
const int mod = 1e9 + 7;

ll fastpow(int n, int i) {
	ll r = 1;
	ll x = n;
	while (i) {
		if (i & 1) {
			r *= x;
			r %= mod;
		}
		i >>= 1;
		x *= x;
		x %= mod;
	} 
	return r;
}

void solve(void) {
	int n, k;
	cin >> n >> k;
	
	ll r = 0;
	for (int i = 0; i < 31; i++) {
		if (k & (1 << i)) {
			r += fastpow(n, i);
			r %= mod;
		}
	}
	cout << r << endl;

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



# [A. Consecutive Sum Riddle](https://codeforces.com/contest/1594/problem/A)

- 标签：模拟
- 分析：假如我们想要得到$x$，我们只需要构造从$-(x-1)$到$x$的等差数列即可，这样$[1,x-1]$的每个数都会被相反数抵消
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
	ll n;
	cin >> n;
	cout << -(n - 1) << " " << n << endl;
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

