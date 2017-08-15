---
title: JavaScript提升机制
date: 2016-01-08 15:28:40
categories: JavaScript
tags: [JavaScript]
---
## 引子
在讨论这个问题之前，我们需要先弄清楚一个语文问题：什么是声明，什么是定义？在很多讲编程语言的书籍中喜欢混淆这两个概念，读者也就无所谓了，大概就是“新建一个变量”。
通俗的来讲，`var i;`是声明一个变量i，但没有赋予初值，`var i = 0;`是声明一个变量i，并定义它的初值为0。后面统一使用该标准来描述。为什么要啰嗦这些呢，因为语义的原因，在JavaScript中出现了令人不解的现象。首先来看一段代码：
{%codeblock lang:javascript%}
console.log(a);    // Output: ReferenceError: a is not defined
{%endcodeblock%}
很显然在打印之前并不存在变量a，会报ReferenceError。那么再看下面这段代码：
{%codeblock lang:javascript%}
var a;
console.log(a);    // Output: undefined
{%endcodeblock%}
这里有人就会说，我明明在打印前定义了变量a，为什么还说没有定义呢。因为`var a;`只是声明了变量，而并没有定义它的值是什么。完整的声明并定义应该是`var a = 'hello';`，这就是语义所产生的误会。
<!--more-->
## 什么是提升
### 变量提升
一般我们总是习惯先声明变量然后再使用它，事实上也只能如此，但在JavaScript中却不一定。且看下面的情况：
{%codeblock lang:javascript%}
console.log(a);    // Output: undefined
var a = 'hello';
{%endcodeblock%}
这个现象出乎寻常，按照第一个例子应该报ReferenceError才对，但是却为undefined。根据第二个例子来看，说明第二行代码对于变量a的声明是有效的，相当于将变量a提到了打印语句的前面，这就是变量提升。但是为什么不是输出字符串'hello'呢？因为JavaScript解释器将上述代码解释为如下形式：
{%codeblock lang:javascript%}
var a;
console.log(a);
a = 'hello';
{%endcodeblock%}
所以后面将a定义为'hello'也就无力回天了，如果再加一行`console.log(a)`，就会打印'hello'。

### 函数提升
同样的函数声明也具有类似特性，但是这里的函数声明不同于变量声明。函数可以两种形式存在，函数声明与函数表达式。
{%codeblock lang:javascript%}
// 函数声明
function add(num1, num2){
    return num1 + num2;
}

// 函数表达式
var add = function(num1, num2){
    return num1 + num2;
}
{%endcodeblock%}
很容易得到提升的结果：
{%codeblock lang:javascript%}
console.log(add(1, 2));    // Output: 3

function add(num1, num2){
    return num1 + num2;
}
{%endcodeblock%}
函数中的提升相对于变量提升还有一些有趣的事情，比如下面的情况：
{%codeblock lang:javascript%}
console.log(add(1, 2));    // Output: TypeError: add is not a function

var add = function(num1, num2){
    return num1 + num2;
}
{%endcodeblock%}
等等，这里怎么变成TypeError了。按第一个例子说没有做函数提升，在调用add函数时应该提示`ReferenceError: add is not defined`才对。我们来慢慢分析，虽然函数表达式不会提升，但是别忘了表达式前半部分的变量声明`var add`，按照上述结论，解释器会对其做变量提升，最终代码解释为如下形式：
{%codeblock lang:javascript%}
var add;
console.log(add(1, 2));

add = function(num1, num2){
    return num1 + num2;
}
{%endcodeblock%}
这样就可以解释为什么console.log函数中仍然可以调用add函数了。实际上改为`console.log(add)`，会打印`undefined`，这就更加证明上述提升后的代码的正确性了。由于add并没有赋值，引擎执行add时并不知道它是一个函数，因而报TypeError。

## 为什么要提升
既然这个东西看起来并不那么美好，并且和一般处理逻辑不符，为什么还要做提升呢？存在即合理，我们来一次追根溯源。JavaScript是一门解释性的语言，其执行过程离不开以下三部分：
- 引擎：负责整个JavaScript的编译和执行
- 解释器：代码解释与生成
- 作用域：收集和维护所有声明的标识符，并确保当前作用域对标识符的访问权限
边解释边执行的一个很严重的问题在于标识符的处理，如果在引擎执行到刚好需要使用它的时候，再去申请内存空间并分配作用域，这将带来灾难性的后果，引擎将变得无比缓慢。所以在代码真正执行前，我们仍需要适当“编译”源代码，也就是解释器的分内职责。其处理代码分为两个时期：
    1.在代码执行前，处理函数声明以及变量声明，也就是将函数以及变量提升。提升后类似于如下效果：
{%codeblock lang:javascript%}
var a;
var str;
var sub;
function add(){
    // ...
};
function doSomething(){
    // ...
};
{%endcodeblock%}
    2.在代码运行时，处理函数表达式以及未声明变量。类似于如下情形：
{%codeblock lang:javascript%}
a = 12;
str = 'world';
sub = function(){
    // ...
};
{%endcodeblock%}

## 总结
声明的提升只是将变量或函数提升到它所在作用域顶部，不会影响其他作用域。尽管函数和变量的提升看起来比较炫酷，但是对代码的阅读性不友好，同时也会影响代码的逻辑，特别是函数声明。因此编码时还需遵照编程规范，提高阅读性。