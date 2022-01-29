---
title: CF Edu 104
date: 2022-01-26 15:05:23
tags: Codeforces
mathjax: true
categories: Algorithm
---

# 总结

- 未掌握multiset的erase方法的正确用法，导致了E题多次TLE
- 竞赛时间完成了A-D题

# multiset的erase方法

- 假如移除值等于val的所有元素

  ```c++
  multiset<int> s;
  s.erase(val);
  ```

- 移除值等于val的其中一个元素

  - 使用此方法时，必须保证val存在于s中，否则`s.find(val)`返回`s.end()`，此时程序将会出错（TLE）

  ```c
  multiset<int> s;
  if (s.find(val) != s.end())
  	s.erase(s.find(val));
  ```

# [E. Cheap Dinner](https://codeforces.com/contest/1487/problem/E)

- 标签：动态规划、有序列表
- 分析
  - $dp(i,j)$表示考虑前$i$道菜，且第$i$道菜选择第$j$种的最小花费
  - 状态转移时，只要第$i$道菜的第$j$种可以和第$i-1$道菜的第$k$种配合，则有$dp(i,j)=min\{dp(i,j),dp(i-1,k)+cost(i,j)\}$
  - 若直接进行动态规划，则每次状态转移的复杂度都是$O(N^2)$
  - 我们可以对上一道菜的$dp$数组进行排序，每次都贪心地选能够被选择的最小者。该过程可以通过multiset实现，将上一道菜的dp数组元素存储于multiset中，遍历到$dp(i,j)$时，先将其不能配合的菜对应的dp元素从multiset移除，然后选择当前的最小元素进行状态转移，之后再将刚移除的元素重新加入到multiset中即可
- 坑点：multiset的正确使用，详见上文
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
// #include <cstring>
using namespace std;

typedef long long ll;

void solve(void) {
	vector<int> n(4);
	for (int i = 0; i < 4; i++)
		cin >> n[i];
	vector<vector<int>> nums(4);
	for (int i = 0; i < 4; i++) {
		nums[i].resize(n[i]);
		for (int j = 0; j < n[i]; j++)
			cin >> nums[i][j];
	}
	vector<vector<vector<int>>> ban(3);
	for (int i = 0; i < 3; i++) {
		ban[i].resize(n[i + 1]);
		int m;
		cin >> m;
		for (int j = 0; j < m; j++) {
			int x, y;
			cin >> x >> y;
			x--; y--;
			ban[i][y].push_back(x);
		}
	}
	for (int i = 0; i < 3; i++) {
		multiset<int> s;
		for (int j = 0; j < n[i]; j++) {
			if (nums[i][j] >= 0)
				s.insert(nums[i][j]);
		}
			
		for (int j = 0; j < n[i + 1]; j++) {
			for (auto& k: ban[i][j]) {
				if (nums[i][k] >= 0)
					s.erase(s.find(nums[i][k]));
			}
			if (s.empty()) {
				nums[i + 1][j] = -1;
			} else {
				nums[i + 1][j] += *s.begin();
			}
			for (auto& k: ban[i][j]) {
				if (nums[i][k] >= 0)
					s.insert(nums[i][k]);
			}
		}
	}
	int r = -1;
	for (int j = 0; j < n[3]; j++) {
		if (nums[3][j] >= 0) {
			if (r == -1 || nums[3][j] < r)
				r = nums[3][j];
		}
	}
	cout << r << '\n';
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

# [D. Pythagorean Triples](https://codeforces.com/contest/1487/problem/D)

- 标签：二分查找、数学
- 分析
  - 同时满足$a^2+b^2=c^2$和$a^2=b+c$两条公式，变换后可得$b+1=c$
  - 因此，对于任意奇数的平方，必定可以拆分为两个数满足$b+1=c$
  - 我们提前计算好奇数的平方数（注意不能只计算到$10^9$，因为只要分解后的$c\lt10^9$即可，因此我们需要计算到$2\times10^9+1$）
  - 然后进行二分查找即可
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
vector<ll> square;

void solve(void) {
	int n;
	cin >> n;
	
	int left = 0, right = (int)square.size() - 1;
	while (left <= right) {
		int mid = left + (right - left) / 2;
		if ((square[mid] + 1) / 2 > n) {
			right = mid - 1;
		} else {
			left = mid + 1;
		}
	}

	cout << right + 1 << '\n';
}

int main(void) {
	ios::sync_with_stdio(false);
  	cin.tie(nullptr);
  	for (ll i = 3; i * i <= (ll)4e9; i+=2) {
  		square.push_back(i * i);
  	}
	int t;
	cin >> t;
	while(t--) {
		solve();
	}
	return 0;
}
```

# [C. Minimum Ties](https://codeforces.com/contest/1487/problem/C)

- 标签：构造性问题
- 分析
  - 假如有奇数只队伍，则每支队伍需要进行$n-1$场比赛，为偶数，只需要赢一半输一半即可
  - 假如有偶数只队伍，则每只队伍需要进行$n-1$场比赛，为奇数，需要平一场后，赢一半输一半
  - 关键是如何分配胜负场次，以下方法是通过尝试得出的
    - 假如$n$为奇数，则每个队都输给大于自己的相同奇偶性编号队伍，赢大于自己的不同奇偶性编号队伍
    - 假如$n$为偶数，则（1）若队伍编号为奇数，它将与自己编号加1的队伍战平；（2）其余规则同上，即每个队都输给大于自己的相同奇偶性编号队伍，赢大于自己的不同奇偶性编号队伍

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

	if (n % 2) {
		for (int i = 1; i <= n - 1; i++) {
			for (int j = i + 1; j <= n; j++) {
				if (i % 2 != j % 2) {
					cout << 1 << " ";
				}
				else {
					cout << -1 << " ";
				}
			}
		}
	} else {
		for (int i = 1; i <= n - 1; i++) {
			for (int j = i + 1; j <= n; j++) {
				if (i % 2 && j == i + 1)
					cout << 0 << " ";
				else if (i % 2 != j % 2)
					cout << 1 << " ";
				else
					cout << -1 << " ";
			}
		}
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

# [B. Cat Cycle](https://codeforces.com/contest/1487/problem/B)

- 标签：模拟
- 分析
  - 若$n$为偶数，则两只猫不会碰面，直接计算即可
  - 若$n$为奇数，则除了第一次见面需要经过$n/2+1$小时后，此后的见面都只相隔$n/2$小时。假设见面次数为$x$，由于右边的猫没有额外的行动，其移动距离即为$k$，而左边的猫移动距离为$x+k$，由此可计算其位置
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
	if (n % 2) {
		k += (k - 1) / (n / 2);
	}
	int r = k % n;
	if (r == 0)
		r = n;
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



# [A. Arena](https://codeforces.com/contest/1487/problem/A)

- 标签：模拟
- 分析：只要不是最小值，则有可能获胜
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
	for (int i = 0; i < n; i++) {
		cin >> a[i];
	}
	sort(a.begin(), a.end());
	int r = 0;
	for (int i = 0; i < n; i++) {
		if (a[i] > a[0]) {
			r = n - i;
			break;
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

