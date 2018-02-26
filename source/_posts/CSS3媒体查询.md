---
title: CSS3媒体查询
date: 2018-01-09 16:22:15
categories: CSS
tags: [CSS3]
---
在CSS中使用媒体查询，我们可以针对不同的媒体类型定义不同的样式。使用最为广泛的是在设计响应式页面时，针对不同大小的屏幕尺寸，定义不同的高度、宽度、间距等。著名的前端响应式框架[Bootstrap](https://getbootstrap.com/)就是通过媒体查询来实现响应式布局的。

## 属性分析
CSS中通过@media规则来实现，语法如下：
{%codeblock lang:css%}
@media mediatype and|not|only (media feature) {
    /* css code */
}
{%endcodeblock%}
或者针对不同媒体类型，引入不同的CSS文件：
{%codeblock lang:html%}
<link rel="stylesheet" media="mediatype and|not|only (media feature)" href="mystyle.css">
{%endcodeblock%}
除了将查询规则写在样式表顶部，还可以将其嵌套在CSS class或者其他查询规则中：
{%codeblock lang:css%}
/* Nested media query */
@supports (display: flex) {
    @media screen and (min-width: 900px) {
        article {
            display: flex;
        }
    }
}
/* Nested class */
.mydiv {
    position: fixed;
    top: 85px;
    left: 0;
    @media screen and (max-width: 360px) {
        top: 76px;
    }
}
{%endcodeblock%}
在JavaScript中，我们还可以通过`CSSMediaRule`来访问`@media`媒体查询规则。
<!--more-->

## 媒体类型
媒体类型(mediatype)描述了设备的类别。如果没有使用not、only等逻辑操作符，那么该值是可选的并且默认为`all`。

| 名称 | 描述 |
| :- | :- |
| all | 用于所有设备 |
| print | 用于打印机和打印预览 |
| screen | 用于电脑屏幕、平板电脑、智能手机等 |
| speech | 应用于屏幕阅读器等发声设备 |

> 在CSS 2.1中定义了一些媒体类型现在已经被废弃，包括：tty，tv，projection，handheld，braille，embossed，aural等。

## 媒体功能
媒体功能表达式会检测用户代理、输出设备或者环境等具体特征。表达式是必须的，并且需要用括号括起来。标准列出的值较多，常用的有主要是和设备宽度、高度、分辨率、纵横向状态、屏幕比例等相关的值，如：width、height、resolution、orientation、aspect-ratio。另外配合`min`、`max`前缀可以达到大于等于、小于等于的筛选效果，如查询宽度小于等于360px的屏幕：
{%codeblock lang:css%}
@media screen and (max-width: 360px) {
    html {
        font-size: 90px;
    }
}
{%endcodeblock%}

## 逻辑操作
使用not、and、only等操作符来结合多个查询条件，其用法和其他编程语言的逻辑操作符类似。
{%codeblock lang:css%}
@media only screen
    and (min-width: 320px)
    and (max-width: 480px)
    and (resolution: 150dpi) {
        body {
            line-height: 1.4;
        }
}
{%endcodeblock%}
