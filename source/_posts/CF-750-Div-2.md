---
title: CF 750 (Div.2)
date: 2022-01-21 17:33:17
tags: Codeforces
categories: Algorithm
mathjax: true
---

# 总结

- 完成了A-F1题，题目比较简单，思路比较直接
- E题因为一个stupid mistake导致的bug卡了较久
- 不过即使不卡，也很难做出F2和G题
- 规律规律：在一次Div2竞赛中，若完成该题的人数小于500，则该题我几乎不可能做出，若小于1000，则有一定几率可以做出



# [E. Pchelyonok and Segments](https://codeforces.com/contest/1582/problem/E)

- 标签：动态规划
- 分析：$dp(i,j)$表示考虑第$i-n$个元素构成$j$个segment的情况下，最左边的segment（即由$j$个元素组成的segment）的最大和
- 状态转移时
  - 若不考虑位置$i$的segment，则有$dp(i,j)=dp(i+1,j)$
  - 若考虑位置$i$的segment，则如果$sum(i,i+j-1)<dp(i+j,j-1)$，$dp(i,j)=sum(i,i+j-1)$，其中$sum(l,r)$表示从$l$到$r$的segment的和
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
	
	vector<ll> a(n);
	for (int i = 0; i < n; i++)
		cin >> a[i];
	vector<ll> prefix(n);
	for (int i = 0; i < n; i++) {
		if (i == 0)
			prefix[i] = a[i];
		else
			prefix[i] = prefix[i - 1] + a[i];
	}
	vector<vector<ll>> dp(n, vector<ll> (451, -1));
	for (int i = 0; i < n; i++)
		dp[i][0] = 0;
	for (int i = n - 1; i >= 0; i--) {
		if (i == n - 1)
			dp[i][1] = a[i];
		else {
			for (int j = 1; j < 451; j++) {
				if (i + 1 < n)
					dp[i][j] = max(dp[i][j], dp[i + 1][j]);
				if (j == 1) {
					dp[i][j] = max(dp[i][j], a[i]);
				} else {
					if (i + j < n) {
						ll val = prefix[i + j - 1] - prefix[i] + a[i];
						if (dp[i + j][j - 1] > val) {
							dp[i][j] = max(dp[i][j], val);
						}
					}
				}
			}
		}
	}
	int r = 0;
	for (int j = 1; j < 451; j++) {
		if (dp[0][j] > 0)
			r = j;
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

# [F1. Korney Korneevich and XOR (easy version)](https://codeforces.com/contest/1582/problem/F1)

- 标签：动态规划
- 分析：$dp(i)$表示当前想要获得**异或值为$i$的子序列末尾最小值**，我们遍历数组a，对于其元素$a_j$，我们只考虑$dp(i)<a_j$的值$i$，得到异或值$a_j \ xor \ i$并进行状态转移

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

void solve(void) {
	uint n;
	cin >> n;
	vector<uint> a(n);
	for (uint i = 0; i < n; i++)
		cin >> a[i];
	vector<int> dp(512, -1);
	dp[0] = 0;
	for (uint& v: a) {
		for (uint i = 0; i < 512; i++) {
			if (dp[i] >= 0 && dp[i] < (int)v) {
				uint val = i ^ v;
				if (dp[val] == -1 || dp[val] > (int)v)
					dp[val] = (int)v;
			}
		}
	}
	vector<int> res{};
	for (int i = 0; i < 512; i++) {
		if (dp[i] >= 0) {
			res.push_back(i);
		}
	}
	cout << (int)res.size() << endl;
	for (auto& v: res)
		cout << v << " ";
	cout << endl;
}

int main(void) {
	ios::sync_with_stdio(false);
  	cin.tie(nullptr);
	int t = 1;
	while(t--) {
		solve();
	}
	return 0;
}
```



# [A. Luntik and Concerts](https://codeforces.com/contest/1582/problem/A)

- 标签：贪心
- 分析：分类讨论，从3开始分配，尽量使两者相等

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
	int a, b, c;
	cin >> a >> b >> c;
	int d = 0;
	if (c % 2 == 1)
		d = 3;
	else
		d = 0;
	if (b % 2 == 1) {
		if (d == 3) {
			d = 1;
		} else {
			if (c == 0) {
				d = 2;
			} else {
				d = 0;
			}
		}
	} else {
		if (d == 3) {
			if (b == 0)
				d = 3;
			else
				d = 1;
		}
	}
	if (d == 0) {
		d = a % 2;
	} else if(d == 1) {
		d = 1 - a % 2;
	} else if (d == 2) {
		if (a % 2)
			d = 1;
		else if(a == 0)
			d = 2;
		else
			d = 0;
	}
	cout << d << endl;
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

# [B. Luntik and Subsequences](https://codeforces.com/contest/1582/problem/B)

- 标签：数学
- 分析：由于目标子序列的和是$s-1$，因此我们只需要关注数组中的0和1，只有这些元素是不确定的，其他元素都必须选择
  - 假设有$a$个0，$b$个1，则每个0我们都可选可不选，而1只能选择一个，因此总共的方案数是$2^a \cdot b$

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
#include <math.h>
using namespace std;
typedef long long ll;

void solve(void) {
	int n;
	cin >> n;
	
	vector<int> a(n);
	ll zeros = 0, ones = 0;
	for (int i = 0; i < n; i++) {
		cin >> a[i];
		if (a[i] == 0)
			zeros++;
		else if(a[i] == 1)
			ones++;
	}

	ll r = ones * pow(2, zeros);
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



# [C. Grandma Capa Knits a Scarf](https://codeforces.com/contest/1582/problem/C)

- 标签：双指针
- 分析：由于我们只能移除同一个字母，因此对26个字母分别考虑，每次都从两端开始向中间遍历
  - 初始时$l=1$、$r=n$
  - 若`s[l]==s[r]`，则直接向中间移动
  - 若不等，则如果其中存在一个选定的可移除字母，则将其移除
  - 若不存在可移除字母，则表示在只移除该字母的情况下无法构成回文字符串
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
	string s;
	cin >> s;
	int r = -1;
	for (char c = 'a'; c <= 'z'; c++) {
		int t = 0;
		int left = 0, right = n - 1;
		bool flag = true;
		while (left <= right) {
			if (s[left] == s[right]) {
				left++;
				right--;
			} else if (s[left] == c) {
				left++;
				t++;
			} else if (s[right] == c) {
				right--;
				t++;
			} else {
				flag = false;
				break;
			}
		}
		if (flag) {
			if (r == -1 || r > t)
				r = t;
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



# [D. Vupsen, Pupsen and 0](https://codeforces.com/contest/1582/problem/D)

- 标签：构造性问题
- 分析：注意到虽然要求$\sum_{i=1}^{n}|b_i| \leq 10^9$，但是$|a_i|\leq 10^4$，且$n \leq 10^5$，也就是说数组$a$必定满足$\sum_{i=1}^{n}|a_i| \leq 10^9$
- 可以利用这个性质，利用数组a中的元素给b赋值，具体地，如果
  - $n$是偶数，则对于每一对元素，我们令$b_i=-a_{i+1}$，$b_{i+1}=a_i$即可
  - $n$是奇数，则对于除最后三个元素外的元素，仍然按照上述规律赋值，对于最后的三个元素，总是存在两个元素相加不等于0，设为$x$、$y$，剩下的元素设为$z$，则将其对应的b中的元素分别赋值为$z$、$z$、$-(x+y)$
- 注意点：即上述的$n$为奇数的情况，必须判断最后三个元素是否有两个元素相加为0的情况，若有我们不能将其作为$-(x+y)$，否则会导致$b$中出现元素0，不符合题目要求（因为这个点WA了一发）

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
	vector<int> b(n);
	if (n % 2 == 0) {
		for (int i = 0; i < n; i += 2) {
			b[i] = -a[i + 1];
			b[i + 1] = a[i];
		}
	} else {
		for (int i = 0; i < n - 3; i += 2) {
			b[i] = -a[i + 1];
			b[i + 1] = a[i];
		}
		if (a[n - 3] + a[n - 2] != 0) {
			b[n - 3] = a[n - 1];
			b[n - 2] = a[n - 1];
			b[n - 1] = -(a[n - 3] + a[n - 2]);
		} else if (a[n - 3] + a[n - 1] != 0) {
			b[n - 3] = a[n - 2];
			b[n - 1] = a[n - 2];
			b[n - 2] = -(a[n - 3] + a[n - 1]);
		} else {
			b[n - 2] = a[n - 3];
			b[n - 1] = a[n - 3];
			b[n - 3] = -(a[n - 2] + a[n - 1]);
		}
	}
	for (auto& v: b) {
		cout << v << " ";
	}
	cout << endl;

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

