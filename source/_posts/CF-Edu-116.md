---
title: CF Edu 116
date: 2022-01-16 19:55:02
tags: Codeforces
mathjax: true
categories: Algorithm
---

# 总结

- 完成了A-C题，D-F题都很难
- E题是组合数学的动态规划，但是状态定义太难想到

# [A. AB Balance](https://codeforces.com/contest/1606/problem/A)

- 标签：模拟
- 分析：ab和ba的数量最多只相差1个，若ab更多则将结尾的b改为a，若ba更多则将结尾的a改为b

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
// const ll INF = 1e18;

void solve(void) {
	string s;
	cin >> s;
	int ab = 0, ba = 0;
	string t;
	t += s[0];
	for (int i = 1; i < (int)s.size(); i++) {
		if (s[i] == 'a' && s[i - 1] == 'b')
			ba++;
		else if(s[i] == 'b' && s[i - 1] == 'a')
			ab++;
		if (i + 1 < (int)s.size())
			t += s[i];
	}
	if (ab == ba) {
		cout << s << endl;
	} else if(ab > ba) {
		cout << t << "a" << endl;
	} else {
		cout << t << "b" << endl;
	}
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

<!--more-->



# [B. Update Files](https://codeforces.com/contest/1606/problem/B)

- 标签：模拟、贪心
- 分析：若当前机器的数量$x$未超过传输线数量$k$，则`x*=2`，否则只能每次加$k$，可直接计算结果

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
// const ll INF = 1e18;

void solve(void) {
	ll n, k;
	cin >> n >> k;
	ll x = 1;
	ll r = 0;
	while (x < n) {
		if (x < k) {
			x += x;
			r += 1;
		} else {
		    if ((n - x) % k)
		        r++;
			r += (n - x) / k;
			break;
		}
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





# [C. Banknotes](https://codeforces.com/contest/1606/problem/C)

- 标签：数位dp
- 分析：由于所有支票的面额都是$10^x$，因此我们可以对十进制数逐位考虑
  - 首先求出每位数字所需使用的支票数量，如只有面值为1、10的支票，则百位加一需要消耗10张支票
  - 我们的目标是求出需要消耗$k+1$张支票的最小数
  - 从低位开始遍历，因为我们的目标是找到最小的数，因此从低位开始尽可能地使用支票

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
// const ll INF = 1e18;

void solve(void) {
	int n, k;
	cin >> n >> k;
	vector<int> a(n, 0);
	for (int i = 0; i < n; i++)
		cin >> a[i];
	vector<ll> cost(18, 0);
	int j = 0;
	for (int i = 0; i < 18; i++) {
		if (j < n && i == a[j]) {
			cost[i] = 1;
			j++;
		} else {
			cost[i] = cost[i - 1] * 10;
		}
	}
	ll c = k + 1;	
	ll r = 0;
	for (int i = 0; i < 18; i++) {
		ll c1 = c / cost[i];
		ll c2 = c % cost[i];
		if (c1 >= 9) {
			c -= cost[i] * 9;
			r += 9 * (ll)pow(10, i);
		} else {
			c -= cost[i] * c1;
			r += c1 * (ll)pow(10, i);
			if (c2) {
				r += (ll)pow(10, i);
				ll c3 = cost[i] - c2;
				int j = i - 1;
				while (c3) {
					ll d = min((ll)9, c3 / cost[j]);
					c3 -= d * cost[j];
					r -= d * (ll)pow(10, j);
					j--;
				}
			}
			break;
		}
	}
	cout << r << endl;


}

int main(void) {
	ios::sync_with_stdio(false);
  	cin.tie(nullptr);
// 	int t = 1;
    int t;
	cin >> t;
	while(t--) {
		solve();
	}
	return 0;
}
```

