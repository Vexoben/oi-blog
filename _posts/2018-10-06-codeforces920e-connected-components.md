---
layout: post
title: codeforces920E Connected Components?
date: 2018-09-16 17:51:00 +0800
categories: training
tags: 图论 补图
img: https://vexoben.github.io/assets/images/Blog2/2018-10-06-codeforces920e-connected-components.png
---

题目链接:[codeforces920E Connected Components?][1]

### **题意**

给定一个n个点的图的补图，补图中存在m条边，求出原图中联通块的个数。

n, m <= 200000

### **题解**

补图bfs。

将所有的点加入到一个set中，重复下面的操作：

1、取出set中的第一个元素，加入queue中，然后在set中删除它。

2、取出队首，检查set中每一个点是否和它有边。如有边，将他们放到同一个联通块中，从set中删除这个点，并在queue中加入它。

3、处理到队列为空。这些入队的点位于同一个联通块中。

这样的复杂度是O((n + m) logn)的。因为检查是否有边的次数不会超过O(n + m)

如果用链表和哈希表代替掉set和map，复杂度可以做到O(n + m)

```cpp
#include<bits/stdc++.h>
#define LL long long
using namespace std;
const int N = 2e5 + 10;

int n, m, ans[N];
map<LL, int> Map;
set<int> Set;
queue<int> Q;

int bfs() {
	int x = *Set.begin(), res = 1;
	Set.erase(x);
	Q.push(x);
	set<int>::iterator it, tmp;
	while (!Q.empty()) {
		int x = Q.front(); Q.pop();
		for (it = Set.begin(); it != Set.end();) {
			if (!Map[1LL * x * n + (*it)]) {
				++res;
				Q.push(*it);
				tmp = it;
				it++;
				Set.erase(tmp);
			}
			else ++it;
		}
	}
	return res; 
}

int main() {
	scanf("%d%d", &n, &m);
	for (int i = 1; i <= m; ++i) {
		int x, y;
		scanf("%d%d", &x, &y);
		Map[1LL * x * n + y] = 1;
		Map[1LL * y * n + x] = 1;
	}
	for (int i = 1; i <= n; ++i)
		Set.insert(i);
	int cnt = 0;
	while (Set.size())
		ans[++cnt] = bfs();
	sort(ans + 1, ans + cnt + 1);
	printf("%d\n", cnt);
	for (int i = 1; i <= cnt; ++i)
		printf("%d ", ans[i]);
	puts("");
	return 0;
}
```

[1]: http://codeforces.com/contest/920/problem/E