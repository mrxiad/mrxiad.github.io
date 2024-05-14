---
title: FFT&NTT
date: 2024-5-6 22:34:50
tags: 算法
categories: 
- 算法
- 数论
---







# [FFT板子](https://www.luogu.com.cn/problem/P3803)

> 构造两个vector，答案存储到第三个vector中

```cpp
#include<iostream>
#include<cstring>
#include<vector>
#include<algorithm>
#include<cmath>
using namespace std;
typedef long long ll;
typedef double db;
const int N = 5e6 + 5;


//**************************************以下是辅助变量*****************************************
struct C {
	long double r, i;//real,imaginary
	C() {}
	C(long double a, long double b) { r = a, i = b; }
	C operator + (C x) { return C(r + x.r, i + x.i); }
	C operator - (C x) { return C(r - x.r, i - x.i); }
	C operator * (C x) { return C(r * x.r - i * x.i, r * x.i + i * x.r); }
}w[N], A[N], B[N];
int R[N];//蝴蝶变换
const db PI = acos(-1);//圆周率
//*********************************************************************************************


//*****************************************FFT板子*********************************************
void FFT(C a[], int n) {
	for (int i = 0; i < n; i++)
		if (i < R[i])
			swap(a[i], a[R[i]]);
	for (int t = n >> 1, d = 1; d < n; d <<= 1, t >>= 1)
		for (int i = 0; i < n; i += (d << 1))
			for (int j = 0; j < d; j++) {
				C tmp = w[t * j] * a[i + j + d];
				a[i + j + d] = a[i + j] - tmp;
				a[i + j] = a[i + j] + tmp;
			}
}
//答案存储到c中
void FFT_times(const vector <ll>& a,const vector <ll>& b, vector<ll>& c) {
	int n, d;
	for (int i = 0; i < a.size(); i++)
		A[i] = C(a[i], 0);
	for (int i = 0; i < b.size(); i++)
		B[i] = C(b[i], 0);
	for (n = 1, d = 0; n < a.size() + b.size() - 1; n <<= 1, d++);
	for (int i = 0; i < n; i++) {
		R[i] = (R[i >> 1] >> 1) | ((i & 1) << (d - 1));
		w[i] = C(cos(2 * PI * i / n), sin(2 * PI * i / n));
	}
	for (int i = a.size(); i < n; i++)
		A[i] = C(0, 0);
	for (int i = b.size(); i < n; i++)
		B[i] = C(0, 0);
	FFT(A, n), FFT(B, n);
	for (int i = 0; i < n; i++)
		A[i] = A[i] * B[i], w[i].i *= -1.0;
	FFT(A, n);
	int up = a.size() + b.size() - 1;
	c.resize(up);//重置
	for (int i = 0; i < up; i++)
		c[i] = ((ll)(A[i].r / n + 0.5));
}
//*********************************************************************************************




void solve()
{
	int n, m;
	vector<ll>va, vb, vc;
	cin >> n >> m;
	for (int i = 0; i <= n; i++) {
		ll x;
		cin >> x;
		va.push_back(x);
	}
	for (int i = 0; i <= m; i++) {
		ll x;
		cin >> x;
		vb.push_back(x);
	}
	FFT_times(va, vb, vc);
	for (int i = 0; i < n + m + 1; i++) {
		cout << vc[i] << " ";
	}
}
int main()
{
	int t = 1;
	//cin >> t;
	while (t--)
		solve();
	return 0;
}
```





# [高尔夫球](https://vjudge.net/problem/Gym-100783C)

## 题意

给n个数，m组询问，数的范围2e5。

n个数中，每个数x代表“机器人可以将球往前推x米”，m组询问，每次给一个y，问1次或者2次推球，能否推进y



## 做法

将2e5个点初始化为0，输入n个数，对应位置设为1，求y的卷积是否大于0，如果大于0，则说明可以到达，否则到不了



## 代码

