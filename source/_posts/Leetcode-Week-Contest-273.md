---
title: Leetcode Week Contest 273
date: 2021-12-26 12:06:35
tags: Leetcode
mathjax: true
categories: Algorithm
---

# [5963. 反转两次的数字](https://leetcode-cn.com/problems/a-number-after-a-double-reversal/)

- 字符串
- 调用`str()`转为字符串反转，调用`int()`转为数字，按照题意实现即可
- 代码

```python
class Solution:
    def isSameAfterReversals(self, num: int) -> bool:
        return int(str(int(str(num)[::-1]))[::-1]) == num
```

# [5964. 执行所有后缀指令](https://leetcode-cn.com/problems/execution-of-all-suffix-instructions-staying-in-a-grid/)

- 暴力

- 由于n为500，因此两次循环即可求得各index开始可以执行的最长指令长度，位置变化只需记录一下是否出界即可

  - 小优化：使用哈希表预存不同指令的位移变化量

  ```python
  data = {'L': (0, -1), 'R': (0, 1), 'U': (-1, 0), 'D':(1, 0)}
  ```

- 代码

```python
class Solution:
    def executeInstructions(self, n: int, startPos: List[int], s: str) -> List[int]:
        ans = [0] * len(s)
        data = {'L': (0, -1), 'R': (0, 1), 'U': (-1, 0), 'D':(1, 0)}
        for i in range(len(s)):
            x, y = startPos
            for j in range(i, len(s)):
                x += data[s[j]][0]
                y += data[s[j]][1]
                if 0 <= x < n and 0 <= y < n:
                    continue
                else:
                    ans[i] = j - i
                    break
            else:
                ans[i] = len(s) - i
        return ans
```

<!--more-->

# [5965. 相同元素的间隔之和](https://leetcode-cn.com/problems/intervals-between-identical-elements/)

- 哈希表 + 动态规划
- 思路
  - 使用哈希表存储相同值的下标数组，`data[val]`是值为`val`的所有下标组成的数组`arr`
  - 对于每个下标数组`arr`，从左到右每移动一个位置，若移动距离为`delta`，则其与左边的数的总距离会增加`delta * left`，与右边的数的总距离会减小`delta * right`，其中`left`是当前位置左边的数的个数，`right`是当前位置右边（包含当前位置）的右边的数的个数，满足`left+right=n`
- 时间复杂度：`O(n)`
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

# [5966. 还原原数组](https://leetcode-cn.com/problems/recover-the-original-array/)

- 排序 + 哈希表（本质上是暴力）
- 思路
  - 分析：数组中最小的数必定属于lower，同理最大的数必定属于higher，假设我们确定了k，则可以还原整个数组
  - **已知k的还原方法**：首先哈希表统计每个数的个数，因为最小的数`min`所对应的数必定为`min + 2*k`，假若该数存在将其计数值减一，若不存在则返回false。这样从小到大遍历整个数组即可，时间复杂度为$O(N)$
  - **确定k的方法**：由于n为1000，且最小的数`arr[0]`必定为lower，只需要考虑其对应的数依次为`arr[1]-arr[n-1]`，则k的取值最多只可能有n-1种，由此可在$O(N^2)$的时间内还原数组
- 时间复杂度：$O(N^2)$

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

- 国服19，全球34
