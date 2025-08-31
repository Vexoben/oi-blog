---
layout: post
title: POJ3131 Cubic Eight-Puzzle
date: 2018-02-09 16:19:49 +0800
categories: training
tags: 搜索
img: https://vexoben.github.io/oi-blog/assets/images/Blog/2018-02-09-poj3131-cubic-eight-puzzle.JPG
---

题目链接：[http://poj.org/problem?id=3131][1]

## **题意**

给你八个正方体，放于九宫格内，每个正方体相对面同色。每次可以将一个正方体翻滚到相邻空位，求使从高处向下看为给定的目标状态的最小移动次数。如果30步内不能到达判定无解。

## **题解**

三维八数码问题，明显是一道搜索题。

先考虑状态数：空位有9种位置，每个正方体有6种放置方法，共15116544种状态。如果只有一组数据的话广搜就可以直接搜过去。

那么可以考虑双向广搜。显然末状态有256种可能。我们可以考虑从初状态搜20层，从末状态搜索10层，取交汇点即可。

但是还有一种更高妙的做法：深搜！

这个是在别人的博客上看到的。当时很惊讶深搜+估价函数（不在目标位置的格子数-1）居然可以过，就把代码拖下来跑了一发，对于15组全部无解的数据，开O2后在学校电脑上只跑了6s。

考虑不加估价函数时的状态数：由于不会往回走，即便在中间也最多有3种方案可以选择。但状态总数并不是3^30，而是从中心走到旁边，再走到角落，走到旁边，再走回中心，这样一次循环（走4步）的方案数只有3*2*1*2=12种，也就是即便跑满状态数也不会超过286654464种，而事实上并不会每次走回旁边后直接走到中心（可能又走到角落），再加上估价函数的剪枝，状态数又远小于这个数字 ！

最后在POJ上跑出来是1157MS，虽不及双向广搜，但也是十分理想的算法了。

```cpp
#pragma GCC optimize(2)
#include<cstdio>
#include<utility>
using namespace std;

char s[3];
int x,y,H,ans,goal[5][5];
pair<int,int> a[5][5];     //a[i][j].first 为向上的一面，second为向前的一面

void Init() {
	H=0; ans=31;
	for (int i=1;i<=3;i++)
		for (int j=1;j<=3;j++) {
			scanf("%s",s+1);
			if (s[1]=='E') goal[i][j]=0;
			else if (s[1]=='W') goal[i][j]=1;
			else if (s[1]=='R') goal[i][j]=2;
			else goal[i][j]=3;
			if (i==x&&j==y) a[i][j]=make_pair(0,0);
			else a[i][j]=make_pair(1,2);
		}
	for (int i=1;i<=3;i++)
		for (int j=1;j<=3;j++)
			if (a[i][j].first!=goal[i][j])
				H++;     // 估价函数
}

void dfs(int tim,int las) {
	if (tim+H-1>=ans) return;
	if (!H) return (void) (ans=tim);
	if (y>1&&las!=1) {   // 从左边翻
		pair<int,int> u=a[x][y],v=a[x][y-1];
		a[x][y]=make_pair(6-a[x][y-1].first-a[x][y-1].second,a[x][y-1].second);
		a[x][y-1]=make_pair(0,0);
		int del=(goal[x][y]==a[x][y].first)+(goal[x][y-1]==a[x][y-1].first)-(goal[x][y]==u.first)-(goal[x][y-1]==v.first);
		H-=del; y--; dfs(tim+1,0); H+=del; y++;
		a[x][y]=u; a[x][y-1]=v;
	}
	if (y<3&&las!=0) {   // 从右边翻
		pair<int,int> u=a[x][y],v=a[x][y+1];
		a[x][y]=make_pair(6-a[x][y+1].first-a[x][y+1].second,a[x][y+1].second);
		a[x][y+1]=make_pair(0,0);
		int del=(goal[x][y]==a[x][y].first)+(goal[x][y+1]==a[x][y+1].first)-(goal[x][y]==u.first)-(goal[x][y+1]==v.first);
		H-=del; y++; dfs(tim+1,1); H+=del; y--;
		a[x][y]=u; a[x][y+1]=v;
	}
	if (x>1&&las!=3) {   // 从上面翻
		pair<int,int> u=a[x][y],v=a[x-1][y];
		a[x][y]=make_pair(a[x-1][y].second,a[x-1][y].first);
		a[x-1][y]=make_pair(0,0);
		int del=(goal[x][y]==a[x][y].first)+(goal[x-1][y]==a[x-1][y].first)-(goal[x][y]==u.first)-(goal[x-1][y]==v.first);
		H-=del; x--; dfs(tim+1,2); H+=del; x++;
		a[x][y]=u; a[x-1][y]=v;
	}
	if (x<3&&las!=2) {   // 从下面翻
		pair<int,int> u=a[x][y],v=a[x+1][y];
		a[x][y]=make_pair(a[x+1][y].second,a[x+1][y].first);
		a[x+1][y]=make_pair(0,0);
		int del=(goal[x][y]==a[x][y].first)+(goal[x+1][y]==a[x+1][y].first)-(goal[x][y]==u.first)-(goal[x+1][y]==v.first);
		H-=del; x++; dfs(tim+1,3); H+=del; x--;
		a[x][y]=u; a[x+1][y]=v;
	}
}

int main() {
	while (scanf("%d%d",&y,&x)&&(x+y)) {
		Init();
		dfs(0,-1);
		if (ans<=30) printf("%d\n",ans);
		else puts("-1");
	}
}
```

[1]:http://poj.org/problem?id=3131
