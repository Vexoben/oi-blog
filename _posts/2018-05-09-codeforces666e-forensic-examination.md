---
layout: post
title: Codeforces666E Forensic Examination
date: 2018-05-09 18:51:30 +0800
categories: training
tags: 字符串 后缀自动机 数据结构 线段树及可持久化
img: https://vexoben.github.io/assets/images/Blog/2018-05-09-codeforces666e-forensic-examination.JPG
---

题目链接：[http://codeforces.com/contest/666/problem/E][1]

## **题意**

给定一个长为l(l<=500000)的字符串s，再给m(m<=50000)个总长不超过50000的字符串ti，q(q<=500000)组询问，每组询问(l,r,pl,pr)表示询问字符串s的子串s[l,r]，在字符串ti(pl<=i<=pr)中最多的出现次数。要求输出出现次数和出现次数最多的串的标号。

## **题解**

考虑把所有的询问串和s都加入到一个广义后缀自动机中。对于每一个节点，用线段树记录下它在每一个串ti中的出现次数。直接开满线段树会炸，需要动态开节点+线段树合并。对于每组询问，只需要快速找到这个子串所对应的节点，然后在线段树上查询即可。考虑s[1,r]对应的节点x和s[l,r]对应的节点y，因为y的Right一定包含x的Right，所以从x开始不停沿着pre指针跳一定会跳到y，这样只需要对每个r预先处理s[1,r]对应的节点，然后从这个节点开始倍增地向前跳到节点y即可。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=2e6+10;
const int inf=0x3f3f3f3f;

char s[N];
int n,m,ver[N];

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

namespace SAM {
	int root=1,tot=1,las=1;
	int len[N],deg[N],pre[N][22],ch[N][26];
	vector<int> son[N];

	int nw(int l) {
		len[++tot]=l;
		return tot;
	}

	void insert(int c) {
		if (ch[las][c]&&len[ch[las][c]]==len[las]+1) return (void) (las=ch[las][c]);
		int p=las,np=las=nw(len[p]+1);
		for (;!ch[p][c];p=pre[p][0]) ch[p][c]=np;
		if (!p) pre[np][0]=root;
		else {
			int q=ch[p][c];
			if (len[q]==len[p]+1) pre[np][0]=q;
			else {
				int nq=nw(len[p]+1);
				memcpy(ch[nq],ch[q],sizeof(ch[nq]));
				pre[nq][0]=pre[q][0]; pre[q][0]=pre[np][0]=nq;
				for (;ch[p][c]==q;p=pre[p][0]) ch[p][c]=nq;
			}
		}
	}
	
	void dfs(int x) {
		for (int i=1;pre[x][i-1]&&i<=21;i++) pre[x][i]=pre[pre[x][i-1]][i-1];
		for (int i=0;i<son[x].size();i++) dfs(son[x][i]);
	}
}
using namespace SAM;

namespace SGT {
	int tot,rt[N];
	struct Node {
		int mx,id,ch[2];
	}tree[N<<3];
	
	#define lch (tree[x].ch[0])
	#define rch (tree[x].ch[1])
	
	inline void Pushup(int x,int u,int v) {
		if (tree[u].mx>tree[v].mx) {
			tree[x].id=tree[u].id;
			tree[x].mx=tree[u].mx;
		}
		else if (tree[u].mx<tree[v].mx) {
			tree[x].id=tree[v].id;
			tree[x].mx=tree[v].mx;
		}
		else {
			tree[x].id=min(tree[u].id,tree[v].id);
			tree[x].mx=tree[u].mx;
		}
	}
	
	void Updata(int &x,int l,int r,int p) {
		if (!x) x=++tot;
		if (l==r) return (void) (tree[x].mx++,tree[x].id=p);
		int mid=(l+r)>>1;
		if (mid>=p) Updata(lch,l,mid,p);
		else Updata(rch,mid+1,r,p);
		Pushup(x,lch,rch);
	}
	
	pair<int,int> Query(int x,int l,int r,int lt,int rt) {
		if (!x) return make_pair(0,-lt);
		if (lt<=l&&r<=rt) return make_pair(tree[x].mx,-tree[x].id);
		int mid=(l+r)>>1;
		pair<int,int> ans=make_pair(0,-lt);
		if (mid>=lt) ans=max(ans,Query(lch,l,mid,lt,rt));
		if (mid<rt) ans=max(ans,Query(rch,mid+1,r,lt,rt));
		return ans;
	}
	
	int Merge(int x,int y,int l,int r) {
		if (!x||!y) return (x+y);
		int z=++tot;
		if (l==r) {
			tree[z].mx=tree[x].mx+tree[y].mx;
			tree[z].id=tree[x].id;
			return z;
		}
		int mid=(l+r)>>1;
		tree[z].ch[0]=Merge(lch,tree[y].ch[0],l,mid);
		tree[z].ch[1]=Merge(rch,tree[y].ch[1],mid+1,r);
		Pushup(z,tree[z].ch[0],tree[z].ch[1]);
		return z;
	}

	#undef lch
	#undef rch
}
using namespace SGT;

void Push(char *s,int now) {
	las=root;
	int len=strlen(s+1);
	for (int i=1;i<=len;i++) {
		insert(s[i]-'a');
		Updata(rt[las],1,n,now);
	}
}

void GetTree() {
	queue<int> Q;
	for (int i=2;i<=SAM::tot;i++) deg[pre[i][0]]++;
	for (int i=1;i<=SAM::tot;i++)
		if (!deg[i]) Q.push(i);
	while (!Q.empty()) {
		int x=Q.front(); Q.pop();
		rt[pre[x][0]]=Merge(rt[pre[x][0]],rt[x],1,n);
		if (!--deg[pre[x][0]]) Q.push(pre[x][0]);
	}
}

void Solve(int l,int r,int pl,int pr) {
	int x=ver[r];
	for (int i=21;~i;i--) {
		if (len[pre[x][i]]>=r-l+1) x=pre[x][i];
	}
	pair<int,int> ans=Query(rt[x],1,n,pl,pr);
	write(-ans.second); putchar(' '); writeln(ans.first);
}

int main() {
	scanf("%s",s+1);
	int len=strlen(s+1);
	for (int i=1;i<=len;i++) insert(s[i]-'a'),ver[i]=las;;
	read(n);
	for (int i=1;i<=n;i++) {
		scanf("%s",s+1);
		Push(s,i);		
	}
	GetTree();
	for (int i=2;i<=SAM::tot;i++) son[pre[i][0]].push_back(i);
	dfs(root);
	for (read(m);m;m--) {
		int l,r,pl,pr; 
		read(l); read(r); read(pl); read(pr);
		Solve(pl,pr,l,r);
	}
	return 0;
}
```

[1]: http://codeforces.com/contest/666/problem/E

