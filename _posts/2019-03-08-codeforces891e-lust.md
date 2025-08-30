---
layout: post
title: Codeforces891E Lust
date: 2019-03-08 13:08:26 +0800
categories: training
tags: 生成函数 组合数学
img: https://vexoben.github.io/assets/images/Blog2/2019-03-08-codeforces891e-lust.png
---

题目链接:[CF891E Lust][100]

又被神仙题D没了

(刷新以获取数学公式)

## **题意**

给定$$n$$个数,第$$i$$个数为$$a_i$$.进行$$k$$次操作,每次随机选择一个数$$i$$,将答案加上$$\prod_{x≠i}a_x$$,并$$a_i-=1$$,求最后答案的期望.

数据范围: $$n≤5000,k≤1e9$$,模$$1e9+7$$输出

加强:$$n≤100000$$

## **题解**

选到数$$i$$对答案的贡献是$$\prod_{x≠i}a_x$$,这不好计算.但是我们惊奇地发现,这个式子等于:

$$\prod_{x=1}^n a_x - \prod_{x=1}^n a'_x$$

其中$$a'​$$是$$a_i-=1​$$后的$$a​$$数组.

显然可以发现操作顺序对答案没有影响.

那么我们枚举每个数被选到多少次.设第$$i​$$个数被选到的次数为$$b_i​$$.

可以发现第一个数对答案的贡献是$$(\prod a_i - \prod a'_i)+(\prod a'_i - \prod a''_i) +... = \prod a_i - \prod a^{(b1)}_i$$

最后可以发现游戏的答案就是 $$\prod a_i - \prod (a_i - b_i)$$

因此我们的答案就是以下式子:

$$ \frac{1}{n^k} \sum_{\sum b_i = k} \frac{(\prod_{i=1}^{n} a_i - \prod_{i=1}^{n} (a_i - b_i))k!}{\prod_{i=1}^{n}b_i!} $$

稍微处理一下就是:

$$ \prod_{i=1}^n a_i - (\frac{k!}{n^k}\sum_{\sum b_i = k}\frac{\prod_{i=1}^n (a_i - b_i) }{\prod_{i=1}^n b_i!}) ​$$

这步之后用生成函数进行优化,设

$$f(x) = \sum_{\sum b_i = k}\frac{\prod_{i=1}^n (a_i - b_i) }{\prod_{i=1}^n b_i!}$$

那么它的生成函数就是:

$$ F(x) = \prod_{i=1}^{n} (\sum_{j=1}^n (a_i - j) \frac{x^j}{j!}) $$

下面不加证明地给出最近学习生成函数得到的几个式子:

$$ e^{ax^b} = \sum_{k≥0} \frac{(ax^b)^k}{k!} .............(1)$$

$$ -ln(1-ax^b) = \sum_{k≥1} \frac{(ax^b)^k}{k} ....(2)$$

$$ xe^x = \sum_{k≥0} \frac{x^k}{(k-1)!} ............(3)$$

根据式$$(1)(3)$$,最终可以将式子化简为:

$$ \prod_{i=1}^{n}a_i - (\frac{k!}{n^k} (e^{nk}\prod_{i=1}^n(a_i-x))[x^k]) $$

其中 $$f(x)[x^k]$$ 表示 $$f(x)$$ 的 $$k$$ 次项系数.

将 $$\prod_{i=1}^n(a_i-x)$$暴力(或者分治NTT)算出来,$$e^{nx}$$脑补一下泰勒展开就可以计算了.

```cpp
#include<bits/stdc++.h>
#define LL long long
using namespace std;
const int N = 5005;
const int mod = 1e9 + 7;

int n, k;
int a[N];
LL f[N], g[N], s[N];

LL Qpow(LL x, int p) {
	LL ans = 1;
	for (; p; p >>= 1) {
		if (p & 1) ans = ans * x % mod;
		x = x * x % mod;
	}
	return ans;
}

LL Inv(LL x) {
	return Qpow(x, mod - 2);
}

void upd(LL &x, LL y) {
	x = (x + y >= mod) ? (x + y - mod) : (x + y);
}

int main() {
	cin >> n >> k;
	for (int i = 1; i <= n; ++i)
		cin >> a[i];
	s[0] = 1; s[1] = k;
	for (int i = 2; i <= n; ++i) {
		s[i] = s[i - 1] * (k - i + 1) % mod;
	}
	LL ans = 1;
	for (int i = 1; i <= n; ++i) {
		ans = ans * a[i] % mod;
	}
	f[0] = 1;
	for (int i = 1; i <= n; ++i) {
		for (int j = n; j >= 0; --j) {
			g[j] = 0;
			upd(g[j], f[j] * a[i] % mod);
			if (j) upd(g[j], mod - f[j - 1]);
		}
		for (int j = 0; j <= n; ++j) {
			f[j] = g[j];
		}
	}
	LL ans2 = 0;
	for (int i = 0; i <= n && i <= k; ++i) {
		LL tmp = Qpow(n, k - i) * s[i] % mod * f[i] % mod;
		upd(ans2, tmp);
	}
	ans2 = ans2 * Inv(Qpow(n, k)) % mod;
	upd(ans, mod - ans2);
	printf("%lld\n", ans);
	return 0;
}
```

[100]: http://codeforces.com/problemset/problem/891/E