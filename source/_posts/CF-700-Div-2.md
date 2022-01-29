---
title: CF 700 (Div 2)
date: 2022-01-29 17:23:10
tags: Codeforces
categories: Algorithm
mathjax: true
---

# 总结

- 稳定发挥，完成了A-D1题



# [D1. Painting the Array I](https://codeforces.com/contest/1480/problem/D1)

- 标签：数学

- 分析

  - 假如没有连续字符，则每个字符都能够增加计数值

  - 对于连续字符，最多只有两个有效，因为超过两个的字符必定会由于与前一个字符重复而被合并
  - 我们维护队列$q$记录所有连续字符，队首元素`q.front()`代表下一个连续字符
  - 此外维护计数值$cnt$，代表当前不等于`q.front()`的序列数（取值范围是0-2）
  - 从左到右遍历，对于非连续字符，我们都将更新总数，以及计数值`cnt`。假如当前字符并非`q.front()`，则`cnt`可以加一（若等于2则不加）；若等于`q.front()`，则`cnt`减一

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
	for (int i = 0; i < n; i++)
		cin >> a[i];
	vector<vector<int>> b{};
	queue<int> q{};
	for (int i = 0; i < n; i++) {
		if (i > 0 && a[i] == a[i - 1]) {
			if (b.back()[1] == 1) {
				b.back()[1]++;
				q.push(a[i]);
			}
		} else {
			b.push_back(vector<int>{a[i], 1});
		}
	}
	int r = 0;
	int cnt = 2;
	for (int i = 0; i < (int)b.size(); i++) {
		if (b[i][1] == 1) {
			r++;
			if (!q.empty() && b[i][0] == q.front()) {
				cnt--;
			} else if (cnt < 2) {
				cnt++;
			}
		}
		else {
			if (cnt == 1) {
				r++;
			} else {
				r += 2;
			}
			q.pop();
			if (!q.empty() && q.front() == b[i][0])
				cnt = 0;
			else
				cnt = 2;
		}
	}
	cout << r << '\n';
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

# [C. Searching Local Minimum](https://codeforces.com/contest/1480/problem/C)

- 标签：交互性问题、二分查找
- 分析
  - 对比任意元素$a_i$与其前一个元素$prev$和后一个元素$next$
  - 假如$a_i \lt prev$且$a_i \lt next$，则$a_i$即为local minimum
  - 假如$a_i \gt prev$，则$a_i$左侧必定存在local minimum
  - 假如$a_i \gt next$，则$a_i$右侧必定存在local minimum
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
#include <memory.h>
using namespace std;
typedef long long ll;
int memo[100003];

int get(int index) {
	if (memo[index])
		return memo[index];
	cout << "? " << index << "\n";
	cout.flush();
	int x;
	cin >> x;
	memo[index] = x;
	return x;
}

void solve(void) {
	memset(memo, 0, sizeof(memo));
	int n;
	cin >> n;
	int left = 1, right = n;
	int prev, next, x;

	while (left <= right) {
		int mid = left + (right - left) / 2;
		x = get(mid);
		if (mid > 1) {
			prev = get(mid - 1);
		} else {
			prev = 1e9;
		}
		if (mid + 1 <= n) {
			next = get(mid + 1);
		} else {
			next = 1e9;
		}
		if (x < prev && x < next) {
			cout << "! " << mid << '\n';
			return;
		} else if (x > prev) {
			right = mid - 1;
		} else {
			left = mid + 1;
		}
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



# [B. The Great Hero](https://codeforces.com/contest/1480/problem/B)

- 标签：模拟
- 分析：将各个monster按攻击值从小到大排序，总是先打攻击值最小的
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
	ll A, B, n;
	cin >> A >> B >> n;
	vector<vector<ll>> a(n, vector<ll>(2));
	for (int i = 0; i < n; i++)
		cin >> a[i][0];
	for (int i = 0; i < n; i++)
		cin >> a[i][1];
	sort(a.begin(), a.end());
	for (int i = 0; i < n; i++) {
		if (B < 0) {
			cout << "NO" << '\n';
			return;
		}
		ll x = a[i][1] / A;
		if (a[i][1] % A)
			x++;
		B -= x * a[i][0];
		if (B <= -a[i][0]) {
			cout << "NO" << '\n';
			return;
		}
	}
	cout << "YES" << '\n';
	return;
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



# [A. Yet Another String Game](https://codeforces.com/contest/1480/problem/A)

- 标签：贪心
- 分析：Alice和Bob都会优先修改当前最开始的字符
  - Alice想让字典序最小，因此改为`a`是最佳选择，若本来为`a`则改为`b`
  - Bob想让字典序最大，因此改为`z`是最佳选择，若本来为`z`则改为`y`
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
	string r;
	for (int i = 0; i < (int)s.size(); i++) {
		if (i % 2 == 0) {
			if (s[i] == 'a')
				r += 'b';
			else
				r += 'a';
		} else {
			if (s[i] == 'z')
				r += 'y';
			else
				r += 'z';
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

