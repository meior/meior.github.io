---
title: 剑指offer-机器人的运动范围
date: 2018-01-07 16:31:58
categories: 算法
tags: [Backtracking, Interview]
---
剑指offer上的题目，在[牛客网](https://www.nowcoder.com/)上刷题通过，下面记录相关思路。

## 题目描述
地上有一个m行和n列的方格。一个机器人从坐标0,0的格子开始移动，每一次只能向左，右，上，下四个方向移动一格，但是不能进入行坐标和列坐标的数位之和大于k的格子。 例如，当k为18时，机器人能够进入方格（35,37），因为3+5+3+7 = 18。但是，它不能进入方格（35,38），因为3+5+3+8 = 19。请问该机器人能够达到多少个格子？

## 问题分析
这是一个典型的回溯法问题，首先我们需要明确问题的解空间并确定解空间的结构，以便回溯搜索求解。解空间是一个m x n的向量组，表示对应方格中的每一个格子是否能够达到，分别用0和1标记这两种状态，解空间的结构采用二维数组极为方便，递归前将其初始化为全0状态。

回溯的过程采用递归实现时较为容易，思路如下：
- 判断当前坐标是否可到达，需要满足一下两个条件：
  - 当前所在坐标数位和不超过阈值
  - 解向量中该坐标处元素值为0
- 若不可到达返回0，若可到达将向量组中该处元素置为1
- 继续对该点上(i,j+1)下(i,j-1)左(i-1,j)右(i+1,j)四个点递归上述步骤
- 递归出口是坐标值不超过矩阵四个边界
<!--more-->
## 程序代码
代码采用JavaScript编写，运行在Node 6.11.4环境中。
{%codeblock lang:javascript%}
/**
 * 构造二维数组
 *
 * @param {Number} rows 数组行数
 * @param {Number} cols 数组列数
 * @param {Number} initial 元素初始值
 * @returns {Array} 构造的新数组
 */
function matrix(rows, cols, initial) {
  const arr = [];
  for (let i = 0; i < rows; i++) {
    const columns = [];
    for (let j = 0; j < cols; j++) {
      columns[j] = initial;
    }
    arr[i] = columns;
  }
  return arr;
}

/**
 * 计算整数各个数位的数字之和
 *
 * @param {Number} number 待计算的整数
 * @returns {Number} 数位和
 */
function getSum(number) {
  let res = 0;
  let num = number || 0;
  while (num > 0) {
    res += num % 10;
    num = parseInt(num / 10, 10);
  }
  return res;
}

/**
 * 回溯法主体
 *
 * @param {NUmber} i 横坐标
 * @param {NUmber} j 纵坐标
 * @param {Number} rows 矩阵行数
 * @param {Number} cols 矩阵列数
 * @param {Array} flag 解向量组
 * @param {Number} threshold 题目限制条件阀值
 * @returns {Number} 可到达点的个数
 */
function backTrack(i, j, rows, cols, flag, threshold) {
  const flags = flag || [];
  // 递归出口为整个矩阵的四个边界
  // 可到达满足两个条件：1.数位和小于阀值，2.该位置未被标记过
  if (i < 0 || i >= rows || j < 0 || j >= cols ||
    getSum(i) + getSum(j) > threshold || flags[i][j] === 1
  ) {
    return 0;
  }

  // 标记当前位置为可到达
  flags[i][j] = 1;
  // 继续递归上下左右的方格
  return backTrack(i - 1, j, rows, cols, flags, threshold) +
    backTrack(i + 1, j, rows, cols, flags, threshold) +
    backTrack(i, j - 1, rows, cols, flags, threshold) +
    backTrack(i, j + 1, rows, cols, flags, threshold) +
    1;
}

/**
 * 机器人的运动范围，采用回溯法
 *
 * @param {Number} threshold 题目限制条件阀值
 * @param {Number} rows 矩阵行数
 * @param {Number} cols 矩阵列数
 * @returns {Number} 可到达点的个数
 */
function movingCount(threshold, rows, cols) {
  const flag = matrix(rows, cols, 0);
  return backTrack(0, 0, rows, cols, flag, threshold);
}

module.exports = movingCount;
{%endcodeblock%}

## 补充说明
该问题旨在求能够到达多少个格子，而机器人在上下左右四个方向上都可以运动，并且整块方格又是连通的，所以不需要记录路径只需要统计个数，即不满足要求的点跳过即可不需要回溯，故在代码思路中没有回溯的步骤，但这并不妨碍对回溯法整体的理解。另外该问题还可以采用其他方法求解，如bfs、dp等。