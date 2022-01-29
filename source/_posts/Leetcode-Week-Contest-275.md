---
title: Leetcode Week Contest 275
date: 2022-01-09 15:15:40
tags: Leetcode
categories: Algorithm
mathjax: true
---



# 总结

- 排名国服45，全服86

- 第三题一开始看错题，虽然通过测试用例发现了错误并改正，但程序中一处细节未修改过来还是出错了，WA了一发导致排名降低
- 第四题的思路很容易想出，但正确性却并非显而易见，抱着试一试的心态AC了
- 经历了连续两场百名开外，这场终于又回到了前50



# [5977. 最少交换次数来组合所有的 1 II](https://leetcode-cn.com/problems/minimum-swaps-to-group-all-1s-together-ii/)

- 标签：滑动窗口
- 分析：首先统计数组中1的个数$m$，滑动窗口统计各个大小为$c$的窗口中1数量的最大值$max\_c$，最终答案为$m-max\_c$
  - 由于数组首尾相接，因此总共包含$n$个窗口
- 代码

```c++
class Solution {
public:
    int minSwaps(vector<int>& nums) {
        int m = 0;
        for (auto &v: nums)
            m += v;
        int c = 0;
        for (int i = nums.size() - m; i < nums.size(); i++)
            c += nums[i];
        int r = c;
        for (int i = 0; i < nums.size(); i++) {
            c += nums[i];
            c -= nums[(i + nums.size() - m) % nums.size()];
            r = max(r, c);
        }
        return m - r;
    }
};
```

<!--more-->



# [5978. 统计追加字母可以获得的单词数](https://leetcode-cn.com/problems/count-words-obtained-after-adding-a-letter/)

- 标签：位运算 + 哈希表
- 分析
  - 需要注意到操作只能添加一个小写字母，且每个小写字母在一个单词中最多只会出现一次
  - 因此对于一个单词可用bitmask表示，使用位运算表示该字母是否出现，并将`startWords`的每个单词的bitmask结果存于哈希表中
  - 对于`targetWords`中的每个单词，计算其bitmask后尝试将每个为1的位变为0，并检查变换后的数是否存在于哈希表中，若存在则该单词可通过转换操作获得
  - 注意必须至少添加1个字母，因此bitmask本身出现在哈希表中不算数
- 代码

```c++
class Solution {
public:
    int wordCount(vector<string>& startWords, vector<string>& targetWords) {
        unordered_set<int> data;
        for (auto &s: startWords) {
            int mask = 0;
            for (auto &c: s) {
                mask |= 1 << (c - 'a');
            }
            data.insert(mask);
        }
        int r = 0;
        for (auto &s: targetWords) {
            int mask = 0;
            for (auto &c: s) {
                mask |= 1 << (c - 'a');
            }

            for (auto &c: s) {
                int mask1 = mask ^ (1 << (c - 'a'));
                if (data.count(mask1)) {
                    r += 1;
                    break;
                }
            }
        }
        return r;
    }
};
```



# [5979. 全部开花的最早一天](https://leetcode-cn.com/problems/earliest-possible-day-of-full-bloom/)

- 标签：排序
- 分析：按花的`growTime`降序排列，从`growTime`最长的开始种植
- 正确性证明：参考[灵茶山艾府dl的题解](https://leetcode-cn.com/problems/earliest-possible-day-of-full-bloom/solution/tan-xin-ji-qi-zheng-ming-by-endlesscheng-hfwe/)
- 代码

```python
class Solution:
    def earliestFullBloom(self, plantTime: List[int], growTime: List[int]) -> int:
        a = []
        for i in range(len(plantTime)):
            a.append([plantTime[i], growTime[i]])
        a.sort(key=lambda s: s[1], reverse=True)
        t = 0
        r = 0
        # print(a)
        for i in range(len(a)):
            t += a[i][0]
            r = max(r, t + a[i][1])
        return r
```



# [5976. 检查是否每一行每一列都包含全部整数](https://leetcode-cn.com/problems/check-if-every-row-and-column-contains-all-numbers/)

- 标签：模拟
- 代码

```c++
class Solution {
public:
    bool checkValid(vector<vector<int>>& matrix) {
        int m = matrix.size(), n = matrix[0].size();
        for (int i = 0; i < m; i++) {
            unordered_set<int> s;
            for (int j = 0; j < n; j++)
                s.insert(matrix[i][j]);
            if (s.size() != n)  
                return false;
        }
        for (int j = 0; j < n; j++) {
            unordered_set<int> s;
            for (int i = 0; i < m; i++)
                s.insert(matrix[i][j]);
            if (s.size() != m)
                return false;
        }
        return true;
    }
};
```

