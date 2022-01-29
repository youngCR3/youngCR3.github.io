---
title: CF Edu 120
date: 2021-12-28 16:42:43
tags: Codeforces
categories: Algorithm
mathjax: true
---

# A Construct a Rectangle

- 标签：`math`
- 分析：有两种可能的情况，一种是两个边分别为邻边（长、宽），第三条边可以切割为长、宽；一种是两个边为对边，第三条边可以切割为两条相等的边

- 代码

  ```python
  for _ in range(int(input())):
      arr = list(map(int, input().split()))
      arr.sort()
      a, b, c = arr
      if a + b == c or (a == b and c % 2 == 0) or (b == c and a % 2 == 0):
          print("YES")
      else:
          print("NO")
  ```

<!--more-->

# B Berland Music

- 标签：`greedy`

- 分析：like总是比dislike高分，假设有m首dislike，则其占据的分数为1~m，like占据的分数为m+1~n。为此分别将其按从小到大排序，然后赋分即可

- 代码

  ```python
  for _ in range(int(input())):
      n = int(input())
      arr = list(map(int, input().split()))
      res = arr.copy()
      s = input()
      like = []
      dislike = []
      for i in range(n):
          if s[i] == '1':
              like.append((arr[i], i))
          else:
              dislike.append((arr[i], i))
      like.sort()
      dislike.sort()
      
      for i in range(len(dislike)):
          res[dislike[i][1]] = i + 1
      for i in range(len(like)):
          res[like[i][1]] = i + 1 + len(dislike)
      print(*res)
  ```

  

# C Set or Decrease

- 标签：`math`

- 分析：一开始想到的是令最小值降到恰好满足等式的值x，然后其他数都赋值x，从而保证$x \times n \leq k$，这种方法忽略了可以让最小值降到比x更小，从而不用将所有其他值都赋值x的情况

- 正确思路：首先将数组排序，考虑除了最小值外，将最大的m个数都进行赋值，此时为了满足该等式最小值所要达到的值target，即满足$target \times (m+1) + sum((n - m - 1)numbers) \leq k$的最大值target，该值可以通过数学方式求得，然后考虑m从0到n-1变化的过程，即可求得最小操作次数

- 代码

  ```python
  for _ in range(int(input())):
      n, k = map(int, input().split())
      arr = list(map(int, input().split()))
      arr.sort()
      minVal = arr[0]
      s = sum(arr) - minVal
      
      res = max(0, minVal + s - k)
      for i in range(n - 1, 0, -1):
          s -= arr[i]
          
          target = (k - s) // (1 + n - i)
          if target >= minVal:
              tmp = n - i
          else:
              tmp = n - i + minVal - target
          res = min(res, tmp)
      print(res)
  ```

  

# D Shuffle

- 标签：`排列组合数`、`sliding windows`
- 分析：此题的解法是通过对测试用例的观察、找规律得到的，首先通过滑动窗口求得每一个包含k个1的子串。当滑动窗口移动时，假设找到的字串长度为y，包含x个1
  - 若最右边的字符为0，则打乱后其必为1才能与前面的不同，因此有`C(y-1, x-1)`种情况
  - 若最右边的字符为1，则打乱后其必为0才能与前面的不同，因此有`C(y-1, x)`种情况
  - 特别地，对于第一个字符，由于不用考虑和前面的重复情况，因此有`C(y, x)`种情况

- 坑点：由于本题需要对MOD取余，当结果恰好是其倍数时则答案为0。开始时我忽略了这个情况，认为答案至少为1，在答案为0时（若完全没有包含k个1的子串）直接返回1，从而出错并被Hack了。
  - 正确做法：增加flag判断，若出现了包含k个1的子串，则答案原始值必不为0（因此出现了0必定为取余导致，直接返回0；否则，说明答案原始值就是0，返回1）

- 代码：本题涉及在某一个取余值下，快速求大数的阶乘以及排列组合数，涉及**费马小定理**等基础知识

  ```python
  from functools import lru_cache
  
  @lru_cache(None)
  def fac(x):
      if x == 0:
          return 1
      else:
          return x * fac(x - 1) % MOD
          
  @lru_cache(None)
  def ifac(x):
      return pow(fac(x), MOD - 2, MOD)
  
  def C(x, y):
      if x < y:
          return 0
      return fac(x) * ifac(y) * ifac(x - y) % MOD
  
  
  MOD = 998244353
  for i in range(5000):
      fac(i)
  n, k = map(int, input().split())
  s = input()
  if k == 0:
      print(1)
  else:
      left = 0
      cnt = 0
      res = 0
      start = False
      flag = False
      for i in range(len(s)): 
          if s[i] == '1':
              cnt += 1
          while cnt > k:
              if s[left] == '1':
                  cnt -= 1
              left += 1
          if cnt == k:
              if not start:
                  res += C(i - left + 1, k)
              else:
                  if s[i] == '1':
                      res += C(i - left, k)
                  else:
                      res += C(i - left, k - 1)
              start = True
              flag = True
              res %= MOD
      if res == 0 and not flag:
          res = 1
      print(res)
  ```

  
