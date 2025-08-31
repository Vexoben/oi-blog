---
layout: post
title: CodeForces367C Sereja and the Arrangement of Numbers
date: 2018-02-10 20:50:12 +0800
categories: training
tags: 脑洞题 图论 欧拉路径
img: assets/images/Blog/2018-02-10-codeforces367c-sereja-and-the-arrangement-of-numbers.JPG
---

题目链接：[http://codeforces.com/problemset/problem/367/C][1]

## **题意**

给出m个不同的数，并且每个数都有个费用，现在要在m个数中选择一些数（每个数可使用任意多次），用这些数组成一个长度为n的数列，并且满足任意两个不同种类的数都相邻。问最大的费用是多少。

m<=1e5,n<=2e6

## **题解**

关键在于确定：使数列中有k个不同的数，数列的最小长度是多少。由于任意两个不同种类的数都相邻，联想到完全图的欧拉回路，即走过多少条边可以使每条边都经过一次，显然次数+1就是最小长度。有奇数个点的时候可以一笔画，只需(k-1)*k/2次即可，如果是偶数则需要额外的(k-2)/2次。

```cpp
#include<bits/stdc++.h>
#define int long long
using namespace std;
const int N=2e5+10;

int n,m,val[N];

int check(int mid) {
	int x=mid*(mid-1)/2;
	if (mid%2==0) x+=(mid-2)/2;
	return n>=x+1;
}

signed main() {
	cin>>n>>m;
	for (int i=1;i<=m;i++) {
		int x,y; cin>>x>>y;
		val[i]=y;
	}
	sort(val+1,val+m+1);
	int l=1,r=m;
	while (l<r) {
		int mid=(l+r+1)>>1;
		if (check(mid)) l=mid;
		else r=mid-1;
	}
	int ans=0;
	for (int i=1;i<=l;i++) ans+=val[m-i+1];
	cout<<ans;
	return 0;
}
```

[1]: http://codeforces.com/problemset/problem/367/C;