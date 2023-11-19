---
title: 中位数第k大二分法
date: 2023-11-19 21:47:40
tags: 算法
categories: 二分
---





# 本文介绍利用二分法求中位数的一般思路



如果一个题目要求中位数，二分答案，判定是否符合条件即可





# [D. Max Median](https://codeforces.com/contest/1486/problem/D)



## 题意

给一个n和一个k，n表示有一个数组有n个元素，求所有“区间长度大于等于k”的区间中，中位数最大的多少



## 做法

二分答案，检查是否存在一个大于等于k的区间满足即可，显然此时原数组元素`具体值`是多少已经不重要，重要的是元素的`相对大小`，设新数组b，若`a[i]>=x`,则`b[i]=1`，否则`b[i]=0`，设s为b的前缀和。



然后考虑对于每一个`i`，是否存在`j (0<=j<=i-k)`,满足`s[i]-s[j]>0`,这是一个很常见的处理方式（称之为划分集合法），具体见代码



## 代码

```cpp
#include <iostream>
using namespace std;
typedef long long ll;
const ll mol=12345678;
const int N=2e5+5;
int a[N];
int b[N];
int s[N];
int n,k;
bool check(int x){
	for(int i=1;i<=n;i++){
        //处理b数组
		if(a[i]>=x){
			b[i]=1;
		}
		else b[i]=-1;
		//前缀和数组
		s[i]=s[i-1]+b[i];
	}
	//s[i]-s[j]>0即可，i-j>=k 
	int minn=0;
	for(int i=k;i<=n;i++){
		minn=min(minn,s[i-k]);
        
        //注意是>0，想想为什么
		if(s[i]-minn>0)
			return 1;	
	}
	return 0;
}


int main() {
	cin>>n>>k;
	for(int i=1;i<=n;i++){
		cin>>a[i];
	} 
	int l=1,r=n;
	int ans=0;
	while(l<=r){
		int mid=l+r>>1;
		if(check(mid)){
			ans=mid;
			l=mid+1;
		}
		else r=mid-1;
	}
	cout<<ans<<endl;
	return 0;
}
```





# [D. Chat Program](https://codeforces.com/gym/104128/problem/D)

> icpc银牌题



## 题意

给一个长度为n的数组，问最多给`一段`区间添加等差数列后,序列第 k 大最大是多少。

等差数列首项为c， 公差为d，长度为m。



## 思路

这题虽然不是中位数，其实也差不多。都是二分答案检查是否符合即可

1. 设二分答案x，若序列中某个数`a[i]<x`，若使这个数`>=x`，则必须选一个位置作为开头，加等差数列。
2. 对于所有数操作完后，我们`只可以`选一个位置作为开头，那么要满足更多的`a[i]>=x`。很容易想到，**对于每一个小于x的位置，这个开头是一段区间**，我们差分维护即可，这样对所有数字操作完后，对差分数组求前缀和
3. 最后，选出一个满足尽可能多的`a[i]`的位置，如果此时可以满足的`a[i]>=x`还是大于等于k个，那么符合要求，否则不符合要求



## 代码

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=1e6+5;
typedef long long ll;
 
ll a[N];
ll n,m;
ll k,c,d;//c+d[i]
ll cha[N];//差分数组
bool check(ll x) { //第k大至少是x,大于等于x的数目需要大于等于k
	int num=0; //符合的数目
	for(int i=1; i<=n; i++) {
		if(a[i]<x) {
			//c+ (i-pos)*d +a[i] >= x//最右边的pos
			//pos=(c+a[i]-x+i*d)/d
 
			if(d==0) {
				if(a[i]+c>=x) {
					cha[max(1ll,i-m+1)]++;
					cha[i+1]--;
					continue;
				}
			} else {
				ll ans=(c+a[i]-x+i*d)/d;//寻找起始位置
				ans=min(ans,(ll)i);
				if(ans>=1&&ans>=i-m+1) {
					cha[max(1ll,i-m+1)]++;
					cha[ans+1]--;
				}
			}
		} else num++;
	}
	ll maxn=0;
	for(int i=1; i<=n; i++) {
		cha[i]+=cha[i-1];
		maxn=max(maxn,cha[i]);
	}
	for(int i=1; i<=n+1; i++)cha[i]=0;
	num+=maxn;
	return num>=k;
 
}
 
void solve() {
	cin>>n>>k>>m>>c>>d;
 
	for(int i=1; i<=n; i++) {
		cin>>a[i];
	}
 
	ll ans=0;
	ll begin=1,end=2e15;
	while(begin<=end) {
		ll mid=begin+end>>1;
		if(check(mid)) {
			ans=mid;
			begin=mid+1;
		} else end=mid-1;
	}
	cout<<ans<<endl;
 
}
 
int main() {
	//关闭C++和C的输入输出流的同步，提高C++的输入输出效率
	std::ios::sync_with_stdio(false);
//解除cin和cout的绑定，让它们可以独立缓冲
	std::cin.tie(0);
	std::cout.tie(0);//这一句可以不要，效果一样
	int t=1;
	//cin>>t;
	while(t--) {
		solve();
	}
	return 0;
}
```







# 总结

碰到这种题先像二分吧，中位数，第k大