---
layout: post
title: Codeforces976D Degree Set
date: 2018-05-03 20:29:28 +0800
categories: training
tags: 构造
img: https://vexoben.github.io/assets/images/Blog/2018-05-03-codeforces-976d Degree Set.JPG
---

题目链接：[http://codeforces.com/problemset/problem/976/D][1]

## **题意**

升序给出一个大小为n的集合d，构造一张无向图，它所有点度数组成的集合恰好等于集合d

n<=1000,di<=1000

## **题解**

考虑如果得到了一张图G，他的度数集合是{d[2]-d[1]，d[3]-d[1]，……，d[n-1]-d[1]}，点数为(d[n-1]-d[1]+1)我们可以用如下的方式构造出集合d

1、构造一个点数为d[1]的完全图K，那么K中所有点的度数均为d[1]；

2、K中所有点向G中是所有点连一条边，G的度数集合变成{d[2]，d[3]，……，d[n-1]}，K中点的度数变成d[n-1]；

3、将剩下尚未使用的(d[n]-d[n-1])个点向K中所有点连边，K中点度数变为d[n],其余点度数变为d[1]。

那么当n>=2时用上面的方法递归，边界条件是：

a.若n=0，返回一个点；

b.若n=1，返回(d[1]+1)个点的完全图

```cpp
#include<bits/stdc++.h>
using namespace std;

int n,tot;
vector<pair<int,int> > ans;
vector<int> d;

void Solve(vector<int> d) {
	if (d.size()==0) return (void) (++tot);
	if (d.size()==1) {
		int fir=tot+1;	tot+=d[0]+1;
		for (int i=fir;i<=tot;i++)
			for (int j=i+1;j<=tot;j++)
				ans.push_back(make_pair(i,j));
		return;
	}
	vector<int> dx; dx.clear();
	for (int i=1;i<d.size()-1;i++) dx.push_back(d[i]-d[0]);
	Solve(dx);
	int now=tot; tot+=d[0];
	for (int i=now+1;i<=tot;i++)
		for (int j=i+1;j<=tot;j++)
			ans.push_back(make_pair(i,j));
	for (int i=1;i<=now;i++)
		for (int j=now+1;j<=tot;j++)
			ans.push_back(make_pair(i,j));
	int K=tot; tot=d[d.size()-1]+1;
	for (int i=now+1;i<=K;i++)
		for (int j=K+1;j<=tot;j++)
			ans.push_back(make_pair(i,j));
}

int main() {
	scanf("%d",&n);
	for (int i=1;i<=n;i++) {
		int x; scanf("%d",&x);
		d.push_back(x);
	}
	Solve(d);
	printf("%d\n",ans.size());
	for (int i=0;i<ans.size();i++) 
		printf("%d %d\n",ans[i].first,ans[i].second);
	return 0;
}
```

[1]: http://codeforces.com/problemset/problem/976/D