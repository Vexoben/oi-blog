---
layout: post
title: BZOJ4299 Codechef FRBSUM
date: 2018-06-19 10:26:41 +0800
categories: training
tags: 结论题 数据结构 线段树及可持久化
img: assets/images/Blog/2018-06-19-bzoj4299-codechef-frbsum.JPG
---

题目链接:[https://www.lydsy.com/JudgeOnline/problem.php?id=4299][1]

## **题意**

数集S的ForbiddenSum定义为无法用S的某个子集（可以为空）的和表示的最小的非负整数。
给定一个序列A，你的任务是回答该数列的一些子区间所形成的数集的ForbiddenSum是多少。

n,m<100000

## **题解**

先考虑S的FRBsum怎么求。

将所有数排序，从小到大加入。假设当前可以表示出的数是0~mx(mx初值为0)，现在加入的数是x，如果x<=mx+1，那么加入后就可以表示0~mx+x；如果x>mx+1，那么x及以后的数都对答案没有影响，答案就是mx+1。

考虑怎么处理区间查询。建出原序列的权值主席树，每次将mx更新为[l,r]内不大于mx+1的数的和。如果mx不变，那么mx+1就是答案。显然更新两次之后mx至少翻一番，所以更新次数为O(logn)次。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=1e5+10;
const int inf=1e9+10;

int n,m,tot;
int a[N],rt[N];
int ch[N*50][2],sum[N*50];

inline void Pushup(int x) {
	sum[x]=sum[ch[x][0]]+sum[ch[x][1]];
}

int ins(int las,int l,int r,int val) {
	int now=++tot;
	if (l==r) {
		sum[now]=sum[las]+val;
		return now;
	}
	int mid=(l+r)>>1;
	if (mid>=val) {
		ch[now][1]=ch[las][1];
		ch[now][0]=ins(ch[las][0],l,mid,val);
	}
	else if (mid<val) {
		ch[now][0]=ch[las][0];
		ch[now][1]=ins(ch[las][1],mid+1,r,val);
	}
	Pushup(now);
	return now;
}

int Query(int x,int l,int r,int val) {
	if (l==r) return sum[x]*(l<=val);
	int mid=(l+r)>>1,ans=0;
	if (mid<val) {
		ans+=sum[ch[x][0]];
		ans+=Query(ch[x][1],mid+1,r,val);
		return ans;
	}
	else if (mid>=val) {
		return Query(ch[x][0],l,mid,val);
	}
}

int main() {
	scanf("%d",&n);
	for (int i=1;i<=n;i++) scanf("%d",&a[i]);
	for (int i=1;i<=n;i++) {
		rt[i]=ins(rt[i-1],1,inf,a[i]);
	}
	scanf("%d",&m);
	while (m--) {
		int l,r; scanf("%d%d",&l,&r);
		int ans=0;
		do {
			int ax=Query(rt[r],1,inf,ans+1)-Query(rt[l-1],1,inf,ans+1);
			if (ax==ans) break;
			ans=ax;
		} while (1);
		printf("%d\n",ans+1);
	}
	return 0;
}
```

[1]:https://www.lydsy.com/JudgeOnline/problem.php?id=4299