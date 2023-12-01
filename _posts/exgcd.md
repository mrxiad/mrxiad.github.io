---
title: exgcd
date: 2023-11-30 22:34:50
tags: 算法
categories: 
- 算法
- 数论
---



# exgcd反向应用



## [ICPC2022杭州A. Modulo Ruins the Legend](https://codeforces.com/gym/104090/problem/A)

### 题意

给一个n和m，给一个数列a，求一个等差数列b，满足$\sum(a_i+b_i)$模m最小，求s和d，s>=0,d>=0

其中$b_i=s+d*i$.



### 思路

1. 假设sum=$\sum(a_i)$,则可以看成求$sum+n*s+(n+1)*n/2*d=k*m+ans$

	其中ans就是我们要求的最小值

2. 先`sum%=m`，设`A=n,B=(n+1)*n/2`，则利用exgcd化成$A*x+B*y+sum=k*m+ans$

3. 设$A*x+B*y=k_1*gcd(A,B)$,则化成$k_1*gcd(A,B)-k_2*m=ans-sum$

4. 容易知道$ans\in[0,m-1]$,则$ans-sum\in[-sum,-sum+m-1]$,在确定最小的ans后，解出k1

5. 解出k1，反解出x和y，再利用m求解即可，具体看代码



### 代码

```cpp
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;
const int N=1e3+5;
const int M=1e4+5;
ll n,m;
ll sum;
ll exgcd(ll a,ll b,__int128 &x,__int128 &y)
{
    if(!b)
    {
        x=1,y=0;
        return a;
    }
    ll d=exgcd(b,a%b,y,x);
    y-=(a/b*x);
    return d;
}
int main()
{
	ll n,m;
	cin>>n>>m;
	for(int i=1;i<=n;i++){
		ll x;
		cin>>x;
		sum+=x;
	}
	sum%=m;
	ll A=n,B=(n+1)*n/2;
	ll g=__gcd(A,B);
	
	//k1*g+sum=k2*m+ans;
	//k1*g-k2*m=ans-sum=k3*gcd(g,m)
	
	__int128 x,y;
	ll t=exgcd(g,m,x,y);//t>0
	ll ans=sum%t;
	x*=(ans-sum)/t;//k1
	
	ll k1=x;
	//A*x+B*y=k1g
	t=exgcd(A,B,x,y);//t>0
	x*=k1;
	y*=k1;
	
	//A*x+B*y-k*m=ans-sum
	x=(x%m+m)%m;
	y=(y%m+m)%m;
	
	cout<<ans<<endl;
	cout<<(ll)x<<" "<<(ll)y<<endl;
	return 0;
}
```



### 注意

有几点特别重要！！！

1. 我们注意到，题目要求s和d都为`非负数`（代码中是x和y），但是我们在求解的过程中，没有管k1，k2的正负，更没有管x和y的正负
2. 而且x和y最后都利用m缩小到\[0,m-1]的范围内，这好像也没什么逻辑
3. 关于第2点，注意到，**换元k1相当于**$A*x+B*y-k_2*m=ans-sum$，第一次求出的x和y一定是方程的解，但是正负不确定。但是，我可以使$x=x+k3*m,y=y+k4*m$,再利用k2削减掉多出来的m，此时方程仍然成立，也就保证了正确性
4. 注意exgcd只可以处理$A*x+B*y=ans-sum$,不可以处理$A*x+B*y-k*m=ans-sum$,不可以跳步！！



