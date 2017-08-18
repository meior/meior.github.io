---
title: 剑指offer-旋转数组的最小数字
date: 2016-02-05 18:43:43
categories: 算法
tags: [Array, Binary Search, Interview]
---
剑指offer上的题目，在[牛客网](https://www.nowcoder.com/)上刷题通过，下面记录相关思路。

## 题目描述
把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。 输入一个非递减排序的数组的一个旋转，输出旋转数组的最小元素。 例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。 NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。

## 问题分析
印象中，排序数组的元素查找一向以二分法最优，这道题虽然略有不同，但基本思想是一致的。数组旋转前是非递减的，则第一个元素即为最小数字、旋转后最小数字位于数组中间部分，将数组分割为两部分，最小数字左边的元素所构成的数组是非递减的，而它和右边的元素所构成的数组也是非递减的，并且前面子数组的元素都大于后面子数组的元素。题目所给的数组是排序的，并且最小元素是两个子数组的分界点，可以尝试用二分法查找最小元素。
<!--more-->
先考虑没有重复元素的情况。使用两个变量left，right分别保存数组第一个元素和最后一个元素的索引，若为旋转数组，则第一个元素应该大于最后一个元素。可以很快定位中间元素的索引为`left + (right - left) / 2`，比较它与数组第一个元素的大小：
- 中间元素大于第一个元素，则中间元素位于前面的子数组，最小元素位于后面的子数组，在中间元素的后面。此时，将left索引设为中间元素的索引。
- 中间元素小于第一个元素，则中间元素位于后面的子数组，最小元素位于前面的子数组，在中间元素的前面。此时，将right索引设为中间元素的索引。

重复上述步骤，一直缩小范围。left所指的元素始终位于前面的子数组，right所指元素始终位于后面的子数组，最终left与right相邻，即left指向前面数组的最后一个元素，right指向后面数组中的第一个元素，而最小元素在后面的子数组中，所以right所指向的元素就是最小元素。

现在考虑数组中有重复元素的情况。例如非递减数组{1, 2, 2, `2`, 2, 2, 2}，其旋转数组中的两个{2, 2, 2, `2`, 1, 2, 2}，{2, 1, 2, `2`, 2, 2, 2}，在这两个数组中，第一个元素、最后一个元素、中间元素均为2。第一个旋转数组中，中间元素位于后面的子数组，第二个旋转数组中，中间元素位于前面的子数组。因此当两个索引指向的元素和中间元素相等时，无法确定中间元素`2`属于前面的子数组还是属于后面的子数组，也就无法调整索引来缩小查找范围。

在有重复元素的情况下，二分搜索无能为力，只能采取顺序搜索的方法。从前往后遍历数组，如果出现前一个元素比后一个元素大的情况，则后一个元素即为最小元素；如果没有，则说明数组每一个元素都相等，返回任意元素均可。

## 程序代码
代码采用JavaScript编写，运行在V8 6.0.0环境中。
{%codeblock lang:javascript%}
/**
 * 顺序查找旋转数组最小元素
 * 分界点即为最小元素
 * @param  {Array}  rotateArray 非递减数组的旋转
 * @return {Number}             最小的元素或者0（空数组）
 */
function searchInOrder(rotateArray) {
  if (rotateArray.length === 0) {
    return 0;
  }

  for (let i = 0; i < rotateArray.length - 1; i++) {
    // 发现分界点，返回后者
    if (rotateArray[i] > rotateArray[i + 1]) {
      return rotateArray[i + 1];
    }
  }
  return rotateArray[0];
}

/**
 * 利用二分法求旋转数组的最小元素
 * 如果中间元素与两端元素都相等，只能采取顺序遍历查找
 * @param  {Array} rotateArray 非递减数组的旋转
 * @return {Number}            最小的元素或者0（空数组）
 */
function minNumberInRotateArray(rotateArray) {
  if (rotateArray.length === 0) {
    return 0;
  }

  let left = 0;
  let right = rotateArray.length - 1;
  let mid = 0;

  // 确定是旋转数组
  while (rotateArray[left] >= rotateArray[right]) {
    // 左右索引相邻，二分搜索结束，right所指即为最小元素
    if (right - left === 1) {
      mid = right;
      break;
    }

    // 计算中间元素索引，js中除操作会产生浮点数，需取整
    mid = left + Math.floor((right - left) / 2);
    // 如果第一个元素、中间元素、最后一个元素都相等，则只能顺序查找
    if (rotateArray[left] === rotateArray[right] && rotateArray[left] === rotateArray[mid]) {
      return searchInOrder(rotateArray);
    }

    if (rotateArray[mid] >= rotateArray[left]) {
      // 中间元素大于或等于左端元素，最小元素位于中间元素后面，left后移
      left = mid;
    } else {
      // 中间元素小于或等于左端元素，最小元素位于中间元素前面，right前移
      right = mid;
    }
  }

  return rotateArray[mid];
}

const array1 = [3, 4, 5, 1, 2];
const array2 = [2, 2, 2, 2, 1, 2, 2];
console.log(minNumberInRotateArray(array1));  // Output: 1
console.log(minNumberInRotateArray(array2));  // Output: 1
{%endcodeblock%}

## 时间复杂度
顺序查找时间复杂度为：`O(n)`，二分查找时间复杂度为：`O(logn)`。