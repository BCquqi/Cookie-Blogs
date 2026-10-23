---
title: 莫比乌斯反演
date: 2026-10-23 09:00:00 +0800
categories: [algorithm, math]
tags: [math, number-theory]
description: 莫比乌斯反演原理、推导、算法竞赛套路与模板代码
toc: true
comments: true
pin: false
math: true
mermaid: true
---

参考：[OI Wiki — 莫比乌斯反演](https://oi-wiki.org/math/number-theory/mobius/)。本文面向算法竞赛，保留推导与结论，去冗。

## 1 莫比乌斯函数

定义：

$$
\mu(n)=
\begin{cases}
1, & n=1,\\
0, & n\text{ 存在平方因子（即 }\exists p,\ p^2\mid n\text{)},\\
(-1)^k, & n\text{ 是 }k\text{ 个互异质数的积}.
\end{cases}
$$

性质：
- 积性函数，非完全积性；
- **核心恒等式**：$\displaystyle \sum_{d\mid n}\mu(d)=[n=1]$。

证明：令 $n=\prod_{i=1}^k p_i^{e_i}$，设 $n'=\prod p_i$（去重后的质数积）。$\mu(d)\ne0$ 当且仅当 $d\mid n'$ 且无平方因子，故

$$
\sum_{d\mid n}\mu(d)=\sum_{d\mid n'}\mu(d)=\sum_{i=0}^{k}\binom{k}{i}(-1)^i=(1-1)^k=[k=0]=[n=1].
$$

利用 Dirichlet 卷积，即 $\varepsilon=1*\mu$，$\mu$ 是常值函数 $1$ 的 Dirichlet 逆。

常用结论：$[\gcd(i,j)=1]=\displaystyle\sum_{d\mid\gcd(i,j)}\mu(d)=\sum_d[d\mid i][d\mid j]\mu(d)$，可将互素条件转化为求和。

### 1.1 求法

单个 $n$：试除分解，复杂度 $O(\sqrt n)$：

```cpp
int mu(int n) {
    int res = 1;
    for (int i = 2; i * i <= n; ++i) {
        if (n % i == 0) {
            n /= i;
            if (n % i == 0) return 0;
            res = -res;
        }
    }
    if (n > 1) res = -res;
    return res;
}
```

前 $N$ 个：线性筛，$O(N)$：

```cpp
vector<int> get_mu(int n) {
    vector<int> mu(n + 1), primes;
    vector<bool> not_prime(n + 1);
    mu[1] = 1;
    for (int x = 2; x <= n; ++x) {
        if (!not_prime[x]) { primes.push_back(x); mu[x] = -1; }
        for (int p : primes) {
            if (x * p > n) break;
            not_prime[x * p] = true;
            if (x % p == 0) { mu[x * p] = 0; break; }
            else mu[x * p] = -mu[x];
        }
    }
    return mu;
}
```

## 2 莫比乌斯反演

### 2.1 基本形式

设 $f,g$ 为数论函数，则

$$
f(n)=\sum_{d\mid n}g(d)\iff g(n)=\sum_{d\mid n}\mu\left(\frac nd\right)f(d).
$$

证明：直接验证，

$$
\sum_{d\mid n}\mu\left(\frac nd\right)f(d)
=\sum_{d\mid n}\mu\left(\frac nd\right)\sum_{k\mid d}g(k)
=\sum_{k\mid n}g(k)\sum_{d\mid n}\left[\frac nd\mid\frac nk\right]\mu\left(\frac nd\right)
=\sum_{k\mid n}g(k)\left[\frac nk=1\right]=g(n).
$$

关键步骤：交换求和次序，并利用 $\sum_{t\mid m}\mu(t)=[m=1]$。

### 2.2 倍数和形式

$$
f(n)=\sum_{n\mid d}g(d)\iff g(n)=\sum_{n\mid d}\mu\left(\frac dn\right)f(d).
$$

证明与基本形式对偶。

### 2.3 拓展：Dirichlet 前缀和与差分

由 $f=1*g$ 得 $g=\mu*f$，即 $f(n)=\sum_{d\mid n}g(d)$ 为 $g$ 在 $n$ 的因数上的“前缀和”，称 **Dirichlet 前缀和**；其逆称 **Dirichlet 差分**。

可在 $O(N\log\log N)$ 内预处理（与 Eratosthenes 筛遍历顺序一致）：逐个素数 $p$，将 $n$ 处的值累加到 $np$ 处（前缀和）；逆过程则累减（差分）。

## 3 算法竞赛常用套路

### 3.1 提取公因数并转化互素条件

典型式子：

$$
\sum_{i=1}^{n}\sum_{j=1}^{m}[\gcd(i,j)=k].
$$

设 $i=ki', j=kj'$，得 $\sum_{i=1}^{\lfloor n/k\rfloor}\sum_{j=1}^{\lfloor m/k\rfloor}[\gcd(i,j)=1]$，再代入

$$
[\gcd(i,j)=1]=\sum_d\mu(d)[d\mid i][d\mid j],
$$

交换求和次序：

$$
\sum_d\mu(d)\left\lfloor\frac{\lfloor n/k\rfloor}{d}\right\rfloor\left\lfloor\frac{\lfloor m/k\rfloor}{d}\right\rfloor
=\sum_d\mu(d)\left\lfloor\frac{n}{kd}\right\rfloor\left\lfloor\frac{m}{kd}\right\rfloor.
$$

固定 $\ell=kd$ 再次交换求和次序，配合 $\mu$ 前缀和与**数论分块**，复杂度 $O(\sqrt N)$ 单次询问。

### 3.2 lcm 转 gcd

$$
\operatorname{lcm}(i,j)=\frac{ij}{\gcd(i,j)}.
$$

枚举公因数 $k$，将 $\gcd(i,j)=k$ 转化为互素问题，再按套路 1 处理。

## 4 模板代码

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;

// 线性筛预处理 mu 及其前缀和，N 为上界
const int N = 1e7 + 5;
int mu[N], primes[N / 10], cnt;
bool not_prime[N];
LL sum_mu[N];

void init(int n) {
    mu[1] = 1;
    for (int i = 2; i <= n; ++i) {
        if (!not_prime[i]) { primes[cnt++] = i; mu[i] = -1; }
        for (int j = 0; j < cnt && i * primes[j] <= n; ++j) {
            not_prime[i * primes[j]] = true;
            if (i % primes[j] == 0) { mu[i * primes[j]] = 0; break; }
            mu[i * primes[j]] = -mu[i];
        }
    }
    for (int i = 1; i <= n; ++i) sum_mu[i] = sum_mu[i - 1] + mu[i];
}

// 数论分块求 sum_{d=1}^{n} mu(d) * floor(n/d) * floor(m/d)
// 可用于 sum_{i<=n,j<=m} [gcd(i,j)=1] 的变形
LL calc(int n, int m) {
    if (n > m) swap(n, m);
    LL ans = 0;
    for (int l = 1, r; l <= n; l = r + 1) {
        r = min(n / (n / l), m / (m / l));
        ans += (LL)(sum_mu[r] - sum_mu[l - 1]) * (n / l) * (m / l);
    }
    return ans;
}
```

## 5 典型例题

- Luogu P2522 [HAOI2011] Problem b：$\sum\sum[\gcd(i,j)=k]$，数论分块 + $\mu$ 前缀和。
- Luogu P3312 [SDOI2014] 数表：$\sum\sum\sigma_0(ij)$，利用 $\sigma_0(ij)=\sum_{x\mid i}\sum_{y\mid j}[x\perp y]$。
- Luogu P3768 简单的数学题：$\sum\sum\gcd(i,j)$。
- BZOJ 2154 Crash 的数字表格：$\sum\sum\operatorname{lcm}(i,j)$，乘积版莫比乌斯反演。
- AtCoder AGC038 C - LCMs：$\sum\operatorname{lcm}(a_i,a_j)$，积性函数预处理。
