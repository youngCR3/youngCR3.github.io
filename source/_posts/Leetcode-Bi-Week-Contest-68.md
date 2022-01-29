---
title: Leetcode Bi-Week Contest 68
date: 2021-12-26 13:16:13
tags: Leetcode
categories: Algorithm
---

# [5946. 句子中的最多单词数](https://leetcode-cn.com/problems/maximum-number-of-words-found-in-sentences/)

- 字符串
- 直接调用`split()`将每个句子划分为单词，然后统计单词数的最大值作为答案即可
- 代码

```python
class Solution:
    def mostWordsFound(self, sentences: List[str]) -> int:
        res = 0
        for s in sentences:
            res = max(res, len(s.split()))
        return res
```

# [5947. 从给定原材料中找到所有可以做出的菜](https://leetcode-cn.com/problems/find-all-possible-recipes-from-given-supplies/)

- 拓扑排序
- 一道菜的材料可能是原材料supplies或者其他菜recipes，由此可以想到拓扑排序，原材料supplies的degree均为0，recipes的degree等于其原材料数目，只有当degree为0时该道菜才可以被制作，拓扑排序可求得能做的菜的数目
- 代码

```python
class Solution:
    def findAllRecipes(self, recipes: List[str], ingredients: List[List[str]], supplies: List[str]) -> List[str]:
        n = len(recipes)
        degree = {}
        data = {}
        for i in range(n):
            degree[recipes[i]] = len(ingredients[i])
            for j in ingredients[i]:
                if j not in data:
                    data[j] = set()
                data[j].add(recipes[i])
        q = []
        h = 0
        for i in supplies:
            degree[i] = 0
        for i in degree:
            if degree[i] == 0:
                q.append(i)
        while h < len(q):
            if q[h] in data:
                for i in data[q[h]]:
                    degree[i] -= 1
                    if degree[i] == 0:
                        q.append(i)
            h += 1
        res = []
        recipes = set(recipes)
        for i in q:
            if i in recipes:
                res.append(i)
        return res
```

<!--more-->

# [5948. 判断一个括号字符串是否有效](https://leetcode-cn.com/problems/check-if-a-parentheses-string-can-be-valid/)

- 贪心
- s的长度为奇数必定无效，若为偶数，则统计其**degree的可能范围**`[minDegree, maxDegree]`
  - 对于无法改变的字符，则`(`会使degree加1，`)`会使degree减1
  - 对于能够改变的字符，则既可使degree加1，也可使degree减1
  - 当`minDegree`为0时，无法再减
  - 当`maxDegree`小于0时，则为无效，可以立刻返回false

- 只有当degree范围包括0时才是有效字符串
- 代码

```python
class Solution:
    def canBeValid(self, s: str, locked: str) -> bool:
        if len(s) % 2:
            return False
        # 当前可达到的最大度和最小度
        minVal = maxVal = 0
        for i in range(len(s)):
            if locked[i] == '1':
                if s[i] == '(':
                    maxVal += 1
                    minVal += 1
                else:
                    if maxVal == 0:
                        return False
                    else:
                        minVal = max(0, minVal - 1)
                        maxVal -= 1
            else:
                maxVal += 1
                minVal = max(0, minVal - 1)
        return minVal == 0
```

# 写在T4之前（此题由于精度问题，暂无解）

- 目前，T4的所有解法事实上都是错的，运算的误差无法通过加上eps弥补，加上`1e-11`只是碰巧通过了所有示例，参考该文章https://leetcode-cn.com/problems/abbreviating-the-product-of-a-range/solution/yi-ge-shu-ju-tuan-mie-jue-da-bu-fen-dai-234yd/
- 以下是当时的解法思路

# [5949. 一个区间内所有数乘积的缩写](https://leetcode-cn.com/problems/abbreviating-the-product-of-a-range/)

- 数学

- 本题可分解为以下几部分

  - 后缀0个数求解
  - 前5个数求解
  - 后5个数求解

- 阶乘后缀0个数求解

  - 一个数的阶乘的后缀0个数，可以将其不断除以5，并将商累加
  - `left*...*right`的后缀0个数，等于`right`阶乘的后缀0个数，减去`left - 1`的阶乘的后缀0个数

  ```python
  def get(n):
      res = 0
      while n:
          res += n // 5
          n //= 5
          return res
  ```

- 后5个数求解

  - 从left到right不断累乘back
  - 若back有后缀0则去掉（若back能被10整除，则除以10）
  - 对100000取余保证为**5位数**

- 前5个数求解

  - 方法一：与后5个数方法类似，但是由于进位的存在，因此运算过程需要保留12位数而不是5位数，最后再取5位数，运算速度较慢
  - 方法二：**取对数`log10`**，累加`[log10(left), log10(right)]`得到和s，则`10^(s - int(s))`即为科学计数法中的有效数字部分，将其乘以10000即可得到前5位数
  - 方法二的精度问题，实现时发现浮点数运算存在精度问题，导致前5位数计算存在偏差，因此可以考虑通过**加一个误差项eps**，经试验**加上1e-11可以通过所有测试用例**

- 代码

```python
class Solution:
    def abbreviateProduct(self, left: int, right: int) -> str:
        # 计算后缀0
        def get(n):
            res = 0
            while n:
                res += n // 5
                n //= 5
            return res
        
        if left == right and left % 10:
            zero = 0
        else:
            zero = get(right) - get(left - 1)
        ans = 0
        back = 1
        for i in range(left, right + 1):
            ans += log(i, 10)
            back *= i
            while back and back % 10 == 0:
                back //= 10
            back %= 1000000000
        back %= 100000
        x = int(ans)
        m = pow(10, ans - int(ans) + 1e-11)
        if x + 1 - zero > 10:
            front = int(m * 10000)
            front = str(front)
            back = str(back)
            back = '0' * (5 - len(back)) + back
            return front + "..." + back + "e" + str(zero)
        else:
            front = int(m * pow(10, x + 1 - zero - 1))
            return str(front) + "e" + str(zero)
```

# 排名情况

- 国服45，全球67
