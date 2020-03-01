# AGC 034 F RNG and XOR

> 题目连接: <https://atcoder.jp/contests/agc034/tasks/agc034_f>
> 
> 道理我都懂，可是不看题解不到啊……

$$\newcommand \xor{\mathbin{\mathbf{xor}}}$$

## 1 题意

给定一个数字 $n$ 

对于 $0 \cdots 2^n - 1$ 中每个数字都有一个 $a_i$

现在有一个数 $X$, 一开始为 $0$ 

每一次操作会随机选择一个数字 $i$ （其中 $0 \leq i \leq 2^n - 1$，选中 $i$ 的概率为 $\frac{a_i}{\sum a}$）

然后令 $X = X \xor i$

问使 $X=i$ 的期望次数

## 2 思路

先令 
$$p_i=\frac{a_i}{\sum a}$$

令 $f_i$ 表示从 $i$ 到 $0$ 的期望次数，这和从 $0$ 到 $i$ 显然是一样的

接着有一个非常显然的式子 

$$ 
\begin{aligned}
f_i &= f_{i \xor j} \times p_j + 1 ( i \neq 0 )\\ 
f_i - 1 &= f_{i \xor j } \times p_j ( i \neq 0 )
\end{aligned}
$$

然后我们就可以得到这个式子

$$ (f_0, f_1, f_2, \cdots, f_{2^{n-1}}, f_{2^n-1}) \xor (p_0, p_1, p_2, \dots, p_{2^{n-1}}, p_{2^n}) \\ = ( x, f_1 - 1, f_2 - 1, \cdots, f_{2^{n-1}} - 1, f_{2^n-1} - 1 ) $$

但是  $x$ 是未知的

注意到 $\sum p = 1$ 所以左边右边的和是相等的

所以
$$ 
\begin{aligned}
x &= \sum_{i=0}^{2^n-1} f_i - \sum_{i=1}^{2^n-1} f_i - 1 \\
&= f_0 + 2^n - 1
\end{aligned} \\
(f_0, f_1, f_2, \cdots, f_{2^{n-1}}, f_{2^n-1}) \xor (p_0, p_1, p_2, \dots, p_{2^{n-1}}, p_{2^n}) \\ = ( f_0 + 2^n - 1, f_1 - 1, f_2 - 1, \cdots, f_{2^{n-1}} - 1, f_{2^n-1} -1 )
$$

我们需要的使其中两个式子没有未知数

注意到如果使 $p_0 = p_0 - 1$

那么右边每一个数都会减去 $f_i$

即 

$$
(f_0, f_1, f_2, \cdots, f_{2^{n-1}}, f_{2^n-1}) \xor (p_0 - 1, p_1, p_2, \dots, p_{2^{n-1}}, p_{2^n}) \\ = ( 2^n - 1, - 1, - 1, \cdots, -1, -1 )
$$

于是就可以放进 FWT 里直接做

然后你就发现这东西跑出来是错的……

为什么呢

令 
$$
\begin{aligned}
\mathbf{P} & = (p_0 - 1, p_1, p_2, \dots, p_{2^{n-1}}, p_{2^n}) \\
\mathbf{A} & = ( 2^n - 1, - 1, - 1, \cdots, -1, -1 )
\end{aligned}
$$

因为 $\mathbf{P}$ 和 $\mathbf{A}$ FWT 后第一位都是 $0$ 

没有办法倒推出来 $f_0$

但是可以发现 $f$ 的值是可以平移的

即 

$$ f_i + k = \sum_{i=0}^{2^n-1} p_j(f_{i \xor j} + k) + 1 = \sum_{i=0}^{2^n-1} f_{i \xor j} \times p_j + k + 1 $$

那么我们又知道 $f_i = 0$

所以将 $f$ 每一位都减去 $f_0$ 即可

## 3 Code

```cpp
#include <cstdio> 

const int N = 1 << 20;
const int mod = 998244353;

int n;
int a[N], b[N], p[N], sum;

inline int add( int a, int b ) { return ( a + b ) % mod; }
inline int mul( int a, int b ) { return ( 1LL * a * b ) % mod; }
inline void add_eq( int &a, int b ) { a = ( a + b ) % mod; }
inline void mul_eq( int &a, int b ) { a = ( 1LL * a * b ) % mod; }

int ksm( int a, int p ) {
	int res = 1;
	while( p ) {
		if( p & 1 )
			res = mul( res, a );
		a = mul( a, a );
		p >>= 1;
	}
	return res;
}
inline int inv( int a ) { return ksm( a, mod - 2 ); }

void XOR( int *f, int len, int x = 1 ) {
	for( int o = 2, k = 1; o <= len; o <<= 1, k <<= 1 ) {
		for( int i = 0; i < len; i += o ) {
			for( int j = 0; j < k; j ++ ) {
				add_eq( f[ i + j ], f[ i + j + k ] );
				f[ i + j + k ] = add( f[ i + j ], add( - f[ i + j + k ], - f[ i + j + k ] ) );
				mul_eq( f[ i + j ], x ); mul_eq( f[ i + j + k ], x );
			}
		}
	}
}

int main() {
#ifdef woshiluo
	freopen( "F.in", "r", stdin );
	freopen( "F.out", "w", stdout );
#endif
	scanf( "%d", &n );
	n = 1 << n;
	for( int i = 0; i < n; i ++ ) {
		scanf( "%d", &a[i] );
		sum += a[i];
		b[i] = mod - 1;
	}
	sum = inv(sum);
	for( int i = 0; i < n; i ++ ) {
		p[i] = mul( a[i], sum );
	}

	b[0] = n - 1; p[0] -= 1;
	XOR( b, n ); XOR( p, n );
	for( int i = 0; i < n; i ++ ){
		mul_eq( b[i], inv( p[i] ) );
	}
	XOR( b, n, inv(2) );
	for( int i = 0; i < n; i ++ ) {
		printf( "%d\n", ( ( add( b[i], -b[0] ) + mod ) % mod ) );
	}
}
```
