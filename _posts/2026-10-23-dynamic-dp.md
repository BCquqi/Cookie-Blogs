---
title: 动态DP与DP套DP详解

date: 2026-10-23 14:00:00 +0800
categories: [algorithm,DP]
tags: [DP, algorithm, divide-and-conquer]
description: 详细讲解动态DP算法原理与DP套DP（DP-in-DP）思想，包括定义、区别、模板代码与典型应用

math: true
mermaid: true
toc: true
comments: true
pin: false
---

# 动态DP概述

动态DP (Dynamic Programming on DAG) 和 DP-in-DP (DP套DP) 是两种高级DP技术，用于解决复杂的最优化问题。

- **动态DP**：指在图结构（通常是DAG）上进行DP，或者指DP的状态空间随时间变化（动态问题）
- **DP-in-DP (DP套DP)**：指在DP算法内部套用另一层DP，用于解决更复杂的优化问题

## 动态DP

### 定义

动态DP用于在**动态变化的图结构**上维护最优解，常见场景包括：
- 带权图上的最短路径
- 随着时间推移而变化的状态空间
- 需要频繁插入/删除元素的场景

### 典型应用：最短路DP

最短路DP通过**状态迭代**的方式求解最短路径：

```
dist[v] = min(dist[v], dist[u] + w(u, v))
```

**SPFA (Shortest Path Faster Algorithm)** 示例：

```cpp
bool spfa() {
    queue<int> q;
    for (int i = 1; i <= n; i++) {
        dist[i] = INF;
        inq[i] = false;
    }
    dist[s] = 1;
    q.push(s);
    inq[s] = true;
    
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        inq[u] = false;
        
        for (int i = head[u]; i; i = e[i].next) {
            int v = e[i].to;
            if (dist[v] > dist[u] + e[i].w) {
                dist[v] = dist[u] + e[i].w;
                if (!inq[v]) {
                    inq[v] = true;
                    q.push(v);
                }
            }
        }
    }
    return dist[t] != INF;
}
```

## DP-in-DP (DP套DP)

### 定义

DP-in-DP 是指**在DP状态转移过程中，嵌套使用另一层DP算法**。这种技术通常用于解决具有**多重约束**或**复杂决策**的最优化问题。

### 典型应用场景

1. **二维/多维背包问题**：外层DP枚举物品，内层DP处理约束
2. **复杂费用计算**：外层DP枚举决策，内层DP计算费用
3. **多阶段优化**：外层DP处理阶段间状态，内层DP处理阶段内细节

### 典型问题模型

#### 问题1：二维背包
**题目**：给定n个物品，每个物品有重量w、体积v和价值c，背包容量为W，体积为V，求最大价值。

**DP-in-DP思路**：
- 外层DP枚举物品序号
- 内层DP枚举重量和体积的组合状态

```cpp
// dp[i][j] 表示体积为j时的最小价值，外层枚举重量
for (int k = 1; k <= n; k++) {
    // 外层DP：第k个物品
    for (int i = W; i >= w[k]; i--) {
        // 内层DP：剩余容量下的最优解
        for (int j = V; j >= v[k]; j--) {
            dp[i][j] = min(dp[i][j], dp[i - w[k]][j - v[k]] + c[k]);
        }
    }
}
```

#### 问题2：分组背包
**题目**：有n组物品，每组物品互斥，选择恰好一个物品进入背包，求最大价值。

**DP-in-DP思路**：
- 外层DP枚举组号
- 内层DP枚举背包容量

```cpp
// 外层DP枚举组
for (int i = 1; i <= n; i++) {
    // 内层DP倒序枚举容量
    for (int j = V; j >= 0; j--) {
        // 枚举当前组选择的物品
        for (int k = 0; k < items[i].size(); k++) {
            int w_item = items[i][k].first;
            int v_item = items[i][k].second;
            if (j >= w_item) {
                dp[j] = max(dp[j], dp[j - w_item] + v_item);
            }
        }
    }
}
```

#### 问题3：树上DP套DP
**题目**：在树结构中，外层DP处理子树，内层DP处理节点选择状态。

**DP-in-DP思路**：
- 外层DP：DFS遍历处理每个子树
- 内层DP：处理当前节点被选中/不被选中状态

```cpp
void dfs(int u, int pa) {
    // 外层DP初始化：叶子节点的状态
    dp[u][0] = 0;  // 不选当前节点
    dp[u][1] = a[u]; // 选当前节点
    
    for (int v : g[u]) {
        if (v == pa) continue;
        dfs(v, u); // 递归处理子树
        
        // 内层DP：合并子树信息
        for (int j = ...); // 枚举容量/约束
            dp[u][...] = max/min(...); // 状态转移
    }
}
```

