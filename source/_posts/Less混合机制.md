---
title: Less混合机制
date: 2018-01-10 19:29:55
categories: CSS
tags: [Less, CSS3]
---
Less的混合机制赋予了CSS继承和函数调用的功能，通过该机制我们可以轻松的将一个已定义的样式引入到另一个样式中，提供了样式抽象和复用的特性，使样式文件具有更好的可读性和可维护性。

## 普通混合
我们可以通过类选择器或者id选择器定义样式，然后在其他样式中引入：
{%codeblock lang:css%}
.mydiv, #div1 {
    color: red;
}
.mixin-class {
    .mydiv();
}
.mixin-id {
    #div1();
}
{%endcodeblock%}
编译后可到结果如下：
{%codeblock lang:css%}
.mydiv, #div1 {
    color: red;
}
.mixin-class {
    color: red;
}
.mixin-id {
    color: red;
}
{%endcodeblock%}
调用混合集时，括号可加可不加，其最终编译效果是一致的，后续有一种情况例外。
<!--more-->
## 隐式混合
从上述编译产出可以看到，mydiv和div1这两个样式定义是多余的，在HTML代码中实际引用的还是编译后的mixin-class和mixin-id，这就造成了浪费。我们可以在混合集定义后加上括号，使其具有类似函数的计算功能而不会被输出到CSS样式表中：
{%codeblock lang:css%}
.mixin {
    color: yellow;
}
.mixin-no-output() {
    background: white;
}
.class {
    .mixin;
    .mixin-no-output;
}
{%endcodeblock%}
输出结果：
{%codeblock lang:css%}
/* mixin属性未加括号，被保留 */
.mixin {
    color: yellow;
}
.class {
  color: yellow;
  background: white;
}
{%endcodeblock%}

## 选择器混合
混合集不仅可以包含各种属性，而且可以包括各种选择器，如：子选择器、伪类选择器等。使用`&`标识符表示调用该混合集的元素。
{%codeblock lang:css%}
.my-hover-mixin() {
    &:hover {
        border: 1px solid red;
    }
}
button {
    .my-hover-mixin();
}
{%endcodeblock%}
输出结果：
{%codeblock lang:css%}
button:hover {
    border: 1px solid red;
}
{%endcodeblock%}
另外还可以使用!important关键字，改变选择器优先级，它将会应用到mixin中的每一个属性。
{%codeblock lang:css%}
.foo (@bg: #f5f5f5, @color: #900) {
    background: @bg;
    color: @color;
}
.unimportant {
    .foo();
}
.important {
    .foo() !important;
}
{%endcodeblock%}
输出结果：
{%codeblock lang:css%}
.unimportant {
    background: #f5f5f5;
    color: #900;
}
.important {
    background: #f5f5f5 !important;
    color: #900 !important;
}
{%endcodeblock%}

## 嵌套混合
当样式混合调用情况比较复杂时，容易产生样式冲突，我们可以将混合属性嵌套在多层id或者class中，这种用法的效果相当于我们熟知的命名空间，确保不和其他库产生冲突。
{%codeblock lang:css%}
#my-library {
    .my-mixin() {
        font-size: 16px;
    }
}
/* 可以这样调用，且>符号可以省略 */
.class {
    #my-library > .my-mixin();
}
{%endcodeblock%}

## 参数混合
我们可以向混合集中传入参数，类似于函数参数，在它进行mixin操作时会将变量传递给选择器代码块。
{%codeblock lang:css%}
.border-radius(@radius) {
    border-radius: @radius;
    -moz-border-radius: @radius;
    -webkit-border-radius: @radius;
}
#header {
    .border-radius(4px);
}
.button {
    .border-radius(6px);
}
{%endcodeblock%}
我们也可以为其添加默认参数：
{%codeblock lang:css%}
.border-radius(@radius: 5px) {
    border-radius: @radius;
    -moz-border-radius: @radius;
    -webkit-border-radius: @radius;
}
{%endcodeblock%}
然后像这样调用它时，仍然会包含一个5px的border-radius：
{%codeblock lang:css%}
#header {
    .border-radius;
}
{%endcodeblock%}

### 多参数混合
参数可以用分号或者逗号分割。但是推荐使用分号分割，因为逗号符号有两个意思：它可以解释为mixins参数分隔符或者CSS列表分隔符。使用逗号作为mixin的分隔符则无法用它创建逗号分割的参数列表。换句话说，如果编译器在mixin调用或者声明中看到至少一个分号，它会假设参数是由分号分割的，而所有的逗号都属于CSS列表。参数分隔规则如下：
- 两个参数，并且每个参数都是逗号分割的列表：.name(1, 2, 3; something, ele)，
- 三个参数，并且每个参数都包含一个数字：.name(1, 2, 3)，
- 使用伪造的分号创建mixin，调用的时候参数包含一个逗号分割的css列表：.name(1, 2, 3;)，
- 逗号分割默认值：.name(@param1: red, blue)。

