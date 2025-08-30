---
layout: post
title: 2015 Multi-University Contest 4
date: 2018-04-19 18:27:22 +0800
categories: contest
tags: 脑洞题 模拟 构造 基环树 DP 数据结构 树链剖分 LCT
img: https://vexoben.github.io/assets/images/Blog/2015-Multi-University-Contest-4.JPG
---

今天World Final。蒟蒻连去看直播的资格都没有啊只好来VP多校了。

2015年的XJ题，大佬辈出的年代啊。

题目：HDU5327~5338

## **A. Olympiad**

### **题意**

问[l,r]区间内有多少各位数字不相同的数。l,r<=1e5。

### **题解**

签到题。模拟即可。

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
#define R register
#define LL long long
using namespace std;
const int N=1e5+10;
const int mod=1e9+7;
const int inf=0x3f3f3f3f;

int n,sum[N],bit[12];

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

int check(int x) {
	memset(bit,0,sizeof(bit));
	while (x) {
		int r=x%10;
		if (bit[r]) return 0;
		bit[r]=1;
		x/=10;
	}
	return 1;
}

int main() {
	int t; read(t);
	for (int i=1;i<N;i++) {
		sum[i]=sum[i-1]+check(i);
	}
	for (int i=1;i<=t;i++) {
		int a,b; read(a); read(b); cout<<sum[b]-sum[a-1]<<endl;
	}
	return 0;
}
```

## **B. Problem Killer**

### **题意**

给定一个长为n的串，求最长的等差或等比子串。

### **题解**

签到题*2，模拟即可。

```cpp
#include <bits/stdc++.h>
#define int long long
#define fo(i, n) for(int i = 1; i <= (n); i ++)
#define out(x) cerr << #x << " = " << x << "\n"
#define type(x) __typeof((x).begin())
#define foreach(it, x) for(type(x) it = (x).begin(); it != (x).end(); ++ it)
using namespace std;
// by piano
template<typename tp> inline void read(tp &x) {
  x = 0;char c = getchar(); bool f = 0;
  for(; c < '0' || c > '9'; f |= (c == '-'), c = getchar());
  for(; c >= '0' && c <= '9'; x = (x << 3) + (x << 1) + c - '0', c = getchar());
  if(f) x = -x;
}
template<typename tp> inline void arr(tp *a, int n) {
  for(int i = 1; i <= n; i ++)
    cout << a[i] << " ";
  puts("");
}
namespace one {
  static const int N = 2e6 + 233;
  int n, ans, a[N], l, r;

  inline void upd(void) {
    ans = max(ans, r - l + 1);
  }

  int main(void) {
    ans = 0;
    read(n);
    for(int i = 1; i <= n; i ++) read(a[i]);
    if(n <= 1) return cout << "1\n", 0;
    l = 1, r = 2;
    upd();
    for(r = 3; r <= n; r ++) {
      int delta = a[r] - a[r - 1];
      while(l < r && a[l + 1] - a[l] != delta)
        ++ l;
      upd();
    }
    l = 1, r = 2;
    upd();
    for(r = 3; r <= n; r ++) {
      while(l < r && a[l + 1] * a[r - 1] != a[r] * a[l])
        ++ l;
      upd();
    }
    cout << ans << "\n";
  }
}
main(void) {
  int T; for(read(T); T --;) one::main();
}
```


## **C. Question for the Leader**

### **题意**

给定一棵n个节点的基环树，问存在多少个k，使得可以将其分为n/k个联通块，每个联通块中包含k个节点。

n<=100000

### **题解**

对于树，有一个结论：如果可以将这棵树分为大小相同的n/k份，那么子树大小是k的倍数的节点数有n/k个。（根可以任选）

但这是一棵基环树。那我们先把环扣出来，暴力枚举删掉哪条边，然后就可以套用上面的结论了。

直接模拟是O(n^2)的，考虑优化。

先把非环上的点的size算好，然后考虑环上的点。将环上所有的点按序标号，size[i]表示第i个点挂下的子树大小，sum[i]表示前i个点挂下的子树大小和。

当我们将第i个点和第(i-1)个点之间的边断开时，j号点的子树大小为(sum[j]-sum[i])。对于一种约数x，只要(sum[j]%x==sum[i]%x),j号点的子树大小就是x的倍数了。在预处理后可以O(1)统计。

注意要加上非环点的贡献。最终答案应该是所有断边方案中可取约数的并集，复杂度O(n^1.5)

代码挺恶心的……

```cpp
#include<bits/stdc++.h>
#define R register
using namespace std;
const int N=1e5+10;

int n,m,E,tim;
int fa[N],fir[N],nex[N<<1],arr[N<<1];
int c[N],a[N],siz[N],num[N],dfn[N],sum[N],md[150][N],cnt[N],can[N];
vector<int> vec,cir;

inline void Add_Edge(int x,int y) {
	nex[++E]=fir[x];
	fir[x]=E; arr[E]=y;
}

void Init() {
	tim=E=0;
	memset(fir,0,sizeof(int)*(n+5));
	memset(num,0,sizeof(int)*(n+5));
	memset(c,0,sizeof(int)*(n+5));
	memset(dfn,0,sizeof(int)*(n+5));
	vec.clear(); cir.clear();
	for (R int i=1;i<=n;i++)
		if (n%i==0) vec.push_back(i);	
	for (R int i=1;i<=n;i++) {
		int x; scanf("%d",&x);
		Add_Edge(i,x); Add_Edge(x,i);
	}
	for (int i=0;i<vec.size();i++)
		memset(md[i],0,sizeof(int)*(vec[i]+3));
	memset(can,0,sizeof(int)*(vec.size()+3));
}

