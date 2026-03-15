---
title: CookieOI[2026.01] Div.1 题解
date: 2026-02-26 20:05:00 +0800
categories: [solution,CookieOI]
tags: [C++,simulating]
description: CookieOI[2026.01] Div.1 题解
toc: true
comments: true
pin: false
math: true
mermaid: true
---

# CookieOI[2026.01] Div.1 题解

## A Poker

本题考验模拟。

我们可以使用一个 $bool$ 数组记录每张牌是否出现过。输入一张牌的信息后将其出现字母的花色和点数全部统一为数字形式，然后标记出现过的牌即可。最后遍历看多少张牌没有被标记。

你也可以使用 $map$ 记录每一张牌，方便存储、计算答案，时间复杂度会节省一点。

```cpp
#include<iostream>
#include<map>
using namespace std;

bool flag[5][14];

int main()
{
    int n;
    cin >> n;
    while (n--) {
        string tmp;
        cin >> tmp;
        char a = tmp[0],b = tmp[1];
        int i,j;
        if (a == 'D') i = 1;
        else if (a == 'C') i = 2;
        else if (a == 'H') i = 3;
        else i = 4;
        if ('2' <= b && b <= '9') j = b - '0';
        else if (b == 'A') j = 1;
        else if (b == 'T') j = 10;
        else if (b == 'J') j = 11;
        else if (b == 'Q') j = 12;
        else j = 13;
        flag[i][j] = true;
    }
    int ans = 52;
    for (int i = 1;i <= 4;i++)
        for (int j = 1;j <= 13;j++)
            if (flag[i][j]) ans--;
    cout << ans << endl;
    return 0;
}
```