---
title: CF 729 (Div 2)
date: 2022-01-25 17:25:57
tags: Codeforces
categories: Algorithm
mathjax: true
---

# 总结

- 难度较高，完成了A-C题
- 题解都非常巧妙，饱含数学原理、思维性，解出来后有一种爽快的成就感
- 对于这种高难度且思维性的竞赛题，以当前水平，能完成前三题已经很满足了



# [B. Plus and Multiply](https://codeforces.com/contest/1542/problem/B)

- 标签：数论、暴力
- 分析
  - 考虑我们只使用相加操作，则当$(n-1)$可以被$b$整除时则满足条件
  - 否则，我们需要使用乘以$a$的操作，无论相乘、相加顺序如何，必定可以转化为先乘以$a$数次，再加上$b$数次的形式（这一点并非显而易见，可以通过列公式化简后观察得到）
  - 如果$a=1$，则退化为上述第一种情况
  - 如果$a=2$，则对于最大值$10^9$，我们也可以在最多31次乘以$a$的操作内找到答案。我们从$x=1$开始不断乘以$a$，并检查$n-x$能否被$b$整除，若可以则返回YES；否则，返回NO
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
	int n, a, b;
	cin >> n >> a >> b;
	if ((n - 1) % b == 0) {
		cout << "YES" << endl;
		return;
	}
	if (a == 1) {
		cout << "NO" << endl;
		return;
	} else {
		ll i = a;
		while (i < n) {
			if (i % b == n % b) {
				cout << "YES" << endl;
				return;
			}
			i *= a;
		}
		if (i == n) {
			cout << "YES" << endl;
			return;
		}
	}
	cout << "NO" << endl;

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



# [C. Strange Function](https://codeforces.com/contest/1542/problem/C)

- 标签：数学、最小公倍数、暴力
- 分析
  - 从1开始求自然数序列的最小公倍数，可以发现当增大到42时，最小公倍数超过了$10^{16}$，即$lcm([1,42])\gt10^{16}$。换言之，对于本题的数据范围，$f(n)<42$必定成立
  - 令$l(i)=lcm([1,i])$代表从1到i的自然数序列的最小公倍数。我们从$i=1$开始不断增大，每次都计算$s(i)=n-n/l(i)$的数量。这代表$[1,n]$中的数有$n/l(i)$个是能被$[1,i]$中的所有数整除的，而$s(i)$个数的最小非因子是小于等于$i$的
  - 通过$s(i)-s(i-1)$我们可以计算出最小非因子等于$i$的数的个数，此时总计数值$r=r+i\times[s(i)-s(i-1)]$
  - 不断遍历，直至$s(i)\geq n$即可，此时所有$[1,n]$范围内的数的$f(i)$函数值都已经累加到答案中
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
const int mod = 1e9 + 7;

ll gcd(ll x, ll y) {
	if (x < y) {
		ll t = y;
		y = x;
		x = t;
	}
	while (y) {
		ll t = x % y;
		x = y;
		y = t;
	}
	return x;
}

ll lcs(ll x, ll y) {
	return x * y / gcd(x, y);
}


void solve(void) {
	ll n;
	cin >> n;
	if (n == 1) {
		cout << 2 << endl;
		return;
	}
	ll r = 0;
	ll f = 2;
	int i = 2;
	ll s = 0;
	while (s < n) {
		r += i * (n - n / f - s) % mod;
		r %= mod;
		s = n - n / f;
		i++;
		f = lcs(f, i);
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

# [A. Odd Set](https://codeforces.com/contest/1542/problem/A)

- 标签：模拟
- 分析：只有奇数、偶数的数量相等时才满足
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
	int even = 0, odd = 0;
	for (int i = 0; i < 2 * n; i++) {
		int t;
		cin >> t;
		if (t % 2 == 0)
			even++;
		else
			odd++;
	}
	if (even == odd) 
		cout << "YES" << '\n';
	else
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



