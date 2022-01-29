---
title: CF 705 (Div 2)
date: 2022-01-27 16:29:50
tags: Codeforces
categories: Algorithm
mathjax: true
---

# 总结

- 竞赛时间做出了A、B题，D题完整思路都是对的，但是对整数因式分解的结果进行记忆化时犯了愚蠢的错误，产生了bug。由于该错误较为低级，因此检查时直接忽略了，检查过程中完全没有想到是因式分解的函数出现了bug
- 还没看C题



# [D. GCD of an Array](https://codeforces.com/contest/1493/problem/D)

- 标签：质因数分解、有序列表、埃氏筛
- 分析
  - 将每个元素进行质因数分解，并存储于hashmap中，其最大公因数可以通过hashmap的“交集”求得
  - 但如果每次更新，我们都重新求每个元素的质因数hashmap的交集，则时间复杂度是$O(N^2)$。注意到最大公因数中每个质因数的次数是由所有元素的质因数次数最小者决定的，因此我们可以使用有序列表（multiset）存储各个元素的质因数的次数
  - 求最大公因式时，遍历所有质因数的有序列表，若列表大小为$N$，说明每个元素都包含其质因数$i$，其次数的最小值为multiset的第一个元素$k$，因此乘以$i^k$即可
  - 而元素更新时，总是乘以某个数$x$，因此其质因数的变化与$x$的质因数一致，这时我们只需要更新这部分质因数的有序列表即可。我们对比有序列表更新前后的最小次数$k_1$和$k_2$，若有变化则乘以$i^{k_2-k_1}$
  - 利用埃氏筛可以快速求解$2\times10^5$以下的质数
- 比赛时犯的愚蠢错误
  - 利用记忆化防止对同一元素重复因式分解，但忽略了质因数分解时$x$会变化，因此应该预存储$y=x$，最后才能正确保存分解结果
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
vector<int> primes;
unordered_map<int, unordered_map<int, ll>> memo;
const int mod = 1e9 + 7;
const int MAX_A = 2e5 + 1;

unordered_map<int, ll> factor(int x) {
	if (memo.find(x) != memo.end())
		return memo[x];
	int i = 0;
	unordered_map<int, ll> r{};
    int y = x;
	while (x > 1) {
		while (x % primes[i] == 0) {
			r[primes[i]]++;
            x /= primes[i];
		}
		i++;
	}
	memo[y] = r;
	return r;
}

void merge(unordered_map<int, ll>& d1, unordered_map<int, ll>& d2) {
	for (auto& item: d2) {
		d1[item.first] += item.second;
	}
}

ll fastpow(ll x, ll y) {
	ll r = 1;
	while (y) {
		if (y & 1) {
			r *= x;
			r %= mod;
		}
		y >>= 1;
		x *= x;
		x %= mod;
	}
	return r;
}

void solve(void) {
	int n, q;
	cin >> n >> q;
	vector<int> a(n);
	vector<unordered_map<int, ll>> data(n);
	unordered_map<int, multiset<ll>> d;
	for (int i = 0; i < n; i++) {
		cin >> a[i];
		data[i] = factor(a[i]);
		for (auto& item: data[i]) {
			d[item.first].insert(item.second);
		}
	}
	ll r = 1;
	for (auto& item: d) {
		if ((int)item.second.size() == n) {
			r *= fastpow(item.first, *item.second.begin());
			r %= mod;
		}
	}

	for (int i = 0; i < q; i++) {
		int j, x;
		cin >> j >> x;
		j--;
		auto dx = factor(x);
		merge(data[j], dx);
		for (auto& item: dx) {
			int v = item.first;
			ll cnt = item.second;
			if (cnt == data[j][v]) {
				d[v].insert(cnt);
				if ((int)d[v].size() == n) {
					r *= fastpow(v, *d[v].begin());
					r %= mod;
				}
			} else {
				ll c1 = *d[v].begin();
				ll c0 = data[j][v] - cnt;
				d[v].erase(d[v].find(c0));
				d[v].insert(c0 + cnt);
				if ((int)d[v].size() == n) {
                    ll c2 = *d[v].begin();
                    if (c2 > c1) {
                        r *= fastpow(v, c2 - c1);
                        r %= mod;
                    }
                }
			}
		}
		cout << r << '\n';
	}
}


int main(void) {
	ios::sync_with_stdio(false);
  	cin.tie(nullptr);
  	primes = vector<int>{};
  	vector<int> sieve(MAX_A, 0);
  	for (int i = 2; i < MAX_A; i++) {
  		if (sieve[i] == 0) {
  			primes.push_back(i);
  			for (int j = 2 * i; j < MAX_A; j+= i) {
  				sieve[j] = 1;
  			}
  		}
  	}
	int t = 1;
	while(t--) {
		solve();
	}
	return 0;
}
```

<!--more-->

# [B. Planet Lapituletti](https://codeforces.com/contest/1493/problem/B)

- 标签：二分查找、模拟
- 分析
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
	int h, m;
	cin >> h >> m;
	string s;
	cin >> s;
	vector<int> valid{};
	vector<int> digits{1, 1, 1, 0, 0, 1, 0, 0, 1, 0};
	vector<int> trans{0, 1, 5, 0, 0, 2, 0, 0, 8, 0};
	for (int i = 0; i < h; i++) {
		for (int j = 0; j < m; j++) {
			if (digits[i / 10] && digits[i % 10] && digits[j / 10] && digits[j % 10]) {
				if (trans[j % 10] * 10 + trans[j / 10] < h && trans[i % 10] * 10 + trans[i / 10] < m) {
					valid.push_back(i * m + j);
				}
			}
		}
	}
	int t = ((s[0] - '0') * 10 + (s[1] - '0')) * m + (s[3] - '0') * 10 + s[4] - '0';
	int l = 0, r = (int)valid.size() - 1;
	while (l <= r) {
		int mid = (r - l) / 2 + l;
		if (valid[mid] >= t)
			r = mid - 1;
		else
			l = mid + 1;
	}
	int idx;
	if (r + 1 == (int)valid.size()) {
		idx = 0;
	} else {
		idx = r + 1;
	}
	int hour = valid[idx] / m;
	int min = valid[idx] % m;
	cout << hour / 10  << hour % 10 << ":" << min / 10 << min % 10 << '\n';
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



# [A. Anti-knapsack](https://codeforces.com/contest/1493/problem/A)

- 标签：分析、贪心
- 分析：大于$k$的数都是可选的，对于小于$k$的数，总是选$[(k+1)/2,k-1]$之间的数，这种策略下能选到最多数量的数
- 但其正确性证明有待补充
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
	vector<int> r{};
	for (int i = (k + 1) / 2; i <= n; i++) {
		if (i != k)
			r.push_back(i);
	}
	cout << r.size() << '\n';
	for (auto& v: r)
		cout << v << " ";
	if (r.size() > 0)
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

