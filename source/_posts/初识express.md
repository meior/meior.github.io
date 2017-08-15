---
title: 初识express
date: 2015-09-12 12:42:40
categories: Node.js
tags: [Express]
---
## express与http
express是一款基于Node.js的服务器端开发框架，它能够让我们轻松灵活地构建轻量级web应用以及移动端服务，让mean真正的在JavaScript栈上统一和联系了起来。出自[tj大神](https://github.com/tj)之手，自然是非同凡响。

### 使用http搭建简易web服务器
Node.js自身有http的模块，同样可以提供web服务，为什么还要用express呢？我们首先来看看使用Node.js底层的http如何来搭建一个web服务：
{%codeblock server.js lang:javascript%}
// 加载http模块
var http = require('http');

// 创建Server
var app = http.createServer(function(req, res) {
  res.writeHead(200, {
    "Content-Type": "text/plain"
  });
  res.end("Welcome to Node.js!\n");
});

// 启动Server
app.listen(8000);
{%endcodeblock%}
运行server.js，在浏览器端访问[http://localhost:8000](http://localhost:8000)即可看到“Welcome to Node.js!”。
<!--more-->
如果用户请求不同的url，如/home，/about，/user:username等页面，自己实现可能就需要作如下处理了：
{%codeblock server.js lang:javascript%}
// 加载http模块
var http = require('http');

// 创建Server
var app = http.createServer(function(req, res) {
  if(req.url == '/'){
    res.writeHead(200, { "Content-Type": "text/plain" });
    res.end("Home Page!\n");
  } else if(req.url == '/about'){
    res.writeHead(200, { "Content-Type": "text/plain" });
    res.end("About Page!\n");
  } else if(req.url == '/user:john'){
    if(req.method == 'get'){
        res.writeHead(200, { "Content-Type": "text/plain" });
        res.end("This is john!\n");
    }
    if(req.method == 'delete'){
        // 具体删除操作省略
        delete('john');
        res.writeHead(200, { "Content-Type": "text/plain" });
        res.end("john has been deleted!\n");
    }
  } else {
    res.writeHead(404, { "Content-Type": "text/plain" });
    res.end("404 Not Found!\n");
  }
});

// 启动Server
app.listen(8000);
{%endcodeblock%}
这样看来简单的几个页面请求就带来了如此多的if else语句，不仅在代码量上冗杂，而且是一种非正规的web服务思想，很不利于开发。

### express之用户登录
那么我们看看express是如何处理这些请求的，做一个简单的登录认证，建立两个文件：
{%codeblock server.js lang:javascript%}
var express = require('express');
var app = express();

/**
*express依赖body parser中间件对请求的包体进行解析，默认支持三种类型的类型：
*application/json, application/x-www-form-urlencoded, multipart/form-data。
*/
var bodyParser = require('body-parser');

//express应用程序的配置
//使用“body-parser”中间件抓取客户端的POST请求的信息
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());

//设置返回数据的响应头信息（未设置状态码）
app.use(function(req, res, next) {
    res.setHeader('Access-Control-Allow-Origin', '*');
    res.setHeader('Access-Control-Allow-Methods', 'GET, POST');
    res.setHeader('Access-Control-Allow-Headers', 'X-Requested-With,content-type, Authorization');
    //使用next方法前往下一个中间件
    next();
});

//设置api路由
var apiRoutes = require('./api')(app, express);
app.use('/api', apiRoutes);

// 启动Server
app.listen(8000);
{%endcodeblock%}
路由文件：
{%codeblock api.js lang:javascript%}
var jwt = require('jsonwebtoken');
module.exports = function(app, express) {

    //定义一个路由
    var apiRouter = express.Router();
    //认证路由
    apiRouter.post('/authenticate', function(req, res) {
        /**
        *这里省略在数据库中查询用户，假定user即为数据库返回的数据
        *如果未检索到含提交的用户名的记录，则返回一个json文件，包含认证状态及页面显示提示信息
        */
        if (!user) {
          res.json({
            success: false,
            message: 'Authentication failed. User not found.'
            });
        } else if (user) {

            //如果检索到提交的用户名的记录，则比较数据库中保存的密码和客户端提交的密码的一致性
            if (req.body.password != user.password) {
            //如果密码匹配不一致，返回json文件
            res.json({
                success: false,
                message: 'Authentication failed. Wrong password.'
            });
            } else {

            //如果用户存在且密码匹配正确
            //则创建一个token并记录此次认证的用户名及密码
            var token = jwt.sign({
                name: user.name,
                username: user.username
            }, 'superSecret', {
                //设置认证状态有效时间为1440分钟(24小时)
                //认证后24内可直接进入系统无需登录，24小时后需重新认证登录
                expiresInMinutes: 1440
            });

            //返回认证登录成功信息，并将此用户的token也返回
            res.json({
              success: true,
              message: 'Enjoy your token!',
              token: token
            });
          }
        }
      });

    //为路由自定义token认证中间件
    apiRouter.use(function(req, res, next) {

          console.log('Somebody just came to our app!');
          //检查post请求解析出来的token字段、post地址的token字段或者请求头中包含的token字段
          var token = req.body.token || req.param('token') || req.headers['x-access-token'];
          if (token) {

            //拿到token后按照自定义密码对其进行解码
            jwt.verify(token, 'superSecret', function(err, decoded) {
              if (err) {
                //解码失败，向客户端返回错误错误代码和信息
                res.status(403).send({
                    success: false,
                    message: 'Failed to authenticate token.'
                });
              } else {

                //返回解码成功结果
                req.decoded = decoded;
                //token认证成功后进入下一个中间件
                next();
              }
            });
          } else {

            //如果从请求中未找到token，说明保存的token认证失效
            //返回错误代码和信息
            res.status(403).send({
                success: false,
                message: 'No token provided.'
            });
          }
    });
    return apiRouter;
}
{%endcodeblock%}
运行server.js，向[http://localhost:8000/authenticate](http://localhost:8000/authenticate)提交post请求，请求内容为username、password。这里可以使用Chrome插件`Postman`进行post测试。

## express特性
上述代码中独特的异步处理、路由管理（`Route`）和中间件（`Middleware`），是express的特点，更加符合web服务开发思维，帮助我们更便捷的构建web应用。另外视图（`View`）也是express机制之一，为客户端设置静态目录并返回基础的index.html界面，那么所有其他客户端页面都是基于该基础页面来渲染的，如在上述代码中express应用的基础上为客户端返回首页：
{%codeblock lang:javascript%}
var path = require('path');
/**
*设置静态文件的查找目录
*public目录下存放前端资源页面
*/
app.use(express.static(__dirname + '/public'));

/**
*假设index.html为首页
*所有get请求都返回首页文件，页面布局处理交给服务端
*/
app.get('*', function(req, res) {
    res.sendFile(path.join(__dirname + '/public/index.html'));
});
{%endcodeblock%}
如果使用像`jade`一样的引擎，也可以类似的设置模板目录：
{%codeblock lang:javascript%}
// 模板目录：./public
app.set('views', __dirname + '/public');

// 使用jade引擎
app.set('view engine', 'jade');
 
// 寻找views/index.html，提交jade渲染，并返回结果
app.get('/', function(req, res) {
    res.render('index', {message: 'This is root page!'});
});
{%endcodeblock%}

## 参考资料
* [express 4.x文档](http://expressjs.com/en/4x/api.html)
* [express 3.x文档](http://expressjs.com/en/3x/api.html)
* [Understanding Express.js](http://evanhahn.com/understanding-express/)
* [Node.js - Express Framework](http://www.tutorialspoint.com/nodejs/nodejs_express_framework.htm)
* [A short guide to Connect Middleware](https://stephensugden.com/middleware_guide/)