---
title: 剑指offer-跳台阶
date: 2016-01-23 22:11:51
categories: 算法
tags: [Fibonacci, Recursion, Interview]
---
剑指offer上的题目，在[牛客网](https://www.nowcoder.com/)上刷题通过，下面记录相关思路。

## 题目描述
一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

## 问题分析
设跳到第n级台阶的方法有f(n)种。则最后一次跳跃有两种选择：
- 从第n-2级台阶跳上2级
- 从第n-1级台阶跳上1级

则可得到递推公式：
> f(n) = f(n-1) + f(n-2)
> 且f(0) = 1，f(1) = 1

有了公式很容易构造程序，不难看出这是一个斐波那契数列。
<!--more-->
## 程序代码
代码采用JavaScript编写，运行在V8 6.0.0环境中。
{%codeblock lang:javascript%}
/**
 * 一次可以跳上1级台阶，也可以跳上2级
 * 计算到达第n级台阶的跳法数
 * @param  {Number} number 终点台阶数
 * @return {Number}        跳法数
 */
function jumpFloor(number) {
  // 初始化
  const steps = [];
  steps[0] = 1;
  steps[1] = 1;

  // 根据递推公式依次构造结果
  for (let i = 2; i <= number; i++) {
    steps[i] = steps[i - 1] + steps[i - 2];
  }
  return steps[number];
}

console.log(jumpFloor(8));  // Output: 34
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