void getcir(int x) {
	dfn[x]=++tim;
	for (int i=fir[x];i;i=nex[i]) {
		if (arr[i]==fa[x]) continue;
		if (!dfn[arr[i]]) fa[arr[i]]=x,getcir(arr[i]);
		else if (dfn[arr[i]]>dfn[x]) {
			int y=arr[i];
			cir.push_back(x); c[x]=1;
			do {
				cir.push_back(y);
				c[y]=1;
				y=fa[y];
			}while (x!=y);
		}
	}
}

void dfs(int x,int fa) {
	siz[x]=1;
	for (int i=fir[x];i;i=nex[i]) {
		if (!c[arr[i]]&&arr[i]!=fa) {
			dfs(arr[i],x); siz[x]+=siz[arr[i]];
		}
	}
}

void check(int x) {
	int v=vec.size();
	for (int i=0;i<v;i++) {
		cnt[i]=md[i][(sum[x]+vec[i])%vec[i]]+num[i];
		if (cnt[i]*vec[i]==n) can[i]=1;
	}
}

void Solve() {
	Init();
	getcir(1);
	for (int i=1;i<=n;i++) 
		if (c[i]) dfs(i,0);
	for (R int i=1;i<=n;i++)
		if (!c[i]) {
			int v=vec.size();
			for (R int j=0;j<v;j++) {
				if (siz[i]%vec[j]==0) num[j]++;
			}
		}
	int v=vec.size();
	for (int i=0;i<cir.size();i++) {
		sum[i]=(i)?sum[i-1]+siz[cir[i]]:siz[cir[i]];
		for (int j=0;j<v;j++) md[j][sum[i]%vec[j]]++;
	}
	for (int i=0;i<cir.size();i++) check(i);
	int ans=0;
	for (int i=0;i<vec.size();i++) ans+=can[i];
	printf("%d\n",ans);
}

int main() {
	while (cin>>n) Solve();
	return 0;
}
```


## **D. Route Statistics**

### **题意**

给定m维坐标中的n个点，坐标范围是{0,1,2}，对于每个k∈[0,2m]，求出有多少个点对的曼哈顿距离是k。

m<=11,n<=300000

### **题解**

神奇的状压DP……

我们要计算的是到每个串距离为x的串有多少个。用**f[i][s][j]**表示与串**s**的前**i**位都相同的那些串，到**s**距离为**j**的个数。

初始状态**f[m][s][0]=串s出现次数**。只要转移出**f[0][s][x]**就可以了。

假设串**s**第**i**位是**c**，那么就有：

![][6]

最后统计答案的时候注意特判x=0

复杂度O(m^2*3^m)

```cpp
#include<bits/stdc++.h>
#define LL long long
using namespace std;
const int N=300005;

LL ans[30];
int n,m,T,pw[20],cnt[N],f[2][N][30];
char s[30];

void Solve() {
	scanf("%d%d",&n,&m);
	memset(cnt,0,sizeof(int)*(pw[m]+3));
	for (int i=1;i<=n;i++) {
		scanf("%s",s+1);
		int x=0;
		for (int i=1;i<=m;i++) x=x*3+s[i]-'0';
		cnt[x]++;
	}
	for (int i=0;i<m;i++) 
		for (int j=0;j<pw[m];j++)
			memset(f[0][j],0,sizeof(int)*(2*m+2));
	for (int i=0;i<pw[m];i++) f[0][i][0]=cnt[i];
	int now=1;
	for (int i=0;i<m;i++) {
		for (int j=0;j<pw[m];j++)
			memset(f[now][j],0,sizeof(int)*(2*m+2));
		for (int j=0;j<pw[m];j++)
			for (int k=0;k<=2*m;k++) {
				if (!f[now^1][j][k]) continue;
				int x=j/pw[i]%3;
				for (int l=0;l<=2;l++) {
					int to=j+(l-x)*pw[i];
					f[now][to][k+abs(x-l)]+=f[now^1][j][k];
				}
			}
		now^=1;
	}
	memset(ans,0,sizeof(ans));
	for (int i=0;i<pw[m];i++) {
		if (!cnt[i]) continue;
		ans[0]+=1LL*cnt[i]*(cnt[i]-1);
		for (int j=1;j<=2*m;j++) ans[j]+=1LL*cnt[i]*f[now^1][i][j];
	}
	for (int i=0;i<=2*m;i++) printf("%lld\n",ans[i]/2);
}

