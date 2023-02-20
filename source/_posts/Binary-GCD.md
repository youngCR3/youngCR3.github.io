---
title: Binary GCD
date: 2023-02-07 11:25:06
tags: Algorithm
Categories: Algorithm
mathjax: true
---

# Conventional GCD

```c++
// aka 辗转相除法
int gcd(int u, int v) {
	if (u == 0)
        return v;
    if (v == 0)
        return u;
    if (u > v) {
        swap(&u, &v);
    }
    while (u) {
        r = v % u;
        v = u;
        u = r;
    }
    return v;
}
```

# Binary GCD

## Math basic

- `gcd(0, v) = v`
- `gcd(2u, 2v) = 2gcd(u, v)`
- `gcd(2u, v) = gcd(u, v)` if v is odd
- `gcd(u, v) = gcd(|u-v|, min(u, v))` if u and v are both odd

## Implementation

```cpp
int gcd(int u, int v) {
    if (u == 0)
        return v;
    if (v == 0)
        return u;
    int i = 0, j = 0;
    while (u && (u & 1) == 0) {
        u >>= 1;
        ++i;
    }
    while (v && (v & 1) == 0) {
        v >>= 1;
        ++j;
    }
    int k = min(i, j);
    while (v > 0) {
        // u and v are both odd at the start of the loop
        assert(u % 2 == 1);
        assert(v % 2 == 1);
        // swap to make u <= v
        if (u > v) {
            swap(&u, &v);
        }
        // gcd(u, v) = gcd(|u -v|, min(u, v))
        v -= u;
        // v is now even, u is still odd
        while (v && (v & 1) == 0) {
            v >>= 1;
        }
    }
    return u * pow(2, k);
}
```



# Application

[LC2543. 判断一个点是否可以到达](https://leetcode.cn/problems/check-if-point-is-reachable/description/)