```cpp
#include<iostream>
#include<cstring>
#include<vector>
#include<algorithm>
#include<cmath>
using namespace std;
typedef long long ll;
typedef double db;
const int N = 5e6;


//**************************************以下是辅助变量*****************************************
struct C {
	long double r, i;//real,imaginary
	C() {}
	C(long double a, long double b) { r = a, i = b; }
	C operator + (C x) { return C(r + x.r, i + x.i); }
	C operator - (C x) { return C(r - x.r, i - x.i); }
	C operator * (C x) { return C(r * x.r - i * x.i, r * x.i + i * x.r); }
}w[N], A[N], B[N];
int R[N];//蝴蝶变换
const db PI = acos(-1);//圆周率
//*********************************************************************************************


//*****************************************FFT板子*********************************************
void FFT(C a[], int n) {
	for (int i = 0; i < n; i++)
		if (i < R[i])
			swap(a[i], a[R[i]]);
	for (int t = n >> 1, d = 1; d < n; d <<= 1, t >>= 1)
		for (int i = 0; i < n; i += (d << 1))
			for (int j = 0; j < d; j++) {
				C tmp = w[t * j] * a[i + j + d];
				a[i + j + d] = a[i + j] - tmp;
				a[i + j] = a[i + j] + tmp;
			}
}
//答案存储到c中
void FFT_times(const vector <ll>& a,const vector <ll>& b, vector<ll>& c) {
	int n, d;
	for (int i = 0; i < a.size(); i++)
		A[i] = C(a[i], 0);
	for (int i = 0; i < b.size(); i++)
		B[i] = C(b[i], 0);
	for (n = 1, d = 0; n < a.size() + b.size() - 1; n <<= 1, d++);
	for (int i = 0; i < n; i++) {
		R[i] = (R[i >> 1] >> 1) | ((i & 1) << (d - 1));
		w[i] = C(cos(2 * PI * i / n), sin(2 * PI * i / n));
	}
	for (int i = a.size(); i < n; i++)
		A[i] = C(0, 0);
	for (int i = b.size(); i < n; i++)
		B[i] = C(0, 0);
	FFT(A, n), FFT(B, n);
	for (int i = 0; i < n; i++)
		A[i] = A[i] * B[i], w[i].i *= -1.0;
	FFT(A, n);
	int up = a.size() + b.size() - 1;
	c.resize(up);//重置
	for (int i = 0; i < up; i++)
		c[i] = ((ll)(A[i].r / n + 0.5));
}
//*********************************************************************************************




void solve()
{
	ll n, m;
	cin >> n;
	const int maxn = 2e5+1;
	vector<ll>va(maxn, 0);
	va[0]++;
	for (int i = 1; i <= n; i++) {
		ll x;
		cin >> x;
		va[x]++;
	}
	vector<ll>vb = va;
	vector<ll>vc;

	FFT_times(va, vb, vc);
	cin >> m;
	ll ans = 0;
	while (m--) {
		ll x;
		cin >> x;
		if (vc[x])ans++;
	}
	cout << ans << endl;
}
int main()
{
	int t = 1;
	//cin >> t;
	while (t--)
		solve();
	return 0;
}
```







# [Pass the Ball! （2021icpc澳门）](https://www.luogu.com.cn/problem/P9660?contestId=134300)

## 题意

给俩数，n，q。分别表示n个人，q组询问。

每个人会有一个指向，表示他会将手中的球给指向的人。初始球的编号和人的编号都为i

n个人会形成若干个环。对于每一个询问，给定一个x，表示每个人将球给旁边的人，给x次。求$\sum i*b[i]$，i表示人的编号，$b[i]$表示拿到的球的编号



## 思路

一眼卷积，对于每一个圈单独考虑，假设大小为size，设$p=x\%size$，则求$\sum circle[i]*circle[(i+p)\%size]$

这里有**两个技巧**

1. 对于$\%size$操作，将第二个数组扩大到原来的两倍即可，这样就可以避免掉取模操作，原式改为$\sum circle[i]*circle[i+p]$
2. 反转第一个数组，则原式改为$\sum circle[siz-1-i]*circle[p+i]$ 

> 这里将siz-1-i换元为j，得到$f(p)=\sum circle[j]*circle[(size-1+p)-j] (p为交换次数)$

对这两个数组求完卷积，操作p次的答案最终为$ans[siz-1+p]$,然后对于**所有长度相同的圈**合并即可



## 代码

