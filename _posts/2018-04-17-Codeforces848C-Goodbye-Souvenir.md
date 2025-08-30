---
layout: post
title: Codeforces848C Goodbye Souvenir
date: 2018-04-17 18:37:35 +0800
categories: training
tags: 数据结构 cdq分治 
img: https://vexoben.github.io/assets/images/Blog/2018-04-17-Codeforces848C-Goodbye-Souvenir.JPG
---

题目链接：[http://codeforces.com/problemset/problem/848/C][1]

## **题意**

给定一个长为n的序列，m个操作，每个操作形如：

1、修改一个数；

2、询问一段区间的价值。对于一种数，如果它在区间中出现，它对区间价值产生最后一次出现位置-第一次出现位置的贡献。

n,m<=1e5  权值<=n

## **题解**

一个关键的思想是差分。

我们记录第x个位置上的数，上一次出现的位置为pre(x),如果第一次出现,pre(x)=0

对于每个数，定义它的权值val=x-pre(x)

对于一次询问[l,r]，我们只需要确定l<=pre(x)<x<=r的x的权值和，也就是满足pre(x)>=l,x<=r的权值和

这是一个带修改的二维偏序，我们可以用cdq分治套树状数组解决

首先我们把为题目简化为：对于二维平面上的点(x,y)，它的权值是x-y，动态加入删除点，每次询问一个矩阵的点权和。

对于原来的n个数，我们把它当做n个修改操作。对每个点权修改操作x->y，我们把它拆成六个修改操作：

1、删除点(x,pre(x))；2、删除点(succ(x),x)；3、加入点(succ(x),pre(x))；

4、加入点(y,pre(y))；3、加入点(succ(y),y)；4、删除点(succ(y),pre(y))；

外层对时间分治，内层对x的位置排序，将pre(x)用树状数组维护即可。

```cpp
#include<bits/stdc++.h>
#define int long long
using namespace std;
const int N=6e5+10;

int n,m,tot,las[N],a[N],tree[N];
set<int> Set[N];
pair<int,int> del[N];
struct Q {
	int tim;
	int x,y,val,type;
}q[N],tmp[N];

void Updata(int x,int y) {
	if (a[x]==y) return;
	set<int>::iterator it;
	it=Set[a[x]].find(x);
	if (it!=Set[a[x]].begin()) {
		--it; ++tot; 
		q[tot]=(Q) {tot,x,*it,-(x-*it),1}; //Delete point (x,pre(x))
		++it;
	}
	++it;
	if (it!=Set[a[x]].end()) {
		++tot; q[tot]=(Q) {tot,*it,x,-(*it-x),1}; // Delete point (succ(x),x);
		int p=*it; --it;
		if (it!=Set[a[x]].begin()) {
			--it; ++tot;
			q[tot]=(Q) {tot,p,*it,p-*it,1}; // Insert point (succ(x),pre(x))
		}
	}
	Set[a[x]].erase(x); a[x]=y;
	it=Set[y].insert(x).first;
	if (it!=Set[y].begin()) {
		--it; ++tot;
		q[tot]=(Q) {tot,x,*it,x-*it,1}; // Insert point (x,pre(x))
		++it;
	}
	++it;
	if (it!=Set[y].end()) {
		++tot; q[tot]=(Q) {tot,*it,x,*it-x,1}; // Insert point (succ(x),x)
		int p=*it; --it;
		if (it!=Set[y].begin()) {
			--it; ++tot;
			q[tot]=(Q) {tot,p,*it,-(p-*it),1}; // Delete point (succ(x),pre(x))
		}
	}
}

void Init() {
	scanf("%lld%lld",&n,&m);
	for (int i=1;i<=n;i++) {
		scanf("%lld",&a[i]);
		Set[a[i]].insert(i);
		++tot; q[tot]=(Q) {tot,i,las[a[i]],i-las[a[i]],1};
		las[a[i]]=i;
	}
	for (int i=1;i<=m;i++) {
		int t,x,y; scanf("%lld%lld%lld",&t,&x,&y);
		if (t==1) Updata(x,y);
		else if (t==2) {
			++tot; q[tot]=(Q) {tot,y,x,0,2};
		}
	}
}

void Add(int x,int val) {
	while (x<=n+1) tree[x]+=val,x+=x&-x; 
}

int Query(int x) {
	int ans=0;
	while (x) ans+=tree[x],x-=x&-x;
	return ans;
}

void Solve(int l,int r) {
	if (l==r) return;
	int mid=(l+r)>>1;
	int u=l,v=mid+1;
	for (int i=l;i<=r;i++) {
		if (q[i].tim<=mid) tmp[u++]=q[i];
		else tmp[v++]=q[i];
	}
	for (int i=l;i<=r;i++) q[i]=tmp[i];
	Solve(l,mid);
	int cnt=0;
	for (int now=l,i=mid+1;i<=r;i++) {
		if (q[i].type==1) continue;
		while (now<=mid) {
			if (q[now].type==2) ++now;
			else if (q[now].x<=q[i].x) {
				del[++cnt]=make_pair(q[now].y+1,q[now].val);
				Add(q[now].y+1,q[now].val);
				now++;
			}
			else break;
		}
		q[i].val+=Query(n+1)-Query(q[i].y);
	}
	for (int i=1;i<=cnt;i++) Add(del[i].first,-del[i].second);
	Solve(mid+1,r);
	u=l,v=mid+1;
	for (int i=l;i<=r;i++) {
		if (u>mid) tmp[i]=q[v++];
		else if (v>r) tmp[i]=q[u++];
		else if (q[u].x<q[v].x) tmp[i]=q[u++];
		else tmp[i]=q[v++];
	}
	for (int i=l;i<=r;i++) q[i]=tmp[i];
}

bool cmp(Q u,Q v) {
	return u.x<v.x;
}

bool cmp2(Q u,Q v) {
	return u.tim<v.tim;
}

signed main() {
	Init();
	sort(q+1,q+tot+1,cmp);
	Solve(1,tot);
	sort(q+1,q+tot+1,cmp2);
	for (int i=1;i<=tot;i++)
		if (q[i].type==2) printf("%lld\n",q[i].val);
	return 0;
}
```

[1]: http://codeforces.com/problemset/problem/848/C