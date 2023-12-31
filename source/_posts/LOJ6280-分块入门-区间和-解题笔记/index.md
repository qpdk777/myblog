---
title: LOJ6280 分块入门 区间和 解题笔记
date: 2023-07-26 22:30:35
categories: 题解
tags: ["算法数据结构", "题解", "数据结构", "分块"]
---

# 题目来源

[LibreOJ 6280 数列分块入门 4](https://loj.ac/problem/6280)

# 题意简述

序列区间加与查询区间和， $n\le5\times 10^4$ 。

# 解题思路

分块，操作区间中不是整块的部分暴力求解，是整块的部分打标记。

重点在时间复杂度的计算上。

最坏复杂度为操作区间几乎覆盖整个序列时，此时中间的整块数量近似于 $\displaystyle\frac{n}{s}$ ，两端的非整块长度为 $2s$ 。忽略常数，单次操作最坏时间复杂度为 $\displaystyle O(\frac{n}{s}+s)$ 。

运用基本不等式 $\displaystyle\frac{n}{s}+s\ge2\sqrt{\frac{n}{s}\cdot s}=2\sqrt{n}$ ，当且仅当 $s=\sqrt{n}$ 时取等，由此确定了最优块长。

关于代码实现，本题的 std 代码为分块题目提供了基本的范式。

# 注意事项

即使是根号分块，块长和块数也不一定相等。

例如，当 $n=15$ 时，块长 $s=\lfloor \sqrt{n}\rfloor=3$ ，但块数为 $5$ 。

本题在代码实现上规避了块数计算，但这并不代表 `len` 既是块长也是块数。

# AC代码

```cpp
#include<bits/stdc++.h>
#define N 50010
using namespace std;

long long n, opt, l, r, c;
long long id[N], len;
long long a[N], tag[N], s[N];

// id[i] 表示第 i 个元素所在的块的序号
// len 表示总的块数
// a[i] 表示数组
// tag[i] 表示第 i 个块中所有元素的整体赋值情况
// s[i] 表示第 i 个块中所有元素的和，已经包含了tag

inline void add(long long l, long long r, long long x){
	int lid = id[l], rid = id[r];
	if(lid == rid) { // 操作只在一个块内部
		for(int i = l; i <= r; i++){
			a[i] += x;
			s[lid] += x;
		}
		// 注意：这里并不需要动 tag，因为操作可能只作用于此块的一部分
		return;
	}
	// 如果操作跨越了多个块
	// 最左侧的块暴力
	for(int i = l; id[i] == lid; i++){ // 维持循环的条件很巧妙
		a[i] += x;
		s[lid] += x;
	}
	// 中间完整的块打标记
	for(int i = lid+1; i <= rid-1; i++){
		tag[i] += x;
		s[i] += x*len;
	}
	// 最右侧的块暴力
	for(int i = r; id[i] == rid; i--){ // 注意循环顺序
		a[i] += x;
		s[rid] += x;
	}
}

inline long long query(long long l, long long r, long long m){
	int lid = id[l], rid = id[r];
	long long ans = 0;
	if(lid == rid) { // 同一个块内暴力求和
		for(int i = l; i <= r; i++){
			ans += (a[i] + tag[lid]); // 注意这里要把tag加上，因为a没有包括tag
			ans %= m;
		}
		return ans;
	}
	// 如果跨越多个块
	// 最左侧的块暴力求和
	for(int i = l; id[i] == lid; i++){
		ans += (a[i] + tag[lid]);
		ans %= m;
	}
	// 中间的块逐块求和
	for(int i = lid+1; i <= rid-1; i++){
		ans += s[i];
		ans %= m;
	}
	// 最右侧的块暴力求和
	for(int i = r; id[i] == rid; i--){ // 注意循环顺序
		ans += (a[i] + tag[rid]);
		ans %= m;
	}
	return ans;
}

int main(){
	cin >> n;
	
	len = sqrt(n); // 一共只分n个块，最后如果有多余的部分不算做块，直接暴力
	for(int i = 1; i <= n; i++) {
		cin >> a[i]; 
		id[i] = (i-1) / len + 1; // 此行标记为关键，需想清楚并记住
		s[id[i]] += a[i];
	}
	
	while(n--){
		cin >> opt >> l >> r >> c;
		if(opt == 0){
			add(l, r, c);
		}
		else{
			cout << query(l, r, c+1) << endl;
		}
	}
	
	
	return 0;
}
```

# 相关优化

如果使用对原序列构造前缀和，对块也构造前缀和，就可以把查询的时间复杂度降低到 $O(1)$ 。然而，修改时连同前缀和一起修改，时间复杂度依然是 $O(\sqrt{n})$ 。