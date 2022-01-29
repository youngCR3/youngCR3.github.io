---
title: CF 767 (Div 2)
date: 2022-01-23 16:04:56
tags: Codeforces
categories: Algorithm
mathjax: true
---

# 总结

- 完成了A-E题，其中存在多条构造性问题，提交时并不确定（幸运的是都AC了），但A-E题并未存在难度特别高的题目
- 排名Div 2的97！Personal Best！Cheer up！

- Rating到达了1890，还差10分就能升紫！

# [E. Grid Xor](https://codeforces.com/contest/1629/problem/E)

- 标签：构造性问题
- 分析：对于每个格子，其邻近的最多四个格子中，必须**有奇数个格子参与异或**，这样就能保证每个格子的值都参与了异或
  - 如何保证此条件呢，通过观察和猜想发现，（用涂色代表参与了异或），将第一行的格子都涂色，然后此后每行的格子是否涂色是根据其上方格子目前相邻的涂色格子数决定 的
  - 假如有上方格子有奇数个格子涂色，则此格子不涂色
  - 假如上方格子有偶数个格子涂色，则此格子涂色
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
	vector<vector<int>> grid(n, vector<int>(n));
	for (int i = 0; i < n; i++) {
		for (int j = 0; j < n; j++)
			cin >> grid[i][j];
	}
	vector<vector<int>> dp(n, vector<int>(n));
	int r = 0;
	for (int i = 0; i < n; i++) {
		for (int j = 0; j < n; j++) {
			if (i == 0) {
				dp[i][j] = 1;
				r ^= grid[i][j];
			} else {
				int x = i - 1, y = j;
				int cnt = 0;
				if (x - 1 >= 0)
					cnt += dp[x - 1][y];
				if (y - 1 >= 0)
					cnt += dp[x][y - 1];
				if (y + 1 < n)
					cnt += dp[x][y + 1];
				if (cnt % 2 == 0) {
					r ^= grid[i][j];
					dp[i][j] = 1;
				}
			}
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



<!--more-->

# [D. Peculiar Movie Preferences](https://codeforces.com/contest/1629/problem/D)

- 标签：字符串、哈希表
- 分析：利用字符串长度不大于3的特点
  - 假如存在字符串本身为回文串，则返回YES；注意到，长度为1的字符串必为回文串
  - 否则，数组中只存在长为2或3的字符串。此时用哈希表存储各个字符串的下标
    - 对于长为2的字符串，查找其反转字符串是否存在，若存在则返回YES
    - 对于长为3的字符串，有三种情况：首先查找其反转字符串是否存在；然后查找`s[1]+s[0]`构成的字符串是否存在，且下标比其下标大；然后查找`s[2]+s[1]`构成的字符串是否存在，且下标比其下标小。这三种情况任一符合，则返回YES
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
	vector<string> a(n);
	unordered_map<string, vector<int>> data;
	for (int i = 0; i < n; i++) {
		cin >> a[i];
		data[a[i]].push_back(i);
	}
	int i = 0;
	for (auto& s: a) {
		if (s.size() == 1) {
			cout << "YES"<< endl;
			return;
		} else if (s.size() == 2) {
			string t;
			t += s[1];
			t += s[0];
			if (data.find(t) != data.end()) {
				cout << "YES"<< endl;
				return;
			}
		} else if (s.size() == 3) {
			string t1;
			t1 += s[2];
			t1 += s[1];
			t1 += s[0];
			if (data.find(t1) != data.end()) {
				cout << "YES"<< endl;
				return;
			}
			string t2;
			t2 += s[1];
			t2 += s[0];
			if (data.find(t2) != data.end()) {
				for (auto& v: data[t2]) {
					if (v > i) {
						cout << "YES"<< endl;
						return;
					}
				}

			}
			string t3;
			t3 += s[2];
			t3 += s[1];
			if (data.find(t3) != data.end()) {
				for (auto& v: data[t3]) {
					if (v < i) {
						cout << "YES"<< endl;
						return;
					}
				}

			}
		}
		++i;
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



# [C. Meximum Array](https://codeforces.com/contest/1629/problem/C)

- 标签：队列、构造性问题、贪心
- 分析：题目希望要求最大的数组b，因此我们总是希望当前的MEX尽可能地大。
  - 注意到MEX随数组长度不断增加单调递增，因此我们总是先求整个数组的MEX（第一次遍历）
  - 然后（第二次遍历）从当前首位置不断往右遍历，直至当前的MEX值等于整个数组，那么此时我们可以截断数组，并添加该MEX值至答案数组
  - 不断重复该过程，直至到达该数组尾部
- 具体实现
  - 第一次遍历，我们将当前数组每个元素的值$a_i$都添加到该值对应的队列$q(a_i)$中，多个队列通过一个map保存
  - 第二次遍历，我们遍历map中的每个元素（注意map是有序的），若发现当前元素大于目标值（目标值即从0开始每次增加1），则找到了当前整个数组的MEX值。遍历过程中，我们还需要保存当前这些队列的队首元素（其含义是值为val的元素在当前数组中的最小下标）的最大值$index$。然后我们将MEX值添加到答案数组，并更新$index++$代表我们截断了这部分数组
  - 截断数组时，我们只需要遍历每个队列，若队首元素小于$index$则将其出队即可
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
	vector<queue<int>> q(n + 1);
	set<int> data;
	for (int i = 0; i < n; i++) {
		cin >> a[i];
		q[a[i]].push(i);
		data.insert(a[i]);
	}
	int index = 0;
	vector<int> r{};
	while (index < n) {
		auto tmp = data;
		for (auto& v: tmp) {
			while (!q[v].empty() && q[v].front() < index)
				q[v].pop();
			if (q[v].empty())
				data.erase(v);
		}
		int val = 0;
		bool flag = false;
		for (auto it = data.begin(); it != data.end(); it++) {
			if (*it != val) {
				r.push_back(val);
				index++;
				flag = true;
				break;
			} else {
				index = max(index, q[val].front());
				val++;
			}
		}
		if (!flag) {
			r.push_back(val);
			break;
		}
	}
	cout << (int) r.size() << endl;
	for (auto& v: r)
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



# [B. GCD Arrays](https://codeforces.com/contest/1629/problem/B)

- 标签：数学、观察
- 分析：需要注意到，在连续序列中，最频繁的因数必定是2，其频数为序列中偶数个数。换言之，我们需要计算序列中奇数的数量，并将其与$k$对比即可
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
	int l, r, k;
	cin >> l >> r >> k;
	if (l == r && l > 1)
		cout << "YES" << endl;
	else {
		int x = (r + 1) / 2 - (l) / 2;
		if (x > k) {
			cout << "NO" << endl;
		} else {
			cout << "YES" << endl;
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



# [A. Download More RAM](https://codeforces.com/contest/1629/problem/A)

- 标签：排序、模拟
- 分析：我们总是从$a_i$最小的RAM开始，每次加上$b_i$，直至$a_i$大于我们当前的$k$，此时我们无法再添加任何RAM
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
	vector<vector<int>> a(n, vector<int>(2));
	for (int i = 0; i < n; i++)
		cin >> a[i][0];
	for (int i = 0; i < n; i++)
		cin >> a[i][1];
	sort(a.begin(), a.end());
	for (int i = 0; i < n; i++) {
		if (a[i][0] <= k)
			k += a[i][1];
		else
			break;
	}
	cout << k << endl;
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

