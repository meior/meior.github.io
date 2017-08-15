---
title: MongoDB初学笔记
date: 2015-09-20 15:49:39
categories: 数据库
tags: [MongoDB]
---
在学习了一段时间的基于nodejs的全栈开发技术---mean框架后，对于mongo数据库也有了一定程度的认识。在接触mean之前，我使用过一段时间的sql server，对于我个人来讲至少在体验上没有mongo方便，当然这不应该是一个开发者的态度（为了追求方便）。

## 简介
[MongoDB](https://www.mongodb.org/) 是一个基于分布式文件存储的开源数据库。分布式数据库的特点在于，在高负载的情况下，添加更多的节点，可以保证服务器性能。MongoDB 将数据存储为一个文档（document），数据结构由键值对`(key=>value)`组成。MongoDB 文档类似于`JSON`对象。字段值可以包含其他文档，数组及文档数组。
{%codeblock lang:javascript%}
{
    name: 'john',
    age: 21,
    sex: 'male',
    tel: '12345678',
    hobby: ['basketball', 'soccerball', 'chess'],
    score: {
                chinese: 96,
                math: 85,
                physical: 92
           }
}
{%endcodeblock%}
Mongo支持丰富的查询表达式，查询指令使用JSON形式的标记，可轻易查询文档中内嵌的对象及数组。增删改查操作语法采用`javascript`，与`JSON`可谓是异曲同工，相比于sql语句，数据操作比较简单和容易。
<!--more-->

## 安装配置
首先在MongoDB[官网下载页](https://www.mongodb.org/downloads#production)下载对应版本，这里用`3.2.0 Windows 64-bit 2008 R2+`版来介绍。
1. 安装完毕后将安装目录下的`bin`目录加入系统`Path`环境变量中；
2. 指定数据库文件目录以及日志文件目录：
    2.1. 这里我选定`D:\Codes\MongoDB`为数据库文件根目录；
    2.2. 依次新建`data/db`目录和`data/log/mongolog.log`文件；
    2.3. 新建mongodb.bat作为mongoDB的启动脚本，添加如下内容：
    {%codeblock lang:cmd%}
    mongod --dbpath "D:\Codes\MongoDB\data\db" --logpath "D:\Codes\MongoDB\data\log\mongodb.log"
    {%endcodeblock%}
3. 双击该脚本启动mongoDB服务，以后每次都可以使用该脚本启动数据库；
4. 另开一个命令提示符窗口，输入`mongo`回车进入mongo命令行模式；
5. 为数据库建立超级用户，依次执行以下命令：
{%codeblock lang:javascript%}
use admin
db.createUser({user: 'sa', pwd: 'admin', roles: [{role: 'userAdminAnyDatabase', db: 'admin'}]})
{%endcodeblock%}
6. 将两个命令提示符窗口都关闭，编辑`mongoDB.bat`，在末尾加上`" -auth"`参数以启用用户验证。

## 简单使用
首先运行之前创建的bat脚本启动mongoDB服务，在另一个cmd窗口通过mongo命令来执行简单的增删查改操作。
这里不再新建另外的数据库，操作均在admin数据库中进行。首先使用`sa`账户登录admin数据库：
{%codeblock lang:cmd%}
mongo admin -usa -padmin
{%endcodeblock%}
新建一个use文档集合（类似于SQL中表的概念）：
{%codeblock lang:cmd%}
db.createCollection('user')
{%endcodeblock%}
查看是否创建成功：
{%codeblock lang:cmd%}
show collections
{%endcodeblock%}

### 插入数据
插入数据：
{%codeblock lang:javascript%}
db.user.insert({name: 'James', age: 20, sex: 'male'})
db.user.insert({name: 'Mary', age: 18, sex: 'female'})
db.user.insert({name: 'Taylor', age: 26, sex: 'female'})
{%endcodeblock%}
这时user集合中已经有了3条数据，分别记录着3人的姓名等信息，使用find函数查询数据：

### 查询数据
显示所有用户：
{%codeblock lang:javascript%}
db.user.find()
{%endcodeblock%}
查询年龄在20-30之间的用户：
{%codeblock lang:javascript%}
db.user.find({age: {'$gte': 18, '$lte': 30}})
{%endcodeblock%}

### 删除数据
将Mary从集合中删除：
{%codeblock lang:javascript%}
db.user.remove({name: 'Mary'})
{%endcodeblock%}

### 修改数据
修改Taylor的年龄为32：
{%codeblock lang:javascript%}
db.user.update({name: 'Taylor'}, {$set: {age: 32}})
{%endcodeblock%}

## 高级查询及数据操作
上述简单使用仅仅是针对一些单值，然而在实际开发中这些是远远不够的，实际的数据场景要复杂得多，为此MongoDB内置了一些操作运算符以满足各种需求。
### 操作运算符
* 条件比较：大于`$gt`、大于等于`$gte`、小于`$lt`、小于等于`$lte`、不等于`$ne`、非`$not`。
关系判断：包含于`$in`、不包含于`$nin`、全部属于`$all`。
* 高级查找：取模`$mod`、或`$or`、且`$and`、异或`$nor`、数量`$size`、字段存在`$exists`、字段类型`$type`、条件组合查询`$elemMatch`、自定义函数查询`$where`。
* 操作符：截取数组元素`$slice`、增减数值型的值`$inc`、添加数组元素`$push`、无重复添加数组元素`$addToSet`、头或尾删除数组元素`$pop`、删除指定数组元素`$pull`、更新创建键值`$set`、删除键`$unset`。

### 返回数据处理
可以限制结果集返回部分字段，字段值等于1时表示需要返回,0时表示不需要返回：
{%codeblock lang:javascript%}
db.person.find({}, {status:1,age:1})
{%endcodeblock%}
find返回json对象数据，可以采用以下方法遍历：
{%codeblock lang:javascript%}
// next方式
var myCursor = db.inventory.find( { type: 'food' } );
var myDocument = myCursor.hasNext() ? myCursor.next() : null;
if (myDocument) {
    var myItem = myDocument.item;
    print(tojson(myItem));
}

// forEach循环遍历
var myCursor = db.inventory.find( { type: 'food' } );
myCursor.forEach(printjson);
{%endcodeblock%}
### 嵌套查询
MongoDB还支持文档数组以及内嵌文档的数据结构。如果采用嵌套方式存储数据，使用`update`修改数据时应该留意。假设现有如下一条记录在数据库中：
{%codeblock lang:javascript%}
{
    name:'李四',
    sex:'男',
    age:23,
    score:[
       {
           semester:1,
            chinese:95,
            math:89,
            english:83,
            program:90
        },
       {
           semester:2,
            chinese:90,
            math:96,
            english:86,
            program:90
        }
    ]
}
{%endcodeblock%}
那么修改李四第一学期的英语成绩并在控制台打印他的所有成绩，可以采用以下方法：
{%codeblock '在Nodejs的mongoose模块下操作示例代码' lang:javascript%}
Model.update({name:'李四',"score.semester":1},{$set:{"score.$.english":92}},function(err,res){
    if(!err) {
        console.log("修改成功！");
        Model.find({name:"李四"},function(err,res){
            console.log(res);
        });
    }
});
{%endcodeblock%}

更多详细资料请参看MongoDB[官方文档](https://docs.mongodb.org/manual/)。
