---
title: Leetcode Bi-Week Contest 69
date: 2022-01-09 13:30:38
tags: Leetcode
categories: Algorithm
mathjax: true
---

# 总结

- 排名国服126，全服251
- 第四题在思路不清晰的情况下提交了一两次，导致了不必要的罚时。另外，中间陷入了错误的解题思路中，幸好后面及时想到了反例，回到了正确的轨道上，及时止损



# [5931. 用邮票贴满网格图](https://leetcode-cn.com/problems/stamping-the-grid/)

## 错误思路

- 算法
  - 统计每一行的连续空格子宽度，若某个空格宽度小于$stampWidth$则返回False
  - 统计每一列的连续空格子高度，若某个空格高度小于$stampHeight$则返回False
  - 否则返回True

- 反例：$stampWidth=2$，$stampHeight=2$，则根据上述算法将返回True，但实际上无法用邮票填充所有空格子

![](case.png)

## 正确解法

- 分析
  - 从左到右、从上到下遍历每个格子，若发现以该格子为右下角能放下一张邮票，则放置
  - 标记被邮票覆盖的空格子
  - 最后统计每个空格子是否都被标记
- 判断能否放下邮票的算法：二维前缀和
  - 对于格子$(i,j)$，若发现以$(i-stampHeight+1,j-stampHeight+1)$为左上角，以$(i,j)$为右下角的区域内都是0，则可以放下一张邮票，通过二维前缀和可以实现在$O(mn)$时间复杂度内遍历所有格子
- 标记空格子的算法：二维差分
  - 若每次放邮票都标记其覆盖的每个格子，则时间复杂度为$O(m^2n^2)$，是不可行的
  - 维护一个计数值$cnt$，若当前格子是邮票右边界则$cnt$加一，若当前格子是邮票左边界的左侧格子则$cnt$减一
  - 同时还需要维护邮票的行信息，若当前行已经超过了该邮票行的上边界则将该邮票移除
  - 当遍历到某个空格子时，若邮票计数值$cnt$为0则说明该空格子没被覆盖到，返回False
  - 若遍历完成，则返回True
- 代码

```c++
class Solution {
public:
    bool possibleToStamp(vector<vector<int>>& grid, int stampHeight, int stampWidth) {
        int m = grid.size(), n = grid[0].size();
        vector<vector<int>> prefix(m, vector<int>(n, 0));
        vector<vector<int>> stamps(m, vector<int>(n, 0));

        for (int i = 0; i < m; i++) {
            int s = 0;
            for (int j = 0; j < n; j++) {
                s += grid[i][j];
                if (i > 0)
                    prefix[i][j] += prefix[i - 1][j];
                prefix[i][j] += s;
                if (i >= stampHeight - 1 && j >= stampWidth - 1) {
                    int v = prefix[i][j];
                    if (i - stampHeight >= 0)
                        v -= prefix[i - stampHeight][j];
                    if (j - stampWidth >= 0)
                        v -= prefix[i][j - stampWidth];
                    if (i - stampHeight >= 0 && j - stampWidth >= 0)
                        v += prefix[i - stampHeight][j - stampWidth];
                    if (v == 0) 
                        stamps[i][j] = 1;
                }

            }
        }

        int c = 0;
        vector<unordered_set<int>> valid1(m, unordered_set<int>{});
        vector<unordered_set<int>> valid2(n, unordered_set<int>{});
        for (int i = m - 1; i >= 0; i--) {
            c = 0;
            for (int j = n - 1; j >= 0; j--) {
                if (stamps[i][j]) {
                    valid2[j].insert(i);
                    valid1[i].insert(j);
                }
                c += valid2[j].size();
                if (j + stampWidth < n) {
                    c -= valid2[j + stampWidth].size();
                }
                if (grid[i][j] == 0 && c == 0)
                    return false;
            }
            if (i + stampHeight < m) {
                for (auto &y: valid1[i + stampHeight]) {
                    valid2[y].erase(i + stampHeight);
                }
            }
        }
        return true;

    }
};
```

<!--more-->

# [5960. 将标题首字母大写](https://leetcode-cn.com/problems/capitalize-the-title/)

- 标签：模拟
- 代码

```c++
class Solution {
public:
    string capitalizeTitle(string title) {
        vector<string> a;
        string t;
        for (auto &c: title) {
            if (c == ' ') {
                if (t.size()) {
                    a.push_back(t);
                    t = "";
                }
            } else {
                t += c;
            }
        } 
        if (t.size())
            a.push_back(t);
        string r;
        for (auto &s: a) {
            if (r.size())
                r += " ";
            transform(s.begin(),s.end(),s.begin(),::tolower);
            if (s.size() > 2) {
                transform(s.begin(),s.begin() + 1,s.begin(),::toupper);
            } 
            r += s;
        }
        return r;
    }
};
```



# [5961. 链表最大孪生和](https://leetcode-cn.com/problems/maximum-twin-sum-of-a-linked-list/)

- 标签：模拟
- 分析：用数组保存链表的值然后双指针即可
- 代码

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    int pairSum(ListNode* head) {
        vector<int> a{};
        while (head != nullptr) {
            a.push_back(head->val);
            head = head->next;
        }
        int l = 0, r = a.size() - 1;
        int v = 0;
        while (l < r) {
            v = max(v, a[l] + a[r]); 
            l++;
            r--;
        }
        return v;
    }
};
```



# [5962. 连接两字母单词得到的最长回文串](https://leetcode-cn.com/problems/longest-palindrome-by-concatenating-two-letter-words/)

- 标签：哈希表
- 分析
  - 所有单词都是两字母构成的，因此一对互为逆字符串的单词可以分布于字符串两侧从而构成回文串
  - 此外，如果一个字符串本身就是回文串，那么它也可以作为最中心的两个字母构成回文串
  - 哈希表统计所有单词数量，并对中心回文串进行特殊判断即可
- 代码

```c++
class Solution {
public:
    int longestPalindrome(vector<string>& words) {
        unordered_map<string, int> c;
        for (auto &w: words) {
            c[w]++;
        }        
        int r = 0;
        bool flag = false;
        unordered_set<string> vis;
        for (auto &w: words) {
            if (vis.find(w) != vis.end())
                continue;
            vis.insert(w);
            if (w[0] == w[1]) {
                r += 4 * (c[w] / 2);
                if (!flag && c[w] % 2) {
                    r += 2;
                    flag = true;
                }
            } else {
                string w1 = {w[1], w[0]};
                vis.insert(w1);
                r += 4 * min(c[w], c[w1]);
            }
        }
        return r;
    }
};
```