int main() {
	pw[0]=1;
	for (int i=1;i<=12;i++) pw[i]=pw[i-1]*3; 
	scanf("%d",&T);
	for (int i=1;i<=T;i++) Solve();
	return 0;
}
```


## **E. Simple Problem**

### **题意**

一开始有一个点0，进行(n-1)次加点操作，每次保证加入后的图仍是一棵树，输出每次加点后树的最大独立集。

n<=100000

### **题解**

因为树是二分图，只需求出最大匹配数就可以了。

如果树的形态不变，显然有这样的贪心策略：用**f[i]**表示i号点是否和它的一个孩子匹配，那么，

![][7]

再考虑加点操作。我们用**a[i]**表示i号点的孩子中f为0的个数，那么加入一个点x后会影响的点是满足以下条件的点y组成的链：

1、y是x的祖先；

2、y到x的路径中，距离x为奇数的点a为0，为偶数的点a为1。

那我们就可以事先把树建好，树链剖分，每次直接修改即可。

线段树需要记录：f的和，深度为奇数、偶数的a的最大值。

一段x的祖先的区间，如果距离x为奇数的点a最大值为0，为偶数的点a最大值为1，这段区间会被影响。这是因为一个点如果有孩子的a是0，这个点的a至少是1。

修改的时候，被影响区间的f被翻转，a按深度奇偶性加减。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=2e5+10;

int n,m,E,y,tim;
int fir[N],nex[N<<1],arr[N<<1];
int pos[N],son[N],siz[N],fa[N],top[N],dep[N],dfn[N];
int len[N<<2],val[N<<2],rev[N<<2],mx[N<<2][2],tag[N<<2][2];

inline void Add_Edge(int x,int y) {
	nex[++E]=fir[x];
	fir[x]=E; arr[E]=y;
}

void dfs1(int x) {
	siz[x]=1; son[x]=0;
	for (int i=fir[x];i;i=nex[i]) {
		if (arr[i]==fa[x]) continue;
		dfs1(arr[i]);
		siz[x]+=siz[arr[i]];
		if (siz[arr[i]]>siz[son[x]]) son[x]=arr[i];
	}
}

void dfs2(int x) {
	dfn[pos[x]=++tim]=x;
	if (son[x]) top[son[x]]=top[x],dfs2(son[x]);
	for (int i=fir[x];i;i=nex[i]) {
		if (arr[i]==fa[x]||arr[i]==son[x]) continue;
		else top[arr[i]]=arr[i],dfs2(arr[i]);
	}
}

void Build(int x,int l,int r) {
	val[x]=mx[x][0]=mx[x][1]=tag[x][0]=tag[x][1]=rev[x]=0;
	len[x]=r-l+1;
	if (len[x]==1) return;
	int mid=(l+r)>>1;
	Build(x<<1,l,mid); Build(x<<1|1,mid+1,r);
}

inline void Pushdown(int x) {
	for (int i=0;i<2;i++) {
		if (!tag[x][i]) continue;
		mx[x<<1][i]+=tag[x][i];
		mx[x<<1|1][i]+=tag[x][i];
		tag[x<<1][i]+=tag[x][i];
		tag[x<<1|1][i]+=tag[x][i];
		tag[x][i]=0;
	}
	if (rev[x]) {
		val[x<<1]=len[x<<1]-val[x<<1];
		val[x<<1|1]=len[x<<1|1]-val[x<<1|1];
		rev[x<<1]^=1; rev[x<<1|1]^=1;
		rev[x]=0;
	}
}

inline void Pushup(int x) {
	for (int i=0;i<2;i++) mx[x][i]=max(mx[x<<1][i],mx[x<<1|1][i]);
	val[x]=val[x<<1]+val[x<<1|1];
}

void Query(int x,int l,int r,int lt,int rt,int t) {
	if (lt<=l&&r<=rt) {
		if (mx[x][t]<=1&&mx[x][t^1]<=0) return (void) (y=l);
		if (l==r) return (void) (y=(!y)?(n+1):y);
	}
	Pushdown(x);
	int mid=(l+r)>>1;
	if (mid<rt) {
		Query(x<<1|1,mid+1,r,lt,rt,t);
		if (y>mid+1) return;
	}
	if (lt<=mid) Query(x<<1,l,mid,lt,rt,t);
}

void Updata(int x,int l,int r,int lt,int rt,int t) {
	if (lt<=l&&r<=rt) {
		tag[x][t]--; tag[x][t^1]++;
		mx[x][t]--; mx[x][t^1]++;
		return;
	}
	Pushdown(x);
	int mid=(l+r)>>1;
	if (mid>=lt) Updata(x<<1,l,mid,lt,rt,t);
	if (mid<rt) Updata(x<<1|1,mid+1,r,lt,rt,t);
	Pushup(x);
}

void Reverse(int x,int l,int r,int lt,int rt) {
	if (lt<=l&&r<=rt) {
		val[x]=len[x]-val[x];
		rev[x]^=1;
		return;
	}
	Pushdown(x);
	int mid=(l+r)>>1;
	if (mid>=lt) Reverse(x<<1,l,mid,lt,rt);
	if (mid<rt) Reverse(x<<1|1,mid+1,r,lt,rt);
	Pushup(x);
}

void Modify(int x,int t) {
	for (;x;x=fa[top[x]]) {
		y=0;
		Query(1,1,n,pos[top[x]],pos[x],t);
		if (y==n+1) return (void) Updata(1,1,n,pos[x],pos[x],t);
		y=dfn[y];
		Updata(1,1,n,pos[y],pos[x],t);
		Reverse(1,1,n,pos[y],pos[x]);
		if (y!=top[x]) {
			y=fa[y];
			Updata(1,1,n,pos[y],pos[y],t);
			return;
		}
	}
}

void Solve() {
	E=tim=0;
	memset(fir,0,sizeof(int)*(n+3));
	for (int i=2;i<=n;i++) {
		scanf("%d",&fa[i]);
		Add_Edge(i,++fa[i]);
		Add_Edge(fa[i],i);
		dep[i]=dep[fa[i]]+1;
	}
	dfs1(1); top[1]=1; dfs2(1);
	Build(1,1,n);
	for (int i=2;i<=n;i++) {
		Modify(fa[i],dep[i]&1);
		printf("%d\n",i-val[1]);
	}	
}

int main() {
	while (~scanf("%d",&n)) Solve();
	return 0;
}
```

## **F. Test for Rikka**

### **题意**

构造一个n*n的矩阵A，令B=A^m,使得B[1][n]=k，读入k，n和m可以任选。

k<=1e18,n<=30,m<=30。

### **题解**

神题……

显然如果将这个矩阵看做图的邻接矩阵，就是点1走m步到点n的方案数为k。

