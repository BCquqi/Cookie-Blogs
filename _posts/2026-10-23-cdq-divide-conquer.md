---
title: CDQ分治详解

date: 2026-10-23 14:00:00 +0800
categories: [algorithm,math]
tags: [CDQ, divide-and-conquer, data-structure]
description: CDQ分治算法详解：原理、应用场景、模板代码与典型例题

math: true
mermaid: true
toc: true
comments: true
pin: false
---

# CDQ分治

CDQ分治（CDQ Divide and Conquer）是一种用于解决**偏序问题**的离线算法，由陈东强提出。它本质上是**分治 + 数据结构**的结合体，能够将多维偏序问题转化为一维偏序问题，配合树状数组/线段树求解。

## 基本思想

对于三维偏序问题：给定点集 $(x_i, y_i, z_i)$，求每个点满足 $x_j \le x_i, y_j \le y_i, z_j \le z_i$ 的点的个数（或权值和）。

暴力复杂度 $O(n^2)$。CDQ分治利用分治将三维偏序拆解为一维偏序，复杂度 $O(n \log^2 n)$（若用树状数组）或 $O(n \log n)$（若用归并排序）。

## 核心原理

### 一维偏序
- 给定数组 $a[1..n]$，求每个 $i$ 有多少 $j < i$ 使得 $a_j \le a_i$
- 排序 + 树状数组即可

### 二维偏序
- 给定点 $(x_i, y_i)$，求每个点满足 $x_j \le x_i, y_j \le y_i$ 的点的个数
- 按 $x$ 排序，树状数组维护 $y$

### 三维偏序 - CDQ分治
- 给定点 $(x_i, y_i, z_i)$
- 第一维 $x$ 用**分治**处理（左半部分对右半部分的贡献）
- 第二维 $y$ 用**排序**处理
- 第三维 $z$ 用**树状数组**处理

## 算法流程

1. **分治**：按 $x$ 坐标分治，递归处理左半边和右半边
2. **合并**：处理“左半边的点对右半边点的贡献”
   - 将左半边和右半边的点按 $y$ 排序
   - 扫描过程中，遇到左半边的点，在树状数组中 $z$ 位置加权值
   - 遇到右半边的点，查询树状数组前缀和，累加到答案
3. **清理**：撤销树状数组的修改（或用时间戳/版本号技巧）

## 适用场景

| 场景 | 描述 |
|------|------|
| 多维偏序 | 三维/四维偏序，求个数/最大值/和 |
| 动态逆序对 | 插入/删除元素，维护逆序对数 |
| 区间 Kth | 区间第 K 大，配合二分/整体二分 |
| 矩阵乘法优化 | 某些 DP 转移可用 CDQ 优化 |
| 计数/统计 | 组合数学、多项式乘法等 |

## 模板代码

### 三维偏序计数

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;

struct Node {
    int x, y, z, id;
    bool operator<(const Node& o) const {
        return y < o.y || (y == o.y && z < o.z);
    }
};

const int N = 200005;
Node a[N], tmp[N];
int n, bit[N];
LL ans[N];

void add(int x, int v) {
    for (; x <= n; x += x & -x) bit[x] += v;
}

int query(int x) {
    int res = 0;
    for (; x; x -= x & -x) res += bit[x];
    return res;
}

void cdq(int l, int r) {
    if (l == r) return;
    int mid = (l + r) >> 1;
    cdq(l, mid);
    cdq(mid + 1, r);
    
    // 合并：左半边对右半边的贡献
    int i = l, j = mid + 1, k = l;
    while (i <= mid && j <= r) {
        if (a[i].y <= a[j].y) {
            add(a[i].z, 1);
            tmp[k++] = a[i++];
        } else {
            ans[a[j].id] += query(a[j].z);
            tmp[k++] = a[j++];
        }
    }
    while (i <= mid) tmp[k++] = a[i++];
    while (j <= r) {
        ans[a[j].id] += query(a[j].z);
        tmp[k++] = a[j++];
    }
    // 撤销修改
    for (int t = l; t <= mid; t++) add(a[t].z, -1);
    // 归并回原数组
    for (int t = l; t <= r; t++) a[t] = tmp[t];
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> a[i].x >> a[i].y >> a[i].z;
        a[i].id = i;
    }
    sort(a + 1, a + n + 1, [](const Node& A, const Node& B) {
        return A.x < B.x || (A.x == B.x && A.y < B.y);
    });
    cdq(1, n);
    for (int i = 1; i <= n; i++) cout << ans[i] << "\n";
    return 0;
}
```

### 三维偏序求最大值/最小值

```cpp
void cdq(int l, int r) {
    if (l == r) return;
    int mid = (l + r) >> 1;
    cdq(l, mid);
    cdq(mid + 1, r);
    
    int i = l, j = mid + 1, k = l;
    while (i <= mid && j <= r) {
        if (a[i].y <= a[j].y) {
            add(a[i].z, a[i].val);  // 存储值
            tmp[k++] = a[i++];
        } else {
            ans[a[j].id] = max(ans[a[j].id], query(a[j].z));
            tmp[k++] = a[j++];
        }
    }
    while (i <= mid) tmp[k++] = a[i++];
    while (j <= r) {
        ans[a[j].id] = max(ans[a[j].id], query(a[j].z));
        tmp[k++] = a[j++];
    }
    for (int t = l; t <= mid; t++) add(a[t].z, 0); // 清理
    for (int t = l; t <= r; t++) a[t] = tmp[t];
}
```

### 整体二分 + CDQ

```cpp
// 查询：区间第 K 大
struct Query {
    int op; // 1: 修改, 2: 询问
    int l, r, k, id, sign;
    // 修改: l=位置, r=值
    // 询问: l=左端点, r=右端点, k=第K大
} q[N];

