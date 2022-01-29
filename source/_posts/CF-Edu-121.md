---
title: CF Edu 121
date: 2022-01-17 01:13:45
tags: Codeforces
categories: Algorithm
mathjax: true
---

# 总结

![](sad.jpg)

- 做出了A-D题，且速度不错
- 其中D题直接遍历无法满足时间复杂度，利用题目中的条件，根据目标值是$2^i$的特性，直接对目标值的次数$i$进行遍历，从而解决了问题。这道题体现出了CF典型题目的特点：需要根据题目的具体条件分析，需要进行时间复杂度分析，需要一定的数学思维（贪心性的思考）
- 但是由于电力原因CF的服务器shutdown了一小时，因此unrated了，QAQ



# [D. Martial Arts Tournament](https://codeforces.com/contest/1626/problem/D)

- 标签：贪心，枚举
- 分析：显然若直接模拟时间夫再度为$O(N^2)$。因为我们的目标是让三段的数量都尽量接近$2^i$，因此我们可以考虑直接对该数的次数$i$进行枚举
- 贪心：首先要考虑的一件事是，假设我们的数量目标值$target=2^i$已经确定的情况下，是否从原数组取越多的数，使其尽量接近但不超过$target$越好，还是保留一部分数留给后面的mid或者heavy更好。答案是的确取得越多越好，这是这道题贪心的地方
- 算法：由于总数不超过2e5，因此我们最多取$target=2^{18}$，我们只需要枚举前两个low和mid的目标值，剩下的heavy可由剩下的元素数量决定，同时通过前缀和 + 二分查找可加速查找过程
- 时间复杂度：$O(K^2logN)$，其中$K=19$表示target次数的可能取值为$[0,18]$
- 代码

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <set>
#include <map>
#include <unordered_set>
#include <utility>
#include <unordered_map>
#include <queue>
#include <math.h>
using namespace std;
typedef long long ll;

