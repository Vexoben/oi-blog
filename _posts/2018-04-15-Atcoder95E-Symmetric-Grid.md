---
layout: post
title: Atcoder95E Symmetric Grid
date: 2018-04-15 19:13:03 +0800
categories: training
tags: 搜索 暴力
img: https://vexoben.github.io/assets/images/Blog/2018-04-15-Atcoder95E-Symmetric-Grid.JPG
---

题目链接：[https://arc095.contest.atcoder.jp/tasks/arc095_c][1]

不是很难的一道搜索题，感觉搜索不怎么会……

## **题意**

给你一个n*m的矩阵，每个格点中有一个小写字母。每次可以交换两行或两列，问是否可以使这个矩阵中心对称。

n,m<=12

## **题解**

思路是暴枚行的排布方式，然后判列是否能合法交换。

对于已经确定好的行排布，如果两列正好相反(s[i][x]==s[n-i+1][y])，那么这两列可以匹配，我们直接匹配他们。

如果有一列不能和其他列匹配，如果列数是奇数，我们看它能否和自己匹配（就是放在中间的那一列）。

注意两个细节：

1、暴枚行的排布时，我们枚举它和哪一行匹配。这样我们就只需要枚举11*9*7*5*3*1次。

2、单独的列要求能够和自己匹配（事实上不判这个也能过）。但是暴枚行排布时单独的行不需要能和自己匹配。

注释中的数据是atc上AC了之后把自己叉掉的数据……

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=15;

char a[N][N],b[N][N];
int n,m,match[N],use[N];

inline int Equal(int x,int y) {
	for (int i=1;i<=n;i++) {
		if (b[i][x]!=b[n-i+1][y]) return 0;
	}
	return 1;
}

inline void check() {
	int res=m&1;
	memset(use,0,sizeof(use));
	int now=1;
	for (int i=1;i<=n;i++) {
		if (i==match[i]) {
			for (int j=1;j<=m;j++)
				b[(n+1)>>1][j]=a[i][j];
		}
		if (i<match[i]) {
			for (int j=1;j<=m;j++) {
				b[now][j]=a[i][j];
				b[n-now+1][j]=a[match[i]][j];
			}
			++now;		
		}
	}		
	for (int i=1;i<=m;i++) {
		if (use[i]) continue;
		for (int j=i+1;j<=m;j++)
			if (!use[j]&&Equal(i,j)) {
				use[i]=j; use[j]=i;
				break;
			}
		if (!use[i]) {
			int flag=1;
			for (int j=1;j<=n;j++) 
				if (b[j][i]!=b[n-j+1][i]) flag=0;
			if (!flag) return;
			if (--res<0) return;
		}
	}
	puts("YES"); exit(0);
}

void dfs(int x,int res) {
	if (x==n+1) return (void) (check());
	if (match[x]) return (void) (dfs(x+1,res));
	if (res) {
		match[x]=x;
		dfs(x+1,0);
		match[x]=0;
	}
	for (int i=x+1;i<=n;i++) {
		if (!match[i]) {
			match[i]=x; match[x]=i;
			dfs(x+1,res);
			match[i]=0; match[x]=0;
		}
	}
}

int main() {
	scanf("%d%d",&n,&m);
	for (int i=1;i<=n;i++) scanf("%s",a[i]+1);
	dfs(1,n&1);
	puts("NO");
	return 0;
}
/*
3 3
aac
bab
cba
*/
```

[1]:https://arc095.contest.atcoder.jp/tasks/arc095_c