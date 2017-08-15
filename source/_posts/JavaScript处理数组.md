---
title: JavaScript处理数组
date: 2015-09-07 12:31:28
categories: JavaScript
tags: [Array, JavaScript]
---
JavaScript只支持一维数组，但其作为一门弱类型语言，数据类型相对灵活，如果使保存在数组中的元素是数组的话，就可以轻松创建多维数组。

## 二维数组
在C语言中我们已经熟悉二维数组的概念，二维数组类似一种由行和列构成的数据表格。
### 创建二维数组
在JavaScript中创建二维数组，按照我们刚才的想法需要先创建一个数组，然后让数组的每个元素也是一个数组。最起码，我们需要知道二维数组要包含多少行，有了这个信息，就可以创建一个n行1列的二维数组了：
{%codeblock lang:javascript%}
var array = [];
var rows = 5;
for (var i = 0; i < rows; i++) {
    array[i] = [];
}
{%endcodeblock%}
这样做的弊端在于数组中的每一个元素都是undefined，这是不规范的。为此`JavaScript的大宗师`[Douglas Crockford](http://baike.baidu.com/link?url=vu5nHoLOHZPWozaaib9FATFTuma1cFI93Tl29FXCl1oWCo6bYqKsKqRdfEg6ZXJwZuT0iX7m5rAgG5POVzs9wK)通过扩展JavaScript数组对象，为其增加了一个新方法来创建二维数组并初始化数组元素，该方法根据传入的参数，设定了数组的行数、列数和初始值。
<!--more-->
{%codeblock lang:javascript%}
Array.matrix = function(numrows, numcols, initial) {
    var arr = [];
    for (var i = 0; i < numrows; ++i) {
        var columns = [];
        for (var j = 0; j < numcols; ++j) {
            columns[j] = initial;
        }
        arr[i] = columns;
    }
    return arr;
}
{%endcodeblock%}
测试一下代码：
{%codeblock lang:javascript%}
var nums = Array.matrix(5, 5, 0);
console.log(nums[1][1]); //输出 0

var names = Array.matrix(3, 3, "");
names[1][2] = "MeiYL";
console.log(names[1][2]); //输出 MeiYL
{%endcodeblock%}
其实还可以仅用一行代码就创建并且使用一组初始值来初始化一个二维数组（更适用于小规模数据）：
{%codeblock lang:javascript%}
var scores = [[86, 77, 78],[76, 85, 81],[92, 94, 89]];
console.log(scores[1][2]); //输出 81
{%endcodeblock%}

### 处理二维数组元素
处理二维数组元素有两种最基本的的方式：按列访问和按行访问。对于两种方式，我们均使用一组嵌入式的for循环。对于按列访问，外层循环对应行，内层循环对应列。以数组scores为例，每一行对应一个学生的成绩记录。我们可以将该学生的所有成绩相加，然后除以科目数得到该学生的平均成绩。
{%codeblock lang:javascript%}
var scores = [[89, 77, 78],[76, 82, 81],[91, 94, 89]];
var total = 0;
var average = 0.0;
for (var row = 0; row < scores.length; row++) {
    for (var col = 0; col < scores[row].length; col++) {
        total += scores[row][col];
    }
    average = total / scores[row].length;
    console.log("Student " + parseInt(row+1) + " average: " + average.toFixed(2));
    total = 0;
    average = 0.0;
}
{%endcodeblock%}
测试输出结果：
{%codeblock lang:javascript%}
Student 1 average: 81.33
Student 2 average: 79.67
Student 3 average: 91.33
{%endcodeblock%}
对于按行访问，只需要稍微调整for循环的顺序，使外层循环对应列，内层循环对应行即可。

### 参差不齐的数组
参差不齐的数组是指数组中每行的元素个数彼此不同。有一行可能包含三个元素，另一行可能包含五个元素，有些行甚至只包含一个元素。很多编程语言在处理这种参差不齐的数组时表现都不是很好，但是JavaScript却表现良好，因为每一行的长度是可以通过计算得到的。为了给个示例，假设数组scores中，每个学生成绩记录的个数是不一样的，不用修改代码，依然可以正确计算出正确的平均分：
{%codeblock lang:javascript%}
var scores = [[89, 77],[76, 82, 81],[91, 94, 89, 99]];
var total = 0;
var average = 0.0;
for (var row = 0; row < scores.length; row++) {
    for (var col = 0; col < scores[row].length; col++) {
        total += scores[row][col];
    }
    average = total / scores[row].length;
    console.log("Student " + parseInt(row+1) + " average: " + average.toFixed(2));
    total = 0;
    average = 0.0;
}
{%endcodeblock%}
在内层的for循环中计算了每个数组的长度，即使数组中每一行的长度不一，程序依然不会出什么问题。该段程序的输出为：
{%codeblock lang:javascript%}
Student 1 average: 83.00
Student 2 average: 79.67
Student 3 average: 93.25
{%endcodeblock%}

## 对象数组
数组还可以包含对象，数组的方法和属性对对象依然适用。数组常用的操作如：`push`、`pull`、`shift`、`unshift`、`splice`等。
{%codeblock lang:javascript%}
function Point(x,y) {
    this.x = x;
    this.y = y;
}
function displayPts(arr) {
    for (var i = 0; i < arr.length; ++i) {
        print(arr[i].x + ", " + arr[i].y);
    }
}
var p1 = new Point(1,2);
var p2 = new Point(3,5);
var p3 = new Point(2,8);
var p4 = new Point(4,4);
var points = [p1,p2,p3,p4];
for (var i = 0; i < points.length; ++i) {
    console.log("Point " + parseInt(i+1) + ": " + points[i].x + ", " + points[i].y);
}
var p5 = new Point(12,-3);
points.push(p5);
console.log("After push: ");
displayPts(points);
points.shift();
console.log("After shift: ");
displayPts(points);
{%endcodeblock%}
测试代码输出：
{%codeblock lang:javascript%}
Point 1: 1, 2
Point 2: 3, 5
Point 3: 2, 8
Point 4: 4, 4
After push:
1, 2
3, 5
2, 8
4, 4
12, -3
After shift:
3, 5
2, 8
4, 4
12, -3
{%endcodeblock%}

## 对象中的数组
JavaScript对象非常灵活，可以使用数组存储复杂的数据。我们创建一个用于保存观测到的周最高气温的对象，该对象有两个方法，一个方法用来增加一条新的气温记录，另外一个方法用来计算存储在对象中的平均气温。代码如下所示：
{%codeblock lang:javascript%}
function weekTemps() {
    this.dataStore = [];
    this.add = add;
    this.average = average;
}
function add(temp) {
    this.dataStore.push(temp);
}
function average() {
    var total = 0;
    for (var i = 0; i < this.dataStore.length; i++) {
        total += this.dataStore[i];
    }
    return total / this.dataStore.length;
}
var thisWeek = new weekTemps();
thisWeek.add(52);
thisWeek.add(55);
thisWeek.add(61);
thisWeek.add(65);
thisWeek.add(55);
thisWeek.add(50);
thisWeek.add(52);
thisWeek.add(49);
console.log(thisWeek.average()); //输出 54.875
{%endcodeblock%}