---
layout: post
title: Codeforces 177G2 Fibonacci Strings
date: 2018-07-11 15:50:22 +0800
categories: training
tags: 字符串 矩阵快速幂
img: https://vexoben.github.io/oi-blog/assets/images/Blog/2018-07-11-codeforces-177g2-fibonacci-strings.JPG
---

[codeforces177G2 Fibonacci Strings][5]

## **题意**

定义斐波那契字符串为:s[1]="a"，s[2]="b"，s[i]=s[i-1]+s[i-2](i>=3)

m组询问，每次给定一个字符串t，询问t在s[n]中的出现次数

m<=10000，n<=1e18，询问总串长<=100000

## **题解**

首先可以发现，s[i]是s[i+1]的前缀，s[i+2]的后缀。

那么当len(s[i])>len(t)的时候，两字符串拼接对答案产生的贡献只有两种情况。

不妨设s[x]是第一个长度大于t的斐波那契字符串。

记Kmp(s,t)为t在s中的出现次数。

令a=Kmp(s[x],t)，b=Kmp(s[x+1],t )，c=Kmp(s[x+2])-Kmp(s[x+1])-Kmp(s[x])，d=Kmp(s[x+3]-Kmp(s[x+2])-Kmp(s[x+1]))

那么c是s[x]与s[x+1]拼接产生的额外贡献，d是s[x+1]与s[x+2]拼接产生的额外贡献。

显然在后续的拼接过程中，交替产生c和d两种贡献。

这时就可以使用矩阵快速幂求解了。

![][6]

```cpp
#include<bits/stdc++.h>
#define LL long long
using namespace std;
const int N=1e5+10;
const int mod=1e9+7;

LL n,m;
int nex[N];
string tmp,s[30];
struct Matrix {
	int n,m;
	int a[5][5];
}u,v,f;

Matrix operator * (Matrix x,Matrix y) {
	Matrix z;
	z.n=x.n; z.m=y.m;
	for (int i=1;i<=x.n;i++) {
		for (int j=1;j<=y.m;j++) {
			int ans=0;
			for (int k=1;k<=x.m;k++) {
				ans+=1LL*x.a[i][k]*y.a[k][j]%mod;
				if (ans>=mod) ans-=mod;
			}
			z.a[i][j]=ans;
		}
	}
	return z;
}

Matrix operator ^ (Matrix x,LL p) {
	Matrix ans;
	ans.n=ans.m=x.n;
	memset(ans.a,0,sizeof(ans.a));
	for (int i=1;i<=x.n;i++) ans.a[i][i]=1;
	while (p) {
		if (p&1) ans=ans*x;
		x=x*x; p>>=1;
	}
	return ans;
}

void Kmp_init() {
	int k=-1,j=0;
	nex[0]=-1;
	int plen=tmp.size();
	while (j<plen) {
		while (k!=-1&&tmp[k]!=tmp[j]) k=nex[k];
		nex[++j]=++k;
	}
}

int Kmp(string &s) {
	int j=0,k=0,ans=0,slen=s.size(),plen=tmp.size();
	while (j<slen) {
		while (k!=-1&&s[j]!=tmp[k]) k=nex[k];
		j++; k++;
		if (k==plen) ans++,k=nex[k];
	}
	return ans;
}

void Solve() {
	cin>>tmp;
	int x; for (x=1;s[x].size()<tmp.size();x++);
	if (x>n) return (void) (puts("0"));
	Kmp_init();
	int a=Kmp(s[x]),b=Kmp(s[x+1]),c=Kmp(s[x+2]),d=Kmp(s[x+3]);
	d=d-b-c; c=c-a-b; 
	if (x==n) return (void) (printf("%d\n",a));
	if (x==n+1) return (void) (printf("%d\n",b));
	LL k=n-x-1;
	f.n=1; f.m=3; f.a[1][1]=a; f.a[1][2]=b; f.a[1][3]=1;
	u.n=u.m=3;
	u.a[1][1]=0; u.a[1][2]=1; u.a[1][3]=0;
	u.a[2][1]=1; u.a[2][2]=1; u.a[2][3]=0;
	u.a[3][1]=0; u.a[3][2]=c; u.a[3][3]=1;
	v.n=v.m=3;
	v.a[1][1]=0; v.a[1][2]=1; v.a[1][3]=0;
	v.a[2][1]=1; v.a[2][2]=1; v.a[2][3]=0;
	v.a[3][1]=0; v.a[3][2]=d; v.a[3][3]=1;
	Matrix t=u*v;
	t=t^(k/2); if (k&1) t=t*u;
	f=f*t;
	printf("%d\n",f.a[1][2]);
}

int main() {
	scanf("%lld%lld",&n,&m);
	s[1]="a"; s[2]="b";
	for (int i=3;i<=29;i++) s[i]=s[i-1]+s[i-2];
	while (m--) Solve();
	return 0;
}
```

[5]:http://codeforces.com/contest/177/problem/G2
[6]:https://vexoben.github.io/oi-blog/assets/images/Blog/2018-07-11-codeforces-177g2-fibonacci-strings(2).JPG