---
title: 悬线法dp(玉蟾宫+ICPC银川K)
date: 2023-11-07 15:52:40
tags: 算法
categories: 
- 算法
- dp
---



# 玉蟾宫

[洛谷P4147玉蟾宫](https://www.luogu.com.cn/problem/P4147)

给一个矩阵，一些点有障碍物，求最大子矩阵



## 做法

`悬线法dp`，第一次听说。

## 步骤

（1）结论：答案一定是一个矩形（废话。。。）
（2）最大矩形一定是：由其中`某个点`，`先`向上扩展到最大，`然后`再分别向左、向右走到最远。
（3）由于（2）的结论对所有点这样操作，一定可以找到最大矩形
  (4)  注意`先初始化h，L，R`，然后在`h=1的时候预处理L,R`，然后`再更新h，同时更新L，R`，并且统计答案   



## 具体代码：

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=1e3+5;
char a[N][N];
int n,m;
int h[N][N];//每个点往上延申的最大长度
int L[N][N];//在h保证的情况下，每个点往左走最远到哪
int R[N][N];//在h保证的情况下，每个点往右走最远到哪

int ans=0; 
int main()
{
	cin>>n>>m;
	for(int i=1;i<=n;i++){
		for(int j=1;j<=m;j++){
			cin>>a[i][j];
			if(a[i][j]=='F')h[i][j]=1;//自己
			L[i][j]=R[i][j]=j;//自己 
		}
	}
	
	//在h=1的情况下处理L和R 
	for(int i=1;i<=n;i++){
		//处理L 
		for(int j=1;j<=m;j++){
			if(a[i][j]=='F'&&a[i][j-1]=='F'){
				L[i][j]=L[i][j-1];
			} 
		} 
		//处理R 
		for(int j=m;j>=1;j--){
			if(a[i][j]=='F'&&a[i][j+1]=='F'){
				R[i][j]=R[i][j+1];
			} 
		}
	}
	for(int i=1;i<=n;i++){
		for(int j=1;j<=m;j++){
			if(a[i][j]=='F'&&a[i-1][j]=='F'){
				h[i][j]=h[i-1][j]+1;//h数组扩展 
				
				//L、R向内收缩 
				L[i][j]=max(L[i][j],L[i-1][j]);
				R[i][j]=min(R[i][j],R[i-1][j]); 
			}
			
			if(a[i][j]=='F'){
				ans=max(ans,h[i][j]*(R[i][j]-L[i][j]+1));
			}
		}
	} 
	cout<<ans*3<<endl;
	return 0;
} 
```









# ICPC银川

[Largest Common Submatrix](https://codeforces.com/gym/104021/problem/K)

给定一个矩阵A和一个矩阵B，求最大子矩阵，满足最大子矩阵同时是A的最大子矩阵和B的最大子矩阵

## 做法

悬线法dp，注意转移条件，以及`边界问题`！！！



## 代码

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=1e3+5;
int a[N][N];
int b[N][N];
int n,m;
int h[N][N];//每个点往上延申的最大长度
int L[N][N];//在h保证的情况下，每个点往左走最远到哪
int R[N][N];//在h保证的情况下，每个点往右走最远到哪
pair<int,int>pos[N*N];
int ans=0;
int main() {
	ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
	
	cin>>n>>m;
	for(int i=1; i<=n; i++) {
		for(int j=1; j<=m; j++) {
			cin>>a[i][j];
			h[i][j]=1;
			L[i][j]=R[i][j]=j;//自己
		}
	}

	for(int i=1; i<=n; i++) {
		for(int j=1; j<=m; j++) {
			cin>>b[i][j];
			pos[b[i][j]]= {i,j};
		}
	}


	//在h=1的情况下处理L和R
	for(int i=1; i<=n; i++) {
		//处理L
		for(int j=2; j<=m; j++) {
			int x=a[i][j];//a中当前数字
			int y=a[i][j-1];//a中当前数字的左边那个数字

			
			//当前数字在b数组中的位置
			int bx=pos[x].first;
			int by=pos[x].second;

			//如果可以向左边扩展
			if(b[bx][by-1]==y) {
				L[i][j]=L[i][j-1];
			}

		}
		//处理R
		for(int j=m-1; j>=1; j--) {
			int x=a[i][j];//a中当前数字
			int y=a[i][j+1];//a中当前数字的右边那个数字


			//当前数字在b数组中的位置
			int bx=pos[x].first;
			int by=pos[x].second;

			//如果可以向右边扩展
			if(b[bx][by+1]==y) {
				R[i][j]=R[i][j+1];
			}
		}
	}

	//更新h,L,R ，统计答案
	int ans=0;
	for(int i=1; i<=n; i++) {
		for(int j=1; j<=m; j++) {
			int x=a[i][j];//a中当前数字
			int y=a[i-1][j];//a中当前数字的上边那个数字


			//当前数字在b数组中的位置
			int bx=pos[a[i][j]].first;
			int by=pos[a[i][j]].second;

			//如果可以向上边扩展
			if(y&&b[bx-1][by]==y) {
				h[i][j]=h[i-1][j]+1;
				L[i][j]=max(L[i][j],L[i-1][j]);
				R[i][j]=min(R[i][j],R[i-1][j]);
			}
			ans=max(ans,h[i][j]*(R[i][j]-L[i][j]+1));
		}
	}
	cout<<ans<<endl;
	return 0;
}
```



