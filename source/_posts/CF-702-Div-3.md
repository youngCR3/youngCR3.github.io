---
title: CF 702 (Div 3)
date: 2022-01-24 21:19:13
tags: Codeforces
categories: Algorithm
mathjax: true
---

# 总结

- 题目较为简单，竞赛时间完成了A-F题
- 赛后完成了G题，事实上G题存在一些特殊情况，以及一些long long类型的细节问题，竞赛时不一定能想到



# [G. Old Floppy Drive](https://codeforces.com/contest/1490/problem/G)

- 标签：二分查找
- 分析
  - 如果数组总和小于等于0，那么除非我们能在第一趟遍历就达到目标值，否则我们永远也无法达到目标值
  - 对于一次遍历，我们维护前缀和数组$data$，且保证其是单调递增的，这是因为如果当前前缀和小于之前的值，那么我们可以通过更短的时间的达到该值，因此我们不需要保存当前的前缀和
  - 保存前缀和数组中的最大值$maxv$，我们需要的前置遍历次数为$c=ceil((x-maxv)/s)$，$s$为整个数组的和
  - 然后我们的目标值变为$x-c\times s$，可以通过二分查找在前缀和数组$data$中找到最早的时间
  - 注意使用long long数据类型
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
	vector<int> a(n);
	vector<vector<ll>> data{};
	ll s = 0;
	for (int i = 0; i < n; i++) {
		cin >> a[i];
		s += a[i];
		if (data.empty() || s > data.back()[0])
			data.push_back(vector<ll>{s, i});
	}
	sort(data.begin(), data.end());
	ll maxv = data.back()[0];
	vector<int> x(m);
	for (int i = 0; i < m; i++)
		cin >> x[i];
	vector<ll> res(m);
	for (int i = 0; i < m; i++) {
		// s negative, the only chance to reach x is the first iteration
		ll c = 0;
		ll t = 0;
		if (s <= 0) {
			c = 0;
			t = x[i];
		} else {
			ll target = x[i] - maxv;
			if (target < 0)
				target = 0;
			ll tmp = max((ll)0, target / s);
			if (target % s)
				tmp += 1;
			c = tmp * n;
			t = x[i] - tmp * s;
		}
		int l = 0, r = (int)data.size() - 1;
		while (l <= r) {
			int mid = l + (r - l) / 2;
			if (data[mid][0] >= t)
				r = mid - 1;
			else
				l = mid + 1;
		}
		if (l == (int)data.size()) {
			res[i] = -1;
		} else {
			res[i] = c + data[r + 1][1];
		}
	}
	for (auto& v: res)
		cout << v << " ";
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

<!--more-->



# [F. Equalize the Array](https://codeforces.com/contest/1490/problem/F)

- 标签：排序
- 分析
  - 首先统计数组出现的次数，并存于数组$cnt$中，并对其进行排序
  - $C$的最优值，必定是$cnt$中某个元素的值
  - 假设取$C=cnt_i$，则比其他的数组元素都将减小为$cnt_i$，比其小的数组元素都将减小为0。遍历整个数组即可计算最小操作次数
- 时间复杂度：$O(N)$
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
	unordered_map<int, int> data;
	for (int i = 0; i < n; i++) {
	    int t;
	    cin >> t;
		data[t]++;
	}
	vector<int> c{};
	for (auto& item: data)
		c.push_back(item.second);
	sort(c.begin(), c.end());
	int s1 = n;
	int r = n;
	int s2 = 0;
	int m = (int)c.size();
	for (int i = m - 1; i >= 0; i--) {
		s1 -= c[i];
		s2 += c[i];
		r = min(r, s1 + s2 - c[i] * (m - i));
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



# [E. Accidental Victory](https://codeforces.com/contest/1490/problem/E)

- 标签：排序
- 分析
  - 将原数组排序后，从大到小遍历
  - 遍历到某个元素时，考虑其加上所有小于等于自己的元素后，能否大于自己右侧的第一个元素，若可以则说明该元素可能成为赢家
  - 否则，只有该元素右侧的元素可能成为赢家
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
	vector<vector<int>> a(n, vector<int>(2));
	for (int i = 0; i < n; i++) {
		cin >> a[i][0];
		a[i][1] = i;
	}
	sort(a.begin(), a.end());
	ll s = 0;
	for (auto& v: a)
		s += v[0];	
	vector<int> r{};
	for (int i = n - 1; i >= 0; i--) {
		if (i == n - 1 || s >= a[i + 1][0]) {
			s -= a[i][0];
			r.push_back(a[i][1]);
		} else {
			sort(r.begin(), r.end());
			cout << n - i - 1 << endl;
			for (auto& v: r) {
				cout << v + 1 << " ";
			}
			cout << '\n';
			return;
		}
	}
	cout << n << endl;
	for (int i = 0; i < n; i++)
		cout << i + 1 << " ";
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

# [D. Permutation Transformation](https://codeforces.com/contest/1490/problem/D)

- 标签：模拟、DFS
- 分析：直接DFS即可
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

void dfs(vector<int>& a, vector<int>& dep, int l, int r, int d) {
	if (l > r)
		return;
	int maxi = -1, maxv = -1;
	for (int i = l; i <= r; i++) {
		if (a[i] > maxv) {
			maxv = a[i];
			maxi = i;
		}
	}
	dep[maxi] = d;
	dfs(a, dep, l, maxi - 1, d + 1);
	dfs(a, dep, maxi + 1, r, d + 1);
}

void solve(void) {
	int n;
	cin >> n;
	vector<int> a(n);
	for (int i = 0; i < n; i++)
		cin >> a[i];
	vector<int> dep(n);
	dfs(a, dep, 0, n - 1, 0);
	for (auto& v: dep) 
		cout << v << " ";
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



# [C. Sum of Cubes](https://codeforces.com/contest/1490/problem/C)

- 标签：哈希表
- 分析：预处理所有1e4以下的数的立方，存储于哈希表中
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


void solve(unordered_set<ll>& data) {
	ll n;
	cin >> n;
	ll i = 1;
	while (i * i * i * 2 <= n) {
		if (data.find(n - i * i * i) != data.end()) {
			cout << "YES" << endl;
			return;
		}
		i++;
	}
	cout << "NO" << endl;
}

int main(void) {
	ios::sync_with_stdio(false);
  	cin.tie(nullptr);
  	unordered_set<ll> data;
  	for (ll i = 1; i < 1e4; i++) {
  		data.insert(i * i * i);
  	}

	int t;
	cin >> t;
	while(t--) {
		solve(data);
	}
	return 0;
}
```

