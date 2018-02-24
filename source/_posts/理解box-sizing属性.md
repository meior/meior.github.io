---
title: 理解box-sizing属性
date: 2018-01-08 15:22:21
categories: CSS
tags: [CSS3]
---
该属性是CSS3中新增的属性，具体作用在于改变容器的盒模型组成方式。由于各种原因很多开发者并不知道该属性的存在，兼容性问题可能是其中的原因之一。常规意义上的盒子模型是指标准w3c盒子模型、IE盒子模型，他们之间的区别也是老生常谈的远古原理了，这里不再赘述。

## 属性分析
box-sizing语法如下：
{%codeblock lang:html%}
box-sizing: content-box|border-box|inherit;
{%endcodeblock%}

### content-box
该值将元素盒模型组成方式设置为标准w3c盒子模型，即元素实际占用宽度与高度包括content、padding、border。浏览器会优先根据为元素指定的width、height值来绘制内容部分，之后在其外部绘制元素的内边距和边框，这样就会造成子元素将父元素撑开甚至内容超出的现象。可以将该模型概括为先放内容再扩展盒子，目前现代浏览器多采用这种布局方式。

### border-box
该值将元素盒模型组成方式设置为IE盒子模型，即元素实际占用宽度与高度就是为元素指定的width、height属性值。浏览器会先依据该值优先绘制整个盒子，之后在盒子内部绘制元素的内边距和边框，最后剩余的空间放内容部分，不论是否放得下，这种情况下子元素无法撑开父元素，只能在盒模型剩余空间中绘制实际元素内容。但是存在特殊情况，下面将进一步讨论。
<!--more-->
### inherit
规定应从父元素继承box-sizing属性的值。

## 实验室
我们在Chrome上实验对比一下这个属性，建立两个div盒子，其各个属性均一致，只保持box-sizing不一致。
{%codeblock lang:html%}
.content-box {
    width: 50px;
    height: 50px;
    border: solid 50px green;
    padding: 50px;
    background-color: yellow;
    box-sizing: content-box;
    /* box-sizing: border-box; */
}
{%endcodeblock%}
其效果如下，可以看到边框、内边距、内容块的大小均展现为css样式中设置的值。最终盒子实际展现大小为这三部分的和，即50x2 + 50x2 + 50 = 250。

![pic1](http://7xs1tt.com1.z0.glb.clouddn.com//blog/%E7%90%86%E8%A7%A3box-sizing%E5%B1%9E%E6%80%A7/pic1.JPG)
将box-sizing属性改为border-box后，盒子大小明显变小但还是不符合我们之前分析的预期，并且盒子中的内容也被覆盖了。

![pic2](http://7xs1tt.com1.z0.glb.clouddn.com/blog/%E7%90%86%E8%A7%A3box-sizing%E5%B1%9E%E6%80%A7/pic2.JPG)
根据上述分析，我们给定了width后，盒子的最终大小应该是50px才对，后续浏览器在往盒子内部依次填充边框、内边距、内容，但现在盒子的最终展现大小却为200px。这是因为我们给定的width太小只有50px，浏览器在向盒子里绘制边框和内边距时发现明显不够用，会以边框和内边距填充完毕后的大小去将盒子撑开，然后再绘制边框和内边距，但是不会绘制里面的内容部分，这也就是在上图中我们并没有看到内容块的原因。故此时盒子最终占用大小为50x2 + 50x2 = 200。

## 兼容性
| IE | Firefox | Chrome | Safari | Opera |
| :-: | :-: | :-: | :-: | :-: |
| ![ie](http://www.w3school.com.cn/ui2017/compatible_ie.png) | ![firefox](http://www.w3school.com.cn/ui2017/compatible_firefox.png) | ![chrome](http://www.w3school.com.cn/ui2017/compatible_chrome.png) | ![safari](http://www.w3school.com.cn/ui2017/compatible_safari.png) | ![opera](http://www.w3school.com.cn/ui2017/compatible_opera.png) |
IE、Opera以及Chrome支持box-sizing属性，Firefox支持替代的-moz-box-sizing属性。

## 应用场景
大多数现代浏览器在解析到有doctype声明后，会采用标准盒子模型进行渲染，所以这里只讨论border-box的使用。什么情况下，我们不希望盒子的宽高被自适应撑开而无法掌控页面布局？
我们往往有一个这样需求：页面顶部有一个工具栏，该工具栏是组件化的，其宽度为屏幕宽度百分比，为了适应水平居中布局我们需要加一个左右的内边距。此时使用该属性就很便捷，在加了padding后，盒子剩下的空间会被工具栏组件根据百分比完全填充。当然不排除其他布局实现，这里使用border-box最为合适。