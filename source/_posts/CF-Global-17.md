---
title: CF Global 17
date: 2022-02-10 21:12:35
tags: Codeforces
categories: Algorithm
mathjax: true
---

# 总结

- Global round的难度较高，D题难度就达到了2000，只做出了A-C题



# [C. Keshi Is Throwing a Party](https://codeforces.com/contest/1610/problem/C)

- 标签：二分查找
- 分析
  - 假设要选取k个人，当前是滴$i$个人，则必须满足的条件是$b_i\geq i-1$和$a_i\geq k - i$
  - 由此，可在一次遍历判断能否选取k个人，然后对$k$进行二分搜索即可
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

bool check(vector<int>&a, vector<int>&b, int k) {
	int n = a.size();
	int c = 0;
	for (int i = 0; i < n; i++) {
		if (a[i] >= k - (c + 1) && b[i] >= c) 
			++c;
		if (c >= k)
			return true;
	}
	return false;
}

void solve(void) {
	int n;
	cin >> n;
	vector<int> a(n);
	vector<int> b(n);
	for (int i = 0; i < n; i++) {
		cin >> a[i];
		cin >> b[i];
	}

	int l = 0, r = n;
	while (l < r) {
		int mid = (r - l) / 2 + l;
		if (check(a, b, mid)) {
			l = mid + 1;
		} else {
			r = mid - 1;
		}
	}

	cout << l - 1 << '\n';
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

# [B. Kalindrome Array](https://codeforces.com/contest/1610/problem/B)

- 标签：双指针
- 分析
  - 两个指针$left$、$right$从两端开始同时往中间遍历，当到达第一个不等的位置$s[left]!=s[right]$时，必定需要通过删除其中一个字符使剩下的字符串为回文串，否则原字符串无法变为回文串
  - 因为最多只需要考虑两个字符的情况，时间复杂度为$O(N)$

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
	int i = 0, j = n - 1;
	while (i < j) {
		if (a[i] != a[j]) {
			break;
		} else {
			++i;
			--j;
		}
	}
	if (i >= j) {
		cout << "YES" << '\n';
		return;
	}

	// try to delete a[i]
	int left = i, right = j;
	while (left < right) {
		if (a[left] != a[right]) {
			if (a[left] == a[i])
				++left;
			else if (a[right] == a[i])
				--right;
			else
				break;
		} else {
			++left;
			--right;
		}
	}
	if (left >= right) {
		cout << "YES" << '\n';
		return;
	}

	// try to delete a[j]
	left = i, right = j;
	while (left < right) {
		if (a[left] != a[right]) {
			if (a[left] == a[j])
				++left;
			else if (a[right] == a[j])
				--right;
			else
				break;
		} else {
			++left;
			--right;
		}
	}
	if (left >= right) {
		cout << "YES" << '\n';
		return;
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



# [A. Anti Light's Cell Guessing](https://codeforces.com/contest/1610/problem/A)

- 标签：数学
- 分析：每次猜测过后，都可以列写一条方程，假如原问题存在$k$个未知量，则只需要$k$次猜测。因此问题本质是求未知量的个数
  - 假如是$1\times 1$，则只有1个格子，因此无需猜测
  - 假如是一维，则未知数的个数为1，只需要一次猜测
  - 假如是二维，则未知数的个数为2，需要两次猜测

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
	if (n == 1 && m == 1) {
		cout << 0 << '\n';
		return;
	}
	else if (n == 1 || m == 1) {
		cout << 1 << '\n';
		return;
	} else {
		cout << 2 << '\n';
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

