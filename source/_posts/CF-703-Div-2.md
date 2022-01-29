---
title: CF 703 (Div 2)
date: 2022-01-25 23:40:50
tags: Codeforces
categories: Algorithm
mathjax: true
---

# 总结

- 完成了A-C2题，正常发挥
- 其中C1、C2题是交互性问题，个人认为非常精彩。我的最终做法也是跟出题人思路一致，而且C2的查询次数限制必然意味着不同的二分查找思路，该思路相对C1不那么直接，但只要灵光乍现就很容易实现
- 在解决C1、C2时，由于其为交互式问题，无法直接debug。为了对猜测的可能bug进行验证，故意进行了几次“验证bug式提交”，导致了过多的罚时。在实际竞赛中，对于这种“debug式提交”与自己思考bug之间的权衡，需要再细细斟酌，是否值得用一次WA来验证自己对bug的猜测，是否要再思考一下？



# [C1. Guessing the Greatest (easy version)](https://codeforces.com/contest/1486/problem/C1)

- 标签：二分查找
- 分析
  - 首先查找当前区间的第二小元素的位置$v=query(l,r)$
  - 然后将当前区间二分，计算$mid=l+(r-l)/2$，并根据$v$与$mid$的位置关系查询
    - 假如$v$在右半部分，则$v'=query(mid,r)$，若最大值也在右半部分，则有$v=v'$
    - 同理，假如$v$在左半部分，则$v'=query(l,mid)$，若最大值也在左半部分，则有$v=v'$
  - 不断二分，每次二分所需的查询次数为2，最多的二分次数小于20，因此总查询次数小于40
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
	int l = 1, r = n;
	int cur, last;
	while (l + 1 < r) {
		cout << "? " << l << " " << r << '\n';
		cout.flush();
		cin >> last;
		int mid = l + (r - l) / 2;
		if (last >= mid) {
			cout << "? " << mid << " " << r << '\n';
			
		} else {
			cout << "? " << l << " " << mid << '\n';
		}
		cout.flush();
		cin >> cur;
		if (cur == last) {
			if (last >= mid)
				l = mid;
			else
				r = mid;
		} else {
			if (last >= mid)
				r = mid - 1;
			else
				l = mid + 1;
		}
	}
	if (l == r) {
		cout << "! " << l << '\n';
	} else {
		cout << "? " << l << " " << r << '\n';
		cout.flush();
		cin >> last;
		if (last == l) {
			cout << "! " << r << '\n';
		} else {
			cout << "! " << l << '\n';
		}
	}
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

# [C2. Guessing the Greatest (hard version)](https://codeforces.com/contest/1486/problem/C2)

- 标签：二分查找
- 分析：在C1中，我们对区间进行了二分查找，区间的左右端点将随查找不断变化
  - 实际上，我们可以换一种思路，固定其中一个端点为整个数组的第二大元素的位置$v=query(1,n)$，并对最大元素的位置进行二分查找
  - 现在，第二大元素必定在我们的区间端点上，若区间包含最大元素，则必有$v'=v$，反之则不等。因此我们可以**直接对最大元素的位置进行二分查找**
  - 每次查找只需要进行1次查询，查找次数小于20，因此总的查询次数小于20
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
map<pair<int, int>, int> memo;

int query(int l, int r) {
	auto p = make_pair(l, r);
	if (memo.find(p) != memo.end()) {
		return memo[p];
	}
	int res;
	cout << "? " << l << " " << r << '\n';
	cout.flush();
	cin >> res;
	memo[p] = res;
	return res;
}

void solve(void) {
	int n;
	cin >> n;
	int l, r;
	int last = query(1, n);

	if (last != 1 && query(1, last) == last) {
		l = 1;
		r = last - 1;
		while (l <= r) {
			int mid = l + (r - l) / 2;
			if (query(mid, last) == last) {
				l = mid + 1;
			} else {
				r = mid - 1;
			}
		}	 
		cout << "! " << l - 1 << '\n';
	} else {
		l = last + 1;
		r = n;
		while (l <= r) {
			int mid = l + (r - l) / 2;
			if (query(last, mid) == last) {
				r = mid - 1;
			} else {
				l = mid + 1;
			}
		}	 
		cout << "! " << r + 1 << '\n';
	}
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



<!--more-->

# [B. Eastern Exhibition](https://codeforces.com/contest/1486/problem/B)

- 标签：数学、中位数
- 分析：我们可以将横纵坐标分开考虑
  - 该位置必定出现在所有坐标的中间，若坐标数量为奇数，则是一个固定点；若坐标数量为偶数，则是一段区间中的一个点，可能的位置数量为$r-l+1$，$l,r$为该中间区间的左右端点
  - 因此，我们对横纵坐标分别排序，并按上述思路处理，最后将两个数相乘即为最终结果
  - 需要注意使用long long数据类型
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
	vector<int> x(n);
	vector<int> y(n);
	for (int i = 0; i < n; i++) {
		cin >> x[i] >> y[i];
	}
	if (n % 2 == 1) {
		cout << 1 << endl;
		return;
	}
	sort(x.begin(), x.end());
	sort(y.begin(), y.end());
	ll r = 1;
	r *= x[n / 2] - x[n / 2 - 1] + 1;
	r *= y[n / 2] - y[n / 2 - 1] + 1;
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



# [A. Shifting Stacks](https://codeforces.com/contest/1486/problem/A)

- 标签：模拟
- 分析：令$h_i$的大小为$i$，多余的部分可以用作后面的数
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
	ll s = 0;
	for (int i = 0; i < n; i++) {
		cin >> a[i];
	}
	for (int i = 0; i < n; i++) {
		s += max(0, a[i] - i);
		if (a[i] < i) {
			if (a[i] + s < i) {
				cout << "NO" << endl;
				return;
			} else {
				s -= i - a[i];
			}
		}
	}
	cout << "YES" << endl;

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

