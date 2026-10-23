---
title: 二项式反演

date: 2026-10-23 09:00:00 +0800
categories: [algorithm, math]
tags: [math, combinatorics]
description: 二项式反演原理、推导、算法竞赛套路与模板代码

math: true
mermaid: true
toc: true
comments: true
pin: false
---

参考：[OI Wiki — 二项式定理](https://oi-wiki.org/math/combination/)。本文面向算法竞赛，保留推导与结论，去冗。

## 二项式定理回顾

二项式定理描述了 $(x+y)^n$ 的展开式：

$$
(x+y)^n = \sum_{k=0}^n \binom{n}{k} x^{n-k} y^k
$$

其中 $\binom{n}{k} = \frac{n!}{k!(n-k)!}$ 为组合数。

在算法竞赛中，我们常遇到形如 $\sum_{k=0}^n \binom{n}{k} f(k)$ 或 $\sum_{k=0}^n \binom{n}{k} g(n-k)$ 的求和式，这时候需要 **二项式反演** 将其转化为关于另一个变量的求和。

## 二项式反演

### 基本形式

设 $f(n), g(n)$ 为数列，有如下等价关系：

$$
f(n) = \sum_{k=0}^n \binom{n}{k} g(k) \iff g(n) = \sum_{k=0}^n (-1)^{n-k} \binom{n}{k} f(k).
$$

$$
f(n) = \sum_{k=0}^n \binom{n}{k} g(n-k) \iff g(n) = \sum_{k=0}^n (-1)^k \binom{n}{k} f(n-k).
$$

### 推导

以第一种形式为例，考虑生成函数方法：

设指数生成函数 $F(x) = \sum_{n=0}^\infty f(n) \frac{x^n}{n!}$， $G(x) = \sum_{n=0}^\infty g(n) \frac{x^n}{n!}$。

则有：

$$
\sum_{n=0}^\infty f(n) \frac{x^n}{n!} = \sum_{n=0}^\infty \sum_{k=0}^n \binom{n}{k} g(k) \frac{x^n}{n!}
= \sum_{k=0}^\infty g(k) \sum_{n=k}^\infty \frac{x^n}{k!(n-k)!}
= \sum_{k=0}^\infty g(k) \frac{x^k}{k!} \sum_{m=0}^\infty \frac{x^m}{m!}
= G(x) e^x.
$$

所以 $F(x) = G(x) e^x$。同理，$e^{-x} = \sum_{n=0}^\infty (-1)^n \frac{x^n}{n!}$。

则 $G(x) = F(x) e^{-x}$，展开后得到：

$$
g(n) = \sum_{k=0}^n (-1)^{n-k} \binom{n}{k} f(k).
$$

### 交换形式

也可以写成：

$$
g(n) = \sum_{k=0}^n (-1)^{n-k} \binom{n}{k} f(k) = \sum_{k=0}^n (-1)^k \binom{n}{k} f(n-k).
$$

### 关键性质

- 反演系数为 $(-1)^{n-k}\binom{n}{k}$，呈现交替符号规律
- 当 $n$ 较小时直接代入计算
- 当 $n$ 较大时配合前缀和/分块优化

## 算法竞赛常用套路

### 判型识别

题目中出现 $\sum_{k=0}^n \binom{n}{k} (\text{关于 }k\text{的项})$ 即考察二项式反演。

### 典型变形

#### 形式 A：$\sum_{k=0}^n \binom{n}{k} a_k$

直接反演得 $a_n = \sum_{k=0}^n (-1)^{n-k} \binom{n}{k} \sum_{j=0}^k \binom{k}{j} a_j$，但这通常会导致两重循环。

更常见的是：**利用性质 $\binom{n}{k} = \binom{n-1}{k} + \binom{n-1}{k-1}$** 递推求解。

#### 形式 B：$\sum_{k=0}^n \binom{n}{k} (-1)^k$

这是一个确定值：$\sum_{k=0}^n \binom{n}{k} (-1)^k = (1-1)^n = [n=0]$。

#### 形式 C：$\sum_{k=0}^n \binom{n}{k} k^m$

利用斯廷宁数或前面的反演公式展开。

### 常见题目套路

- **差分**: 已知 $f(n)=\sum_{k=0}^n \binom{n}{k} g(k)$，求 $g(n)$
- **求和**: 已知 $g(n)$，求 $f(n)=\sum_{k=0}^n \binom{n}{k} g(k)$ 的闭式
- **组合恒等式**: 证明两个求和式相等

## 模板代码

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;

// 预处理组合数 C[n][k]
const int MAXN = 1005;
LL C[MAXN][MAXN];
LL f[MAXN], g[MAXN];

void init_comb(int n) {
    for (int i = 0; i <= n; ++i) {
        C[i][0] = C[i][i] = 1;
        for (int j = 1; j < i; ++j)
            C[i][j] = C[i-1][j-1] + C[i-1][j];
    }
}

// 二项式反演：f(n) = sum_{k=0}^n C(n,k) * g(k)
void binomial_inversion(int n) {
    // g(n) = sum_{k=0}^n (-1)^{n-k} * C(n,k) * f(k)
    for (int i = 0; i <= n; ++i) {
        g[i] = 0;
        for (int j = 0; j <= n; ++j) {
            LL sign = ((n - j) & 1) ? -1 : 1;
            g[i] += sign * C[n][j] * f[j];
        }
    }
}

// 例：已知 f(n) = 2^n，求 g(n) 使得 f(n) = sum_{k=0}^n C(n,k) g(k)
// 答案：g(n) = 1 (常数序列)
```

## 典型例题

### 例1：[Luogu P1151](https://www.luogu.com.cn/problem/P1151)
求 $\sum_{k=0}^n \binom{n}{k} k$ 的值。

**解**：利用二项式定理变形，或直接反演。

### 例2：[Luogu P1661](https://www.luogu.com.cn/problem/P1661)
已知 $f(n) = \sum_{k=0}^n \binom{n}{k} g(k)$，其中 $f(n)$ 已知，求 $g(n)$。

**解**：直接使用反演公式 $g(n) = \sum_{k=0}^n (-1)^{n-k} \binom{n}{k} f(k)$。

### 例3：组合恒等式证明
证明 $\sum_{k=0}^n \binom{n}{k} = 2^n$。

**解**：考虑 $(1+1)^n = \sum_{k=0}^n \binom{n}{k} 1^{n-k} 1^k = \sum_{k=0}^n \binom{n}{k}$，所以 $\sum_{k=0}^n \binom{n}{k} = 2^n$。

## 总结

二项式反演是处理含有组合系数求和式的有力工具。其核心公式为：

$$
f(n) = \sum_{k=0}^n \binom{n}{k} g(k) \iff g(n) = \sum_{k=0}^n (-1)^{n-k} \binom{n}{k} f(k).
$$

在算法竞赛中，关键在于：
1. 判断题目是否考察二项式反演
2. 正确识别 $f(n)$ 和 $g(n)$ 的关系
3. 配合组合数预处理和模板快速实现

常见变形包括交替符号处理、指数生成函数方法以及利用帕斯卡矩阵性质进行优化。