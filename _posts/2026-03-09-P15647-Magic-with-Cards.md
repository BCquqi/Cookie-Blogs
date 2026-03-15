---
title: 题解：P15647 [ICPC 2022 Tehran R] Magic with Cards
date: 2026-03-09 16:54:13 +0800
categories: [solution,Luogu]
tags: [BFS]
description: 题解：P15647 [ICPC 2022 Tehran R] Magic with Cards
toc: true
comments: true
pin: false
math: true
mermaid: true
---

# P15647 Magic with Cards

## 题意简述

给定一个有 $2n$ 张牌的牌堆，每次可以对牌堆进行两种操作：

1. **Riffle（交错洗牌）**：把牌堆分成两组（$c_1 \sim c_n$ 和 $c_{n+1} \sim c_{2n}$），交错在一起，得到牌堆 $c_1,c_{n+1},c_2,...,c_{2n}$。
2. **Scuffle（对调洗牌）**：从第一张牌开始，将牌堆两两分成一组，交换顺序。

求仅通过这两种操作，让牌堆第 $i$ 张牌到达位置 $j$ 的最短操作次数。

## 思路概要

本题是一道简单的 bfs 练习题，可以看作一个从位置 $i$ 操作到位置 $j$ 的**最短路问题**。

因为 bfs 算法（广度优先搜索）只要搜索到一项内容就可以得到最短路的特性且时间复杂度 $O(2n)$ 适合本题数据范围，故适合采用。在 bfs 算法中需要用到的数据结构是先进先出的**队列**（ queue ）。

对于每一次队列内的 bfs 操作，我们应该考虑该状态可以到达的两个状态，分别对应两种对牌堆的处理。

通过找规律可以发现，对于 Riffle 操作，位置 $x$ 的后续状态可以归为两类：

$$pos1=\begin{cases}
   2 \times x-1,1 \le x \le n\\
   2 \times x-2 \times n,n+1 \le x \le 2n
\end{cases}$$

对于 Scuffle 操作来说也分两类，无非就是奇数位后移一位、偶数位向前一位：

$$pos2=\begin{cases}
   x \mod 2=1,x+1\\
   x \mod 2=0,x-1
\end{cases}$$

知道每一次的 bfs 状态转换后，算法的设计就较为简单了。

## 正确代码

```cpp
#include<iostream>
#include<queue>
#include<cstring>
using namespace std;

const int N = 2e5 + 5;
int n,s,e,dis[N],vis[N];

/*
在 bfs 中，dis 数组用于记录从起点到达状态的最短路；
vis 数组用于记录是否曾经处理过本状态结点（剪枝，后来计算的答案步数一定比第一次长）
*/

void bfs() {
    memset(dis,0x3f,sizeof dis); // 此处 memset 是为了后面方便特殊判断输出 -1
    queue<int> q;
    q.push(s); // 处理起点
    dis[s] = 0,vis[s] = true;
    while (!q.empty()) { // 不断处理能到达的点
        int x = q.front(); q.pop();
        int pos1 = x <= n ? 2 * x - 1 : 2 * x - 2 * n;
        int pos2 = x % 2 ? x + 1 : x - 1;
        if (!vis[pos1]) {
            dis[pos1] = dis[x] + 1; // 更新最短路
            vis[pos1] = true;
            q.push(pos1);
        } if (!vis[pos2]) {
            dis[pos2] = dis[x] + 1; // 更新最短路
            vis[pos2] = true;
            q.push(pos2);
        }
    }
    return ;
}

int main() {
    cin >> n >> s >> e;
    bfs();
    if (dis[e] == 0x3f3f3f3f) cout << -1 << endl;
    else cout << dis[e] << endl;
    return 0;
}
```

[AC link](https://www.luogu.com.cn/record/266187386)