---
layout: post
title: codeforces713C Sonya and Problem Wihtout a Legend
date: 2018-09-16 15:56:25 +0800
categories: training
tags: 
img: https://vexoben.github.io/oi-blog/assets/images/Blog2/2018-10-06-codeforces713c-sonya-and-problem-wihtout-a-legend.png
---

题目链接： [codeforces713C Sonya and Problem Wihtout a Legend][1]

(刷新以获取数学公式)

### **题意**

给定一组数，每次可以用$$1$$的代价将一个数$$+1$$或$$-1$$，求将数列变为严格上升的最小代价。

n ≤ 3000

### **题解**

首先将$$a[i]$$变为$$a[i]-i$$就可以将题目转化为不严格上升。

然后注意到一个数变肯定变到原来有的$$a[i]$$中，直接上$$O(n^2)$$的DP就可以通过了

但这样没有什么技术含量。这题是有$$O(nlogn)$$的做法的。

定义函数$$f_i(x)$$为将前$$i$$个数变为严格上升，且保证$$a_{i}≤x$$的最小代价。我们有：

![][2]

显然，$$f_{i}(x)$$是一个斜率为非正整数的下凸包。

设$$opt(i)$$是最小的$$x$$使得$$f_i(x)$$取到最小值。那么$$opt(i)$$就是斜率从$$-1$$到$$0$$的转折点，我们可以得到：

![][3]

我们不能直接贪心的原因是$$opt(i)$$不能维护。

那么考虑维护$$f_{i}$$这个下凸包。

假设我们已经得到了$$f_{i-1}$$，现在加入$$a_{i}$$。可以发现，对于$$x﹤a_{i}$$的直线，加入$$a_{i}$$会使斜率$$-1$$，对于$$x>a_{i}$$的直线，加入$$a_{i}$$会使斜率$$+1$$(除非它的斜率已经是$$0$$，那么就不再增加)。

我们用一个priority_queue维护它。堆中第$$i$$大的元素，是斜率由$$-i$$转为$$-i+1$$的转折点的横坐标。

根据这个定义，弹出堆顶使堆中所有元素斜率$$+1$$，加入一个元素使小于它的所有元素斜率$$-1$$

如果$$a_{i} > opt(i - 1)$$，直接将$$a_{i}$$丢进堆即可；

否则，我们会发现加入$$a_{i}$$会使斜率发生断层。

比如，我们将$$a_{i}$$插入到斜率为$$-1$$的直线中间。本应使右边一段斜率变为了$$0$$，而左边那段斜率变成了$$-2$$。

但是事实上，我们将$$a_{i}$$插入，使得左边的斜率$$-1$$。又弹出了$$opt(i - 1)$$，使所有的斜率$$+1$$。这样，原本斜率应该变为$$-2$$的一段斜率还是$$-1$$。

于是就有一个小trick：只需要把$$a_{i}$$再插入一次就可以了。

事实证明这样做是正确的。总复杂度是$$O(nlogn)$$

```cpp
#include<bits/stdc++.h>
#define LL long long
using namespace std;
const int N = 1e5 + 10;

int n, x;
LL res;
priority_queue<int> Q;

int main() {
	scanf("%d", &n);
	for (int i = 1; i <= n; ++i) {
		scanf("%d", &x);
		x -= i;
		Q.push(x);
		if (Q.top() > x) {
			res += Q.top() - x;
			Q.pop();
			Q.push(x);
		}
	}
	cout << res << endl;
}
```

[1]: http://codeforces.com/contest/713/problem/C

[2]: https://vexoben.github.io/oi-blog/assets/images/Blog2/2018-10-06-codeforces713c-sonya-and-problem-wihtout-a-legend(2).png

[3]: https://vexoben.github.io/oi-blog/assets/images/Blog2/2018-10-06-codeforces713c-sonya-and-problem-wihtout-a-legend(3).png