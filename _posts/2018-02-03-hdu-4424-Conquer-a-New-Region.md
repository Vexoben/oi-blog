---
layout: post
title: HDU 4424 Conquer a New Region
date: 2018-02-03 18:38:08 +0800
categories: training
tags: 数据结构
img: https://vexoben.github.io/oi-blog/assets/images/Blog/HDU-4424-Conquer-a-New-Region.JPG
---

题目链接：[http://acm.hdu.edu.cn/showproblem.php?pid=4424][1]

## **题意**  
给定一棵带边权的树，任选一个起点，定义其余所有的点权为起点到该点路径中最短边的长度，最大化点权和。 
## **题解**
点权由最短边决定，所以考虑将边从大到小排序。每加入一条边就是合并两个连通块。设这两个连通块分别为A、B，则合并后的连通块C价值为：max(val[A]+length*size[B],val[B]+length*size[A])。用并查集维护一下即可。

```cpp
#include<bits/stdc++.h>
#define int long long
using namespace std;
const int N=2e5+10;

int n,m,siz[N],val[N],fa[N];
struct Edge {
	int u,v,l;
}a[N];

bool cmp(Edge x,Edge y) {
	return x.l>y.l;
}

inline int Find(int x) {
	return fa[x]==x?x:fa[x]=Find(fa[x]);
}

void Solve() {
	int ans=0;
	sort(a+1,a+n,cmp);
	for (int i=1;i<n;i++) {
		int u=Find(a[i].u);
		int v=Find(a[i].v);
		int l=a[i].l;
		int v1=val[u]+l*siz[v];
		int v2=val[v]+l*siz[u];
		fa[u]=fa[v];
		siz[v]=siz[u]+siz[v];
		val[v]=max(v1,v2);
		ans=max(ans,val[v]);
	}
	printf("%lld\n",ans);
}

signed main() {
	while (~scanf("%lld",&n)) {
		memset(val,0,sizeof(int)*(n+1));
		for (register int i=1;i<=n;i++) siz[i]=1;
		for (register int i=1;i<=n;i++) fa[i]=i;
		for (register int i=1;i<n;i++) scanf("%lld%lld%lld",&a[i].u,&a[i].v,&a[i].l);
		Solve();
	}
	return 0;
}
```

[1]: http://acm.hdu.edu.cn/showproblem.php?pid=4424