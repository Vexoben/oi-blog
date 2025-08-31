---
layout: post
title: Codeforces908E New Year and Entity Enumeration
date: 2018-05-09 21:26:49 +0800
categories: training
tags: 组合数学
img: assets/images/Blog/2018-05-09-codeforces908e-new-year-and-entity-enumeration.JPG
---

题目链接：[http://codeforces.com/contest/908/problem/E][1]

## **题意**

给定正整数n,m和大小为n的集合T，试求好的集合S的个数。一个集合被称为好的，当且仅当：

1、S中仅包含m位的二进制数；

2、S对取反封闭(xor 2^m-1 后仍在S中)；

3、S对AND封闭；

4、T是S的子集。

## **题解**

显然有0和2^m-1属于S。

先不考虑第四个条件。设f(x)为S中，x位上是1的最小数，那么有一个结论是，若f(x)!=f(y)，则f(x)&f(y)=0。

可以这样证明：不妨令b=f(x)&f(y)!=f(x) (f(x)!=f(y)，b至少和其中一个不等)，那么如果x位上b是1，有f(x)&b<f(x)；如果x位上b不是1，令M=2^m-1，有f(x)&(b xor M)<f(x)

这表明，f的取值方案是m位二进制分组的方案数，即B(m)，B表示bell数

同时，由于有了取反和AND，我们也可以实现OR；从而给定f的取值方案，就可以唯一还原集合S

从而不考虑T的情况下，答案就是B(m)

然后考虑T是S的子集。设x[i]是将n个字符串的第i位依次写下得到的字符串，那么数位i和数位j能被分为一组当且仅当x[i]=x[j]。从而可以对所有能被分为一组的数位用bell数计数，然后将所有的答案相乘即可。

```cpp
#include<bits/stdc++.h>
#define LL long long
using namespace std;
const int N=1050;
const int mod=1e9+7;

int n,m;
LL c[N][N],bell[N];
char s[N];
string x[N];
map<string,int> Map;

int main() {
	for (int i=0;i<N;i++) {
		c[i][0]=1;
		for (int j=1;j<=i;j++) c[i][j]=(c[i-1][j]+c[i-1][j-1])%mod;
	}
	bell[0]=bell[1]=1;
	for (int i=2;i<N;i++) {
		for (int j=1;j<=i;j++) {
			bell[i]+=bell[i-j]*c[i-1][j-1];
			bell[i]%=mod;
		}
	}
	cin>>m>>n;
	for (int i=1;i<=n;i++) {
		scanf("%s",s+1);
		for (int j=1;j<=m;j++) x[j]+=s[j];
	}
	for (int i=1;i<=m;i++) Map[x[i]]++;
	LL ans=1;
	map<string,int>::iterator it;
	for (it=Map.begin();it!=Map.end();it++) {
		ans=ans*bell[it->second]%mod;
	}
	cout<<ans;
	return 0;
}
```

[1]: http://codeforces.com/contest/908/problem/E