```cpp
#include<iostream>
#include<cstring>
#include<vector>
#include<algorithm>
#include<cmath>
using namespace std;
typedef long long ll;
typedef double db;
const int N = 3e6;


//**************************************以下是辅助变量*****************************************
struct C {
	long double r, i;//real,imaginary
	C() {}
	C(long double a, long double b) { r = a, i = b; }
	C operator + (C x) { return C(r + x.r, i + x.i); }
	C operator - (C x) { return C(r - x.r, i - x.i); }
	C operator * (C x) { return C(r * x.r - i * x.i, r * x.i + i * x.r); }
}w[N], A[N], B[N];
int R[N];//蝴蝶变换
const db PI = acos(-1);//圆周率
//*********************************************************************************************


//*****************************************FFT板子*********************************************
void FFT(C a[], int n) {
	for (int i = 0; i < n; i++)
		if (i < R[i])
			swap(a[i], a[R[i]]);
	for (int t = n >> 1, d = 1; d < n; d <<= 1, t >>= 1)
		for (int i = 0; i < n; i += (d << 1))
			for (int j = 0; j < d; j++) {
				C tmp = w[t * j] * a[i + j + d];
				a[i + j + d] = a[i + j] - tmp;
				a[i + j] = a[i + j] + tmp;
			}
}
//答案存储到c中
void FFT_times(const vector <ll>& a,const vector <ll>& b, vector<ll>& c) {
	int n, d;
	for (int i = 0; i < a.size(); i++)
		A[i] = C(a[i], 0);
	for (int i = 0; i < b.size(); i++)
		B[i] = C(b[i], 0);
	for (n = 1, d = 0; n < a.size() + b.size() - 1; n <<= 1, d++);
	for (int i = 0; i < n; i++) {
		R[i] = (R[i >> 1] >> 1) | ((i & 1) << (d - 1));
		w[i] = C(cos(2 * PI * i / n), sin(2 * PI * i / n));
	}
	for (int i = a.size(); i < n; i++)
		A[i] = C(0, 0);
	for (int i = b.size(); i < n; i++)
		B[i] = C(0, 0);
	FFT(A, n), FFT(B, n);
	for (int i = 0; i < n; i++)
		A[i] = A[i] * B[i], w[i].i *= -1.0;
	FFT(A, n);
	int up = a.size() + b.size() - 1;
	c.resize(up);//重置
	for (int i = 0; i < up; i++)
		c[i] = ((ll)(A[i].r / n + 0.5));
}
//*********************************************************************************************

const int MAXN = 1e5 + 2;

ll n, q;
ll p[MAXN];
bool biaoji[MAXN];
vector<ll>circle[MAXN];//第i个圈的人
vector<ll>cnt[N];//存储第i个圈的贡献
void solve()
{
	int numCircle = 0;
	cin >> n >> q;
	for (int i = 1; i <= n; i++) {
		cin >> p[i];//指向
	}
	for (int i = 1; i <= n; i++) {
		if (biaoji[i])continue;
		++numCircle;
		int now = i;
		while (!biaoji[now]) {
			biaoji[now] = 1;
			circle[numCircle].push_back(now);
			now = p[now];
		}
		//做一遍fft
		vector<ll>tmp = circle[numCircle];
		tmp.insert(tmp.end(), circle[numCircle].begin(), circle[numCircle].end());//扩大到两倍
		reverse(circle[numCircle].begin(), circle[numCircle].end());//反转(这是fft的一个技巧)
		FFT_times(circle[numCircle], tmp, cnt[numCircle]);//将贡献累计到cnt中
	}

	//此时一共有numCircle个圈,并且贡献都存储到了cnt中,接下来要将(等长度)的cnt汇总

	memset(p, 0, sizeof p);
	vector<ll>si(numCircle+1);//记录每一种圈的原始圈长度
	int now = 0;
	for (int i = 1; i <= numCircle; i++) {
		int siz = circle[i].size();
		if (!p[siz]) {
			cnt[++now] = cnt[i];
			si[now] = siz;
			p[siz] = now;//记录这个siz对应的cnt编号
		}
		else {
			int last = p[siz];
			for (int j = 0; j < cnt[i].size(); j++) {
				cnt[last][j] += cnt[i][j];//累加贡献
			}
		}
	}
	//此时有now种圈，now不超过根号n个

	while (q--) {
		ll z;
		cin >> z;
		ll ans = 0;
		for (int i = 1; i <= now; i++) {//遍历now种圈
			int siz = si[i];
			int p = z%siz;
			ans += cnt[i][siz - 1 + p];
		}
		cout << ans << endl;
	}

}
int main()
{
	int t = 1;
	//cin >> t;
	while (t--)
		solve();
	return 0;
}
```









# [浮点数FFT](https://www.luogu.com.cn/problem/P3338)

## 题意