定义多个具有相同名称和参数数量的mixins是合法的，Less会使用它可以应用的属性。如果使用mixin的时候只带一个参数，比如.mixin(green)，这个属性会导致所有的mixin都会使用强制使用这个明确的参数。
{%codeblock lang:css%}
.mixin(@color) {
    color-1: @color;
}
.mixin(@color; @padding: 2) {
    color-2: @color;
    padding-2: @padding;
}
.mixin(@color; @padding; @margin: 2) {
    color-3: @color;
    padding-3: @padding;
    margin: @margin @margin @margin @margin;
}
.selector div {
    .mixin(#666);
}
{%endcodeblock%}
输出结果：
{%codeblock lang:css%}
.selector div {
    color-1: #666;
    color-2: #666;
    padding-2: 2;
}
{%endcodeblock%}
由于只给mixin传了一个参数，所以color-1正常输出，而具有三个参数的mixin则不会输出。那为什么两个参数的mixin也输出了呢？因为其第二个参数具有默认值，所以也被输出到样式表中。

### @arguments变量
就像JavaScript函数一样，mixin参数也可以使用arguments关键字，它包含所有传入的参数。
{%codeblock lang:css%}
.box-shadow(@x: 0; @y: 0; @blur: 1px; @color: #000) {
    box-shadow: @arguments;
    -moz-box-shadow: @arguments;
    -webkit-box-shadow: @arguments;
}
.big-block {
    .box-shadow(2px; 5px);
}
{%endcodeblock%}
输出结果：
{%codeblock lang:css%}
.big-block {
    box-shadow: 2px 5px 1px #000;
    -moz-box-shadow: 2px 5px 1px #000;
    -webkit-box-shadow: 2px 5px 1px #000;
}
{%endcodeblock%}

### 高级参数和@rest变量
如果希望混合集接受数量不定的参数，可以使用...。在变量名后面使用它，会将这些参数分配给对应的变量。
{%codeblock lang:css%}
.mixin(...)           // 匹配0-N个参数
.mixin()              // 只匹配0个参数
.mixin(@a: 1)         // 匹配0-1个参数
.mixin(@a: 1; ...)    // 匹配0-N个参数
.mixin(@a; ...)       // 匹配1-N个参数
.mixin(@a; @rest...)  // 匹配1-N个参数
{%endcodeblock%}
`@rest...`会将从第二个开始的所有参数全部放到@rest变量中，这样可以很方便的直接放入padding、margin等接受多个值的属性中。

### 参数匹配模式
参数如果不带@符合则不是一个合法的Less变量，会被处理成一个确定的值，而带@的标识符则被处理成一个变量，接受任何类型的值。
{%codeblock lang:css%}
.mixin(dark; @color) {
    color: darken(@color, 10%);
}
.mixin(light; @color) {
    color: lighten(@color, 10%);
}
.mixin(@_; @color) {
    display: block;
}
// 调用
@switch: light;
.class {
    .mixin(@switch; #888);
}
{%endcodeblock%}
最终会得到结果：
{%codeblock lang:css%}
.class {
    /* lighten淡化了颜色 */
    color: #a2a2a2;
    display: block;
}
{%endcodeblock%}
产生上述输出的原因：
- 第一个mixin定义并没有匹配，因为它期望的第一个参数是一个确定的值dark，调用时传入的是light；
- 第二个mixin定义匹配了，因为它接受的参数是预期的light；
- 第三个mixin定义也匹配了，因为它的参数是一个变量，接受任何值。

同样参数的个数也是匹配的中重要依据：
{%codeblock lang:css%}
.mixin(@a) {
    color: @a;
}
.mixin(@a; @b) {
    color: fade(@a; @b);
}
{%endcodeblock%}
现在，如果我们用一个参数来调用.mixin，这将会输出第一个定义，但是如果我们使用两个参数调用它，会获取到第二个定义，这就是@a淡入到@b。以上我们可以看出，Less混合参数的匹配和面向对象编程语言的函数匹配模式很像。

## 函数混合
通过以上研究发现，Less的混合机制怎么看怎么像函数，实际上它完全可以当作是CSS函数来使用，实现一些简单的计算是没有任何问题的。在mixin中定义的变量在外部都是可以见的，如果调用它的作用域中定义了同名变量，那么该变量将会被赋予相同的值，达到了函数返回值的效果。
{%codeblock lang:css%}
.average(@x, @y) {
    @average: ((@x + @y) / 2);
}
div {
    .average(16px, 50px);
    padding: @average;
}
{%endcodeblock%}
输出结果：
{%codeblock lang:css%}
div {
    padding: 33px;
}
{%endcodeblock%}
实际运用中我们可能需要在函数中计算出具体数值，然后手动带上单位，此时就需要字符串拼接。尤其是在计算px与rem相加时，由于rem不确定，需要首先转换不能直接相加。
{%codeblock lang:css%}
.font-size(@sizeValue) {
    @remValue: @sizeValue;
    @pxValue: (@sizeValue * 10);
    font-size: ~"@{pxValue}px"; 
    font-size: ~"@{remValue}rem";
}
p {
    .font-size(13);
}
{%endcodeblock%}