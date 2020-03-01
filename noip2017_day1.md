# NOIP 2017 Day 1 解题报告

> 今天又考的是原题就很真实，这一届的这天在考试的时候七十中居然停电了……   
> 当时我还是~~现在也是~~普及组选手，当时下午准备考试的时候老师进来告诉我们一定要保存好文件  
> 结果当年就出现了各种各样的神仙操作

嘛，老故事就不提了

还是来写解题报告吧

## T1 小凯的疑惑 (math.cpp/in/out)

### 0 吐槽

这个东西当时全世界都知道了，现在大家都知道 $ a + b - a -b $ 就是这道题目的答案了...

不过我觉得要我考场上我也许会推的很迷？

反正在考场上推出来的人感觉都很强

### 1 Code

```cpp
#include <cstdio>

long long a, b;

int main() {
	freopen( "math.in", "r", stdin );
	freopen( "math.out", "w", stdout );

	scanf( "%lld%lld", &a, &b );
	printf( "%lld", a * b - a - b );

	fclose( stdin );
	fclose( stdout );
	return 0;
}
```

## T2 时间复杂度

### 0 吐槽

为什么会有这种毒瘤模拟题目...

模拟就算了吧，老师还不给我们大样例

心态崩了

### 1 Solution

这个东西就很有意思
