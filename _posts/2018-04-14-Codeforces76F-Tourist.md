---
layout: post
title: Codeforces76F Tourist
date: 2018-4-14 17:56
categories: training
tags: DP 二分
img: https://vexoben.github.io/assets/images/Blog/2018-4-14-Codeforces76F-Tourist.JPG
author: Vexoben
---

题目链接：[http://codeforces.com/problemset/problem/76/F][4]

## **题意**  
tourist站在x轴上。有n个事件，第i个时间于Ti的时间出现于Xi的位置。tourist每个单位时间最多移动V个单位，问：  
1、若tourist从0出发，可以看到的最多事件。  
2、若tourist从任意点出发，可以看到的最多事件。  
n≤1e5, Xi≤2e8, Ti≤2e6  

## **题解**

考虑事件i,j。能看完事件i再去看j的充要条件是：

![][1]  
暴力去绝对值得：  

![][2]  
不妨令：

![][3]

那么问题转化为求最长的二维最长不下降序列。先以P为关键字排序，然后求Q的最长不下降序列即得第二问答案。

对于第一问，我们需要强制以P=Q=0的状态为开头。那只需要加入(0,0)，并且删去所有P<0或Q<0的状态即可。  
```cpp
#include<bits/stdc++.h>
#define LL long long
using namespace std;
const int N=1e5+10;
const LL inf=0x3f3f3f3f3f3f3f3f;

int n,v;
LL t[N],x[N],f[N];
pair<LL,LL> a[N];

inline int Find(LL val) {
	int l=0,r=n;
	while (l<r) {
		int mid=(l+r+1)>>1;
		if (f[mid]<=val) l=mid;
		else r=mid-1;
	}
	return r;
}

int Solve(int t) {
	for (int i=1;i<N;i++) f[i]=inf;
	f[0]=-inf;
	for (int i=1;i<=n;i++) {
		if ((t==1)&&(a[i].first<0||a[i].second<0)) continue;
		int las=Find(a[i].second);
		f[las+1]=min(f[las+1],a[i].second);
	}
	for (int i=1;i<N;i++) if (f[i]==inf) return i-1;
}

int main() {
	cin>>n;
	for (int i=1;i<=n;i++) scanf("%lld%lld",&x[i],&t[i]);
	cin>>v;
	for (int i=1;i<=n;i++) {
		a[i].first=x[i]+t[i]*v;
		a[i].second=-x[i]+t[i]*v;
	}
	sort(a+1,a+n+1);
	cout<<Solve(1)<<' '<<Solve(2);
	return 0;
}
```


  [1]: https://vexoben.github.io/assets/images/Blog/2018-4-14-Codeforces76F-Tourist%282%29.JPG
  [2]: https://vexoben.github.io/assets/images/Blog/2018-4-14-Codeforces76F-Tourist%283%29.JPG
  [3]: https://vexoben.github.io/assets/images/Blog/2018-4-14-Codeforces76F-Tourist%284%29.JPG
  [4]: http://codeforces.com/problemset/problem/76/F