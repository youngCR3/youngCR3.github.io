---
title: CF 768 (Div 2)
date: 2022-01-29 17:22:54
tags: Codeforces
categories: Algorithm
mathjax: true
---

# 总结

- 完成了A-D题，但是WA太多次导致罚分较多，最终排名684，掉了7分
- 本来是要掉大分的，幸好最后没放弃思考，做出了D题。这是我还以为自己能涨分，但后来才发现要想升上并稳定在candidate master，应该要在Div2中稳定前五百才行
- 说实话，我对自己能达到candidate master还没信心，能稳定在Div2前1000，rating保持在1800以上已经是我当下的目标了



# [D. Range and Partition](https://codeforces.com/contest/1631/problem/D)

- 标签：二分查找、滑动窗口
- 分析
  - 划分为k个子数组，且每个都要求在范围内的元素比不在范围内的元素多，那么整个数组里，在范围内的元素至少比不在范围内的元素多k个
  - **猜想**：假如整个数组在范围内的元素至少比不在范围内的元素多k个，则**无论如何排列**，必定能按题目要求划分为k个子数组（该猜想并未严格证明，是通过观察得到的）
  - 因此，只需要对取值范围的区间大小$x$进行**二分查找**，将数组从小到大排序，并通过**滑动窗口**判断是否有足够数量的元素在取值范围内
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

vector<int> check(vector<int>& a, int k, int x) {
	int n = (int)a.size();
	int left = 0;
	for (int i = 0; i < n; i++) {
		while (a[i] > a[left] + x) {
			left++;
		}
		if (i - left + 1 - (n - (i - left + 1)) >= k)
			return vector<int> {a[left], a[i]};
	}
	return vector<int>{};
}

void solve(void) {
	int n, k;
	cin >> n >> k;
	vector<int> b(n);
	for (int i = 0; i < n; i++)
		cin >> b[i];
	vector<int> a = b;
	sort(a.begin(), a.end());
	int left = 0, right = n - 1;
	vector<int> r;
	while (left <= right) {
		int mid = left + (right - left) / 2;
		auto arr = check(a, k, mid);
		if (!arr.empty()) {
			right = mid - 1;
		}
		else
			left = mid + 1;
	}
	r = check(a, k, right + 1);
	cout << r[0] << " " << r[1] << "\n";
	vector<vector<int>> res{};
	int s = 0;
	left = 0;
	for (int i = 0; i < n; i++) {
		if (b[i] >= r[0] && b[i] <= r[1]) {
			s++;
		} else {
			s--;
		}
		if (s > 0 && (int)res.size() < k - 1) {
			s = 0;
			res.push_back(vector<int>{left, i});
			left = i + 1;
		}
	}
	res.push_back(vector<int>{left, n - 1});
	for (auto& v: res) {
		cout << v[0] + 1 << " " << v[1] + 1 << '\n';
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



<!--more-->

# [C. And Matching](https://codeforces.com/contest/1631/problem/C)

- 标签：构造性问题

- 分析

  - 通过观察、猜想、尝试可以找到构造方法

  - 假如$k$不是$n-1$，那么只需要将$0$与$\bar{k}$配对，$n-1$与$k$配对，其他任意数$i$与$\bar{i}$配对即可
  - 假如$k$为$n-1$，若$n=4$则不存在构造方法；若$n>4$，那么一种构造方法如下：
    - 如果$i$是2的幂，且$i=2^t$，那么它与$2^{t-1}$按位取反后的数配对
    - 特别地，如果$i$是1，则与$n/2-1$配对
    - 其他数$i$与其按位取反后的数$\bar{i}$配对

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
typedef unsigned int uint;

int count(int x) {
	int r = 0;
	while (x) {
		r++;
		x &= (x - 1);
	}
	return r;
}

void solve(void) {
	uint n, k;
	cin >> n >> k;
	if (k == n - 1) {
		if (n == 4) {
			cout << -1 << '\n';
			return;
		} else {
			int m = count(n - 1);
			vector<int> vis(n);
			for (int i = 0; i < n; i++) {
				int c = count(i);
				if (c == 1) {
					if (i == 1) {
						uint j = (1 << (m - 1)) - 1;
						cout << i << " " << j << '\n';
					} else {
						uint j = n - 1 - (i >> 1);
						cout << i << " " << j << '\n';
					}
				} else if (c == m - 1) {
					continue;
				} else {
					if (vis[i])
						continue;
					vis[i] = 1;
					vis[n - i - 1] = 1;
					cout << i << " " << n - 1 - i << '\n';
				}
			}
			return;
		}
	} else if (k == 0) {
		for (uint i = 0; i < n / 2; i++) {
			cout << i << " " << n - 1 - i << '\n';
		}
		return;
	} else {
		vector<int> vis(n);
		for (uint i = 0; i < n; i++) {
			if (vis[i])
				continue;
			if (i == 0) {
				vis[i] = 1;
				vis[n - 1 - k] = 1;
				cout << i << " " << n - 1 - k << '\n';
			} else if (i == k) {
				vis[i] = 1;
				vis[n - 1] = 1;
				cout << i << " " << n - 1 << '\n';
			} else {
				vis[i] = 1;
				vis[n - 1 -i] = 1;
				cout << i << " " << n - 1 - i << '\n';
			}
		}
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



# [B. Fun with Even Subarrays](https://codeforces.com/contest/1631/problem/B)

- 标签：贪心
- 分析
  - 最后一个元素无法改变，因此必须让整个数组都变为$a_n$
  - 从后往前遍历，假如当前元素不等于$a_n$则执行一次操作，$k$的值为当前元素$a_i$右侧的元素数量$n-i$
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
	int i = n - 1;
	int r = 0;
	while (i >= 0) {
		if (a[i] == a[n - 1]) {
			i--;
		} else {
			i -= (n - i - 1);
			r++;
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



# [A. Min Max Swap](https://codeforces.com/contest/1631/problem/A)

- 标签：贪心
- 分析：对于每个位置，我们都将较大的元素分为一组、较小的元素分为另一组
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
	for (int i = 0; i < n; i++)
		cin >> a[i];
	for (int i = 0; i < n; i++)
		cin >> b[i];
	int max1 = -1, max2 = -1;
	for (int i = 0; i < n; i++) {
		max1 = max(max1, max(a[i], b[i]));
		max2 = max(max2, min(a[i], b[i]));
	}
	cout << max1 * max2 << '\n';
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

