---
priority: 0.6
title: 递归与递推
excerpt: 简单算法系列
categories: [Essay]
background-image: climb.jpeg
tags:
  - 面试
  - 递归
  - 尾递归
  - 递推
---

这些天把腾讯的一道笔试题拿回公司用来面试，出乎意料的难倒了一批人，题目本身很简单，但能够被腾讯拿来当笔试题，其实也是可以挖一挖，可以用来评估面试者算法基础。

题目如下：

> 一楼梯共有n级台阶，规定每步可以迈1级台阶或2级台阶或3级台阶，计算地面到第n级台阶所有不同的走法的总数。

大多数人可以很快写下面的代码：

```c++
int count_way(int n) 
{
	if (n = 1 || n = 2) {
		return n;
	}
	return count_way(n-1) + count_way(n-2);
}
```

这么解是可以的，但是面试官往往会问递归的空间复杂度能否优化，或者直接提出空间复杂度要求O(1)。

那么这么简单的递归改递推就好：

```c++
int count_way(int n)
{
	if (n = 1 || n = 2) {
		return n;
	}
	int a = 1, b = 2;
	for (int i = 3; i <= n; ++i)
	{
		int c = a + b;
		a = b;
		b = c;
	}
	return b;
}
```

如果面试官还想看下你对DP的理解：

```c++
int count_way(int* arr, int n)
{
	if (n == 1 || n == 2) {
		return n;
	}
  if (arr[n-1] == 0) 
    arr[n-1] = count_way(arr, n-1);
  if (arr[n-2] == 0) 
    arr[n-2] = count_way(arr, n-2);
  return arr[n-1] + arr[n-2];
}
```

面试过程中，根据你回答问题的深度，可能还会涉及尾递归优化，DP，大O表示法等相关概念或者更高难度的问题，当然面试官也会对代码中的细节包括代码风格做出评判，良好的基本功是绝佳的敲门砖。

