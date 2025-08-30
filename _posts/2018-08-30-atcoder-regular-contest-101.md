---
layout: post
title: AtCoder Regular Contest 101
date: 2018-08-30 14:28:01 +0800
categories: contest
tags: DP
img: https://vexoben.github.io/assets/images/Blog/2018-08-30-atcoder-regular-contest-101.JPG
---

很妙妙的题，但是网上没有完整题解非常不爽QAQ

(刷新以获取数学公式)

## **E.Ribbons on Tree**

题目链接:[Ribbons on Tree][8]

### **题意**

给定一棵 $$ n $$ 个节点的树( $$ n $$ 为偶数) ,现要将树上的点分为 $$ \frac{n}{2} $$ 组,每一组的两个点由一条路径相连.求出每条边都被至少一条路径覆盖的分组方案数.

$$ n ≤ 5000 $$ 且为偶数

### **题解**

首先我们定义一个函数$$ g(x) $$

当 $$ x mod 2 = 1 $$， $$ g(x) = 0 $$

当 $$ x mod 2 = 0 $$， $$ g(x) = (x - 1) * (x - 3) * ... * 1 $$

即 $$ g(x) $$ 是在一个大小为 $$ x $$的联通块中随意分组的方案数。

设 $$ E $$ 是原图的一个边集， 我们很容易求出使 $$ E $$ 中的边都不被覆盖的方案数：它等于断开 $$ E $$ 中的边后，所有连通块大小的 $$ g $$函数乘积。

最暴力的做法就可以直接容斥了。设 $$ f(E) = E $$ 中的边都不被覆盖的方案数，那么答案就是：

$$ \sum_{E} (-1)^{siz(E)}f(E) $$

而巧妙的地方在于，这个式子可以用树形dp计算。

令 $$ dp[i][j][k] $$ 表示子树 $$ i $$ 中， 根所在联通块大小为 $$ j $$， 切断边数的奇偶性为 $$ k $$ 的答案。

需要注意的是，以下两份代码的复杂度是不同的：

```cpp
void Solve(int x) {
	  siz[x] = 1;
	  for (int i = fir[x]; i; i = nex[i]) {
	  		if (arr[i] == fa[x]) continue;
	  		Solve(arr[i]);
			siz[x] += siz[arr[i]];
	  }
	  // dp
}
```

```cpp
void Solve(int x) {
	  siz[x] = 1;
	  for (int i = fir[x]; i; i = nex[i]) {
	  		if (arr[i] == fa[x]) continue;
			Solve(arr[i]);
			// dp
			siz[x] += siz[arr[i]];
	  }
}
```

第一份代码显然是 $$ O(n ^ 3) $$ 的， 但第二份代码是 $$ O(n ^ 2) $$的。

简要证明一下：

当转移到子树 $$ x $$ 时，设它的子树大小分别为 $$ S_1 $$ ， $$ S_2 $$ ， ... ， $$ S_{n} $$

那么转移子树 $$ x $$ 的复杂度就是：

$$ \sum _ {i =  1} ^ {n} S_{i} * \sum _ {j = 1} ^ {i - 1} S_{j} $$

即 ![][7]

考虑一对点 $$ (i, j) $$ 仅当 $$ i $$ ， $$ j $$ 在不同子树中时对复杂度才有贡献。

而 $$ (i, j) $$ 在不同子树中的次数显然仅有一次，且位于两点的lca处，从而复杂度是 $$ O(n ^ 2) $$。

```cpp
#include<bits/stdc++.h>
#define LL long long
using namespace std;
const int N = 5005;
const int mod = 1e9 + 7;

int n, E;
int fa[N], siz[N];
int fir[N], nex[N << 1], arr[N << 1];
LL g[N], tmp[N][2], f[N][N][2];

inline void Add_Edge(int x, int y) {
	nex[++E] = fir[x];
	fir[x] = E; arr[E] = y;
}

void Solve(int x) {
	siz[x] = f[x][1][0] = 1;
	for (int i = fir[x]; i; i = nex[i]) {
		if (arr[i] == fa[x]) continue;
		fa[arr[i]] = x;
		Solve(arr[i]);
		for (int j = 1; j <= siz[x] + siz[arr[i]]; ++j) tmp[j][0] = tmp[j][1] = 0;
		for (int j = 1; j <= siz[x]; ++j) {
			for (int k = 1; k <= siz[arr[i]]; ++k) {
				for (int l = 0; l < 2; ++l) {
					tmp[j + k][l] += f[arr[i]][k][0] * f[x][j][l];
					tmp[j + k][l] += f[arr[i]][k][1] * f[x][j][l ^ 1];
					tmp[j][l] += f[arr[i]][k][0] * f[x][j][l ^ 1] % mod * g[k];
					tmp[j][l] += f[arr[i]][k][1] * f[x][j][l] % mod * g[k];
					tmp[j][l] %= mod;
					tmp[j + k][l] %= mod;
				}
			}
		}
		for (int j = 1; j <= siz[x] + siz[arr[i]]; ++j) {
			f[x][j][0] = tmp[j][0];
			f[x][j][1] = tmp[j][1];
		}
		siz[x] += siz[arr[i]];
	}
}

int main() {
	cin >> n;
	for (int i = 1; i < n; ++i) {
		int x, y; scanf("%d%d", &x, &y);
		Add_Edge(x, y); Add_Edge(y, x);
	}
	g[0] = 1;
	for (int i = 1; i <= n; ++i) {
		if (i & 1) g[i] = 0;
		else g[i] = g[i - 2] * (i - 1) % mod;
	}
	LL ans = 0;
	Solve(1);
	for (int i = 1; i <= n; ++i) {
		ans += f[1][i][0] * g[i];
		ans -= f[1][i][1] * g[i];
		ans %= mod;
	}
	cout << (ans + mod) % mod << endl;
	return 0;
}
```

