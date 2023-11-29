---
title: 树形dp
date: 2023-11-29 21:16:30
tags: 算法
categories: dp
---





# 前言

本文介绍树形dp比较难的状态设计和考虑方式





# 边贡献型

贡献放置到边上



## [P3177 [HAOI2015] 树上染色](https://www.luogu.com.cn/problem/P3177)

## 题意

有一棵点数为 *n* 的树，树边有边权。给你一个在 0∼*n* 之内的正整数 *k* ，你要在这棵树中选择 *k* 个点，将其染成黑色，并将其他的 *n*−*k* 个点染成白色。将所有点染色后，你会获得**黑点两两之间**的距离加上**白点两两之间**的距离的和的收益。问收益最大值是多少。



## 思路

1. 设dp\[u][j]表示以u为根的子树中，选j个黑色点，获取到的最大价值（这个价值只是在子树**内部**的价值，子树外部的价值不算），答案是dp\[root][k]
2. 可能现在还不理解1，那么看转移方程：
	`dp[u][j]=max(dp[u][j],dp[u][j-k]+dp[v][k]+val)`,这个val是v子树中选k个点，u和v**直连边**可以贡献的价值，这个价值很好算，看代码即可。
3. 代码中计算val没有用到j，只是用到了k，那可能会有疑问，为什么要这样设计j这一维度呢？？因为可以转移，背下来即可。
4. **转移的正确性**:我们说v是u的儿子，但实际上，我们在进行**合并操作**，因为当利用v计算u的时候，**v并没有看成在u中**，所以是合并操作。**这样第3条就解释通了**：我们需要将u”子树“”和v”子树”合并，而合并的之前，u，v内部如何选择黑色点是毫无关系的，而且获取到的最大价值**只**在自己子树内部，而且已经被统计好。dp的含义是**内部边**的价值，所以需要j那一维度。
5. 注意代码中状态计算的方式，只有这种方式才可以保证复杂度是$O(n^2)$



## 代码

```cpp
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;
const int N=2e3+5;
const int M=N*2;
int idx;
ll h[N],e[M],ne[M],w[M];
void add(int a,int b,int c){
	e[++idx]=b;
	w[idx]=c;
	ne[idx]=h[a];
	h[a]=idx;
}

ll n,m;
ll siz[N];
ll dp[N][N];//以u为根的子树，恰好选了j个黑色点的最大收益(收益是内部边的收益)

void dfs(int u,int fa){
	static ll tem[N];//辅助数组
	siz[u]=1;
	dp[u][0]=dp[u][1]=0;
	for(int i=h[u];i;i=ne[i]){//枚举层，所有儿子在同一层，按照访问儿子的先后顺序划分层
		int v=e[i];
		if(v==fa)continue;
		dfs(v,u);//要siz[j]和dp[j][]数组
		
        //必须先初始化
		for(int i=0;i<=siz[u]+siz[v];i++){
			tem[i]=-1e9;
		}
        
        
		for(int j=0;j<=siz[u];j++){//前层子树,一共选j个点
			for(int k=0;k<=siz[v];k++){//新子树选k个
				if(j+k<=m){//只有合法才可以转移
					ll val=(ll)(k*(m-k)+(siz[v]-k)*(n-m-siz[v]+k))*w[i];  //当前情况下连接子节点的边的贡献
					tem[j+k]=max(tem[j+k],dp[u][j]+dp[v][k]+val);
				}
			}
		}	
        
		//合并两颗树
		for(int j=0;j<=siz[u]+siz[v];j++){
			dp[u][j]=tem[j];
		}
		siz[u]+=siz[v];
	}
	
}
int main()
{
	memset(dp,-0x3f,sizeof dp);
	cin>>n>>m;
	for(int i=1;i<n;i++){
		int a,b,c;
		cin>>a>>b>>c;
		add(a,b,c);
		add(b,a,c);
	}
	dfs(1,0);	
	cout<<dp[1][m]<<endl;
	return 0;
}
```





## [牛客多校Tree](https://ac.nowcoder.com/acm/contest/57360/A)

## 题意

给定一棵树，每个点选择黑、白有对应的代价，定义一棵树的收益为所有黑白点对间路径边权最大值的和，问如何选择每个点的颜色使得收益-代价最大？



## 思路

1. 因为边权最大值才可以算贡献，所以建立kruskal重构树，这样**边的贡献只会在子树内部**,所以可以用树形dp做
2. 设dp\[u][j]表示以u为根的子树中选j个黑色点的最大价值，此时价值只会统计到u子树的内部
3. 重构树是二叉堆，这样很好的诠释了“合并”的思想，而且，我们**只需要知道**合并后的那颗树的信息,因为只有这样的信息，才会被后续用到。
4. 上一题，v一旦被合并到u中，v的信息就再也用不到了，实际上我们也可以将u的信息合并到v中，但是不方便，因为我们还要计算u与它的father的边的贡献。
5. 注意代码中状态计算的方式，只有这种方式才可以保证复杂度是$O(n^2)$



## 代码

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=3e3+5;
const int M=N;
typedef long long ll;
ll n;
ll a[N];
ll cost[N];
ll dp[N][N];//以u为根的子树中，选择黑色点的个数为j的最大价值
struct A{
	ll u,v,w;
	bool operator<(const A&b)const{
		return w<b.w;
	}
}ed[M];

int siz[N];
int f[N];
int find(int x){
	if(x==f[x])return x;
	return f[x]=find(f[x]);
}

void merge(ll u,ll v,ll w){
	static ll tem[N];
	u=find(u),v=find(v);
	if(u==v)return;
	if(siz[u]>siz[v])swap(u,v);
	int n=siz[u],m=siz[v];
	for(int i=0;i<=n+m;i++)
		tem[i]=-1e9;
	for(int i=0;i<=m;i++){//枚举前层子树(v)的黑色点个数
		for(int j=0;j<=n;j++){//枚举新子树(u)的黑色点个数	
			ll cnt=(ll)i*(n-j)+(ll)(m-i)*j;
			tem[i+j]=max(tem[i+j],dp[v][i]+dp[u][j]+cnt*w);
		}
	}
	f[u]=v;
	siz[v]+=siz[u];
	for(int j=0;j<=n+m;j++)
		dp[v][j]=tem[j];
}
int main()
{
	cin>>n;
	for(int i=1;i<=n;i++){
		cin>>a[i];
		siz[i]=1;
		f[i]=i;
	}
	for(int i=1;i<=n;i++){
		cin>>cost[i];
		dp[i][0]=(a[i]==0)?0:-cost[i];
		dp[i][1]=(a[i]==1)?0:-cost[i];
	}
	for(int i=1;i<n;i++){
		ll a,b,c;
		cin>>a>>b>>c;
		ed[i]={a,b,c};
	}
	sort(ed+1,ed+n);
	for(int i=1;i<n;i++){
		ll u=ed[i].u;
		ll v=ed[i].v;
		ll w=ed[i].w;
		merge(u,v,w);
	}
	ll ans=0;
	for(int i=0;i<=n;i++)
		ans=max(ans,dp[find(1)][i]);
	cout<<ans<<endl;;
	return 0;
}
```





> 所有边的贡献必须在子树内部才可以用这种方法，尤其注意第一种，为什么要那样设计状态