一开始想的是将图分层，低层向高层连边，终点加自环，这样可以控制步数，但是方案数只能到1e9。

要控制步数，要么让图变成一张DAG，要么就来一张完全图！

这样构造图：从点1出发，连向一个大小为C的团。团后面接一条长为d的链，链的底端是点n。

考虑团中的一个点x，走了i步后到达点x的方案数显然是**C^(i-1)**，那么如果将x向链上距点n距离为t的点连边，就会对答案产生**C^(m-t-1)**的贡献。

不妨取C=9,d=20,m=21，易知可以构造出k<=1e18的所有图。

```cpp
#include<bits/stdc++.h>
using namespace std;

long long k,pw[22];
int a[35][35];

void Solve() {
	memset(a,0,sizeof(a));
	for (int i=2;i<=10;i++) a[1][i]=1;
	for (int i=2;i<=10;i++)
		for (int j=2;j<=10;j++) a[i][j]=1;
	for (int i=11;i<=29;i++) a[i][i+1]=1;
	scanf("%lld",&k);
	int now=19;
	while (k) {
		while (k<pw[now]&&now) now--;
		for (int i=2;k>=pw[now];k-=pw[now],i++) a[i][now+11]=1;
	}
	puts("30 21");
	for (int i=1;i<=30;i++) {
		for (int j=1;j<=30;j++) printf("%d",a[i][j]);
		putchar('\n');
	}
}

int main() {  
	pw[0]=1;
	for (int i=1;i<=19;i++) pw[i]=pw[i-1]*9;
	int T; scanf("%d",&T);
	while (T--) Solve();
	return 0;
}
```


## **G. Undirected Graph**

### **题意**

给定一张n个点m条边的无向图，q次询问，每次询问如果只保留左右端点都在区间[l,r]中的边，原图有几个联通块。

n<=100000,m<=200000,q<=200000

### **题解**

考虑离线。如果所有询问关于右端点排序，可选的边就是右端点不超过询问右端点的边，且这些边中左端点越靠右，越容易减小联通块个数。

那么将边也关于右端点排序。当询问的右端点右移，相应地也将边的右端点右移。每次加入一条边的时候，如果连接的两个点不连通，直接加入；否则，代替掉两点路径上左端点最靠左的边（如果这条边的左端点比加入边的左端点靠左）。这样就需要用一颗LCT维护最小生成树森林。

询问的时候，每多一条左端点在询问左端点右边的边，就会使联通块个数-1，树状数组维护一下即可。

```cpp
#pragma GCC optimize(2)
#include<bits/stdc++.h>
using namespace std;
const int N=3e5+10;
const int V=1e5+5;
const int inf=0x3f3f3f3f;

int n,m,q,tot;
int mi[N],ch[N][2],rev[N],st[N],tree[N],fa[N],ans[N];
struct Pair {
	int u,v,tim;
	bool operator < (Pair &other) const {
		return v<other.v;
	}
}E[N],Q[N];

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

inline int Get(int x) {
	return (ch[fa[x]][1]==x);
}

inline int Isroot(int x) {
	return (ch[fa[x]][0]!=x)&&(ch[fa[x]][1]!=x);
}

inline void Pushup(int x) {
	if (E[mi[ch[x][0]]].u<E[mi[ch[x][1]]].u) mi[x]=mi[ch[x][0]];
	else mi[x]=mi[ch[x][1]];
	if (x<=V) return;
	if (E[x-V].u<E[mi[x]].u) mi[x]=x-V;
}

inline void Pushdown(int x) {
	if (!rev[x]) return;
	swap(ch[x][0],ch[x][1]);
	rev[ch[x][0]]^=1;
	rev[ch[x][1]]^=1;
	rev[x]=0;
}

inline void Rotate(int x) {
	int y=fa[x],z=fa[y],k=Get(x);
	if (!Isroot(y)) ch[z][Get(y)]=x;
	fa[x]=z;
	fa[ch[x][k^1]]=y;
	ch[y][k]=ch[x][k^1];
	ch[x][k^1]=y;
	fa[y]=x;
	Pushup(y); Pushup(x);
}

inline void Splay(int x) {
	int top=0;
	st[++top]=x;
	for (int t=x;!Isroot(t);t=fa[t]) st[++top]=fa[t];
	for (;top;top--) Pushdown(st[top]);
	for (;!Isroot(x);Rotate(x)) {
		if (!Isroot(fa[x])) (Get(x)==Get(fa[x]))?Rotate(fa[x]):Rotate(x);
	}
	Pushup(x);
}

inline void Access(int x) {
	for (int t=0;x;t=x,x=fa[x]) {
		Splay(x); ch[x][1]=t; Pushup(x);
	}
}

inline int Find(int x) {
	Access(x); Splay(x);
	while (ch[x][0]) x=ch[x][0];
	Splay(x); return x;
}

inline void Evert(int x) {
	Access(x); Splay(x); rev[x]^=1;
}

inline void Link(int x,int y) {
	Evert(x); fa[x]=y;
}

inline void Cut(int x,int y) {
	Evert(x); Access(y); Splay(y);
	ch[y][0]=fa[x]=0;
}

inline int Query(int x,int y) {
	Evert(x); Access(y); Splay(y);
	return mi[y];
}

inline void Updata(int x,int del) {
	while (x<=n) tree[x]+=del,x+=(x&-x);
}

inline int ask(int x) {
	int ans=0;
	while (x) ans+=tree[x],x-=(x&-x);
	return ans;
}

void Insert(int x) {
	if (E[x].u==E[x].v) return;
	if (Find(E[x].u)!=Find(E[x].v)) {
		++tot; Updata(E[x].u,1);
		Link(x+V,E[x].u); Link(x+V,E[x].v);
		return;
	}
	int t=Query(E[x].u,E[x].v);
	if (E[t].u>E[x].u) return;
	Updata(E[t].u,-1);
	Cut(t+V,E[t].u); Cut(t+V,E[t].v);
	Updata(E[x].u,1);
	Link(x+V,E[x].u); Link(x+V,E[x].v);
}

void Init() {
	tot=0;
	memset(tree,0,sizeof(int)*(n+5));
	memset(ans,0,sizeof(int)*(q+5));
	for (int i=1;i<=n;i++) ch[i][0]=ch[i][1]=fa[i]=rev[i]=mi[i]=0;
	for (int i=1+V;i<=m+1+V;i++) ch[i][0]=ch[i][1]=fa[i]=rev[i]=mi[i]=0;	
} 

int main() {
	while (~scanf("%d%d%d",&n,&m,&q)) {
		for (int i=1;i<=m;i++) {
			read(E[i].u); read(E[i].v);
			if (E[i].u>E[i].v) swap(E[i].u,E[i].v);
		}
		for (int i=1;i<=q;i++) {
			read(Q[i].u); read(Q[i].v); Q[i].tim=i;
		}
		sort(E+1,E+m+1); sort(Q+1,Q+q+1);
		E[0].u=inf; for (int i=1;i<=m;i++) mi[i+V]=i;
		for (int i=1,j=0;i<=q;i++) {
			while (j<m&&E[j+1].v<=Q[i].v) {
				++j;
				Insert(j);
			}
			ans[Q[i].tim]=n-tot+ask(Q[i].u-1);
		}
		for (int i=1;i<=q;i++) writeln(ans[i]);
		Init();
	}
	return 0;
}
```


