---
layout: post
title: POJ1475 Pushing Boxes
date: 2018-02-04 18:49:08 +0800
categories: training
tags: 搜索 图论
img: assets/images/Blog/POJ-1475-Pushing-Boxes.JPG
---

题目链接：[http://poj.org/problem?id=1475][1]

## **题意**
给你一张地图，把箱子推到目的地，要求在推箱子次数最少的前提下最小化走的路程。输出方案。

## **题解**
这题网上有很多A*和嵌套bfs的做法。事实上Dijkstra解决这道题也十分优越。

用状态dis[a][b][c][d]表示人处于(a,b)这个点，箱子处于(c,d)这个点的最小费用。费用的定义是推箱子的费用+人行走的费用。我们将一次推箱子的费用设置成10000，而人走一次的费用设置成1，就满足了在推箱子次数最少的前提下最小化走的路程的条件。输出方案的话记录一下前驱节点即可。复杂度O(n^2*m^2*log(n^2*m^2))

```cpp
#include<set>
#include<map>
#include<queue>
#include<stack>
#include<cmath>
#include<cstdio>
#include<time.h>
#include<vector>
#include<cstring>
#include<stdlib.h>
#include<iostream>
#include<algorithm>
#define LL long long
using namespace std;
const int N=22;
const int Cost=1e4+10;
const int inf=0x3f3f3f3f;
const int u[4]={1,0,-1,0};
const int v[4]={0,1,0,-1};
const char move[4]={'S','E','N','W'};

char s[N][N];
int n,m,a[N][N],las[N][N][N][N],dis[N][N][N][N],push[N][N][N][N];
int xb,yb,xe,ye,boxx,boxy;
struct Pnt {
	int a,b,c,d,dis;
};
struct cmp {
	bool operator () (const Pnt &x,const Pnt &y) {
		return x.dis>y.dis;
	}
};
priority_queue <Pnt,vector<Pnt>,cmp> Q;

namespace FastIO {
	template<typename tp> inline void read(tp &x) {
		x=0; register char c=getchar(); register bool f=0;
		for(;c<'0'||c>'9';f|=(c=='-'),c = getchar());
		for(;c>='0'&&c<='9';x=(x<<3)+(x<<1)+c-'0',c = getchar());
		if(f) x=-x;
	}
	template<typename tp> inline void write(tp x) {
		if (x==0) return (void) (putchar('0'));
		if (x<0) putchar('-'),x=-x;
		int pr[20]; register int cnt=0;
		for (;x;x/=10) pr[++cnt]=x%10;
		while (cnt) putchar(pr[cnt--]+'0');
	}
	template<typename tp> inline void writeln(tp x) {
		write(x);
		putchar('\n');
	}
}
using namespace FastIO;

inline int Check(int x,int y) {
	if (a[x][y]!=1) return 0;
	if (x==0||x>n) return 0;
	if (y==0||y>m) return 0;
	return 1;
}

void Dijkstra() {
	dis[xb][yb][boxx][boxy]=0;
	Q.push((Pnt){xb,yb,boxx,boxy,0});
	while (!Q.empty()) {
		Pnt t=Q.top(); Q.pop();
		for (int i=0;i<4;i++) {
			if (Check(t.a+u[i],t.b+v[i])) {
				if (t.a+u[i]==t.c&&t.b+v[i]==t.d) {
					if (Check(t.c+u[i],t.d+v[i])) {
						int *r=&dis[t.c][t.d][t.c+u[i]][t.d+v[i]];
						if (*r>t.dis+Cost) {
							*r=t.dis+Cost;
							Q.push((Pnt){t.c,t.d,t.c+u[i],t.d+v[i],*r});
							las[t.c][t.d][t.c+u[i]][t.d+v[i]]=i;
							push[t.c][t.d][t.c+u[i]][t.d+v[i]]=1;
						}
					}
				}
				else {
					int *r=&dis[t.a+u[i]][t.b+v[i]][t.c][t.d];
					if (*r>t.dis+1) {
						*r=t.dis+1;
						Q.push((Pnt){t.a+u[i],t.b+v[i],t.c,t.d,*r});
						las[t.a+u[i]][t.b+v[i]][t.c][t.d]=i;
						push[t.a+u[i]][t.b+v[i]][t.c][t.d]=0;
					}
				}
			}
		}
	}
}

void output(int a,int b,int c,int d) {
	int t=las[a][b][c][d];
	if (~t) {
		if (push[a][b][c][d]) output(a-u[t],b-v[t],c-u[t],d-v[t]),putchar(move[t]);
		else output(a-u[t],b-v[t],c,d),putchar(move[t]-'A'+'a');
	}
}

void Solve() {
	Dijkstra();
	int ans=inf,x=0,y=0;
	for (int i=1;i<=n;i++)
		for (int j=1;j<=m;j++)
			if (dis[i][j][xe][ye]<ans){
				ans=dis[i][j][xe][ye];
				x=i; y=j;
			}
	if (ans==inf) {
		printf("Impossible.");
		return;
	}
	else output(x,y,xe,ye);
}
int main() {
	read(n); read(m);
	for (int cas=1;n+m!=0;cas++) {
		memset(a,0,sizeof(a));
		memset(las,0xff,sizeof(las));
		memset(push,0,sizeof(push));
		memset(dis,0x3f,sizeof(dis));
		printf("Maze #%d\n",cas);
		for (int i=1;i<=n;i++) scanf("%s",s[i]+1);
		for (int i=1;i<=n;i++)
			for (int j=1;j<=m;j++) {
				if (s[i][j]!='#') a[i][j]=1;
				if (s[i][j]=='S') xb=i,yb=j;
				else if (s[i][j]=='T') xe=i,ye=j;
				else if (s[i][j]=='B') boxx=i,boxy=j;
			}
		Solve();
		putchar('\n'); putchar('\n'); 
		read(n); read(m);
	}
	return 0;
}
```

[1]:http://poj.org/problem?id=1475