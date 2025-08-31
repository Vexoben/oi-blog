---
layout: post
title: Dsu on Tree 
date: 2018-07-10 19:40:21 +0800
categories: notes
tags: dsu 
img: https://vexoben.github.io/oi-blog/assets/images/Blog/2018-07-10-dsu-on-tree.JPG
---

dsu on tree,即树上启发式合并，是一种离线算法，结合树链剖分，可以处理与子树有关的许多询问。

## **算法过程**

1、将给定的树轻重链剖分；

2、将整棵树dfs。到点x时，先dfs所有x的轻孩子，并在递归上来时清除他们的影响；然后dfs重孩子，并在递归上来时保留他的影响；

3、现在处理完了所有x的子树并回到了x，保留的信息是重孩子对x的影响。然后我们再暴力将所有轻孩子的树dfs一遍，将他们的影响加上；

4、计算出询问在x的答案。

## **复杂度分析**

显然只需要考虑第三步的复杂度就可以了。

一个节点需要被暴力dfs，仅当它对应的祖先是x的轻孩子。又由树剖复杂度证明知道一个点到根节点的路径上轻孩子不超过O(logn)，所以设修改复杂度是w，总复杂度就是O(n*logn*w)

## **模板题**

[codeforces246E Blood Cousins Return][1]

### **题意**

给出一个n个点的森林，每个点都有一个字符串。每次询问以x为根的子树中距离x为k的节点包含多少不同字符串。

n,m<=100000

### **题解**

对深度开set，直接跑dsu on tree，每次相当于询问dep为dep[x]+k的set大小。

```cpp
#include<bits/stdc++.h>
#define fi first
#define se second
#define mp make_pair
#define pii pair<int,int>
using namespace std;
const int N=1e5+10;

string s;
int n,m,E,tot;
int fa[N],fir[N],nex[N<<1],arr[N<<1];
int dep[N],siz[N],son[N],name[N],ans[N];
set<int> Set[N];
map<int,int> Map[N];
vector<pii> vec[N];
map<string,int> hs;

inline void Add_Edge(int x,int y) {
	nex[++E]=fir[x];
	fir[x]=E; arr[E]=y;
}

void dfs(int x) {
	siz[x]=1;
	for (int i=fir[x];i;i=nex[i]) {
		dep[arr[i]]=dep[x]+1;
		dfs(arr[i]);
		siz[x]+=siz[arr[i]];
		if (siz[arr[i]]>siz[son[x]]) son[x]=arr[i];
	}
}

inline void Add(int x,int d) {
	if (++Map[d][x]==1) Set[d].insert(x);
}

inline void Del(int x,int d) {
	if (--Map[d][x]==0) Set[d].erase(x);
}

void Updata(int x,int type) {
	if (type==1) Add(name[x],dep[x]); else Del(name[x],dep[x]);
	for (int i=fir[x];i;i=nex[i]) Updata(arr[i],type);
}

void Solve(int x,int keep) {
	for (int i=fir[x];i;i=nex[i]) {
		if (arr[i]==son[x]) continue;
		Solve(arr[i],0);
	}
	if (son[x]) Solve(son[x],1);
	for (int i=fir[x];i;i=nex[i]) {
		if (arr[i]==son[x]) continue;
		Updata(arr[i],1);
	}
	Add(name[x],dep[x]);
	for (int i=0;i<vec[x].size();i++) {
		ans[vec[x][i].fi]=Set[vec[x][i].se].size();
	}
	if (!keep) {
		Del(name[x],dep[x]);
		for (int i=fir[x];i;i=nex[i]) Updata(arr[i],-1);
	}
}

int main() {
	scanf("%d",&n);
	for (int i=1;i<=n;i++) {
		cin>>s>>fa[i];
		if (!hs[s]) hs[s]=++tot;
		name[i]=hs[s];
		Add_Edge(fa[i],i);
	}
	scanf("%d",&m);
	for (int i=1;i<=n;i++)
		if (!fa[i]) dep[i]=1,dfs(i);
	for (int i=1;i<=m;i++) {
		int x,d; scanf("%d%d",&x,&d);
		vec[x].push_back(mp(i,dep[x]+d));
	}
	for (int i=1;i<=n;i++)
		if (!fa[i]) Solve(i,0);
	for (int i=1;i<=m;i++) printf("%d\n",ans[i]);
	return 0;
}
```

[1]:http://codeforces.com/contest/246/problem/E