## **H. Virtual Participation**

### **题意**

给定整数k<=1e9,构造一个串，使它不同的子串个数恰好为k，串长不大于100000。

### **题解**

大瞎搞题，糊了个和标算不同的做法。

看到题第一反应就是，每次将相同的数放若干次。

假设我们放第i个数xi次，总共放了r个数，那么不同的子串个数就是：

![][2]

但是这个式子很难处理。于是考虑缩减r的范围。

如果r=2，设放了x个1，y个2，那么子串个数就是(x+y+xy),因式分解一下：

![][3]

这个式子表明如果(k+1)可以分解为两个和不超过100002的整数的乘积，我们就可以构造出解。

这样不能处理(k+1)为质数。考虑把r扩展到3：

![][4]

枚举x=1,2后发现有规律。不妨把x当做常数，那么就有：

![][5]

这时只要k+(x+1)^2-x可以分解为两个和不超过100000的整数的乘积就行了。不妨将x从1开始枚举，只需很少的次数就可枚举出解。

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
#define R register
#define LL long long
using namespace std;
const int N=1e5+10;
const int mod=1e9+7;
const int inf=0x3f3f3f3f;

int n,k,T;

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

void Solve() {
	for (int x=1;x<=1000;x++) {
		int t=k+(x+1)*(x+1)-x;
		for (int i=x+1;i*i<=t;i++) {
			if (t%i) continue;
			int y=i-x-1,z=t/i-x-1;
			if (y<0||z<0) continue;
			if (x+y+z>100000) continue;
			printf("%d\n",x+y+z);
			for (int i=1;i<x;i++) putchar('1'),putchar(' ');
			if (!y) y=z,z=0;
			if (x) {
				putchar('1');
				if (y||z) putchar(' ');
			}
			for (int i=1;i<y;i++) putchar('2'),putchar(' ');
			if (y) {
				putchar('2');
				if (z) putchar(' ');
			}
			for (int i=1;i<z;i++) putchar('3'),putchar(' ');
			if (z) putchar('3'); 
			putchar('\n');
			return;
		}
	}
	throw;
}

