---
layout: post
title: Codeforces196C Paint Tree
date: 2018-07-07 13:45:26 +0800
categories: training
tags: 计算几何 构造
img: https://vexoben.github.io/oi-blog/assets/images/Blog/2018-07-07-codeforces196c-paint-tree.JPG
---

题目链接:[http://codeforces.com/contest/196/problem/C][1]

## **题意**

给定一棵 n 个节点的树和坐标平面上的 n 个点，要求将 n 个点用 n-1 条线段连接起来形成树的结构，树边不能相交，且形成的树要和原树同构，输出方案。

数据满足任意三点不共线。

n<=1500 坐标绝对值<=1e9

## **题解**

解一定存在且可由下述方法构造：

1、找到平面上最靠左的一个点，把它作为根；

2、将其余所有点极角排序;

3、假设根节点有一棵大小为 x 的子树，将前 x 个节点划分为一个子树，并将其中最靠左的点作为根，向下递归。

```cpp
#include<bits/stdc++.h>
#define LL long long
using namespace std;
const int N=1505;

int n,m,E;
int fir[N],arr[N<<1],nex[N<<1],siz[N],ans[N];
vector<int> son[N];
struct Vector {
	int x,y,num;
}p[N],st;
vector<Vector> vec;

LL cross(Vector a,Vector b) {
	return 1LL*a.x*b.y-1LL*a.y*b.x;
}

Vector operator - (Vector a,Vector b) {
	return (Vector) {a.x-b.x,a.y-b.y,0};
}

void Add_Edge(int x,int y) {
	nex[++E]=fir[x];
	fir[x]=E; arr[E]=y;
}

void dfs(int x) {
	siz[x]=1;
	for (int i=fir[x];i;i=nex[i]) {
		if (siz[arr[i]]) continue;
		dfs(arr[i]);
		siz[x]+=siz[arr[i]];
		son[x].push_back(arr[i]);
	}
}

bool cmp1(Vector a,Vector b) {
	if (a.x!=b.x) return a.x<b.x;
	else return a.y<b.y;
}

bool cmp2(Vector a,Vector b) {
	return (cross(a-st,b-st)>0);
}

void Solve(int x,vector<Vector> vec) {
	ans[vec[0].num]=x;
	st=vec[0];
	sort(vec.begin()+1,vec.end(),cmp2);
	vector<Vector> tmp;
	int now=1;
	for (int i=0;i<son[x].size();i++) {
		tmp.clear();
		while (tmp.size()<siz[son[x][i]]) {
			tmp.push_back(vec[now]); now++;
		}
		Solve(son[x][i],tmp);
	}
}

int main() {
	scanf("%d",&n);
	for (int i=1;i<n;i++) {
		int x,y; scanf("%d%d",&x,&y);
		Add_Edge(x,y); Add_Edge(y,x);
	}
	dfs(1);
	for (int i=1;i<=n;i++) scanf("%d%d",&p[i].x,&p[i].y),p[i].num=i;
	sort(p+1,p+n+1,cmp1);
	for (int i=1;i<=n;i++) vec.push_back(p[i]);
	Solve(1,vec);
	for (int i=1;i<=n;i++) printf("%d ",ans[i]);
	puts("");
	return 0;
}
```

[1]:http://codeforces.com/contest/196/problem/C