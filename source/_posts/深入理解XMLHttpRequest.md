---
title: 深入理解XMLHttpRequest
date: 2018-01-12 19:31:52
categories: JavaScript
tags: [JavaScript, Ajax]
---
XMLHttpRequest是一个重要的js对象，它是Ajax技术的核心。Ajax全称是Asynchronous JavaScript and XML，中文翻译是异步的JavaScript和XML，它是CSS、HTML、JavaScript、HTTP等多种技术的组合方案，并不是一种新技术。
通过该功能，我们可以在JavaScript脚本中向服务器请求数据，然后更新到DOM中。这一过程无须卸载页面，会带来更好的用户体验。Ajax大致包括四个步骤：创建Ajax对象、发送HTTP请求、接收处理数据、更新DOM。一个典型的Ajax发送json数据的示例如下：
{%codeblock lang:javascript%}
function ajax() {
    const person = {
        name: 'jack',
        age: 20
    };
    let xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function () {
        if (xhr.readyState == 4) {
            if (xhr.status == 200 || xhr.status == 304) {
                let time = document.getElementById('time');
                console.log(xhr.responseText);
            }
        }
    }
    xhr.open('post', '/time', true);
    xhr.setRequestHeader("Content-type", "application/json;charset=utf-8");
    xhr.send(JSON.stringify(person));
}
{%endcodeblock%}
## 发展历程
Ajax这一名称出现在其技术实现之后，早在1998年，微软Outlook Web Access小组就写成了允许客户端脚本发送HTTP请求(XMLHTTP)的第一个组件，直到2005年该名词才被提出。后来各大浏览器纷纷效仿也提供了这个接口，所以就造成了早期的IE浏览器和其他厂商浏览器对Ajax的实现不一致的情况。后来W3C对其进行了标准化，推出了[XMLHttpRequest标准](https://www.w3.org/TR/XMLHttpRequest/)。XMLHttpRequest标准又分为Level 1和Level 2。<!--more-->
XMLHttpRequest Level 1主要存在以下缺点：
- 受同源策略的限制，不能发送跨域请求；
- 不能发送二进制文件(如图片、视频、音频等)，只能发送纯文本数据；
- 在发送和获取数据的过程中，无法实时获取进度信息，只能判断是否完成；

Level 2对Level 1进行了改进，XMLHttpRequest Level 2中新增了以下功能：
- 可以发送跨域请求，在服务端允许的情况下；
- 支持发送和接收二进制数据；
- 新增formData对象，支持发送表单数据；
- 发送和获取数据时，可以获取进度信息；
- 可以设置请求的超时时间；

## 创建对象
由于规范的问题，在早期版本的IE浏览器中(IE5、IE6)，只能通过`ActiveXObject`对象来创建Ajax对象，后续版本的IE虽然可以使用，但存在诸多限制，以下是兼容性写法：
{%codeblock lang:javascript%}
let xhr;
if (window.XMLHttpRequest) {
    xhr = new XMLHttpRequest();
}
else {
    xhr = new ActiveXObject('Microsoft.XMLHTTP');
}
{%endcodeblock%}
{%note warning%}
对于每一个新请求，原则上都需要使用新的xhr对象。如果重用xhr对象，由于异步的特性，在发送新请求时将会终止之前被挂起的所有请求，造成数据丢失。
{%endnote%}

## 发送请求
首先需要设置请求方式、请求地址、异步或者同步等信息，open函数接收三个参数分别对应前面提供的三种信息：
{%codeblock lang:javascript%}
xhr.open('post', '/time', true);
{%endcodeblock%}
在发送请求之前，我们还需要针对不同的情况设置不同的请求头部信息，包括`content-type`、`connection`、`cookie`、`accept-xxx`等字段，该方法必须要在open方法之后调用，多次调用会将字段追加到HTTP请求头中，例如：
{%codeblock lang:javascript%}
xhr.setRequestHeader("Content-type", "application/json;charset=utf-8");
xhr.setRequestHeader('X-Test', 'one');
xhr.setRequestHeader('X-Test', 'two');
// 最终request header中"X-Test"为: one, two
{%endcodeblock%}
最后调用send方法向服务器发送数据，如果请求方式为GET，该参数为空或者null，如果为POST方法，参数为要发送的数据，且必须为String类型：
{%codeblock lang:javascript%}
const person = {
    name: 'jack',
    age: 20
};
xhr.send(JSON.stringify(person));
{%endcodeblock%}

## 接收数据
服务器返回的数据包括我们期望的真实数据还有HTTP状态码，一般我们需要关注的是以下几个数据：
{%note info%}
responseText：作为响应主体被返回的文本(文本形式)
responseXML：如果响应的内容类型是'text/xml'或'application/xml'，这个属性中将保存着响应数据的XML DOM文档(document形式)
status：HTTP状态码(数字形式)
statusText：HTTP状态说明(文本形式)
{%endnote%}
通过状态码我们可以判断返回的数据的有效性。对于同步Ajax我们只需在send之后直接判断HTTP状态码`xhr.status`即可，200表示数据处理成功并返回，304表示请求的资源并没有被修改，可以直接使用浏览器中缓存的版本，这也是有效数据。
{%codeblock lang:javascript%}
xhr.send();
if (xhr.readyState == 4) {
    if (xhr.status == 200) {
        console.log(xhr.responseText);
    }
}
{%endcodeblock%}
如果需要接收的是异步响应，send方法并不会被阻塞等待数据返回，我们也就不能直接判断状态码就去取数据，这就需要检测XHR对象的readyState属性，该属性表示请求/响应过程的当前活动阶段。这个属性可取的值如下：
{%note info%}
0(UNSENT)：未初始化。尚未调用open()方法
1(OPENED)：启动。已经调用open()方法，但尚未调用send()方法
2(HEADERS_RECEIVED)：发送。己经调用send()方法，且接收到头信息
3(LOADING)：接收。已经接收到部分响应主体信息
4(DONE)：完成。已经接收到全部响应数据，而且已经可以在客户端使用了
{%endnote%}
理论上只要readyState属性值由一个值变成另一个值，都会触发一次readystatechange事件。可以利用这个事件来检测每次状态变化后readyState的值。通常我们对readyState值为4的阶段感兴趣，因为这时所有数据都已就绪。但是需要注意的是，必须在调用open之前指定onreadystatechange事件处理函数，这样才能确保跨浏览器兼容性，否则将无法接收readyState属性为0和1的情况。
{%codeblock lang:javascript%}
xhr.onreadystatechange = function () {
    if (xhr.readyState == 4) {
        if (xhr.status == 200 || xhr.status == 304) {
            console.log(xhr.responseText);
        }
    }
}
{%endcodeblock%}
如果请求过了很久还没有成功，为了不会白白占用的网络资源，我们一般会主动终止请求。XMLHttpRequest提供了timeout属性来允许设置请求的超时时间，该属性默认等于0，表示没有时间限制。如果请求超时，将触发ontimeout事件。IE8浏览器不支持该属性。
{%codeblock lang:javascript%}
xhr.open('get', '/time', true);
xhr.ontimeout = function() {
    console.log('Request timed out.');
}
xhr.timeout = 1000;
xhr.send();
{%endcodeblock%}

## 更新网页
这一步在状态码判断中就可以很方便的完成，通过`responseText`拿到数据后，我们可以对其进行任何操作，更新到DOM元素，或者用于js内部变量的计算等。

## 写在最后
Ajax请求过程中有很多不同阶段的事件、HTTP请求头的多种定制、跨域请求等，另外还有多种高级应用，如传输二进制文件、文件上传下载及其进度等，还有待后续进一步学习和了解。