void solve(vector<int> qs, int L, int R) {
    if (qs.empty()) return;
    if (L == R) {
        for (int id : qs) if (q[id].op == 2) ans[q[id].id] = L;
        return;
    }
    int mid = (L + R) >> 1;
    vector<int> left_qs, right_qs;
    // 左半边修改加入，右半边询问查询
    for (int id : qs) {
        if (q[id].op == 1) {
            if (q[id].r <= mid) {
                add(q[id].l, 1);
                left_qs.push_back(id);
            } else {
                right_qs.push_back(id);
            }
        } else {
            int cnt = query(q[id].r) - query(q[id].l - 1);
            if (cnt >= q[id].k) {
                left_qs.push_back(id);
            } else {
                q[id].k -= cnt;
                right_qs.push_back(id);
            }
        }
    }
    // 清理
    for (int id : left_qs) if (q[id].op == 1 && q[id].r <= mid) add(q[id].l, -1);
    solve(left_qs, L, mid);
    solve(right_qs, mid + 1, R);
}
```

## 关键细节

### 排序稳定性
- 合并时按 $y$ 排序，必须保持稳定性
- `stable_sort` 或手动归并排序
- 若 $y$ 相同，按 $z$ 排序，保证左半边先于右半边

### 树状数组清理
- 方法 1：记录修改过的位置，最后清零
- 方法 2：时间戳/版本号（若支持区间加点查）
- 方法 3：重新建树（复杂度高，不推荐）

### 坐标压缩
- $y$、$z$ 坐标范围大时需离散化
- 在 `cdq` 递归前统一完成，或在合并时动态处理

### 去重处理
- 相同点可能需要合并计数
- 排序时将相同点合并，记录权重

## 典型例题

### 例 1：三维偏序计数
**题目**：给定 $n$ 个点 $(x_i, y_i, z_i)$，求每个点满足 $x_j \le x_i, y_j \le y_i, z_j \le z_i$ 的点的个数（不含自身）。

**解**：标准 CDQ 分治，复杂度 $O(n \log^2 n)$。

### 例 2：动态逆序对
**题目**：维护序列，支持插入/删除，查询当前逆序对数。

**解**：整体二分 + CDQ，时间 $O(n \log^2 n)$。

### 例 3：区间第 K 大
**题目**：静态数组，多次询问区间 $[l,r]$ 第 $k$ 大。

**解**：整体二分 + 树状数组，或主席树 $O(n \log n)$，CDQ 适合动态版本。

### 例 4：矩阵乘法优化 DP
**题目**：$dp_i = \max_{j < i} (dp_j + f(i-j))$，$f$ 为凸/凹函数。

**解**：若 $f$ 可拆解为多项式，用 CDQ + FFT 优化到 $O(n \log^2 n)$。

## 复杂度分析

| 维度 | 复杂度 | 说明 |
|------|--------|------|
| 三维偏序 | $O(n \log^2 n)$ | 树状数组 $\log n$ + 分治 $\log n$ |
| 三维偏序 (归并) | $O(n \log n)$ | 无树状数组，纯归并排序 |
| 整体二分 | $O(n \log n \log V)$ | $V$ 为值域大小 |
| 动态逆序对 | $O(n \log^2 n)$ | CDQ + 整体二分 |

## 易错点与技巧

1. **离线 vs 在线**：CDQ 是离线算法，必须知道所有操作/询问
2. **等于号处理**：$x_j \le x_i$ 还是 $x_j < x_i$ 影响排序稳定性
3. **数组大小**：开 $4n$ 或 $n \times 4$ 避免越界
4. **long long**：答案可能超过 `int`，注意类型
5. **边界**：分治边界 $l == r$ 直接返回
6. **归并排序优化**：三维偏序若无需树状数组（如只求最大值），可用归并代替，降为 $O(n \log n)$

## CDQ vs 其他算法

| 算法 | 适用场景 | 复杂度 |
|------|----------|--------|
| CDQ 分治 | 多维偏序、动态问题 | $O(n \log^2 n)$ |
| 树状数组/线段树 | 1D/2D 偏序、区间查询 | $O(n \log n)$ |
| 主席树 | 静态区间第 K 大 | $O(n \log n)$ |
| 整体二分 | 动态第 K 大、中位数 | $O(n \log n \log V)$ |
| 莫队 | 离线区间查询 | $O(n \sqrt{n})$ |

## 学习建议

1. **先理解一维/二维偏序**：掌握排序 + 树状数组的基本套路
2. **理解分治的“左右贡献”**：左半边点只对右半边产生贡献
3. **手写模板**：多写几遍，熟练掌握归并、清理、稳定排序
4. **刷题积累**：洛谷 P3810、P3391、Codeforces 相关题
5. **拓展学习**：整体二分、树套树、FFT+CDQ 等高级技巧

## 参考资料

1. **原始论文**：陈东强，《一种用于解决三维偏序问题的算法》
2. **OI Wiki - CDQ 分治**：https://oi-wiki.org/ds/cdq-divide-conquer/
3. **《算法竞赛进阶指南》** - 李煜东
4. **常见题目**：洛谷 P3810【模板】三维偏序、P3391 [USACO08MAR] Cow Jog G、CF 868F Yet Another Minimization Problem

# 总结

CDQ 分治是解决高维偏序问题的利器，核心思想是**分治降维 + 数据结构**。它将复杂的多维问题拆解为一维问题，通过排序和树状数组高效求解。掌握 CDQ 需要理解：
- 分治的“左对右”贡献关系
- 多维坐标的处理顺序（分治、排序、数据结构）
- 稳定排序、坐标压缩、树状数组清理等工程细节

熟练掌握 CDQ 后，大多数三维偏序、动态逆序对、区间第 K 大等问题都能迎刃而解。