---
layout: post
title: codeforces718C Sasha and Array
date: 2018-09-16 17:40:08 +0800
categories: training
tags: 矩阵快速幂 线段树及可持久化
img: https://vexoben.github.io/oi-blog/assets/images/Blog2/2018-10-06-codeforces718c-sasha-and-array.png
---

题目链接: [codeforces718C Sasha and Array][1]

(刷新以获取数学公式)

### **题意**

给定一组数$$a$$，要求支持两种操作：

1、将$$[l, r]$$中的数加上$$x$$

2、求$$\sum_{i = l}^{r} f(a[i])$$，其中$$f(x)$$是第$$x$$个斐波那契数。

n,m  ≤ 100000

### **题解**

斐波那契数的第$$i$$项就是$$ \dbinom{0 \;\;1 }{1 \;\; 1}$$的$$i$$次方左下角的元素。

对每个数存下对应的矩阵，求和相当于矩阵求和，区间加相当于区间乘一个矩阵，复杂度为$$O(nlogn)$$

```cpp
#include<bits/stdc++.h>
#define LL long long
using namespace std;
const int N = 1e5 + 10;
const int mod = 1e9 + 7;

int n, m, a[N];
struct Matrix {
	LL a[2][2];
}A, E, tmp, tr[N << 2], tag[N << 2];

Matrix operator * (Matrix a, Matrix b) {
	Matrix ans = (Matrix){0, 0, 0, 0};
	for (int i = 0; i < 2; ++i) {
		for (int j = 0 ;j < 2; ++j) {
			for (int k = 0; k < 2; ++k) {
				ans.a[i][j] += 1LL * a.a[i][k] * b.a[k][j];
			}
			ans.a[i][j] %= mod;
		}
	}
	return ans;
}

Matrix operator + (Matrix a, Matrix b) {
	Matrix ans = (Matrix) {0, 0, 0, 0};
	for (int i = 0; i < 2; ++i) {
		for (int j = 0; j < 2; ++j) {
			ans.a[i][j] = (a.a[i][j] + b.a[i][j]) % mod;
		}
	}
	return ans;
}

bool operator == (Matrix a, Matrix b) {
	for (int i = 0; i < 2; ++i) {
		for (int j = 0; j < 2; ++j) {
			if (a.a[i][j] != b.a[i][j]) return false;
		}
	}
	return true;
}

Matrix Qpow(int k) {
	Matrix res = E, x = A;
	while (k) {
		if (k & 1) res = res * x;
		x = x * x;
		k >>= 1;
	}
	return res;
}

inline void Pushdown(int x) {
	if (tag[x] == E) return;
	tr[x << 1] = tr[x <<  1] * tag[x];
	tag[x << 1] = tag[x << 1] * tag[x];
	tr[x << 1 | 1] = tr[x << 1 | 1] * tag[x];
	tag[x << 1 | 1] = tag[x << 1 | 1] * tag[x];
	tag[x] = E;
}

void Build(int x, int l, int r) {
	if (l == r) {
		tr[x] = Qpow(a[l]);
		tag[x] = E;
		return;
	}
	int mid = (l + r) >> 1;
	Build(x << 1, l, mid);
	Build(x << 1 | 1, mid + 1, r);
	tag[x] = E;
	tr[x] = tr[x << 1] + tr[x << 1 | 1];
}

void Updata(int x, int l, int r, int ql, int qr, int num) {
	if (ql <= l && r <= qr) {
		tr[x] = tr[x] * tmp;
		tag[x] = tag[x] * tmp;
		return;
 	}
	Pushdown(x);
	int mid = (l + r) >> 1;
	if (mid >= ql) Updata(x << 1, l, mid, ql, qr, num);
	if (mid < qr) Updata(x << 1 | 1, mid + 1, r, ql, qr, num);
	tr[x] = tr[x << 1] + tr[x << 1 | 1];
}

Matrix Query(int x, int l, int r, int ql, int qr) {
	if (ql <= l && r <= qr) return tr[x];
	Pushdown(x);
	int mid = (l + r) >> 1;
	Matrix ans = (Matrix) {0, 0, 0, 0};
	if (mid >= ql) ans = ans + Query(x << 1, l, mid, ql, qr);
	if (mid < qr) ans = ans + Query(x << 1 | 1, mid + 1, r, ql, qr);
	return ans;
}

int main() {
	E.a[0][0] = E.a[1][1] = 1;
	A = (Matrix) {0, 1, 1, 1};
	scanf("%d%d", &n, &m);
	for (int i = 1; i <= n; ++i)
		scanf("%d", &a[i]);
	Build(1, 1, n);
	while (m--) {
		int type;
		scanf("%d", &type);
		if (type == 2) {
			int l, r;
			scanf("%d%d", &l, &r);
			Matrix ans = Query(1, 1, n, l, r);
			printf("%lld\n", ans.a[1][0]);
		}
		else {
			int l, r, num;
			scanf("%d%d%d", &l, &r, &num);
			tmp = Qpow(num);
			Updata(1, 1, n, l, r, num);
		}
	}
	return 0;
}
```

[1]: http://codeforces.com/contest/718/problem/C