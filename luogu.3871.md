# P3871 [TJOI2010]中位数

> 题目链接: [https://www.luogu.org/problem/P3871]

因为要帮同学们复习的时候选题，选到堆的时候问了问 StudyingFather 的意见，他给我推荐了这个题目

超级感谢推荐，这个题目真是太有意思了（

## 1 思路

实际上这种方法我也并不是第一个，只是觉得网上绝大多数代码都没有咱写的清晰

~~实际上就是觉得别人代码不好看~~

竟然写了就说说吧

考虑中位数本质是维护最中间的数

将序列一分为二，保持插入后两个数列大小的均衡即可

但是插入的时候也不能暴力插入啊（

那就将左右两个序列分别建立一个堆

然后插入堆即可

## 2 Source 

```cpp
#include <cstdio>
#include <queue>

std::priority_queue<int> _min, _max;

int n, m, left, rig;

void balance() {
	int k = ( ( left + rig ) & 1 );
	while( left - rig > k ) {
		_min.push( - _max.top() ), _max.pop();
		left --, rig ++;
	}
	while( left - rig < k ) { 
		_max.push( - _min.top() ), _min.pop();
		left ++, rig --;
	}

}

void add( int val ) {
	if( _max.empty() == false && val > _max.top() ) {
		_min.push( - val );
		rig ++;
	}
	else {
		_max.push( val );
		left ++;
	}
	balance();
}

int main() {
#ifdef woshiluo
	freopen( "luogu.3871.in", "r", stdin );
	freopen( "luogu.3871.out", "w", stdout );
#endif
	scanf( "%d", &n );
	for( int i = 1, tmp; i <= n; i ++ ) {
		scanf( "%d", &tmp );
		add(tmp);
	}

	scanf( "%d", &m );

	char op[5];
	int tmp;
	while( m -- ) {
		scanf( "%s", op );
		if( op[0] == 'a' ) {
			scanf( "%d", &tmp );
			add(tmp);
		}
		else {
			balance();
			printf( "%d\n", _max.top() );
		}
	}
	return 0;
}
```
