---
title: CF 734 (Div 3)
date: 2022-01-23 20:07:32
tags: Codeforces
categories: Algorithm
mathjax: true
---

# 总结

- Div3竞赛，完成了6/8题



# [E. Fixed Points](https://codeforces.com/contest/1551/problem/E)

- 标签：动态规划
- 分析
  - $dp(i,j)$表示前$i$个元素移除了$j$个的情况下，能得到的最多有效点
  - 状态转移时，我们可以考虑保留第$i+1$个元素，这样需要考虑$a_{i+1}$是否等于$i+1-j$，若等于则$dp(i+1,j)=dp(i,j)+1$；否则，$dp(i+1,j)=dp(i,j)$
  - 此外，还可以移除第$i+1$个元素，则$dp(i+1,j+1)=dp(i,j)$

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
	int n, k;
	cin >> n >> k;
	vector<int> a(n);
	for (int i = 0; i < n; i++)
		cin >> a[i];
	vector<vector<int>> dp(n + 1, vector<int>(n + 1, -1));
	dp[0][0] = 0;
	int r = -1;
	for (int i = 0; i <= n; i++) {
		for (int j = 0; j <= n; j++) {
			if (dp[i][j] >= 0) {
				if (dp[i][j] >= k) {
					if (r == -1 || j < r)
						r = j;
				}
				if (i < n) {
					if (a[i] == i + 1 - j) {
						dp[i + 1][j] = max(dp[i + 1][j], dp[i][j] + 1);
					} else {
						dp[i + 1][j] = max(dp[i + 1][j], dp[i][j]);
					}
				}
				if (i + 1 <= n && j + 1 <= n)
					dp[i + 1][j + 1] = max(dp[i + 1][j + 1], dp[i][j]);
			}
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

<!--more-->

# [D1. Domino (easy version)](https://codeforces.com/contest/1551/problem/D1)

- 标签：分析
- 分析
  - 假如m、n都为偶数，则可以全用水平填满，因此返回YES
  - 假如n为奇数，则最多只能铺$(n-1)/2 \times m$个水平木块，此外$k$必须是偶数
  - 假如$m$为奇数，则可以通过行列转置转化为上述第二种情况
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
	int n, m, k;
	cin >> m >> n >> k;
	if (n % 2 == 0 && m % 2 == 0) {
		if (k % 2) {
			cout << "NO" << endl;
		} else {
			cout << "YES" << endl;
		}
		return;
	} 
	int k1 = k, k2 = n * m / 2 - k1;
	if (m % 2) {
		k1 = n * m / 2 - k;
		k2 = k;
		int t = n;
		n = m;
		m = t;
	}
	if (k1 <= n * m / 2 - m / 2 && k1 % 2 == 0) {
		cout << "YES" << endl;
	} else {
		cout << "NO" << endl;
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



# [C. Interesting Story](https://codeforces.com/contest/1551/problem/C)

- 标签：排序
- 分析：分别对每个字母进行考虑，对于每个字符串，计算该字母的$delta$值，即字符串中该字母的数量减去其他字符的数量，然后进行排序，维护计数值$c$，从大到小不断相加直至$c \leq 0$，取五个字母中的最大数量即可
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
	vector<string> a(n);
	for (int i = 0; i < n; i++)
		cin >> a[i];
	int r = 0;
	for (int j = 0; j < 5; j++) {
		vector<int> delta(n);
		for (int i = 0; i < n; i++) {
			int c = 0;
			for (auto& ch: a[i]) {
				if (ch == 'a' + j)
					c++;
				delta[i] = c - ((int)a[i].size() - c);
			}
		}
		sort(delta.begin(), delta.end());
		int c = 0;
		bool flag = true;
		for (int i = n - 1; i >= 0; i--) {
			c += delta[i];
			if (c <= 0) {
				r = max(n - i - 1, r);
				flag = false;
				break;
			}
		}
		if (flag) {
			r = n;
			break;
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



# [B2. Wonderful Coloring - 2](https://codeforces.com/contest/1551/problem/B2)

- 标签：构造性问题
- 分析：若字母的数量大于等于k，则必定每种颜色都涂上，添加到$r$；否则，添加到$cnt$，最后可以涂色的长度为$r+cnt / k$
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
	int n, k;
	cin >> n >> k;
	vector<int> a(n);
	unordered_map<int, int> c;
	unordered_map<int, int> idx;
	for (int i = 0; i < n; i++) {
		cin >> a[i];
		c[a[i]]++;
		idx[a[i]] = 1;
	}
	int cnt = 0;
	unordered_map<int, vector<int>> idx1;
	for (auto &item: c) {
		if (item.second < k)
			cnt += item.second;
	}
	cnt = (cnt / k) * k;
	for (int i = 0; i < n; i++) {
		if (c[a[i]] < k) {
			idx1[a[i]].push_back(i);
		}
	}
	vector<int> r(n);
	int cnt1 = 0;
	int j = 1;
	for (auto& item: idx1) {
		for (auto& i: item.second) {
			if (cnt1 < cnt) {
				cnt1++;
				r[i] = j;
				j++;
				if (j == k + 1)
					j %= k;
			} else {
				r[i] = 0;
			}
		}
	}

	
	for (int i = 0; i < n; i++) {
		if (c[a[i]] >= k) {
			if (idx[a[i]] <= k) {
				r[i] = idx[a[i]];
				idx[a[i]]++;
			} else {
				r[i] = 0;
			}
		}
	}

	for (auto& v: r) {
		cout << v << " ";
	}
	cout << '\n';
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



# [B1. Wonderful Coloring - 1](https://codeforces.com/contest/1551/problem/B1)

- 标签：构造性问题
- 分析：若字母的数量大于等于2，则必定可以两种颜色都涂上，直接加到$r$，若字母数量为1，则添加到$cnt$，最后可以涂色的长度为$r + cnt/2$
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
	string s;
	cin >> s;
	vector<int> c(26);
	for (auto& ch: s) {
		c[ch - 'a']++;
	}
	int r = 0;
	int one = 0;
	for (auto &v: c) {
		if (v >= 2)
			r++;
		else if (v == 1)
			one++;
	}
	r += one / 2;
	cout <<r <<endl;

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



# [A. Polycarp and Coins](https://codeforces.com/contest/1551/problem/A)

- 标签：模拟
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
	int c1 = n / 3, c2 = n / 3;
	if (n % 3 == 1)
		c1++;
	else if(n % 3 == 2)
		c2++;
	cout << c1 << " " << c2 << '\n';
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