main() {
	while (~scanf("%d",&k)) Solve();
	return 0;
}
```

## **I. Walk Out**

### **题意**

给定一个n*m的迷宫，每个格子上有0或1。可以上下左右移动，求(1,1)到(n,m)的最短路。

路径长度的定义是：将所有走过格子上的数字按次序写下后得到的二进制数的大小。

去前导零后输出二进制数，n,m<=1000。

### **题解**

VP时我切的第二题。不难但挺精彩的一道题。

不能跑最短路的瓶颈在于dist的比较是O(n)的。一开始想优化它但是没有想到好的办法。

后面突然想到：如果(1,1)上的数字是1，那最优解一定也是曼哈顿距离最小的解。

顺着这个思路想下去。如果走过的路径上已经有了1，那么只能向下或向右走；否则可以向四个方向走。

将搜索分为两个阶段：

1、只走0的格子。在可以走到的范围内找出离终点曼哈顿距离最小的1，将他们入队。

2、取出这些1，记录答案，再往下BFS。

显然每个点只会入队出队一次，复杂度O(n*m)。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=1e3+10;
const int u[4]={1,0,-1,0};
const int v[4]={0,1,0,-1};

char s[N];
int T,n,m,cnt,ans_num,type;
int dis[N],a[N][N],ans[N*10],vis[N][N];
struct node {
	int x,y,dis,To_end;
}tmp[N];
queue<node> Q;

void Init() {
	memset(ans,0,sizeof(ans));
	memset(vis,0,sizeof(vis));
	memset(dis,0,sizeof(dis));
	cnt=0; ans_num=0;
}

void Getplace() {
	vis[1][1]=1;
	if (a[1][1]) return (void) (tmp[++cnt]=(node) {1,1,1,n-1+m-1});
	Q.push((node){1,1,1,n-1+m-1});
	while (!Q.empty()) {
		int x=Q.front().x,y=Q.front().y,d=Q.front().dis; Q.pop();
		for (int i=0;i<4;i++) {
			int xx=x+u[i],yy=y+v[i],To_end=n-xx+m-yy;
			if (xx>n||xx<=0||yy>m||yy<=0) continue;
			if (!vis[xx][yy]) {
				vis[xx][yy]=1;
				if (!a[xx][yy]) Q.push((node){xx,yy,d+1,To_end});
				else {
					if (cnt&&To_end<tmp[cnt].To_end) tmp[cnt=1]=(node) {xx,yy,d+1,To_end};
					else if (!cnt||To_end==tmp[cnt].To_end) tmp[++cnt]=(node) {xx,yy,d+1,To_end};
				}
			}
		}
	}
}

void Solve() {
	scanf("%d%d",&n,&m);
	for (int i=1;i<=n;i++) {
		scanf("%s",s+1);
		for (int j=1;j<=m;j++) a[i][j]=s[j]-'0';
	}
	type=a[n][m]; a[n][m]=1;
	Init();
	Getplace();
	while (cnt) {
		ans[++ans_num]=tmp[1].dis;
		for (int i=1;i<=cnt;i++) Q.push(tmp[i]);
		cnt=0;
		while (!Q.empty()) {
			int x=Q.front().x,y=Q.front().y,d=Q.front().dis; Q.pop();
			for (int i=0;i<2;i++) {
				int xx=x+u[i],yy=y+v[i],To_end=n-xx+m-yy;
				if (xx>n||xx<=0||yy>m||yy<=0) continue;
				if (!vis[xx][yy]) {
					vis[xx][yy]=1;
					if (!a[xx][yy]) Q.push((node){xx,yy,d+1,To_end});
					else {
						if (cnt&&To_end<tmp[cnt].To_end) tmp[cnt=1]=(node) {xx,yy,d+1,To_end};
						else if (!cnt||To_end==tmp[cnt].To_end) tmp[++cnt]=(node) {xx,yy,d+1,To_end};
					}
				}
			}
		}
	}
	int now=ans[1];
	for (int i=1;i<=ans_num;i++) {
		if (i>1) for (;now<ans[i];now++) putchar('0');
		if (i!=ans_num) putchar('1');
		else cout<<type;
		now++;
	}
	putchar('\n');
}

int main() {
	scanf("%d",&T);
	while (T--) Solve();
	return 0;
}
```

## **J. XYZ and Drops**

### **题意**

模拟游戏十滴水……

### **题解**

按题意模拟即可

跟DHL看了半天查不出错，一气之下重构代码就AC了

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=105;
const int u[4]={1,-1,0,0};
const int v[4]={0,0,1,-1};

int r,c,n,T,x[N],y[N];
int siz[N],tim[N],id[N][N];

struct drop {
	int x,y,dx,dy;
};
vector<int> wat;
vector<drop> vec,tmp;

void Push(int x,int y) {
	for (int i=0;i<4;i++) tmp.push_back((drop){x,y,u[i],v[i]});
}

void Init() {
	vec.clear(); tmp.clear(); wat.clear();
	memset(id,0,sizeof(id));
	memset(tim,0,sizeof(tim));
	for (int i=1;i<=n;i++) scanf("%d%d%d",&x[i],&y[i],&siz[i]);
	for (int i=1;i<=n;i++) id[x[i]][y[i]]=i;
	int xx,yy; scanf("%d%d",&xx,&yy); Push(xx,yy);
}

void Doit(int T) {
	vec.clear();
	for (int i=0;i<tmp.size();i++) vec.push_back(tmp[i]);
	tmp.clear();
	wat.clear();
	for (int i=0;i<vec.size();i++) {
		drop t=vec[i];
		int xx=t.x+t.dx,yy=t.y+t.dy;
		if (xx<=0||xx>r||yy<=0||yy>c) continue;
		int p=id[xx][yy];
		if (siz[p]) {
			if (++siz[p]>=5&&!tim[p]) {
				tim[p]=T;
				wat.push_back(p);
			}
		}
		else tmp.push_back((drop){xx,yy,t.dx,t.dy});
	}
	for (int i=0;i<wat.size();i++) {
		siz[wat[i]]=0;
		Push(x[wat[i]],y[wat[i]]);
	}
}

void Solve() {
	Init();
	for (int i=1;i<=T;i++) Doit(i);
	for (int i=1;i<=n;i++) {
		if (siz[i]) printf("1 %d\n",siz[i]);
		else printf("0 %d\n",tim[i]);
	}
}

