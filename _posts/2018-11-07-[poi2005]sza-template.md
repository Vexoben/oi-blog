---
layout: post
title: POI2005 SZA-template
date: 2018-11-07 17:20:04 +0800
categories: training
tags: KMP 字符串
img: https://vexoben.github.io/oi-blog/assets/images/Blog2/2018-11-07-[poi2005]sza-template.png
---

## **题意**

给定字符串 S，求出最短的字符串 T 能够覆盖 S。

覆盖的定义是：对于任意 pos <= strlen(S)， 在 S 中存在 l <= pos <= r 使得 S[l..r] = T。

加强：对于 S 的所有前缀输出这个答案。

strlen(S) <= 500000

## **题解**

因为 T 覆盖 S 的第一个字符，所以 T 是 S 的一个前缀； 同理， T 是 S 的一个后缀。

从而可能的答案是满足 "前缀匹配后缀" 的那些串。很容易用KMP求出所有可能的答案。

如果存在两个前缀 X, Y 都可以覆盖 S， 若 X 可以覆盖　Y， X 一定比 Y 优。

从而 S 的答案要么是 next[S] 的答案，要么是它本身。

怎么判断 next[S] 的答案能否覆盖 S 呢？只需要在计算答案时顺便维护一下某个前缀可以覆盖到的最远距离即可。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 1e6 + 10;
 
int n, ans[N], mat[N], nex[N];
char s[N];
 
int main() {
   scanf("%s", s + 1);
    n = strlen(s + 1);
    for (int i = 2, j = 0; i <= n; ++i) {
        while (j && s[j + 1] != s[i]) j = nex[j];
        if (s[j + 1] == s[i]) ++j;
        nex[i] = j;
    }
    for (int i = 1; i <= n; ++i) {
        if (mat[ans[nex[i]]] + ans[nex[i]] >= i) {
            ans[i] = ans[nex[i]];
        }
        else {
            ans[i] = i;
        }
        mat[ans[i]] = i;
        printf("%d\n", ans[i]);
    }
    return 0;
}
```