### DP-in-DP的设计范式

DP-in-DP的设计遵循以下范式：

1. **外层DP**: 定义主要问题的**状态空间**
2. **内层DP**: 解决**子问题约束**或**计算复杂费用**
3. **状态转移**: 外层DP状态通过内层DP计算得到

### 模板框架

```cpp
for (/* 外层DP枚举 */) {
    // 初始化内层DP状态
    init_inner_state();
    
    for (/* 内层DP枚举 */) {
        // 内层状态转移
        inner_dp_transfer();
    }
    
    // 用内层DP结果更新外层DP
    outer_dp_update();
}
```

### 时间复杂度分析

DP-in-DP的时间复杂度是**外层DP复杂度 × 内层DP复杂度**：

| 问题类型 | 外层DP | 内层DP | 总复杂度 |
|----------|--------|--------|----------|
| 二维背包 | O(n×W) | O(V) | O(n×W×V) |
| 树上DP | O(n) | O(V²) | O(n×V²) |
| 复杂费用DP | O(n) | O(k) | O(n×k) |

## 典型应用：树上背包DP（DP-in-DP）

### 题目

给定一棵$n$个节点的树，每个节点有体积$v_i$和价值$w_i$，背包容量为$V$。选择一些节点使其**权值和最大**，且**选择的节点权值之和 ≤ V**，同时**不能选择相邻节点**。

### 思路

这是一个**DP-in-DP**的典型例题：

- **外层DP**：枚举树的节点、背包容量
  - `dp[u][j][0]`：以u为根的子树中，背包容量为j时，**不选u**的最大价值
  - `dp[u][j][1]`：以u为根的子树中，背包容量为j时，**选u**的最大价值

- **内层DP**：枚举子树上的容量分配
  - 将子树的容量分配给不同的子节点

### 状态转移

```
dp[u][j][0] += max(dp[v][k][0], dp[v][k][1])  // 不选u，可以选或不选子节点
dp[u][j][1] += dp[v][k][0]                   // 选u，子节点不能选
```

### 代码模板

```cpp
void dfs(int u, int fa) {
    // 初始化：节点u自身的体积和价值
    for (int i = 0; i <= V; i++) {
        dp[u][i][0] = 0;
        dp[u][i][1] = (i >= v[u]) ? w[u] : -INF;
    }
    
    for (int e = head[u]; e; e = nxt[e]) {
        int v = to[e];
        if (v == fa) continue;
        dfs(v, u); // 递归处理子树
        
        // 内层DP：枚举容量分配
        for (int j = V; j >= 0; j--) {
            int val = dp[u][j][0]; // 不选u时的当前值
            dp[u][j][0] = 0;
            dp[u][j][1] = (j >= v[u]) ? w[u] : -INF;
            
            for (int k = j; k >= 0; k--) {
                // 子节点v的容量分配
                dp[u][j][0] = max(dp[u][j][0], 
                    dp[u][j-k][0] + max(dp[v][k][0], dp[v][k][1]));
                dp[u][j][1] = max(dp[u][j][1],
                    dp[u][j-k][1] + dp[v][k][0]);
            }
        }
    }
}
```

### 复杂度

- 外层DP：$O(n \times V)$
- 内层DP：$O(V^2)$ （容量分配枚举）
- **总复杂度**：$O(n \times V^2)$

## 常见陷阱

1. **枚举顺序**：
   - 外层DP和内层DP的枚举顺序不能混淆
   - 内层DP通常在外层DP状态确定后执行

2. **状态初始化**：
   - 内层DP的状态初始化需要在每次外层DP更新前重置
   - 否则可能导致状态污染

3. **边界条件**：
   - 内层DP的边界条件要与外层DP保持一致
   - 如背包DP中的 `dp[0] = 0`

## 总结

DP-in-DP (DP套DP) 是解决复杂最优化问题的利器：

1. **理解本质**：外层DP处理主问题，内层DP处理子问题
2. **设计方法**：先确定外层DP状态，再设计内层DP
3. **实现技巧**：注意枚举顺序、初始化和边界
4. **复杂度控制**：合理设置内层DP状态维度

**学习建议**：
- 先掌握单层DP基本套路（背包、LIS、区间DP）
- 再学习CDQ分治等进阶优化
- 最后掌握DP-in-DP在复杂图论/树论问题上的应用

DP-in-DP 的关键在于：**“识别何时需要嵌套DP”** — 当外层DP的状态转移需要在另一个维度上进行DP计算时，就需要考虑DP-in-DP。