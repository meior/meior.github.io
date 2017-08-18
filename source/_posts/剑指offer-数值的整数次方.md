---
title: 剑指offer-数值的整数次方
date: 2016-02-12 14:40:31
categories: 算法
tags: [Bitwise, JavaScript]
---
剑指offer上的题目，在[牛客网](https://www.nowcoder.com/)上刷题通过，下面记录相关思路。

## 题目描述
给定一个double类型的浮点数base和int类型的整数exponent。求base的exponent次方。

## 问题分析
求幂运算需要注意以下几种情况：
- 指数为0
- 指数为负
- 底数为0，指数为负时不符合规定（分母不能为0）

常规求幂运算采用循环或者递归的方法，时间复杂度为`O(n)`，效率不高，这里介绍`快速幂`算法。

例如求4^11，将指数11写成二进制形式为1011，11 = 1x2^3 + 1x2^1 + 1x2^0，则4^11 = 4^(2^3) x 4^(2^1) x 4^(2^0) = 4^8 x 4^2 x 4^1。最后得到的表达式的三个乘积项，分别对应于指数的二进制位中的三个非0位。由此可见，对于指数的每一个二进制位，如果该位为1，则将其所对应的项乘进去，而每往高位进一位，对应的项就翻倍（变为前一项的平方），最后连乘所得到的结果即为最终的幂。这就是快速幂算法。

对比与普通的求幂方法，快速幂算法的循环次数为指数的二进制表示的有效位数，即`log(n)`，时间效率较高。
<!--more-->
## 程序代码
代码采用JavaScript编写，运行在V8 6.0.0环境中。
{%codeblock lang:javascript%}
/**
 * 快速幂算法
 * @param  {Number} base     底数
 * @param  {Number} exponent 指数
 * @return {Number}          幂
 */
function power(base, exponent) {
  // 不符合规定，抛出错误
  if (base === 0 && exponent < 0) {
    throw new Error('expression error');
  }

  // 幂运算结果
  let result = 1;
  // 表示上述方法中的项
  let baseNumber = base;
  // 统一以正指数计算，最后再处理负数
  let expNumber = Math.abs(exponent);
  // 一直到指数为0
  while (expNumber) {
    /* eslint-disable no-bitwise */
    // 和1按位与，可读取指数二进制表示的最后一位
    if (expNumber & 1) {
      // 位为1，将项乘进去
      result *= baseNumber;
    }

    // 每向高位前进一位，项就翻倍
    baseNumber *= baseNumber;
    expNumber >>= 1;
  }

  // 负指数则返回1除以result
  return exponent > 0 ? result : 1 / result;
}

console.log(power(4, 0));  // Output: 1
console.log(power(3, 5));  // Output: 243
console.log(power(2, -3));  // Output: 0.125
console.log(power(2.5, 5));  // Output: 97.65625
{%endcodeblock%}

## 时间复杂度
快速幂的时间复杂度为：`O(logn)`。