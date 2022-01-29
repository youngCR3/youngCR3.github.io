---
title: Leetcode 数位动态规划
date: 2022-01-16 12:47:27
tags: Leetcode
mathjax: true
categories: Algorithm
---



# [600. 不含连续1的非负整数](https://leetcode-cn.com/problems/non-negative-integers-without-consecutive-ones/)

- 分析：按数字的二进制逐位进行动态规划，$dp(i,j)$根据$j$的值不同有以下四种含义
  - $j=0$，代表第$i$位是0的数的数量
  - $j=1$，代表第$i$位是1的数的数量
  - $j=2$，代表第$i$位是0，且后缀（即第$0-i$位）也小于等于$n$的数的数量
  - $j=3$，代表第$i$位是1，且后缀（即第$0-i$位）也小于等于$n$的数的数量
- 状态转移
  - $dp(i,0)=dp(i-1,0)+dp(i-1,1)$，因为第$i$位是0，则上一位是0或1都可以
  - $dp(i,1)=dp(i-1,0)$，因为第$i$位是1，则上一位只能是0
  - $dp(i,2)$的更新与$n$的值有关
    - 若$n$的第$i$位为0，则其后的位必须保证后缀小于等于$n$，即$dp(i,2)=dp(i-1,2)+dp(i-1,3)$
    - 若$n$的第$i$位为1，则可以保证后缀小于$n$，因此$dp(i,2)=dp(i-1,0)+dp(i-1,1)$
  - 同理，$dp(i,3)$的更新如下
    - 若$n$的第$i$位为0，则$dp(i,3)=0$
    - 若$n$的第$i$位位1，则$dp(i,3)=dp(i-1,2)$

- 代码

```cpp
class Solution {
public:
    int findIntegers(int n) {
        vector<vector<int>> dp(31, vector<int>(4, 0));
        for (int i = 0; i < 31; i++) {
            if (i == 0) {
                dp[i][0] = 1;
                dp[i][1] = 1;
                if (n & (1 << i)) {
                    dp[i][2] = 1;
                    dp[i][3] = 1;
                } else {
                    dp[i][2] = 1;
                    dp[i][3] = 0;
                }
            } else {
                dp[i][0] = dp[i - 1][0] + dp[i - 1][1];
                dp[i][1] = dp[i - 1][0];
                if (n & (1 << i)) {
                    dp[i][2] = dp[i][0];
                    dp[i][3] = dp[i - 1][2];
                } else {
                    dp[i][3] = 0;
                    dp[i][2] = dp[i - 1][2] + dp[i - 1][3];
                }
            }
        }
        return dp[30][2] + dp[30][3];
    }
};
```

<!--more-->

# [1067. 范围内的数字计数](https://leetcode-cn.com/problems/digit-count-in-range/)

```cpp
class Solution {
public:
    int cnt(int d, int x) {
        // 返回数字d在0-x之间出现的次数
        if (x == 0) {
            if (d == 0)
                return 1;
            return 0;
        }
        long long base = 10;
        int r = 0;
        while (10 * x >= base) {
            int c1 = x / base;
            r += c1 * (base / 10);
            int c2 = x % base;
            if (c2 >= (d + 1) * (base / 10)) {
                r += base / 10;
            } else if (c2 >= d * (base / 10)) {
                r += c2 - d * (base / 10) + 1;
            }
            if (d == 0 && base > 10)
                r -= base / 10;
            base *= 10;
        }
        
        // cout << x << " " << r << endl;
        return r;
    }

    int digitsCount(int d, int low, int high) {
        return cnt(d, high) - cnt(d, low - 1);
    }
};
```





# [902. 最大为 N 的数字组合](https://leetcode-cn.com/problems/numbers-at-most-n-given-digit-set/)

- 分析
  - $dp(i,0)$表示第$0-i$位的数字组合数量，$dp(i,1)$表示第$0-i$位的数字组合中，后缀小于等于$n$的数字组合数量
  - 由于`digits`中不含0，我们需要将各位的dp结果累加。计算答案$r$时，除了最后一位加上$dp(i,1)$外，其他直接加上$dp(i,0)$即可

- 代码

```c++
class Solution {
public:
    int atMostNGivenDigitSet(vector<string>& digits, int n) {
        
        vector<int> data{};
        while (n > 0) {
            data.push_back(n % 10);
            n /= 10;
        }
        int m = data.size();
        vector<vector<int>> dp(m, vector<int>(2));
        int r = 0;
        for (int i = 0; i < m; i++) {
            if (i == 0) {
                dp[i][1] = digits.size();
                for (auto& s: digits) {
                    if (s[0] - '0' <= data[i])
                        dp[i][0]++;
                }
            } else {
                dp[i][1] = digits.size() * dp[i - 1][1];
                for (auto& s: digits) {
                    if (s[0] - '0' < data[i]) {
                        dp[i][0] += dp[i - 1][1];
                    } else if (s[0] - '0' == data[i]) {
                        dp[i][0] += dp[i - 1][0];
                    }
                }
            }
            if (i == m - 1)
                r += dp[i][0];
            else
                r += dp[i][1];

        }
        return r;
    }
};
```

