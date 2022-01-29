---
title: CF 704 (Div 2)
date: 2022-01-25 15:11:20
tags: Codeforces
categories: Algorithm
mathjax: true
---

# 总结

- 题解都非常巧妙、思维性，完成了A-C题，其中B题由于对题意的错误理解，卡了很久
- 英文阅读多少还是对理解题意有所影响，做B题时，误以为在所有操作中$k$都是不变的，但$k$是每次操作都可以改变的
- 赛后尝试做D题，一道构造性题目，且存在较多corner case，若不看题解我几乎不可能想出来

# [C. Maximum width](https://codeforces.com/contest/1492/problem/C)

- 标签：字符串匹配、动态规划
- 分析
  - 维护$next$和$prev$数组，$next(i,j)$表示出现在索引$i$及其右侧的第一个字符$j$所在的位置，$prev(i,j)$表示出现在索引$i$及其左侧的第一个字符$j$所在的位置
  - 动态规划，$dp1(i)$表示匹配$t(0,i)$的最小索引位置，$dp2(i)$表示匹配$t(i,m)$的最大索引位置，遍历所有$j$，并找到$max\{dp2(j+1)-dp1(j)\}$
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
	string s, t;
	cin >> s >> t;
	vector<vector<int>> dp1(n, vector<int>(26));
	vector<vector<int>> dp2(n, vector<int>(26));
	vector<int> c(26, -1);
	for (int i = 0; i < n; i++) {
		c[s[i] - 'a'] = i;
		for (int j = 0; j < 26; j++) {
			dp1[i][j] = c[j];
		}
		
	}
	c = vector<int>(26, -1);
	for (int i = n - 1; i >= 0; i--) {
		c[s[i] - 'a'] = i;
		for (int j = 0; j < 26; j++) {
			dp2[i][j] = c[j];
		}
	}
	vector<int> left(m, -1);
	vector<int> right(m, -1);
	int i = 0;
	for (int j = 0; j < m; j++) {
		i = dp2[i][t[j] - 'a'];
		left[j] = i;
		i++;
	}
	i = n - 1;
	for (int j = m - 1; j >= 0; j--) {
		i = dp1[i][t[j] - 'a'];
		right[j] = i;
		i--;
	}
	int r = 0;
	for (int i = 0; i < m - 1; i++) {
		r = max(r, right[i + 1] - left[i]);
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



# [B. Card Deck](https://codeforces.com/contest/1492/problem/B)

- 标签：贪心
- 分析：每次都找到当前数组中的最大值，然后将其到数组末尾之间的元素加入答案数组$r$
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
	int maxi;
	for (int i = 0; i < n; i++) {
		cin >> a[i];
	}
	vector<int> r{};
	vector<int> vis(n + 1);
	int maxv = n;
	int pos = n;
	for (int i = n - 1; i >= 0; i--) {
		vis[a[i]] = 1;
		if (a[i] == maxv) {
			for (int j = i; j < pos; j++)
				r.push_back(a[j]);
			pos = i;
			while (max[v]) {
				maxv--;
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



# [A. Three swimmers](https://codeforces.com/contest/1492/problem/A)

- 标签：模拟
- 分析：如果$p \ mod \ a$为0则不用等待，否则需要等待$a-p\ mod \ a$分钟，$b$、$c$同理
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
	ll p, a, b, c;
	cin >> p >> a >> b >> c;
	ll r;
	if (p % a == 0) {
		cout << 0 << endl;
		return;
	} else {
		r = a - p % a;
	}
	if (p % b == 0) {
		cout << 0 << endl;
		return;
	} else if (r > b - p % b) {
		r = b - p % b;
	}
	if (p % c == 0) {
		cout << 0 << endl;
		return;
	} else if (r > c - p % c) {
		r = c - p % c;
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