给出 $n$ 个数 $q_1,q_2, \dots q_n$，定义

$$F_j~=~\sum_{i = 1}^{j - 1} \frac{q_i \times q_j}{(i - j)^2}~-~\sum_{i = j + 1}^{n} \frac{q_i \times q_j}{(i - j)^2}$$

$$E_i~=~\frac{F_i}{q_i}$$

对 $1 \leq i \leq n$，求 $E_i$ 的值。

对于 $100\%$ 的数据，$1 \leq n \leq 10^5$，$0 < q_i < 10^9$。



## 思路

推式子，化成两个卷积，即可，注意第二个卷积要换元



## 代码

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
#include<cstdio>
#include<cmath>
using namespace std;
typedef long long ll;
typedef double db;

const int N = 5e6 + 5;
//**************************************以下是辅助变量*****************************************
struct C {
	long double r, i;//real,imaginary
	C() {}
	C(long double a, long double b) { r = a, i = b; }
	C operator + (C x) { return C(r + x.r, i + x.i); }
	C operator - (C x) { return C(r - x.r, i - x.i); }
	C operator * (C x) { return C(r * x.r - i * x.i, r * x.i + i * x.r); }
}w[N], A[N], B[N];
int R[N];//蝴蝶变换
const db PI = acos(-1);//圆周率
//*********************************************************************************************


//*****************************************FFT板子*********************************************
void FFT(C a[], int n) {
	for (int i = 0; i < n; i++)
		if (i < R[i])
			swap(a[i], a[R[i]]);
	for (int t = n >> 1, d = 1; d < n; d <<= 1, t >>= 1)
		for (int i = 0; i < n; i += (d << 1))
			for (int j = 0; j < d; j++) {
				C tmp = w[t * j] * a[i + j + d];
				a[i + j + d] = a[i + j] - tmp;
				a[i + j] = a[i + j] + tmp;
			}
}
//答案存储到c中
void FFT_times(const vector <db>& a, const vector <db>& b, vector<db>& c) {
	int n, d;
	for (int i = 0; i < a.size(); i++)
		A[i] = C(a[i], 0);
	for (int i = 0; i < b.size(); i++)
		B[i] = C(b[i], 0);
	for (n = 1, d = 0; n < a.size() + b.size() - 1; n <<= 1, d++);
	for (int i = 0; i < n; i++) {
		R[i] = (R[i >> 1] >> 1) | ((i & 1) << (d - 1));
		w[i] = C(cos(2 * PI * i / n), sin(2 * PI * i / n));
	}
	for (int i = a.size(); i < n; i++)
		A[i] = C(0, 0);
	for (int i = b.size(); i < n; i++)
		B[i] = C(0, 0);
	FFT(A, n), FFT(B, n);
	for (int i = 0; i < n; i++)
		A[i] = A[i] * B[i], w[i].i *= -1.0;
	FFT(A, n);
	int up = a.size() + b.size() - 1;
	c.resize(up);//重置
	for (int i = 0; i < up; i++)
		c[i] = (A[i].r / n);
}
//*********************************************************************************************



int main()
{
	int n;
	cin >> n;
	vector<db>va(n + 1);//q(i)
	for (int i = 1; i <= n; i++) {
		cin >> va[i];
	}
	vector<db>vb(n + 1);//g(i)=1/i^2
	for (int i = 1; i <= n; i++) {
		vb[i] = (double)1.0 / i / i;
	}
	//先计算前面的,答案保存在vc1[j]中
	vector<db>vc1;
	FFT_times(va, vb, vc1);

	//反转q数组
	reverse(va.begin(), va.end());
	vector<db>vc2;
	//在计算后面的,答案保存在vc2[n-j]中
	FFT_times(va, vb, vc2);

	for (int j = 1; j <= n; j++) {
		printf("%.3lf\n", vc1[j] - vc2[n - j]);
	}
	return 0;
}
```





# [字符串错位匹配](https://codeforces.com/problemset/problem/528/D)









# [NTT板子](https://www.luogu.com.cn/problem/P3803)

> 这个板子可能有问题，慎用！！！

```cpp
#include<cstdio>
#include<vector>
#include<iostream>
#include<algorithm>
using namespace std;
typedef long long ll;


//*************************NTT**************************
const int MAXN = 3 * 1e6 + 10;
const ll  P = 998244353, G = 3, Gi = 332748118;
ll A[MAXN], B[MAXN],R[MAXN];//辅助数组

