---
title: JavaScript字符串替换的应用
date: 2018-01-11 15:26:56
categories: JavaScript
tags: [JavaScript]
---
JavaScript中处理字符串时，如果目标字符组合情况复杂，引入正则表达式将更加方便有效。比如在查找指定子串或者字符并进行替换时，String原型提供了`replace`函数，支持字符串或者正则匹配。

## 语法
{%codeblock lang:javascript%}
str.replace(regexp|substr, newSubstr|function)
{%endcodeblock%}
replace函数会返回替换后的新字符串，但不会改变原字符串的值。这里我们注意到，函数的第二个参数不仅可以传递替换的新字符串，还可以传一个自定义的替换方法。

## 参数
### pattern
如果该参数是一个字符串常量，则str中的存在的这一段字符会被替换成newSubStr，且仅替换第一次出现的字符串。我们也可以使用`RegExp`对象或者是正则的字面量形式，replace函数会将该规则匹配到的字符串替换成newSubstr或者是自定义替换函数的返回值。如果regexp具有全局标志g，那么replace将替换所有匹配的子串。否则它只替换第一个匹配子串。

### replacement
ECMAScript v3规定，replace方法的参数replacement可以是字符串或者。如果传入字符串，每个匹配都将由该字符串替换，它还支持一些以`$`开头的特殊替换规则。如果传入替换函数，每个匹配都调用该函数并将它返回的字符串作为替换文本使用。该函数的第一个参数是匹配模式的字符串。接下来的参数是与模式中的子表达式匹配的字符串，可以有0个或多个这样的参数。接下来的参数是一个整数，声明了匹配在原字符串中出现的位置。最后一个参数是原字符串本身。
<!--more-->
## 描述
这里来讨论特殊的替换参数，主要是函数和含`$`的字符串。

### 字符串
替换字符串参数可以包含一下特殊字符：

| 字符 | 替换文本 |
| :- | :- |
| $$ | 替换成$字符 |
| $& | 替换成匹配到的字符串 |
| $` | 替换成位于匹配子串左侧的字符串 |
| $' | 替换成位于匹配子串右侧的字符串 |
| $n | 替换成与regexp中第n个表达式相匹配的字符串(n从1开始，且n小于100) |

### 函数
你可以指定一个函数作为第二个参数，它会在匹配到字符串后被调用，并将其返回值作为替换字符串。如果是全局匹配，该函数可能会被调用多次。函数的参数列表如下：

| 名称 | 描述 |
| :- | :- |
| match | 匹配到的字符串(对应于上述特殊字符串的`$&`) |
| p1, p2, ... | regexp中的第n个表达式匹配到的字符串(对应于上述特殊字符串的`$n`) |
| offset | 匹配到的字符串在整个字符串中的偏移量 |
| string | 整个字符串本身 |

函数参数的具体个数取决于regexp中有多少组被括号括起来的匹配模式。例如下面例子中的newString将被替换成`abc - 12345 - #$*%`：
{%codeblock lang:javascript%}
function replacer(match, p1, p2, p3, offset, string) {
  // p1：非数字，p2：数字，p3：非字母数字
  return [p1, p2, p3].join(' - ');
}
const str = 'abc12345#$*%';
const newStr = str.replace(/([^\d]*)(\d*)([^\w]*)/, replacer);
console.log(newString);  // abc - 12345 - #$*%
{%endcodeblock%}

## 示例
### 全局和忽略规则
使用`/g`全局替换，`/i`忽略大小写。
{%codeblock lang:javascript%}
const reg = /apples/gi;
var str = 'Apples are round, and apples are juicy.';
const newStr = str.replace(reg, 'oranges');
console.log(newStr);  // oranges are round, and oranges are juicy.
{%endcodeblock%}

### 交换字符串
我们可以使用`$n`来获取指定表达式匹配到的字符串，实现字符串内字符互换。
{%codeblock lang:javascript%}
const reg = /(\w+)\s(\w+)/;
const str = 'Hello World';
const newStr = str.replace(reg, '$2 $1');
console.log(newStr);  // World, Hello
{%endcodeblock%}

### 变量命名转换
在JavaScript中我们使用的是驼峰命名法，而在html标签属性中是无法区分大小写的，我们需要转化成`-`加小写字母的表示方式。在Vue中，会自动对其做转换，转换后还是可以引用JavaScript中的同名变量。
{%codeblock lang:javascript%}
function styleHyphenFormat(propertyName) {
    function upperToHyphenLower(match, offset, string) {
        return (offset > 0 ? '-' : '') + match.toLowerCase();
    }
    return propertyName.replace(/[A-Z]/g, upperToHyphenLower);
}
console.log(styleHyphenFormat('marginTop'));  // margin-top
{%endcodeblock%}
在上述分析中，`$&`同样可以取到匹配到的字符串，直接对其调用toLowerCase不就可以了吗，似乎看起来还更简洁，都不用定义函数了。
{%codeblock lang:javascript%}
const propertyName = 'marginTop';
const newProperty = propertyName.replace(/[A-Z]/g, '-' + '$&'.toLowerCase());
console.log(newProperty);  // margin-Top
{%endcodeblock%}
大小写转换为什么没有生效？因为`'$&'.toLowerCase()`会被先执行，对字符串`'$&'`转换大写结果不会改变，所以最终调用结果等价于：
{%codeblock lang:javascript%}
propertyName.replace(/[A-Z]/g, '-' + '$&');
{%endcodeblock%}

### 温度换算
华氏温度到摄氏温度的转换一个比较常用的换算，可以采用如下实现：
{%codeblock lang:javascript%}
function f2c(x) {
    function convert(str, p1, offset, s) {
        return ((p1 - 32) * 5/9) + 'C';
    }
    const str = String(x);
    const reg = /(-?\d+(?:\.\d*)?)F\b/g;
    return str.replace(reg, convert);
}
console.log(f2c('0F'));  // -17.77777777777778C
{%endcodeblock%}