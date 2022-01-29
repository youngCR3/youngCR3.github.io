---
title: CF 746 (Div 2)
date: 2022-01-22 20:29:14
tags: Codeforces
categories: Algorithm
mathjax: true
---

# 总结

- 本次竞赛难度较高，除了A-C题外，其余题目都只有小于1000人完成；此外，C题需要正确分析出答案，也需花不少时间
- 竞赛时间只完成了A、B题；C题思路不够清晰，没发现程序的bug，赛后补完
- C题：假如一棵子树$T_1$的异或和为0，且该子树的子树中存在异或和为target的情况$T_2$**（比赛时忽略必须存在异或和为target的子树的子树的条件，误以为找到异或和为0的子树即可）**



# [C. Bakry and Partitioning](https://codeforces.com/contest/1592/problem/C)

- 标签：树、dfs
- 分析
  - 假如树中所有节点的值异或和为0，则任意将树分为两部分，其异或和必定相等，返回Yes
  - 假如树中所有节点的值异或和非0，此时必定只能将树分为奇数部分。而且必定可以将树分为3部分，这是因为假如我们分为超过三部分，可以通过不断地将三部分合成一部分使得其变为3部分（相当于不断减2）
- 因此我们需要考虑能否将树分为3部分，使其异或值相等，均等于原来所有节点值的异或和$target$
  - 假如我们存在两棵不重叠的子树，其异或和均为target，则将这两棵子树分开后，剩下的树的异或和也必定为target
  - 此外，假如一棵子树$T_1$的异或和为0，且该子树的子树中存在异或和为target的情况$T_2$**（比赛时忽略了这一点，误以为找到异或和为0的子树即可）**，则该子树的子树$T_2$分离后，该子树的剩下部分$T_1-T_2$的异或和也必定等于target

## 常数优化

- 本题需要用DFS求解，其中一种思路就是返回该节点为根的子树中，任意子树的异或和的集合。但这样DFS函数返回时就要创建一个新的集合，空间复杂度是$O(N^2)$
- 正确的做法应该是只使用一个哈希表data，然后将每个子树的异或和添加到该data中。但这样做会带来新的问题
  - 首先是找到异或和为target的子树时，我们只有在另外一个分支也有异或和为target的子树时才返回true。如果使用同一个data，则无法确保该和为target的子树是否来自其他分支，正确做法是在DFS子树时先记录`data[target]`的数值，若大于0则保证必定有来自其他分支的有效子树
  - 另外，找到异或和为0的子树时，我们需要确认该子树上存在异或和为target的子子树。解决方法同样是在DFS子树时先记录`data[target]`的数值$old$，并在DFS后将其与`data[target]`进行对比，若DFS后数值变大了，说明该分支子树上的确存在异或和为target的子树

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
bool flag;


int dfs(unordered_map<int, int>& data, vector<int>& a, vector<vector<int>>& edges, int node, int pa, int target) {
	int val = 0;
	int cnt = 0;
	if (data.find(target) != data.end())
		cnt = data[target];
	for (auto& v: edges[node]) {
		if (v != pa) {
			auto p = dfs(data, a, edges, v, node, target);
			val ^= p;
			if (flag)
				return 0;
		}
	}
	val ^= a[node];
	if (val == 0 && data.find(target) != data.end() && data[target] > cnt) {
		flag = true;
		return 0;
	} else if (val == target && cnt > 0) {
		flag = true;
		return 0;
	}
	data[val]++;
	return val;
}

void solve(void) {
	flag = false;
	
	int n, k;
	cin >> n >> k;
	vector<int> a(n + 1);
	int v = 0;
	for (int i = 1; i <= n; i++) {
		cin >> a[i];
		v ^= a[i];
	}
	vector<vector<int>> edges(n + 1);
	for (int i = 0; i < n - 1; i++) {
		int u, v;
		cin >> u >> v;
		edges[u].push_back(v);
		edges[v].push_back(u);
	}
	if (v == 0) {
		cout << "YES" << endl;
		return;
	}
	if (k - 1 >= 2) {
		unordered_map<int, int> data;
		dfs(data, a, edges, 1, -1, v);
		if (flag) {
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



<!--more-->

# [B. Hemose Shopping](https://codeforces.com/contest/1592/problem/B)

- 标签：构造性问题
- 分析
  - 假如数组的长度大于等于2x，则任意两个元素都可以交换，返回Yes
  - 假如数组的长度大于等于x，小于2x，且$n \ mod \ x = i$，则第$i+1$个到第$n$个元素无法与任何元素交换，此时检查它们是否与排序后的数值一致
  - 假如数组长度小于x，则所有元素都无法改变，除非数组本身有序才返回Yes
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
	int n, x;
	cin >> n >> x;
	vector<int> a(n);
	vector<int> b(n);
	for (int i = 0; i < n; i++) {
		cin >> a[i];
		b[i] = a[i];
	}
	sort(b.begin(), b.end());
	if (n / x >= 2) {
		cout << "YES" << endl;
		return;
	}
	else if (n / x == 1) {
		for (int i = n % x; i < x; i++) {
			if (a[i] != b[i]) {
				cout << "NO" << endl;
				return;
			}
		}
	} else {
		for (int i = 0; i < n; i++) {
			if (a[i] != b[i]) {
				cout << "NO" << endl;
				return;
			}
		}
	}	
	
	cout << "YES" << '\n';
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



# [A. Gamer Hemose](https://codeforces.com/contest/1592/problem/A)

- 标签：模拟
- 分析：必定选取两把伤害最高的武器$w_1$、$w_2$切换使用，一组的伤害为$w_1+w_2$，计算$H \ mod \ (w_1+w_2)$，若该值大于$w_1$则还需额外两次攻击，若大于0但小于等于$w_1$则还需额外一次攻击，若为0则无需额外攻击
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
	int n, h;
	cin >> n >> h;
	vector<int> a(n);
	for (int i = 0; i < n; i++)
		cin >> a[i];
	sort(a.begin(), a.end());
	int x = a[n - 1] + a[n - 2];
	int r = h / x * 2;
	h %= x;
	if (h > a[n - 1])
		r += 2;
	else if (h > 0)
		r += 1;
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

