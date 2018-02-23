---
title: 剑指offer-调整数组顺序
date: 2018-01-06 14:40:31
categories: 算法
tags: [Array, Interview]
---
剑指offer上的题目，在[牛客网](https://www.nowcoder.com/)上刷题通过，下面记录相关思路。

## 题目描述
输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。

## 问题分析
该问题有一种比较简单的且时间复杂度为O(n)的方法，即从前往后遍历数组，将奇数剪切下来放到新数组中，最后将剩余的偶数放到新数组尾部即可，空间复杂度为O(n)。

这里我们讨论另一种方法。要想保证原有次序，则只能顺次移动或相邻交换。设置一个索引i从左向右遍历，找到第一个偶数，设置另一个索引j从该偶数后开始向后找，直到找到第一个奇数。按照题目要求的排序方式，该奇数应该位于前面整段偶数的前面，将两个索引所包含的这一段(i到j-1)整体后移一位，将找到的奇数插入到前面空出的位置i中并将i加1，此时i到j这一段已经都是偶数了，下一次循环时j不用回退到i+1的位置开始查找下一个奇数。
<!--more-->
## 程序代码
代码采用JavaScript编写，运行在Node 6.11.4环境中。
{%codeblock lang:javascript%}
/**
 * 判断是否为偶数
 *
 * @param {Number} number 待判断的整数
 */
function isEven(number) {
  return number % 2 === 0;
}

/**
 * 将数组中的奇数放在前半部分，偶数放在后半部分
 *
 * @param {Array} array 待重排序数组
 */
function reOrderArray(array) {
  const arr = !array || array.length === 0 ? [] : array;

  let i = 0;
  let j = -1;
  const len = arr.length;
  while (i < len) {
    // 从左往右找到第一个偶数
    while (i < len && !isEven(arr[i])) {
      i++;
    }
    // 在已知偶数后面查找第一个奇数
    if (j <= i && j < len) {
      j = i + 1;
    }
    while (j < len && isEven(arr[j])) {
      j++;
    }
    if (j < len) {
      // 将i到j-1中间这部分偶数整体后移
      const tmp = arr[j];
      for (let end = j - 1; end >= i; end--) {
        arr[end + 1] = arr[end];
      }
      // 将奇数放到空缺位置
      arr[i++] = tmp;
    } else {
      break;
    }
  }
  return arr;
}

module.exports = reOrderArray;
{%endcodeblock%}

## 时间复杂度
在循环过程中，不用将j每次都回退到i+1的位置，因为经过上一次移动后，i到j这一段均为偶数。此方法时间复杂度约为O(n)。