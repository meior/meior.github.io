---
title: 剑指offer-矩形覆盖
date: 2016-01-27 09:05:37
categories: 算法
tags: [Fibonacci, Recursion, Interview]
---
剑指offer上的题目，在[牛客网](https://www.nowcoder.com/)上刷题通过，下面记录相关思路。

## 题目描述
我们可以用2x1的小矩形横着或者竖着去覆盖更大的矩形。请问用n个2x1的小矩形无重叠地覆盖一个2*n的大矩形，总共有多少种方法？

## 问题分析
题目一看就很熟悉，在[hdu](http://acm.hdu.edu.cn/showproblem.php?pid=2046)上做过，同样是一道利用递推求解的题目。借用一下hdu的题目图片：
![矩形覆盖](http://7xs1tt.com1.z0.glb.clouddn.com//blog/%E5%89%91%E6%8C%87offer-%E7%9F%A9%E5%BD%A2%E8%A6%86%E7%9B%96/pic1.jpg)
采取同样的思想，设铺满规模为2*n的矩阵方法有f(n)种。考虑最后一块2x1的小矩阵如何放置：
- 竖着放，即在规模为`2*(n-1)`的矩阵上竖着放一块2x1小矩阵
- 横着放，由于行数为2，则倒数第二块2x1小矩阵也只能是横着放的，即在规模为`2*(n-2)`的矩阵上放一块规模为2x2的矩阵

则可得到递推公式：
> f(n) = f(n-1) + f(n-2)
> 且f(0) = 0，f(1) = 1，f(2) = 2

不难看出这是个斐波那契数列。
<!--more-->
## 程序代码
代码采用JavaScript编写，运行在V8 6.0.0环境中。
{%codeblock lang:javascript%}
/**
 * 求矩阵的覆盖方法
 * @param  {Number} number 矩阵列数
 * @return {Number}        方法总数
 */
function rectCover(number) {
  // 初始化
  const methods = [];
  methods[0] = 0;
  methods[1] = 1;
  methods[2] = 2;

  // 根据递推公式依次构造结果
  for (let i = 3; i <= number; i++) {
    methods[i] = methods[i - 2] + methods[i - 1];
  }
  return methods[number];
}

console.log(rectCover(7));  // Output: 21
{%endcodeblock%}
很容易将迭代代码改为递归形式：
{%codeblock lang:javascript%}
function jumpFloor(number) {
  if (number === 0 || number === 1) {
    return 1;
  }
  return jumpFloor(number - 1) + jumpFloor(number - 2);
}
{%endcodeblock%}

## 时间复杂度
很容易得到时间复杂度：`O(n)`。