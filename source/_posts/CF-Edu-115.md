---
title: CF Edu 115
date: 2022-01-17 19:44:56
tags: Codeforces
mathjax: true
categories: Algorithm
---

# 总结

- Edu121前进行的练习，完成了A-D题



# [D. Training Session](https://codeforces.com/contest/1598/problem/D)

- 标签：组合数学

> Monocarp decided to select exactly 3 problems from n problems for the problemset. The problems should satisfy **at least one** of two conditions (possibly, both):
>
> - the topics of all three selected problems are different;
> - the difficulties of all three selected problems are different.



- 分析：假如我们直接考虑原问题，则较难求解，我们需要求解的是不满足以上情况的问题组合数，即选择3条问题，既存在任意两条拥有相同的topic，又存在任意两条拥有相同的难度，这样的方案有多少种，设为$x$，而总共可选的问题组合为$s=C_n^3=n(n-1)(n-2)/6$，因此最终答案$r=s-x$
- 求解：如何求解变换后的问题，对问题按topic分类，同时维护各个difficulty的计数值$cnt$，$cnt(j)$表示难度为$j$的问题数量
  - 遍历不同topic的每个问题$i$，假设该topic共有m个问题，则从中再选择一个问题$j$的方案数为$m-1$，然后选择第三个问题使其难度等于$d_i$，则有$(m-1) \times (cnt(d_i)-1)$
- 注意点，求解总的问题组合数量$s$时需要使用long long

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
	ll n;
	cin >> n;
	vector<int> a(n);
	vector<int> b(n);
	unordered_map<int, vector<int>> cnt;
	unordered_map<int, ll> data;
	for (int i = 0; i < n; i++) {
		cin >> a[i] >> b[i];
		if (cnt.find(a[i]) == cnt.end()) {
			cnt[a[i]] = vector<int>{};
		}
		cnt[a[i]].push_back(b[i]);
		data[b[i]]++;
	}
	ll r = 0;
	for (auto &item: cnt) {
		for (auto &v: item.second) {
			if ((ll) item.second.size() > 1 && data[v] >= 1)
				r += ((ll)item.second.size() - 1) * (data[v] - 1);
		}
	}
	ll t = n * (n - 1) * (n - 2) / 6;
 	cout << t - r << endl;

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

# [C. Delete Two Elements](https://codeforces.com/contest/1598/problem/C)

- 标签：数学、哈希表
- 分析：由题意可知，删除的两个元素的平均值也必须等于整个数组的平均值，因此该平均值要么是整数，要么乘以二后是整数。然后用哈希表记录数组元素，即可在$O(N)$时间内找到满足条件的元素
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
	vector<ll> a(n);
	unordered_map<ll, ll> data;
	ll sum = 0;
	for (int i = 0; i < n; i++) {
		cin >> a[i];
		sum += a[i];
		data[a[i]]++;
	}
	ll target;
	if (sum % n == 0) {
		target = sum / n * 2;
	} else if ((sum % n) * 2 == n) {
		target = sum / n * 2 + 1;
	} else {
		cout << 0 << endl;
		return;
	}
	ll r = 0;
	for (auto& v: data) {
		if (data.find(target - v.first) != data.end()) {
			if (target - v.first == v.first) {
				r += v.second * (v.second - 1);
			} else {
				r += v.second * data[target - v.first];
			}
		}
	}
	r /= 2;
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

# [B. Groups](https://codeforces.com/contest/1598/problem/B)

- 标签：模拟、哈希表
- 分析：对星期几进行遍历，则一共有$5\times4=20$种组合，对于每种情况都可以通过哈希表计数判断是否满足
- 时间复杂度：$O(20N)$
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

bool solve(void) {
	int n;
	cin >> n;
	vector<vector<int>> data(n, vector<int>(5));
	for (int i = 0; i < n; i++) {
		for (int j = 0; j < 5; j++) {
			cin >> data[i][j];
		}
	}
	for (int i = 0; i < 5; i++) {
		for (int j = i + 1; j < 5; j++) {
			vector<int> cnt(3);
			bool flag = true;
			for (auto&v: data) {
				if (v[i] && v[j])
					cnt[2]++;
				else if (v[i])
					cnt[0]++;
				else if (v[j])
					cnt[1]++;
				else {
					flag = false;
					break;
				}
			}
			if (!flag)
				continue;
			if (cnt[0] <= n / 2 && cnt[1] <= n / 2)
				return true;
		}
	}
	return false;

}

int main(void) {
	ios::sync_with_stdio(false);
  	cin.tie(nullptr);

    int t;
	cin >> t;
	while(t--) {
		if (solve())
			cout << "YES" << endl;
		else
			cout << "NO" << endl;
	}
	return 0;
}
```



# [A. Computer Game](https://codeforces.com/contest/1598/problem/A)

- 标签：模拟
- 分析：除非某一列的两个格子均被阻塞，否则他就可以通过并到达终点

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

bool solve(void) {
	int n;
	cin >> n;
	string s1, s2;
	cin >> s1 >> s2;
	for (int i = 0; i < n; i++) {
		if (s1[i] == '1' && s2[i] == '1') {
			return false;
		}
	}
	return true;

}

int main(void) {
	ios::sync_with_stdio(false);
  	cin.tie(nullptr);

    int t;
	cin >> t;
	while(t--) {
		if (solve())
			cout << "YES" << endl;
		else
			cout << "NO" << endl;
	}
	return 0;
}
```

