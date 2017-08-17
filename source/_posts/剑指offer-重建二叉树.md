---
title: 剑指offer-重建二叉树
date: 2016-01-30 21:16:16
categories: 算法
tags: [Binary Tree, JavaScript]
---
剑指offer上的题目，在[牛客网](https://www.nowcoder.com/)上刷题通过，下面记录相关思路。

## 题目描述
输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。

## 问题分析
首先需要明确二叉树的几种遍历方法：
- 前序遍历：根节点->左子树->右子树
- 中序遍历：左子树->根节点->右子树
- 后序遍历：左子树->右子树->根节点

前序遍历的特点是不管在哪颗（子）树，首先访问其根节点。那么很容易知道，在前序遍历结果中，各个（子）树的根节点总是位于左节点与右节点前面的，也就是说前序遍历序列的第一个元素即为根节点，也就是题目中值为1的节点。由中序遍历顺序可知，首先访问左子树，遍历完根节点左边的所有节点后，再去访问根节点，最后去遍历根节点右边的所有元素。由此可得出规律，在中序遍历序列中，根节点左边的元素为左子树节点值，右边的元素为右子树节点值。

前面已知`1`为根节点，则由中序遍历规律得出左子树节点（中序遍历序列）：`{4,7,2}`，右子树节点（中序遍历序列）：`{5,3,8,6}`，即左子树有3个节点，右子树有4个节点。对比树前序遍历序列，根节点后面紧跟着的3个元素即为左子树节点，再接着的4个元素为右子树节点，可知左子树的前序遍历序列：`{2,4,7}`，右子树的前序遍历序列：`{3,5,6,8}`。

在分别得到左右子树的前序遍历序列和中序遍历序列后，可采取递归的方法，再次对左右子树执行上述分析步骤。最后直到两个遍历序列中只有一个元素时，即达到叶子节点，构建该节点并返回。经过层层递归后，二叉树构造完毕，函数最终返回二叉树的根节点。
<!--more-->
## 程序代码
代码采用JavaScript编写，运行在V8 6.0.0环境中。
{%codeblock lang:javascript%}
/**
 * 二叉树节点构造函数
 * @param {Number} x 节点值
 */
function TreeNode(x) {
  this.val = x;
  this.left = null;
  this.right = null;
}

/**
 * 根据二叉树前序和中序遍历结果，递归构造二叉树
 * @param  {Array}    pre 前序遍历结果数组
 * @param  {Array}    vin 中序遍历结果数组
 * @return {TreeNode}     构造的二叉树
 */
function reConstructBinaryTree(pre, vin) {
  // 当前（子）树的根节点
  let root = {};

  if (pre.length <= 0) {
    // 节点为空，返回null
    root = null;
  } else {
    // 根节点位于前序遍历序列第一个
    const val = pre[0];
    root = new TreeNode(val);

    // 获取左右子树（节点）的前序遍历与中序遍历序列
    const index = vin.indexOf(val);
    const leftPre = pre.slice(1, index + 1);
    const leftVin = vin.slice(0, index);
    const rightPre = pre.slice(index + 1);
    const rightVin = vin.slice(index + 1);

    // 递归构建左右子树（节点）
    root.left = reConstructBinaryTree(leftPre, leftVin);
    root.right = reConstructBinaryTree(rightPre, rightVin);
  }

  // 返回当前构建的节点
  return root;
}

const preorder = [1, 2, 4, 7, 3, 5, 6, 8];
const inorder = [4, 7, 2, 1, 5, 3, 8, 6];
console.log(reConstructBinaryTree(preorder, inorder));
{%endcodeblock%}
程序构建结果如图所示：
![TreeNode](http://7xs1tt.com1.z0.glb.clouddn.com//blog/%E5%89%91%E6%8C%87offer-%E9%87%8D%E5%BB%BA%E4%BA%8C%E5%8F%89%E6%A0%91/pic1.PNG)