ll qmi(ll a, ll k) {//快速幂
	ll ans = 1;
	while (k) {
		if (k & 1) ans = (ans * a) % P;
		a = (a * a) % P;
		k >>= 1;
	}
	return ans % P;
}

//n为数组长度,type=1为正变换,-1为逆变换
void NTT(ll A[], int n,int type) {
	for (int i = 0; i < n; i++)
		if (i < R[i]) 
			swap(A[i], A[R[i]]);

	for (int mid = 1; mid < n; mid <<= 1) {
		ll Wn = qmi(type == 1 ? G : Gi, (P - 1) / (mid << 1));
		for (int j = 0; j < n; j += (mid << 1)) {
			ll w = 1;
			for (int k = 0; k < mid; k++, w = (w * Wn) % P) {
				int x = A[j + k], y = w * A[j + k + mid] % P;
				A[j + k] = (x + y) % P,
					A[j + k + mid] = (x - y + P) % P;
			}
		}
	}
}
//c=a*b
void NTT_times(const vector<ll>& a, const vector<ll>& b, vector<ll>& c) {
	int n, d;
	for (int i = 0; i < a.size(); i++)
		A[i] = a[i];
	for (int i = 0; i < b.size(); i++)
		B[i] = b[i];
	for (n = 1, d = 0; n < a.size() + b.size() - 1; n <<= 1, d++);
	for (int i = 0; i < n; i++) {
		R[i] = (R[i >> 1] >> 1) | ((i & 1) << (d - 1));
	}
	for (int i = a.size(); i < n; i++)
		A[i] = 0;
	for (int i = b.size(); i < n; i++)
		B[i] = 0;
	NTT(A, n, 1), NTT(B, n, 1);
	for (int i = 0; i < n; i++)
		A[i] = A[i] * B[i]%P;
	NTT(A, n, -1);
	int up = a.size() + b.size() - 1;
	c.resize(up);//重置
	ll inv = qmi(n, P - 2);
	for (int i = 0; i < up; i++)
		c[i] =A[i]*inv%P;
}
//**************************************************************
int main() {
	int n, m;
	cin >> n >> m;
	vector<ll>va(n+1),vb(m+1);
	for (int i = 0; i <= n; i++) {
		cin >> va[i];
	}
	for (int i = 0; i <= m; i++) {
		cin >> vb[i];
	}
	vector<ll>vc;
	NTT_times(va, vb, vc);
	for (auto i : vc) {
		cout << i << " ";
	}
	return 0;
}
```





# 最新板子

```cpp
const int G = 3,P = 998244353;
inline int qpow(int a,int b){
	int re = 1;
	for (; b; b >>= 1,a = 1ll * a * a % P)
		if (b & 1) re = 1ll * re * a % P;
	return re;
}
int R[maxn],c[maxn];
void NTT(int* a,int n,int f){
	for (int i = 0; i < n; i++) if (i < R[i]) swap(a[i],a[R[i]]);
	for (int i = 1; i < n; i <<= 1){
		int gn = qpow(G,(P - 1) / (i << 1));
		for (int j = 0; j < n; j += (i << 1)){
			int g = 1,x,y;
			for (int k = 0; k < i; k++,g = 1ll * g * gn % P){
				x = a[j + k],y = 1ll * g * a[j + k + i] % P;
				a[j + k] = (x + y) % P,a[j + k + i] = (x + P - y) % P;
			}
		}
	}
	if (f == 1) return;
	int nv = qpow(n,P - 2); reverse(a + 1,a + n);
	for (int i = 0; i < n; i++) a[i] = 1ll * a[i] * nv % P;
}
//a=a*b
void conv(int* a,int* b,int deg1,int deg2){
	int n = 1,L = 0;
	while (n <= (deg1 + deg2)) n <<= 1,L++;
	for (int i = 1; i < n; i++) R[i] = (R[i >> 1] >> 1) | ((i & 1) << (L - 1));
	for (int i = 1; i <= deg2; i++) c[i] = b[i];
	for (int i = deg2 + 1; i < n; i++) c[i] = 0; c[0] = 0;
	NTT(a,n,1); NTT(c,n,1);
	for (int i = 0; i < n; i++) a[i] = 1ll * a[i] * c[i] % P;
	NTT(a,n,-1);
}
```







# [原根映射](https://www.luogu.com.cn/problem/P3321)