---
title: CF 658 (Div 2)
date: 2022-01-22 11:58:54
tags: Codeforces
categories: Algorithm
mathjax: true
---

# 总结

- 完成了A-D题，E是难度为2600的构造题，直接跳过
- 总体难度比较简单，E题运用了单调栈 + dp，算是一道综合多种算法的题目，但由于题意容易理解，根据用例不难想到正确思路



# [D. Unmerge](https://codeforces.com/contest/1382/problem/D)

- 标签：单调栈 + 动态规划
- 分析
  - 规则：对于元素$x$，其后跟着的小于$x$的元素必定跟它属于同一个数组，直到找到第一个大于$x$的元素$next(x)$
  - 因此，我们可以利用单调栈找到各个元素$a_i$其后的第一个大于它的元素$next(a_i)$
  - 然后，可以将数组“切成”多个片段，并将各个片段的长度保存在$segs$中
  - 然后利用dp判断是否可以将所有片段分为两组使其长度相等，$dp(i,j)$表示将前$i$个片段分为两组，且长度差为$j$的可能性，$j$的取值范围是$[0,n]$
- 时间复杂度：$O(N^2)$

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
#include <stack>
using namespace std;
typedef long long ll;

void solve(void) {
	int n;
	cin >> n;
	
	vector<int> a(2 * n);
	for (int i = 0; i < 2 * n; i++)
		cin >> a[i];

	stack<pair<int, int>> stk;
	vector<int> next(2 * n);
	vector<int> segs{};
	for (int i = 2 * n - 1; i >= 0; i--) {
		while (!stk.empty() && stk.top().first < a[i]) {
			stk.pop();
		}
		if (stk.empty()) {
			next[i] = 2 * n;
		} else {
			next[i] = stk.top().second;
		}
		stk.push(make_pair(a[i], i));
	}
	int i = 0;
	while (i < 2 * n) {
		segs.push_back(next[i] - i);
		i = next[i];
	}
	int m = (int)segs.size();

	vector<vector<int>> dp(m, vector<int>(n + 1));
	if (segs[0] <= n)
		dp[0][segs[0]] = 1;
	for (int i = 0; i < m - 1; i++) {
		for (int j = 0; j <= n; j++) {
			if (dp[i][j]) {
				if (j + segs[i + 1] <= n) {
					dp[i + 1][j + segs[i + 1]] = 1;
				}
				if (abs(j - segs[i + 1]) <= n) {
					dp[i + 1][abs(j - segs[i + 1])] = 1;
				}
			}
		}
	}
	if (dp[m - 1][0]) {
		cout << "YES" << endl;
	} else {
		cout << "NO" << endl;
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

# [C2. Prefix Flip (Hard Version)](https://codeforces.com/contest/1382/problem/C2)

- 标签：双指针、奇偶性

- operation的定义

  从右到左遍历字符串，若相等则无需操作，若当前字符不等，则需要进行操作。由于一次操作会将首字符反转并调换到当前位置，因此如果首字符与当前字符不等则操作后会相等；若首字符与当前字符相等，则首先反转首字符，然后再执行操作

- 确定各个字符当前的值

  - 由于每次操作都涉及反转以及位置调换，若直接模拟复杂度为$O(N^2)$
  - 观察发现，若遍历到第$i$位我们需要对一个长度为$i$子串进行操作，操作后相当于第$i$位的字符被固定，此时该字符必定是该子串中位于两端的字符之一，可能是左端可能是右端
  - 我们维护两个指针left、right，分别表示当前位于两端的字符在原字符串中的索引
  - 同时维护翻转次数op，若为奇数表示发生了位置调换，且0、1状态反转；若为偶数则和原顺序一致，且0、1状态一致
  - 特别地，我们可能需要单独反转首字符，此时我们不更新op，而是直接对原字符串中的字符同步进行反转

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
	string a, b;
	cin >> a >> b;
	vector<int> r{};
	vector<char> s{};
	for (auto& ch: a)
		s.push_back(ch);
	int left = 0, right = n - 1;
	int op = 0;
	for (int i = n - 1; i >= 0; i--) {
		if (op % 2) {
			char ch = '0' + '1' - s[left];
			if (ch != b[i]) {
				char ch1 = '0' + '1' - s[right];
				if (ch1 == b[i]) {
					r.push_back(1);
					s[right] = '0' + '1' - s[right];
				}
				r.push_back(i + 1);
				op++;
				right--;
			} else {
				left++;
			}
		} else {
			char ch = s[right];
			if (ch != b[i]) {
				char ch1 = s[left];
				if (ch1 == b[i]) {
					r.push_back(1);
					s[left] = '0' + '1' - s[left];
				}
				r.push_back(i + 1);
				op++;
				left++;
			} else {
				right--;
			}
		}
	}
	cout << (int)r.size() << " ";
	for (auto& v: r) {
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

# [C1. Prefix Flip (Easy Version)](https://codeforces.com/contest/1382/problem/C1)

- 标签：模拟
- 分析
  - operation的定义：从右到左遍历字符串，若相等则无需操作，若当前字符不等，则需要进行操作。由于一次操作会将首字符反转并调换到当前位置，因此如果首字符与当前字符不等则操作后会相等；若首字符与当前字符相等，则首先反转首字符，然后再执行操作
  - 每次反转时，都直接在原字符串上操作即可
- 时间复杂度：$O(N^2)$
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
	string a, b;
	cin >> a >> b;
	vector<int> r{};
	vector<char> c{};
	for (auto& ch: a)
		c.push_back(ch);
	for (int i = n - 1; i >= 0; i--) {
		if (c[i] != b[i]) {
			if (c[0] == b[i]) {
				r.push_back(1);
				c[0] = '0' + '1' - c[0];
			}

			r.push_back(i + 1);
			int left = 0, right = i;
			while (left <= right) {
				char ch = c[left];
				c[left] = '0' + '1' - c[right];
				c[right] = '0' + '1' - ch;
				left++;
				right--;
			}

		}
 	}
 	cout << (int)r.size() << " ";
 	for (auto& v: r) {
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



# [B. Sequential Nim](https://codeforces.com/contest/1382/problem/B)

- 标签：博弈论、模拟
- 分析
  - 从最后一个石堆分析，若能够先手到达最后一个石堆，则必定可以直接拿走所有石子，使得另一玩家失利，因此先到达最后石堆者必胜
  - 逆序分析，假如先到达第$i+1$个石堆者获胜
    - 而第$i$个石堆有多于一个石子，则先到达第$i$个石堆者获胜，因为他可以拿走其他石子使得第$i$堆只剩一个石子，从而使另一玩家只能拿走该石子，这样他就可以先到达第$i+1$个石堆，保证胜利
    - 若第$i$个石堆只有一个石子，则后到达第$i$个石堆者获胜
  - 假如后到达第$i+1$个石堆者获胜，则先到达第$i$个石堆者必胜，因为它可以拿走所有石子，使得对手必定先到达第$i+1$个石堆，而自己必胜
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
	bool flag = true;
	for (int i = n - 1; i >= 0; i--) {
		if (i == n - 1) {
			flag = true;
		} else {
			if (flag) {
				if (a[i] == 1) {
					flag = false;
				}
			} else {
				flag = true;
			}
		}
	}
	if (flag) {
		cout << "First" << endl;
	} else {
		cout << "Second" << endl;
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


# [A. Common Subsequence](https://codeforces.com/contest/1382/problem/A)

- 标签：贪心
- 分析
  - 若两个数组没有公共元素，则为NO
  - 否则，找到最小的公共元素即可
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
	vector<int> b(m);
	unordered_set<int> as;
	unordered_set<int> bs;
	for (int i = 0; i < n; i++) {
		cin >> a[i];
		as.insert(a[i]);
	}
	for (int j = 0; j < m; j++) {
		cin >> b[j];
		bs.insert(b[j]);
	}
	int r = -1;
	for (auto& v: as) {
		if (bs.find(v) != bs.end()) {
			if (r == -1 || v < r)
				r = v;
		}
	}
	if (r == -1) {
		cout << "NO" << endl;
	} else {
		cout << "YES" << endl;
		cout << 1 << " " << r << endl;
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

