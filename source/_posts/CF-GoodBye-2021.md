---
title: CF GoodBye 2021
date: 2021-12-30 17:00:13
tags: Codeforces
mathjax: true
categories: Algorithm
---

# A Integer Diversity

- 标签：哈希表
- 分析：绝对值相同的数最多只能构成两个数，**若为0则只能构成一个数**，哈希表统计各个数的绝对值的数量并累加即可，注意0

- 代码

  ```python
  for _ in range(int(input())):
      n = int(input())
      arr = list(map(int, input().split()))
      cnt = {}
      for i in arr:
          val = abs(i)
          cnt[val] = cnt[val] + 1 if val in cnt else 1
      res = 0
      for i in cnt:
          if i > 0:
              res += min(2, cnt[i])
          else:
              res += min(1, cnt[i])
      print(res)
  ```

<!--more-->

# B Mirror in the String

- 标签：字符串，规律
- 分析：通过测试用例可发现一下规律
  - 目标前缀必定是一个非递增字符串
  - 除特殊情况外，该非递增字符串越长越好
  - 特殊情况在于开头的第一个字符，若其后的第二个字符与其相等，也应截断。例如对于`aa`，应该选择`a`作为镜像前的子串，这是因为`aa`的字典序小于`aaaa`

- 代码

  ```python
  for _ in range(int(input())):
      n = int(input())
      s = input()
      for i in range(1, len(s)):
          if s[i] > s[i - 1] or (i == 1 and s[i] == s[i - 1]):
              k = i - 1
              break
      else:
          k = len(s) - 1
      print(s[: k + 1] + s[: k+1][::-1])
  ```

# C Representative Edges

- 标签：等差序列

- 分析：通过用例可初步推断，最终形成的数组必定为等差序列，为了尽量少地操作，则要在原数组找到最长的等差序列

- 上述结论的详细推导

  由题目条件可得以下两式
$$
a_{l}+a_{l+1}+\ldots+a_{r}=\frac{1}{2}\left(a_{l}+a_{r}\right) \cdot(r-l+1)
$$

$$
a_{l}+a_{l+1}+\ldots+a_{r+1}=\frac{1}{2}\left(a_{l}+a_{r+1}\right) \cdot(r+1-l+1)
$$

​		下式减上式再整理得到
$$
a_{r+1}-a_{r}=\frac{a_r-a_l}{r-l}
$$
​		下面通过数学归纳法证明命题对于任意$i=1,2,...,n-1$,都有$a_{i+1}-a_{i}=d$

  - 当$i=1$时，由d的定义可知$a_2-a_1=d$成立

  - 假设当$i=k$时，$a_{k+1}-a_{k}=d$成立

  - 则当$i=k+1$时，有
$$
a_{k+2}-a_{k+1}=\frac{a_{k+1}-a_{1}}{k+1-1}=\frac{a_{k+1}-a_{1}}{k}
$$
​		又因为
$$
a_{k+1}-a_{k}=\frac{a_{k}-a_{1}}{k-1}=d
$$
​		因此
$$
a_{k}-a_{1}=(k-1)d
$$
​		则
$$
a_{k+2}-a_{k+1}=\frac{a_{k+1}-a_k+a_k-a_{1}}{k}=\frac{d+(k-1)d}{k}=d
$$
​		**QED**

- 代码：遍历每两个元素，确定一个等差数列，然后计算数组中位于该等差数列的数的个数，时间复杂度为$O(N^3)$

  ```c++
  #include <iostream>
  #include <vector>
  #include <cmath> 
  using namespace std;
  
  int main(void) {
      int t;
      cin >> t;
      while (t--) {
          int n;
          cin >> n;
          vector<int> a(n, 0);
          for (int i = 0; i < n; i++) {
              cin >> a[i];
          }
          if (n == 1) 
              cout << 0 << endl;
          else {
              int res = n;
              for (int i = 0; i < n; i++) {
                  for (int j = i + 1; j < n; j++) {
                      double d = (a[j] - a[i]) * 1.0 / (j - i);
                      // cout << d << endl;
                      int cnt = 0;
                      for (int k = 0; k < n; k++) {
                          if (abs(a[k] - (a[i] + d * (k - i))) < 1e-10)
                              cnt++;
                      }
                      res = min(res, n - cnt);
                  }
                  
              }
              cout << res << endl;
          }
      }
      return 0;
  }            
  ```

# D Keep the Average High

- 标签：动态规划

- 分析：技巧性很强的一道题，考虑计算将各个数与x的差d，在遍历的过程中对d进行累加，累加值为`s`，同时计算s的最大值`max_s`，**假设某个时刻发现`max_s < s`，则必定说明存在一个连续子数组的平均值小于x**

- 限制条件：由于条件规定`l<r`，因此单个元素小于x是不需要考虑的，据此对上述算法进行修改

  - 首先在计算最大值max_s时应该滞后1个元素，即当前的累加值s不要纳入最大值计算，而是在下一个下标的最大值计算时才计算当前的s
  - 最大值的初始值设为0，若第一个元素就小于x，则同样会出现`max_s < s`，此时继续遍历即可
  - 当发现存在一个连续子数组平均值小于x后，只需要不选择最后一个元素即可。此后`max_s`和`s`等变量还原到初始值0

- 代码

  ```c++
  #include <iostream>
  #include <vector>
  using namespace std;
  
  void solve(void) {
      int n;
      cin >> n;
      vector<int> a(n, 0);
      for (int i = 0; i < n; i++) {
          cin >> a[i];
      }
      int x;
      cin >> x;
      vector<long long> prefix(n, 0);
      long long maxVal = 0;
      long long sum = 0;
      int res = n;
      int left = 0;
      for (int i = 0; i < n; i++) {
          sum += a[i] - x;
          prefix[i] = sum;
          if (sum < maxVal && i > left) {
              res -= 1;
              sum = 0;
              maxVal = 0;
              left = i + 1;
          } else {
              if (i - 1 >= left)
                  maxVal = max(maxVal, prefix[i - 1]);
          }
      }
      cout << res << endl;
      
  }
  
  int main(void) {
      int t;
      cin >> t;
      while (t--) {
          solve();
      }
      return 0;
  }            
  ```

  