## **F.Robots and Exits**

题目链接：[Robots and Exits][9]

### **题意**

在 X 轴上有 $$ n $$ 个机器人和 $$ m $$ 个出口，坐标互不相同。每次可以让所有机器人坐标同时加一或减一。当一个机器人位于出口，它将消失。求所有机器人出出口的方案数。

$$ n, m ≤ 100000 $$ ，答案膜 1e9+7。

### **题解**

在最靠左出口左边和最靠右出口右边的机器人可以直接无视。

剩下的机器人可能走到左右两个出口之一，记距离左右两个出口的距离为 $$ (x_{i}, y_{i}) $$

显然对答案会产生影响的是历史向左走最大值和向右走最大值，记为$$(x, y)$$ ， 则每次可以让 $$ x $$ 或 $$ y $$ 加一。

我们的问题就转化为：在一个二维平面上，从 $$ (0, 0) $$ 开始，每次可以向上或向右走， 问先走到 $$ x = x_{i} $$ 或先走到 $$ y = y_{i} $$的方案数是多少。

这个问题可以用DP解决。

首先我们可以发现， 我们走的路线是一条折线， 且每个拐角处一定恰好顶到某组未确定直线中的一条。在满足这样条件的情况下，拐角不同的两条线答案不同。

我们可以枚举最后一个由向上转至向右的拐角在哪里，设转角在 $$ (x_{i}, y_{i}) $$ 的方案数为 $$ dp[i] $$。

那么 $$ dp[i] = (\sum _ {j = 1} ^ {i - 1} [y_{i} > y_{j}] dp[j]) + 1 $$

树状数组优化一下转移就是 $$ O(n * logn) $$了。

```cpp
#include<bits/stdc++.h>
#define fi first
#define se second
#define mp make_pair
#define pii pair<int, int>
using namespace std;
const int N = 1e5 + 10;
const int mod = 1e9 + 7;

int n, m, x[N], y[N], tr[N];
pii a[N];

bool cmp(pii a, pii b) {
	return (a.fi != b.fi) ? a.fi < b.fi : a.se > b.se;
}

void init() {
	int cnt = 0;
	cin >> n >> m;
	for (int i = 1; i <= n; ++i) scanf("%d", &x[i]);
	for (int i = 1; i <= m; ++i) scanf("%d", &y[i]);
	for (int i = 1; i <= n; ++i) {
		if (x[i] < y[1] || x[i] > y[m]) continue;
		int t = lower_bound(y + 1, y + m + 1, x[i]) - y;
		a[++cnt] = mp(x[i] - y[t - 1], y[t] - x[i]);
	}
	sort(a + 1, a + cnt + 1, cmp);
	n = unique(a + 1, a + cnt + 1) - a - 1;
	m = 0;
	for (int i = 1; i <= n; ++i) y[++m] = a[i].se;
	sort(y + 1, y + m + 1);
	m = unique(y + 1, y + m + 1) - y - 1;
	for (int i = 1; i <= n; ++i)
		a[i].se = lower_bound(y + 1, y + m + 1, a[i].se) - y; 
}

inline void Add(int x, int val) {
	for (++x; x < N; x += x & -x) {
		tr[x] += val;
		if (tr[x] >= mod) tr[x] -= mod;
	}
}

inline int Query(int x) {
	int res = 0;
	for (++x; x; x -= x & -x) {
		res += tr[x];
		if (res >= mod) res -= mod;
	}
	return res;
}

void Solve() {
	int res = 1;
	for (int i = 1; i <= n; ++i) {
		int del = Query(a[i].se - 1) + 1;
		res += del;
		if (res >= mod) res -= mod;
		Add(a[i].se, del);
	}
	cout << res << endl;
}

int main() {
	init();
	Solve();
	return 0;
}

```

[7]: https://vexoben.github.io/assets/images/Blog/2018-08-30-atcoder-regular-contest-101(2).png

[8]: https://arc101.contest.atcoder.jp/tasks/arc101_c

[9]: https://arc101.contest.atcoder.jp/tasks/arc101_d