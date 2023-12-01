---
title: 后缀dp
date: 2023-11-21 16:50:10
tags: 算法
categories:
- 算法
- dp
---







# 本文介绍一种dp的设计状态（后缀）



## 处理问题

一个序列中 “`任意一段区间` “满足某个性质，这个时候应该想到 “ 后缀dp”





# [Qu'est-ce Que C'est?](https://ac.nowcoder.com/acm/contest/57358/J)



## 题意

给定长度为 n 的数列 ，要求每个数都在 [ − m , m ]  范围，且任意长度大于等于 2 的区间和都大于等于 0 ，问方案数。

`1 ≤ n , m ≤ 5e3 `



## 思路

1. 后缀dp秒杀，设`dp[i][j]`表示第`i`个数字选完后，所有后缀中`最小值`是`j`的合法方案数目（**j可以是负数**）
2. 如果以i为结尾时，所有后缀最小值满足>=0,那么`以i为结尾的方案`一定是合法方案
3. 计算`dp[i][j]`的时候，利用的i-1层的信息，假设i-1层是正确的，那么第i层肯定是正确的，那么后面都正确。
4. 遍历完所有的i，说明一个很重要的信息：以i为结尾的**任意**后缀都满足“`任意长度>=2的区间满足sum>=0`”,那么任意一段长度大于等于2的区间都满足！！



## 代码

```cpp
#include<iostream>
#include<cstring>
#include<algorithm>
using namespace std;
const int N=1e5+5; 
typedef long long ll; 
const ll mol=998244353;
const int P=5001;
ll a[N],n,m;
ll dp[2][N];//表示前i个，最小后缀和为j的方案数目 
ll dpsum[2][N];//dp的前缀和数组 
int main()
{
	cin>>n>>m;
    
    //初始化dp[1][j],注意偏移量
	for(int j=-m;j<=m;j++)
	dp[1&1][j+P]=1,dpsum[1&1][j+P]=(dpsum[1&1][j-1+P]+dp[1&1][j+P])%mol;
    
	for(int i=2;i<=n;i++){
		for(int j=-m;j<=m;j++){//枚举这一层的最小后缀
			if(j>=0){
				//dp[i&1][j+P]=dp[i-1&1][j-m+P]+...+dp[i-1&1][j+m+P]; 
                
                //当前为j，上一层可以是[j-m,j+m],注意边界
				dp[i&1][j+P]=dpsum[(i-1)&1][min(m,j+m)+P]-dpsum[(i-1)&1][j-m-1+P];
				dp[i&1][j+P]=(dp[i&1][j+P]%mol+mol)%mol;
			}
			else{
				//dp[i&1][j+P]=dp[i-1&1][-j+P]+...+dp[i-1&1][-j+m+P];
                
                //当前为j，上一层可以是[-j,-j+m]，否则此时出现区间<0的情况，注意边界
				dp[i&1][j+P]=dpsum[i-1&1][min(-j+m,m)+P]-dpsum[(i-1)&1][-j-1+P];
				dp[i&1][j+P]=(dp[i&1][j+P]%mol+mol)%mol;
			}
		}
        //前缀和
		for(int j=-m;j<=m;j++){
			dpsum[i&1][j+P]=(dpsum[i&1][j-1+P]+dp[i&1][j+P])%mol;
		}
	}
	cout<<dpsum[n&1][m+P];
}
```



## 注意

1. 当前层`j>=0`，上一层可以是负数（这个时候本位置一定选正数，可以满足题意），但是`j<0`时，上一层必须是正数，而且` k>=-j`,否则，此时一定存在某个以i为结尾的后缀(len>=2)，使得这个后缀的sum<0，不符合题意
2. **边界问题**：为什么当前层为`j(j>=0)`的时候,上一层`最大`从`k=m`转移**而不是**`k=m+j`呢？
	因为以i结尾的任意一个后缀的最小值不可能超过m









# [P2592 [ZJOI2008] 生日聚会](https://www.luogu.com.cn/problem/P2592)

## 题意：

给定n，m，k。
n，m表示有n个男孩和m个女孩，对于任意连续的一段，男孩与女孩的数目之差不超过k，求方案数。
`n , m ≤ 150，k ≤ 20`



## 思路

1. 设`dp[i][j][w][t]`表示选i个男孩，j个女孩，且`此时`任意后缀中男孩比女孩`最多`多w个，女孩比男孩`最多`多t个的方案数目
2. 显然，增加一个男孩或者增加一个女孩，如果上一个状态合法，那此轮状态也一定合法。
3. 具体转移见代码



## 代码

```cpp
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;
const int N=152;
const int M=22;
const ll mol=12345678;
ll dp[N][N][M][M];
int n,m,k;

int main()
{
	cin>>n>>m>>k;
	dp[0][0][0][0]=1;
	for(int i=0;i<=n;i++){
		for(int j=0;j<=m;j++){
			for(int w=0;w<=k;w++){
				for(int t=0;t<=k;t++){
					if(dp[i][j][w][t]){
						ll tem=dp[i][j][w][t];
						(dp[i+1][j][w+1][max(0,t-1)]+=tem)%=mol;
						(dp[i][j+1][max(0,w-1)][t+1]+=tem)%=mol;
					}
				}
			}
		}
	}
	ll ans=0;
	for(int w=0;w<=k;w++){
		for(int t=0;t<=k;t++){
			ans+=dp[n][m][w][t];
			ans%=mol;
		}
	}
	cout<<ans<<endl;
}
```



## 注意

> **是由当前层转移到下一层，并不是由前一层转移到当前层**，原因如下：
>
> 1.若由前一层转移到当前层,那么其中肯定有这样的转移方程：`dp[i][j][w][t]+=dp[i-1][j][max(0,w-1)][t+1]`，这是错的，如何确定选（i-1，j)这个状态下，女生一定比男生多t+1个呢？？如果j为0呢？那是不是可以不需要多t+1个，多t个好像也可以，此时t为0。
>
> 2.理论上从前一层转移到当前层也是可以的，只不过需要特判？？这个作者没有严格证明，可以自己尝试





# 总结

通过上面两个题，发现，任意一段需要满足某一性质，则需要后缀dp。

dp定义：在`某个状态`下，`任意后缀`的`最`大属性（或者最小属性）为k   的方案数



## 注意

> 一定是在某个状态下
>
> 任意后缀
>
> 最大（最小）属性为k