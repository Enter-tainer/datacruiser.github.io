---
title: Leetcode Tencent 50 No.6 78. Subsets
date: 2019-08-08 17:37:08
categories: LeetCode 腾讯精选50题
tags:
- 数据结构
- 算法
- C语言
description: DataWhale暑期学习小组-LeetCode刷题第八期Task6。
---
# 描述

给定一组**不含重复元素**的整数数组 nums，返回该数组所有可能的子集（幂集）。

说明：解集不能包含重复的子集。

**示例:**

```c
输入: nums = [1,2,3]
输出:
[
  [3],
  [1],
  [2],
  [1,2,3],
  [1,3],
  [2,3],
  [1,2],
  []
]

```


# 解题思路

这是一道考察集合和位运算的题目，首先复习一下集合在计算机当中的表示形式。

参考自[离散数学及其应用（Discrete Mathematics and Its Applications ）
](https://item.jd.com/11614251.html)

通常，计算机是利用全集元素当中元素的任何一种顺序来存放集合元素，假定全集$U$是有限的，首先为$U$的元素任意规定一个顺序，例如$a_1,a_2,...,a_n$。于是可以用长度为$n$的二进制位串来表示$U$的任意一个子集$A$：其中位串中第$i$是1时，则$a_i$属于$A$；是0时，则$a_i$不属于$A$。下面举一个具体的例子加以详细说明。

- **例题1**


令$U=\{1,2,3,4,5,6,7,8,9,10\}$，而且$U$的元素以升序排序，即$a_i=i$。表示$U$中所有奇数的子集、所有偶数的子集和不超过5的整数的子集的位串是什么？

表示$U$中所有奇数的子集$\{1,3,5,7,9\}$的位串，其第1、3、5、7、9位（从左往右）为1，其它位为0。即


$$10\,\,1010\,\,1010$$
类似的，$U$中所有偶数的子集，即$\{2,4,6,8,10\}$，可由位串

$$01\,\,0101\,\,0101$$

用位串来表示集合结合布尔运算，有着非常多的好处，便于计算集合的补集、并集、交集和差集。要从表示集合的位串计算它的补集的位串，只需要简单地把每个1改为0，每个0改成1，具体可以看下面的例子。

- **例题2**

我们已经知道集合$\{1,3,5,7,9\}$的位串（全集为$\{1,2,3,4,5,6,7,8,9,10\}$）是

$$10\,\,1010\,\,1010$$

它的补集的位串是什么？

将以上位串按位取反操作，即可得到此集合的补集的位串

$$01\,\,0101\,\,0101$$

这对应于集合$\{2,4,6,8,10\}$

同样的，要想得到两个集合的并集和交集的位串，我们可以对表示这两个集合的位串按位做布尔运算。

了解了集合在计算机中的表示形式以后回到本题，我们把给定的不含重复元素的数组nums当作是全集，则如图中所说的，我们可用长为n（n为nums的size）的位串来表示该全集的所有子集，且子集的个数正好是 $2^n$个，分别为0到 $2^n-1$的正整数的二进制表示正好为所有子集对应的位串。那么现在的问题变成如何通过位运算来取出整数二进制表示的每一位（0或者1）。具体可以看下面这个例子。

```c
S = {1,2,3}
N 	count	bit		Combination
0	0	000		{}
1	1	001		{1}
2	1	010		{2}
3	2	011		{1,2}
4	1	100		{3}
5	2	101		{1,3}
6	2	110		{2,3}
7	3	111		{1,2,3}
```

集合S的size为3，3位位串的二进制可表示的整数范围位0～7，即0～$2^3-1$，count变量为每个集合当中元素的个数，即二进制位串当中1的个数，可以通过下面这个函数统计任意小于k的正整数所表示的二进制当中1的个数。

```c
int count(int num, int k) {
    int count = 0; //初始化
    for(int i = 0; i < k; i++) {
        if(num & 1) //假如num二进制末位为1 则count+1
            count++;
        num >>= 1; //num右移一位 
    }
    return count;
}
```
其它细节的东西将在完整的代码里面给出。

# 代码


```c
int count(int num, int k) {
    int i = 0, count = 0;
    for(i = 0; i < k; i++) {
        if(num & 1)
            count++;
        num >>= 1;
    }
    return count;
}

int** subsets(int* nums, int numsSize, int* returnSize, int** returnColumnSizes)
{
    int n = 1<<numsSize;	//计算2^n，即子集的个数
    *returnSize = n; //这个参数好像在sunbsets函数的其它地方没有用到，但是不加通不过，可能测试用例的其它地方有用到
    int** result = (int**)malloc(n * sizeof(int*)); //初始化范围结果，result为二级指针，可以当作二维数组来考虑
    returnColumnSizes[0] = (int*)malloc(n * sizeof(int));  //这个变量也是比较奇怪的，感觉用一级指针也是够用了
    for(int i = 0; i < n; i++) 
    {
        returnColumnSizes[0][i] = count(i, numsSize);  //获得每个子集的元素个数
        result[i] = (int*)malloc(returnColumnSizes[0][i] * sizeof(int)); //按每个子集的元素个数来分配内存
        int size = 0, k = i;  //初始化临时变量
        for(int j = 0; j < numsSize; j++) 
        {
            //int k = i;
            if(k & 1)	// 取出在第i个子集当中，i所对应的二进制位串当中为1的第j位，并将全集当中的第j位元素放入该子集当中
                result[i][size++] = nums[j];
            k >>= 1;
        }
    }   
    return result;
}
```