void solve(void) {
	int n;
	cin >> n;
	vector<int> a{};
	map<int, int> c{};
	for (int i = 0; i < n; i++) {
		int t;
		cin >> t;
		c[t]++;
	}
	for (auto &item: c) {
		a.push_back(item.second);
	}
	vector<int> pre(a.size());
	for (int i = 0; i < (int)a.size(); i++) {
		if (i == 0)
			pre[i] = a[i];
		else
			pre[i] = pre[i - 1] + a[i];
	}
	int m = (int)a.size();
	int res = -1;
	for (int i = 0; i < 19; i++) {
		int target = pow(2, i);
		int l = 0, r = m - 1;
		while (l <= r) {
			int mid = l + (r - l) / 2;
			if (pre[mid] > target)
				r = mid - 1;
			else
				l = mid + 1;
		}
		int low;
		if (r == -1)
			low = 0;
		else
			low = pre[r];
		for (int j = 0; j < 18; j++) {
			int target1 = pow(2, j);
			int l1 = r + 1, r1 = m - 1;
			while (l1 <= r1) {
				int mid = l1 + (r1 - l1) / 2;
				if (pre[mid] - low > target1)
					r1 = mid - 1;
				else
					l1 = mid + 1;
			}
			int midw;
			if (r1 < r + 1)
				midw = 0;
			else
				midw = pre[r1] - low;
			int high = n - low - midw;
			int target2 = 1;
			while (target2 < high)
				target2 <<= 1;
			int tmp = target - low + target1 - midw + target2 - high;
			if (res == -1 || tmp < res) {
				res = tmp;
			}
		}
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

# [C. Monsters And Spells](https://codeforces.com/contest/1626/problem/C)

- 标签：等差数列、双指针
- 分析：假设在位置为$k_i$处有一个怪物的血量为$h_i$，则为了打败它需要在$k_i-h_i+1$处就开始蓄力。假设在这两点之间还有其他怪物，那我们必须将蓄力过程合并，直到找到一系列怪物的蓄力过程的左端点$left$和右端点$right$，则所需要的力量值是一个从1到$right - left + 1$的等差数列，即$(1+(right-left+1))\times (right - left + 1)/2$
- 代码

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <set>
#include <map>
#include <unordered_set>
#include <utility>
#include <unordered_map>
#include <queue>
using namespace std;
typedef long long ll;

void solve(void) {
	int n;
	cin >> n;
	vector<ll> k(n);
	vector<ll> h(n);
	for (int i = 0; i < n; i++)
		cin >> k[i];
	for (int i = 0; i < n; i++)
		cin >> h[i];
	ll r = 0;
	ll left = k[n - 1] + 1, right = k[n - 1];
	for (int i = n - 1; i >= 0; i--) {
		if (k[i] < left) {
			r += (1 + right - left + 1) * (right - left + 1) / 2;
			right = k[i];
			left = k[i] - h[i] + 1;
		} else {
			left = min(left, k[i] - h[i] + 1);
		}
	}
	r += (1 + right - left + 1) * (right - left + 1) / 2;
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



# [B. Minor Reduction](https://codeforces.com/contest/1626/problem/B)

- 标签：贪心
- 分析：由于合并过程必定会使数字变小，我们应考虑最小化减小量
  - 从后往前遍历，假如存在两个相邻数字相加大于10，则执行该操作。这是因为这种操作不会改变原数字的总位数，且由于总会减小，那么这个发生在越低位越好
  - 假如不存在不减少总位数的操作，则因为两个数相加可能会使第一位的数字变大，因此发生在越高位越好。从前往后遍历，假如两个相邻数字相加大于原来的第一个数，则执行该操作
- 代码

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <set>
#include <map>
#include <unordered_set>
#include <utility>
#include <unordered_map>
#include <queue>
using namespace std;
typedef long long ll;

void solve(void) {
	string x;
	cin >> x;
	for (int i = (int)x.size() - 2; i >= 0; i--) {
		int a = x[i] - '0';
		int b = x[i + 1] - '0';
		if (a + b >= 10) {
			string r;
			for (int j = 0; j < i; j++) {
				r += x[j];
			}
			r += '1';
			r += (a + b) % 10 + '0';
			for (int j = i + 2; j < (int)x.size(); j++)
				r += x[j];
			cout << r << endl;
			return;
		}
	}
	for (int i = 0; i < (int)x.size() - 1; i++) {
		int a = x[i] - '0';
		int b = x[i + 1] - '0';
		if (a + b > a) {
			string r;
			for (int j = 0; j < i; j++) {
				r += x[j];
			}
			r += (a + b) + '0';
			for (int j = i + 2; j < (int)x.size(); j++)
				r += x[j];
			cout << r << endl;
			return;
		}
	}
	string r;
	int n = (int)x.size();
	for (int i = 0; i < (int)x.size() - 2; i++)
		r += x[i];
	int a = (x[n - 2] - '0') + (x[n - 1] - '0');
	r += a + '0';
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

# [A. Equidistant Letters](https://codeforces.com/contest/1626/problem/A)

- 标签：模拟
- 分析：将出现两次的字母按顺序分两次排列，如`abcabc`，出现一次的字母紧跟后面即可
- 代码

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <set>
#include <map>
#include <unordered_set>
#include <utility>
#include <unordered_map>
#include <queue>
using namespace std;
typedef long long ll;

void solve(void) {
	vector<int> a(26, 0);
	string s;
	cin >> s;
	for (auto&c: s)
		a[c - 'a']++;
	string r;
	for (int i = 0; i < 26; i++)
		if (a[i] == 2)
			r += 'a' + i;
	for (int i = 0; i < 26; i++)
		if (a[i] == 1)
			r += 'a' + i;
	for (int i = 0; i < 26; i++)
		if (a[i] == 2)
			r += 'a' + i;
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