int main() {
	while (cin>>r>>c>>n>>T) Solve();
	return 0;
}
```


## **K. Yet Another XYZ Problem**

### **题意**

给定两个由x、y、z组成的字符串S,T,每次可以进行下面三个操作：

1、将S中所有x变成y，代价C0；

2、将S中所有y变成z，代价C1；

3、将S中所有z变成x，代价C2。

问将S变换成T，在代价不超过Maxcost的情况下有多少种方案，对1e9+7取模。

C0,C1,C2,Maxcost<=1e18，输入数据不超过50KB

### **题解**

先吐槽一发……

据说wys的毒瘤是跟xyz学的？？？

惊恐的眼神……

吓得我都打出了：

```cpp
return (void) (printf("%d\n"),getmod(ans));
```

更恐怖的是，调代码的时候第一组把我卡掉的数据是:

```
1
1 1 1 10
xyz
xxy
```

来自大佬的嘲讽……幸运的是身体还算健康

分类讨论题，大致分为7类。用cnt1记S中出现字母种类数，cnt2记T中出现字母种类数，sum=C0+C1+C2

1、cnt1<cnt2，直接puts("0");

2、cnt1=1,cnt2=1。设将S变成T的最小费用是x，那么答案显然就是：

![][8]

3、cnt1=2,cnt2=2。这时每次变换所能改变的字母是确定的，否则改变另一个字母会使两个字母变成相同就变不回来了。然后同情况2计算即可。要注意的是有可能永远不能变成相同，比如"xzz"和"xyx"

4、cnt1=3，cnt2=3。S、T相同答案是1，否则为0

5、cnt1=3，cnt2=2。枚举第一次变换变了什么字母，然后同情况3计算即可。

6、cnt1=2，cnt2=1。显然是两个字母先变，变到某处后变成一个字母，然后再进行若干次变换。因为6是两种变换的循环节，这时要枚举变成一个字母所用步数关于6的余数，这时一个字母变换次数关于6取模有两种可能。不考虑变循环节的费用，假设前者所需费用是Cx，后者是Cy，那么前后可变循环节次数的最大值是：

![][9]

这时对答案产生的贡献是

![][10]

7、cnt1=3，cnt2=1。枚举第一次变换变了什么字母，然后同情况6计算即可。

```cpp
#include<bits/stdc++.h>
#define LL long long
using namespace std;
const int N=1e6+10;
const int mod=1e9+7;

LL c[3],cx;
int n,m,len,to[3],a[3][2];
char s[N],sx[N],t[N];

inline int nex(int x) {
	return (x==2)?0:x+1;
}

inline int getmod(LL x) {
	x%=mod;
	return (x<0)?x+mod:x;
}

int calc1(char *s,char *t,LL cc) {
	int a=s[1]-'x',b=t[1]-'x';
	while (a!=b) cc+=c[a],a=nex(a);
	if (cc>cx) return 0;
	return getmod((cx-cc)/(c[0]+c[1]+c[2])+1);
}

int calc2(char *s,char *t,LL cc) {
	memset(to,0,sizeof(to));
	for (int i=1;i<=len;i++) {
		if (!to[s[i]-'x']) to[s[i]-'x']=t[i];
		if (to[s[i]-'x']!=t[i]) return 0;
	}
	int a=s[1],u=t[1],b,v;
	for (int i=1;i<=len;i++) {
		if (s[i]!=a) b=s[i];
		if (t[i]!=u) v=t[i];
	}
	a-='x'; b-='x'; u-='x'; v-='x';
	while (!(a==u&&b==v)) {
		if (nex(a)==b) cc+=c[b],b=nex(b);
		else if (nex(b)==a) cc+=c[a],a=nex(a);
	}
	if (cc>cx) return 0;
	else return getmod((cx-cc)/(c[0]+c[0]+c[1]+c[1]+c[2]+c[2])+1);
}

int calc3(char *s,char *t) {
	for (int i=1;i<=len;i++)
		if (s[i]!=t[i]) return 0;
	return 1;
}

int calc4(char *s,char *t,LL cc) {
	int a=s[1],u=t[1],b; LL ans=0;
	for (int i=1;i<=len;i++) {
		if (s[i]!=a) b=s[i];
	}
	a-='x'; b-='x'; u-='x';
	for (int i=0;i<6;i++) {
		int x=a,y=b; LL cd=cc;
		for (int j=1;j<=i;j++) {
			if (nex(x)==y) cd+=c[y],y=nex(y);
			else if (nex(y)==x) cd+=c[x],x=nex(x);
		}
		if (nex(x)==y) cd+=c[x],x=y;
		else if (nex(y)==x) cd+=c[y],y=x;
		for (int j=0;j<6;j++) {
			if (cd>cx) break;
			if (x==u) {
				LL mx=(cx-cd)/(c[0]+c[0]+c[1]+c[1]+c[2]+c[2]);
				mx=getmod(mx);
				ans+=(mx+1)*(mx+2)/2;
				ans=getmod(ans);
			}
			cd+=c[x]; x=nex(x);
		}
	}
	return ans;
}

void Solve() {
	memset(a,0,sizeof(a));
	scanf("%lld%lld%lld%lld",&c[0],&c[1],&c[2],&cx);
	scanf("%s",s+1); scanf("%s",t+1);
	len=strlen(s+1);
	for (int i=1;i<=len;i++) {
		a[s[i]-'x'][0]++; a[t[i]-'x'][1]++;
	}
	int cnt1=(a[0][0]>0)+(a[1][0]>0)+(a[2][0]>0);
	int cnt2=(a[0][1]>0)+(a[1][1]>0)+(a[2][1]>0);
	if (cnt1<cnt2) return (void) (puts("0"));
	if (cnt1==1&&cnt2==1) return (void) (printf("%d\n",calc1(s,t,0)));
	if (cnt1==2&&cnt2==2) return (void) (printf("%d\n",calc2(s,t,0)));
	if (cnt1==3&&cnt2==3) return (void) (printf("%d\n",calc3(s,t)));
	if (cnt1==3&&cnt2==2) {
		LL ans=0;
		for (int i=0;i<3;i++) {
			for (int j=1;j<=len;j++) sx[j]=(s[j]==i+'x')?(nex(s[j]-'x')+'x'):s[j];
			ans+=calc2(sx,t,c[i]);
		}
		return (void) (printf("%d\n",getmod(ans)));
	}
	if (cnt1==2&&cnt2==1) return (void) (printf("%d\n",calc4(s,t,0)));
	if (cnt1==3&&cnt2==1) {
		LL ans=0;
		for (int i=0;i<3;i++) {
			for (int j=1;j<=len;j++) sx[j]=(s[j]==i+'x')?(nex(s[j]-'x')+'x'):s[j];
			ans+=calc4(sx,t,c[i]);
		}
		return (void) (printf("%d\n",getmod(ans)));
	}
	return (void) puts("0");
}

