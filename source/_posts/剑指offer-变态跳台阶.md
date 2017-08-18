---
title: 剑指offer-变态跳台阶
date: 2016-01-24 22:33:41
categories: 算法
tags: [Recursion, Interview]
---
剑指offer上的题目，在[牛客网](https://www.nowcoder.com/)上刷题通过，下面记录相关思路。

## 题目描述
一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

## 问题分析
好吧这次真的是变态青蛙，居然可以一次直上n级台阶。
设跳到第n级台阶有f(n)种跳法。不同于普通青蛙，这次最后一跳跃就有了n种选择：
- 从第n-1级台阶跳上1级
- 从第n-2级台阶跳上2级
- 从第n-3级台阶跳上3级
- ......
- 从第0级台阶跳上n级（即从起点起跳）

则可得到递推公式：
> f(n) = f(n-1) + f(n-2) + f(n-3) + ... + f(1) + f(0)
> 且f(0) = 1，f(1) = 1

<!--more-->
由该公式可得到达第n-1级台阶的方法数：
> f(n-1) = f(n-2) + f(n-3) + ... + f(1) + f(0)

综合上述两式，可得：
> f(n) = f(n-1) + f(n-1) = 2 * f(n-1)

有了该递推公式很容易构造递推程序。其实，对该式进一步化简，则可得`f(n) = 2 ^ (n - 1)`，即可在`O(1)`时间复杂度内直接求解。

## 程序代码
代码采用JavaScript编写，运行在V8 6.0.0环境中。
{%codeblock lang:javascript%}
/**
 * 一次可以跳1级台阶，也可以跳2级，也可以跳n级
 * 计算到达第n级台阶的跳法数
 * @param  {Number} number 终点台阶数
 * @return {Number}        跳法数
 */
function jumpFloorII(number) {
  // 初始化
  const steps = [];
  steps[0] = 1;
  steps[1] = 1;

  // 根据递推公式依次构造结果
  for (let i = 2; i <= number; i++) {
    steps[i] = 2 * steps[i - 1];
  }
  return steps[number];
}

console.log(jumpFloorII(8));  // Output: 128
{%endcodeblock%}
很容易将迭代代码改为递归形式：
{%codeblock lang:javascript%}
function jumpFloorII(number) {
  return number === 1 ? 1 : 2 * jumpFloorII(number - 1);
}
{%endcodeblock%}
利用最终求得的公式，可在更小的时间复杂度内直接求得结果，代码也很简洁，一行搞定：
{%codeblock lang:javascript%}
function jumpFloorII(number) {
  return 1 << --number;
}
{%endcodeblock%}

## 时间复杂度
很容易得到迭代方法时间复杂度：`O(n)`，直接求解时间复杂度：`O(1)`。