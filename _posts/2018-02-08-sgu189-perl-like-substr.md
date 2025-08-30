---
layout: post
title: SGU189 Perl-like Substr
date: 2018-02-08 16:14:34 +0800
categories: training
tags: 模拟
img: https://vexoben.github.io/assets/images/Blog/2018-02-08-sgu189-perl-like-substr.JPG
---

题目链接：[http://acm.sgu.ru/problem.php?contest=0&problem=189][1]

## **题意**

见[http://blog.csdn.net/qq_20118433/article/details/42618825][2]

## **题解**

一道比杀蚂蚁清真到不知道哪去的字符串模拟题。

一开始读入字符串的时候整行读入，然后扫一遍去掉空格、分号和换行符。

代码中最核心的是string Substring(string &sx,int l,int r)函数，它将sx字符串中substr函数的串值算出来。l、r分别是substr函数左右两个括号的位置。参数表中的逗号，变量名前的'$'是绝佳的标识符。将变量名分离出来之后根据逗号个数分类讨论即可。 用string编写的话，substr可以直接支持不含负数的情况。

## **几个细节**

没有出现过的变量名值是“”（空串），赋值的时候小心在已有变量中找不到RE

字符串的值中允许有空格，在去空格的时候要判断一下是否在引号内

谨慎处理好第一行的换行符，你可能需要在第一行使用scanf或cin。scanf可以很好地处理这个问题。

未出现过的空串可以赋值给各种串，小心RE

好好读题目

## **代码**

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=1e4+10;

int n,m,tot;
struct String {
	string nam,text;
}s[N];

void Remove_Space(string &s) {        // 移除s中的空格、分号和换行符
	string sx="";
	int t=0;
	for (int i=0;i<s.length();i++) {
		if (s[i]==';') break;
		if (s[i]=='\"') t^=1;
		if (t||s[i]!=' ') sx+=s[i];
	}
	s=sx;
}

int Get_Num(string &s,int l,int r) {  // 读取出s[l,r]中的数字 
	int x=0,f=1;
	for (int i=l;i<=r;i++) {
		if (s[i]=='-') f=-1;
		else x=x*10+s[i]-'0';
	}
	return x*f;
}

int Get_Order(string &sx) {           // 在s[i]中查找名字为sx的字符串的下标
	for (int i=1;i<=tot;i++) {
		if (s[i].nam==sx) return i;
	}
	return 0;
}

string Substring(string &sx,int l,int r) { // 返回sx中substr函数对应的字符串，l、r分别为左右括号的下标 
	int a[4],cnt=0;
	for (int i=l;i<=r;i++) {
		if (sx[i]==',') a[++cnt]=i;
	}
	int l1=l+2,r1=a[1]-1,len1=r1-l1+1;
	string sy=sx.substr(l1,len1);
	int num=Get_Order(sy);
	if (cnt==1) {
		int beg=Get_Num(sx,a[1]+1,r-1);
		if (beg<0) beg=s[num].text.length()+beg;
		return s[num].text.substr(beg);
	}
	else if (cnt==2) {
		int beg=Get_Num(sx,a[1]+1,a[2]-1);
		if (beg<0) beg=s[num].text.length()+beg;
		int len=Get_Num(sx,a[2]+1,r-1);
		if (len<0) len=s[num].text.length()-beg+len;
		return s[num].text.substr(beg,len);
	}
}

void New_String(string &sx) { // 读入一个新字符串
	int a=sx.find('=');
	s[tot+1].nam=sx.substr(1,a-1);
	if (sx[a+1]=='$') {
		string ss=sx.substr(a+2,sx.length()-a-2);
		int num=Get_Order(ss);
		s[tot+1].text=s[num].text;
	}
	else if (sx[a+1]=='\"') {
		s[tot+1].text=sx.substr(a+2,sx.length()-a-3);
	}
	else if (sx[a+1]=='s') {
		s[tot+1].text=Substring(sx,a+7,sx.length()-1);
	}
	tot++;
}

string Equal(string &sx) { // 求出表达式对应的字符串值
	if (sx[0]=='$') {
		string ss=sx.substr(1,sx.length()-1);
		int num=Get_Order(ss);
		return s[num].text;
	}
	else if (sx[0]=='\"') {
		string ss="";
		for (int i=1;i<=sx.length()-2;i++)	ss+=sx[i];
		return ss;
	}
	else if (sx[0]=='s') {
		return Substring(sx,6,sx.length()-1);
	}
}

void Replace(string &sx,string &ss,int l,int r) { 将sx[l,r]中的内容替换成ss
	int a[4],cnt=0;
	for (int i=l;i<=r;i++) {
		if (sx[i]==',') a[++cnt]=i;
	}
	int l1=l+2,r1=a[1]-1,len1=r1-l1+1;
	string sy=sx.substr(l1,len1);
	int num=Get_Order(sy);
	if (cnt==1) {
		int beg=Get_Num(sx,a[1]+1,r-1);
		if (beg<0) beg=s[num].text.length()+beg;
		s[num].text.erase(beg);
		s[num].text=s[num].text.replace(beg,s[num].text.length()-beg,ss);
	}
	else if (cnt==2) {
		int beg=Get_Num(sx,a[1]+1,a[2]-1);
		if (beg<0) beg=s[num].text.length()+beg;
		int len=Get_Num(sx,a[2]+1,r-1);
		if (len<0) len=s[num].text.length()-beg+len;
		s[num].text=s[num].text.replace(beg,len,ss);
	}
}

int main() {
	scanf("%d%d\n",&n,&m); tot=0;
	s[0].text="";
	for (int i=1;i<=n+m;i++) {
		string sx; getline(cin,sx);
		Remove_Space(sx);
		if (sx[0]=='$') {
			int a=sx.find('=');
			string ss=sx.substr(1,a-1);
			int num=Get_Order(ss);
			if (num) {
				string sr=sx.substr(a+1,sx.length()-a-1);
				s[num].text=Equal(sr);
			}
			else New_String(sx);
		}
		
		else if (sx[0]=='p') {
			string sy=sx.substr(6,sx.length()-7);
			cout<<Equal(sy)<<endl;
		}
		else if (sx[0]=='s') {
			int a=sx.find('=');
			string ss=sx.substr(a+1,sx.length()-a-1);
			string sy=Equal(ss);
			Replace(sx,sy,6,a-1);
		}
	}
}
```

## **几组数据**

调试的时候用到的几个数据（非常弱），把我的程序粘下来就知道输出了


```
0 5
$ad = "0123456789"; 
$bxx = "abcdefghigklmn"; 
$c =   $ad;
$xjj = substr($ad,2);
$pqq = substr($ad,2,4)

0 4
$a = "0123456789";
print("abs");
print(substr($a, -5)); 
print(substr($a, -2, -1));

0 4
$a = "0123456789";
$b = "abcdefghigklmn";
substr($b, 0, 1) = $a;
print(substr($b,0));

0 4
$a = "0123456789";
$b = "abcdefghigklmn";
substr($b, -3,-1) = "233";
print(substr($b,0));

0 7
$a = "123456"
$a = $c
print($a)
print($c)
$b = "adsfdfsa"
$a = substr($b,-3,-1)
print($a)

0 2
$ab = "45 6"
print($ab)
```

[1]: http://acm.sgu.ru/problem.php?contest=0&problem=189
[2]: http://blog.csdn.net/qq_20118433/article/details/42618825