int main() {
	int T; scanf("%d",&T);
	while (T--) Solve();
	return 0;
}
/*
2
2 5 3 12
xxx
yyy
1 1 2 2
zzz
yyy

2
1 1 1 10
xy
yz
1 1 1 10
xzz
xyx

2
1 1 1 10
xyz
xyz
1 1 1 10
xyz
yxz

2
1 1 1 10
xyz
xxy 
1 1 1 10
xyz
yyx

3
1 1 1 5
xzx
xxx
1 1 1 10
xxy
xxx
1 1 1 20
xyx
zzz

1
3 2 2 9
xyz
xxx
*/
```

## **L. ZZX and Permutations**

### **题意**

给定一个长为n的全排列，用括号将它分为若干个区间，使变换后全排列的字典序最大。

例如全排列:1 4 5 6 3 2,如果划分为(1,4,5,6)(3,2),表示将1写到6的位置，6写到5的位置，5写到4的位置……

n<=100000

### **题解**

从小到大考虑每个数的位置被哪个数所占据。这有三种情况：

1、被自己占据；

2、被左边可取范围内最大的数占据；

3、被右边第一个数占据。

对于前两种情况，我们可以直接确定一个括号区间。直接将它们写进答案后删除这个区间。注意左边最大数和这个数之前不能有被删除过的区间。

对于第三种情况，我们只能确定一个括号区间的左括号l。右括号的选择又有两种情况：

1、在后面考虑一个数x的时将[l,x]归为一个区间；

2、考虑(l+1)时确定右括号。

那么只需要让(l+1)不能为左括号就可以了。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=1e5+10;

int n,m,T;
int a[N],id[N],mx[N<<2],num[N<<2],ans[N],used[N];
set<int> Set;

void Init() {
	scanf("%d",&n);
	memset(a,0,sizeof(int)*(n+3));
	memset(id,0,sizeof(int)*(n+3));
	for (int i=1;i<=n;i++) scanf("%d",&a[i]),id[a[i]]=i;
	Set.clear(); Set.insert(0);
	memset(used,0,sizeof(int)*(n+3));
}

void Pushup(int x) {
	if (mx[x<<1]<mx[x<<1|1]) mx[x]=mx[x<<1|1],num[x]=num[x<<1|1];
	else mx[x]=mx[x<<1],num[x]=num[x<<1];
}

void Build(int x,int l,int r) {
	if (l==r) return (void) (num[x]=l,mx[x]=a[l]);
	int mid=(l+r)>>1;
	Build(x<<1,l,mid); Build(x<<1|1,mid+1,r);
	Pushup(x);
}

int Query(int x,int l,int r,int lt,int rt) {
	if (rt<l||lt>r) return 0;
	if (lt<=l&&r<=rt) return num[x];
	int mid=(l+r)>>1;
	int u=Query(x<<1,l,mid,lt,rt),v=Query(x<<1|1,mid+1,r,lt,rt);
	return (a[u]<a[v])?v:u;
}

void Updata(int x,int l,int r,int pos) {
	if (l==r) return (void) (num[x]=mx[x]=0);
	int mid=(l+r)>>1;
	if (pos<=mid) Updata(x<<1,l,mid,pos);
	else Updata(x<<1|1,mid+1,r,pos);
	Pushup(x);
}

void Solve() {
	Init();
	Build(1,1,n);
	for (int i=1;i<=n;i++) {
		if (used[id[i]]) continue;
		int x=id[i];
		set<int>::iterator it=Set.lower_bound(x);
		int l=Query(1,1,n,*(--it)+1,x),r=x;
		if (a[l]<a[r+1]&&!used[r+1]) {
			ans[i]=a[r+1];
			Updata(1,1,n,r+1);
		}
		else {
			for (int i=l;i<=r;i++) {
				ans[a[i]]=a[(i==r)?l:i+1];
				Set.insert(i);
				Updata(1,1,n,i);
				used[i]=1;
			}
		}
	}
	for (int i=1;i<=n;i++) {
		printf("%d",ans[i]);
		putchar((i!=n)?' ':'\n');
	}
	memset(num,0,sizeof(int)*(2*n+10));
	memset(mx,0,sizeof(int)*(2*n+10));
}

int main() {
	scanf("%d",&T);
	while (T--) Solve();
	return 0;
}
```

[1]: http://acm.hdu.edu.cn/showproblem.php?pid=5327
[2]: https://vexoben.github.io/assets/images/Blog/2015-Multi-University-Contest-4(2).JPG
[3]: https://vexoben.github.io/assets/images/Blog/2015-Multi-University-Contest-4(3).JPG
[4]: https://vexoben.github.io/assets/images/Blog/2015-Multi-University-Contest-4(4).JPG
[5]: https://vexoben.github.io/assets/images/Blog/2015-Multi-University-Contest-4(5).JPG
[6]: https://vexoben.github.io/assets/images/Blog/2015-Multi-University-Contest-4(6).JPG
[7]: https://vexoben.github.io/assets/images/Blog/2015-Multi-University-Contest-4(7).JPG
[8]: https://vexoben.github.io/assets/images/Blog/2015-Multi-University-Contest-4(8).JPG
[9]: https://vexoben.github.io/assets/images/Blog/2015-Multi-University-Contest-4(9).JPG
[10]: https://vexoben.github.io/assets/images/Blog/2015-Multi-University-Contest-4(10).JPG
