---
title: 经典算法梳理
date: 2025-11-09 11:42:45 +0800
categories: [algorithm]
tags: []
description: 经典算法的全面梳理
toc: true
comments: true
pin: false
math: true
mermaid: true
---

# 经典算法梳理
## 前言
本文仅为经典算法的展示，有模板的直接上代码加注释，没有模板的会简要梳理思想。\
目前更新到大约CSP-J难度，持续更新中。
## 约瑟夫 - 转圈循环
```cpp
bool flag[1005]; // 是否被淘汰

int id = 0; // id记录判断到第几人
for (int i = 1;i <= n;i++) { // 枚举每轮（因为淘汰n个人，所以循环n次）
    for (int j = 1;j <= m;j++) {
        id = (id + 1) % n; // id更新
        if (id == 0) id = n; // 处理没有余数的特殊情况
        while (flag[id]) { // 如果已经被淘汰就要跳过
            id = (id + 1) % n;
            if (id == 0) id = n;
        }
    }
    cout << id << ' '; // m轮结束，id指向的人就淘汰
    flag[id] = true; // 标记被淘汰
}
```
## 前缀和与差分
前缀和数组`sum[i]`代表了`a[1] + a[2] + a[3] + ... + a[i]`，差分数组`diff[i]`则代表`a[i] - a[i - 1]`，差分与前缀和互为逆运算。
### 前缀和
```cpp
sum[0] = 0;
for (int i = 1;i <= n;i++) { // 前缀和数组的预处理
    cin >> a[i];
    sum[i] = sum[i - 1] + a[i]; // 递推求和思路
}
int q;
cin >> q;
for (int i = 1;i <= q;i++) {
    int l,r;
    cin >> l >> r;
    cout << sum[r] - sum[l - 1] << endl; // 前缀和查询，需要注意左端点要减一，可以自行推出
}
```
### 差分
```cpp
for (int i = 1;i <= n;i++) {
    cin >> a[i];
    d[i] = a[i] - a[i - 1]; // 计算差分数组
}
while (m--) {
    int l,r,c;
    cin >> l >> r >> c;
    d[l] += c,d[r + 1] -= c; // 使用差分数组进行区间加法，需要注意右端点要加一，可以自行推出
}
for (int i = 1;i <= n;i++) {
    sumd[i] = sumd[i - 1] + d[i]; // 根据差分和前缀和的逆运算关系还原数组（求差分数组的前缀和）
    cout << sumd[i] << ' ';
}
cout << endl;
```
## dfs - 深度优先搜索
### 一维
```cpp
void dfs(int step) {
	if (step > n) { // 当完成了所有决策后
		for (int i = 1;i <= n;i++) // 进行对决策的处理
            cout << plan[i] << ' ';
		return ; // 勿忘结束算法
	}
	for (int i = 0;i <= 9;i++) { // 枚举所有可能决策
		plan[step] = i; // 设置决策
		dfs(step + 1); // 进行后续决策
        plan[step] = -1; // 回溯（将数组返回初始值防止冗余数据干扰后续决策，本题是否添加并不影响）
	}
}
```
### 二维 - 连通块
```cpp
char a[25][25]; // 地图
int dx[] = {-1,1,0,0},dy[] = {0,0,-1,1}; // 方向移动点枚举（简化代码用）
int m,n,ans = 0;
bool vis[25][25]; // 是否走到过

void dfs(int x,int y) {
	vis[x][y] = true; // 访问过后优先标记
	ans++; // 又可以搜到一个点
	for (int i = 0;i < 4;i++) { // 遍历四个方向
		int nx = x + dx[i],ny = y + dy[i]; // 移动后的(x,y)坐标
		if (nx < 1 || nx > n || ny < 1 || ny > m) continue; // 位置越界
		if (a[nx][ny] == '#') continue; // 有障碍物
		if (vis[nx][ny]) continue; // 遍历过了（阻止走回头路）
		dfs(nx,ny); // 以这个点为下一步的起点，继续搜索
	}
	return ;
}

int tmpx,tmpy; // 起点坐标
for (int i = 1;i <= n;i++)
    for (int j = 1;j <= m;j++) {
        cin >> a[i][j];
        if (a[i][j] == '@') tmpx = i,tmpy = j;
    }
dfs(tmpx,tmpy);
```
## bfs - 广度优先搜索
### 一维
```cpp
void bfs() {
	queue<int> q; // 定义任务队列
	q.push(a); // 添加起点
	dis[a] = 0; // dis数组标记
	vis[a] = true; // vis数组标记
	while (q.size()) { // q没有清空
		int now = q.front(); q.pop();
		if (now == b) return ; // 得到目标值
        // 根据题目内容分类讨论，本题的含义是数值从a开始向上或向下k[now]步最终获得b
		int nxt1 = now + k[now]; // 计算下一个可能到达的地点
		if (1 <= nxt1 && nxt1 <= n && !vis[nxt1]) {
			dis[nxt1] = dis[now] + 1; // 更新dis数组
			vis[nxt1] = true; // 更新vis数组
			q.push(nxt1); // 添加queue任务
		}
		int nxt2 = now - k[now]; // 同理
		if (1 <= nxt2 && nxt2 <= n && !vis[nxt2]) {
			dis[nxt2] = dis[now] + 1;
			vis[nxt2] = true;
			q.push(nxt2);
		}
	}
}
```
### 二维 - 最短路
```cpp
char a[55][55]; // 地图
int dis[55][55];
bool vis[55][55];
int sx,sy,ex,ey; // 记录起点和终点坐标
int dx[] = {-1,1,0,0},dy[] = {0,0,-1,1};
int n,m;

struct Node {
	int x,y;
}; // 记录一个坐标点

void bfs() {
	queue<Node> q; // bfs的坐标点队列
	q.push({sx,sy}); // bfs起点入队初始化
	dis[sx][sy] = 0;
	vis[sx][sy] = true;
	while (q.size()) { // 任务队列非空
		int x = q.front().x,y = q.front().y; q.pop(); // 取出队首坐标
		if (x == ex && y == ey) return ; // 到达终点，结束搜索
		for (int i = 0;i < 4;i++) { // 遍历四个方向
			int nx = x + dx[i],ny = y + dy[i]; // 更新坐标
			if (nx < 1 || nx > n || ny < 1 || ny > m) continue; // 越界
			if (a[nx][ny] == '0') continue; // 障碍物
			if (vis[nx][ny]) continue; // 已经访问过
			q.push({nx,ny}); // 新坐标加入队列
			vis[nx][ny] = true; // 标记访问过
			dis[nx][ny] = dis[x][y] + 1; // 多走了一步，距离更新
		}
	}
}

bfs();
if (vis[ex][ey]) cout << dis[ex][ey] << endl;
else cout << -1 << endl;
```
## 贪心
朴素贪心没有标准代码，其思路是**每步选取当前最优解**；当然，不是所有题目都可以使用贪心算法。使用贪心算法要保证的一条重要性质是**无后效性**。
### 区间选点问题
```cpp
struct Node {
	int b,e;
} s[200005]; // 定义一个区间

bool cmp(Node x,Node y){return x.e < y.e;} // 区间排序规则 - 右端点排序

sort(s + 1,s + n + 1,cmp); // 根据排序规则排序
int ans = 1,pik = 1; // ans记录区间选择个数，pik记录上一个选择的区间位置
for (int i = 2;i <= n;i++)
    if (s[i].b > s[pik].e) ans++,pik = i; // 如果区间与前面的区间没有重叠部分，那么就要加一个点（有重叠部分这里默认的思路是将点设置在有重叠部分中间）
```
### 区间覆盖问题
```cpp
struct Node {
	int b,e;
} a[200005];

bool cmp(Node x,Node y) {return x.b < y.b;} // 区间排序规则 - 左端点排序

sort(a + 1,a + n + 1,cmp); // 根据排序规则排序
int ans = 0,pik = -1e9; // ans记录区间选择个数，pik记录当前覆盖的最远右端点
for (int i = 1;i <= n;i++) {
    if (a[i].b <= s && s <= a[i].e) pik = max(pik,a[i].e); // 区间中包含起点就尝试更新pik
    if (i == n || a[i + 1].b > s) { // 尝试的区间已经超过了尝试的起点，即起点应该更新了
        ans++; // 增加需要选择的区间数量（这里默认的就是区间包含s中终点e最靠后的那个区间）
        s = pik; // 起点更新至目前覆盖区域的最右端
        if (s >= t) break; // 覆盖完成
        pik = -1e9; // 重置pik
    }
}
if (s < t) cout << "impossible" << endl;
else cout << ans << endl;
```
## 快速幂
```cpp
typedef long long LL;
// 快速幂 a ^ b % p
LL qpow(LL a,LL b,LL p)
{
	LL ans = 1; // 答案
	while (b) { // b没有完全被二进制拆分时
		if (b & 1) ans = ans * a % p; // b拆出二进制最后一位1，ans返回值（a^原来的指数）就继续乘以（a^后续统计的指数）
		a = a * a % p; // 位权翻倍，a的次数翻倍
		b >>= 1; // 继续往后推移一位
	}
	return ans;
}
```
## 高精度（可自行封装）
```cpp
void s2BIG (string s,int a[]) // 将输入的字符串倒序存入数组（倒序计算进位方便），特别地，a[0]记录数的长度
{
	int al = s.length();
	for (int i = 1;i <= al;i++) a[i] = s[al - i] - '0';
	a[0] = al;
}

void printBIG(int a[]) // 倒序输出计算后的数组（反过来再反过来完成还原）
{
	int al = a[0];
	for (int i = al;i >= 1;i--) cout << a[i];
}

bool cmpBIG(int a[],int b[]) // 基于倒序数组逐位比较（返回true代表a < b）
{
	int al = a[0],bl = b[0];
	if (al != bl) return al < bl;
	for (int i = al;i >= 1;i--) if (a[i] != b[i]) return a[i] < b[i];
	return false;
}

void addBIG(int a[],int b[],int c[]) // 相加
{
	int al = a[0],bl = b[0];
	int cl = max(al,bl); // 忽略首位进位情况下，得出结果的长度是两数中较长那个数的长度
	int u = 0; // 记录进位
	for (int i = 1;i <= cl;i++)
	{
		int t = u; // 这一位加起来的和
		if (i <= al) t += a[i];
		if (i <= bl) t += b[i];
		c[i] = t % 10; // 记录
		u = t / 10; // 更新进位
	}
	if (u > 0) // 首位进位
	{
		cl++;
		c[cl] = u;
	}
	c[0] = cl; // 更新长度
}

void subBIG(int a[],int b[],int c[]) // 求差（大减小）
{
	int al = a[0],bl = b[0];
	int cl = al; // 默认al较大
	int u = 0; // 记录退位
	for (int i = 1;i <= cl;i++)
	{
		int t = a[i] - u; // 一开始就退位
		if (i <= bl) t -= b[i]; // 逐位减去b
		if (t < 0) // 仍然需要退位
		{
			c[i] = t + 10; // 借位借了10
			u = 1; // 两数相减借位最多为1，这里有借位就必定是1
		}
		else  // 正常计算
		{
			c[i] = t;
			u = 0;
		}
	}
	while (c[cl] == 0 && cl > 1) cl--; // 从最后开始计算退掉多少位
	c[0] = cl; // 更新长度
}

void mulBIG(int a[],int b,int c[]) // 相乘（高精乘单精）
{
	int cl = a[0];
	long long u = 0;
	for (int i = 1;i <= cl;i++)
	{
		int t = 1ll * a[i] * b + u; // 因为a长度默认更长，所以无需判断，直接乘
		c[i] = t % 10;
		u = t / 10;
	}
	while (u > 0)
	{
		cl++;
		c[cl] = u % 10; // 此时进位可能不止1
		u /= 10;
	}
	c[0] = cl;
}

void divBIG(int a[],int b,int c[]) // 相除（以）（高精除（以）单精）
{
	int cl = a[0];
	long long r = 0;
	for (int i = cl;i >= 1;i--)
	{
		r = r * 10 + a[i]; // 位值原理
		c[i] = r / b; // 直接取除以的得数
		r %= b; // 取余
	}
	while (c[cl] == 0 && cl > 1) cl--; // 像减法那样向前偏移位
	c[0] = cl;
}
```
## 二分查找
二分查找是一种`#include<algorithm>`中自带的算法：\
`int pos = lower_bound(a + 1,a + n + 1,b) - a;` 代表找出数组a中第一个 >= b的元素位置；\
`int pos = upper_bound(a + 1,a + n + 1,b) - a;` 代表找出数组a中第一个 > b的元素位置。
## 二分答案
```cpp
int l[100005],n,k;

bool check(int mid) {} // 这个函数里检查的东西依题目而定

int L = 1,R = 1e8,ans = -1;
while (L <= R) // 区间缩小最后剩下的一个ans就是最终答案
{
    int mid = L + R >> 1; // L和R的中间值
    if (check(mid)) { // 这里符合向右边的条件就取右半段，ans更新为目前答案值
        L = mid + 1;
        ans = mid;
    }
    else R = mid - 1; // 否则取左半段
}
```
## 动态规划dp
后续会出文章专门讲解dp，敬请期待！
## 分治法（ex快速排序&归并排序）
```cpp
void quick_sort(int a[],int l,int r) {
    if (l >= r) return ; // 递归终止条件
    int x = a[l + r >> 1]; // 选取基准值
    int i = l,j = r;
    while (i <= j) {
        while (a[i] < x) i++; // i逼近最近的大于等于基准值的位置
        while (a[j] > x) j--; // j逼近最近的小于等于基准值的位置
        if (i <= j) swap(a[i++],a[j--]); // 顺序反了就交换
    }
    quick_sort(a,l,j); // 两侧再分别处理
    quick_sort(a,i,r);
}

void merge_sort(int a[],int l,int r) {
    if (l >= r) return ;
    int mid = l + r >> 1; // 选中间位置
    merge_sort(a,l,mid); // 分别处理
    merge_sort(a,mid + 1,r);
    int i = l,j = mid + 1,cur = l;
    while (i <= mid && j <= r) { // 两侧同时合并
        if (a[i] < a[j]) tmp[cur++] = a[i++]; // 左侧插入
        else tmp[cur++] = a[j++]; // 右侧插入
    }
    while (i <= mid) tmp[cur++] = a[i++]; // 插入剩下的
    while (j <= r) tmp[cur++] = a[j++]; // 插入剩下的
    for (int i = l;i <= r;i++) a[i] = tmp[i]; // 复制回原数组
}
```
持续更新中……
