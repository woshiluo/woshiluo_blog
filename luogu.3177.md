# Luogu P3177 [HAOI 2015] 树上染色

> 题目链接: <https://www.luogu.com.cn/problem/P3177>

## 0 说在之前

树形 DP 要么模板题，要么神仙题

这个题就属于不太正常的

## 1 思路

$f_{i,j}$

其中 $i$ 代表当前节点

$j$ 代表子树中选择 $j$ 个黑点，所能对答案产生的**贡献**

有了这个思路，后面的东西就是标准的树上背包了

## 2 Code

```cpp
#include <cstdio>
#include <cstring>

inline int Min(int a, int b) { return a < b? a : b; }
inline long long Max(long long a, long long b) { return a > b? a : b; }

const int N = 2100;

int n, m;
int size[N];

// Edge Start
struct edge {
	int to, val, next;
} e[N << 1];
int ehead[N], ecnt;

inline void add_edge(int now, int to, int val) {
	ecnt ++;
	e[ecnt].to = to;
	e[ecnt].val = val;
	e[ecnt].next = ehead[now];
	ehead[now] = ecnt;
}
// Edge End

// DP Start
long long f[N][N];
void dp(int now, int la) {
	size[now] = 1;
	f[now][0] = f[now][1] = 0;
	for(int i = ehead[now]; i; i = e[i].next) {
		if( e[i].to == la )
			continue;
		dp(e[i].to, now);
		size[now] += size[ e[i].to ];
		for(int j = Min(size[now], m); j >= 0; j--) {
			for(int k = 0; k <= Min(size[ e[i].to ], j); k++) {
				if( f[now][ j - k ] == -1 )
					continue;
				long long val = 1LL * ( 1LL * k * ( m - k ) + 1LL * ( size[ e[i].to ] - k ) * ( n - m - size[ e[i].to ] + k ) ) * e[i].val;
				f[now][j] = Max( f[now][j], f[now][j - k] + f[ e[i].to ][k] + val);
			}
		}
	}
}
// DP End

int main() {
#ifdef woshiluo
	freopen("luogu.3177.in", "r", stdin);
	freopen("luogu.3177.out", "w", stdout);
#endif
	scanf("%d%d", &n, &m);
	for(int i = 1, u, v, w; i < n; i++) {
		scanf("%d%d%d", &u, &v, &w);
		add_edge(u, v, w);
		add_edge(v, u, w);
	}

	memset(f, -1, sizeof(f));
	dp(1, 0);

	printf("%lld", f[1][m]);
	fclose(stdin);
	fclose(stdout);
}
```
