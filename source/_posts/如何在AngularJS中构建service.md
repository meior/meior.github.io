---
title: '[转]如何在AngularJS中构建service'
date: 2015-10-16 12:50:45
categories: JavaScript
tags: [AngularJS]
---
在AngularJS里面，`services`作为单例对象在需要到的时候被创建，只有在应用生命周期结束的时候(关闭浏览器)才会被清除。而`controllers`在不需要的时候就会被销毁了。这就是为什么使用controllers在应用里面传递数据不可靠的原因，特别是使用routing的时候。就是说services在应用的controllers、 方法、数据之前起到了很关键的作用。
现在我们开始看怎么创建service。每个方法我们都会看到下面两个一样的参数：
* name-我们要定义的service的名字；
* function-service方法；
  他们都创建了相同的底层对象类型。实例化后，他们都创建了一个service，这些对象没有什么功能上的差别。

## factory()
AngularJS里面创建service最简单的方式是使用factory()方法。factory()让我们通过返回一个包含service方法和数据的对象来定义一个service。在service方法里面我们可以注入services，比如`$http`和`$q`等。
{%codeblock lang:javascript%}
angular.module('myApp.services')
.factory('User', function($http) { // injectables go here
  var backendUrl = "http://localhost:3000";  var service = {    // our factory definition
    user: {},
    setName: function(newName) { 
      service.user['name'] = newName; 
    },
    setEmail: function(newEmail) {
      service.user['email'] = newEmail;
    },
    save: function() {
      return $http.post(backendUrl + '/users', {
        user: service.user
      });
    }
  };  return service;
});
{%endcodeblock%}
### 在应用里面使用factory()方法
在应用里面可以很容易地使用factory ，需要到的时候简单地注入就可以了。
<!--more-->
{%codeblock lang:javascript%}
angular.module('myApp')
.controller('MainCtrl', function($scope, User) {
  $scope.saveUser = User.save;
});
{%endcodeblock%}
### 什么时候使用factory()方法
在service里面当我们仅仅需要的是一个方法和数据的集合且不需要处理复杂的逻辑的时候，factory()是一个非常不错的选择。
**_注意：需要使用.config()来配置service的时候不能使用factory()方法。_**

## service()
service()通过构造函数的方式让我们创建service，我们可以使用原型模式替代javaScript原始的对象来定义service。和factory()方法一样我们也可以在函数的定义里面看到服务的注入。
{%codeblock lang:javascript%}
angular.module('myApp.services')
.service('User', function($http) { // injectables go here
  var self = this; // Save reference
  this.user = {};
  this.backendUrl = "http://localhost:3000";
  this.setName = function(newName) {
    self.user['name'] = newName;
  }
  this.setEmail = function(newEmail) {
    self.user['email'] = newEmail;
  }
  this.save = function() {
    return $http.post(self.backendUrl + '/users', {
      user: self.user
    })
  }
});
{%endcodeblock%}
这里的功能和使用factory()方法的方式一样，service()方法会持有构造函数创建的对象。
### 在应用里面使用service()方法
{%codeblock lang:javascript%}
angular.module('myApp')
.controller('MainCtrl', function($scope, User) {
  $scope.saveUser = User.save;
});
{%endcodeblock%}
### 什么时候适合使用service()方法
service()方法很适合使用在功能控制比较多的service里面
**_注意：需要使用.config()来配置service的时候不能使用service()方法。_**

## provider()
provider()是创建service最底层的方式，这也是唯一一个可以使用.config()方法配置创建service的方法。不像上面提到的方法那样，我们在定义的`this.$get()`方法里面进行依赖注入。
{%codeblock lang:javascript%}
angular.module('myApp.services')
.provider('User', function() {
  this.backendUrl = "http://localhost:3000";
  this.setBackendUrl = function(newUrl) {
    if (url) this.backendUrl = newUrl;
  }
  this.$get = function($http) { // injectables go here
    var self = this;
    var service = {
      user: {},
      setName: function(newName) {
        service.user['name'] = newName;
      },
      setEmail: function(newEmail) {
        service.user['email'] = newEmail;
      },
      save: function() {
        return $http.post(self.backendUrl + '/users', {
          user: service.user
        })
      }
    };
    return service;
  }
});
{%endcodeblock%}
### 在应用里面使用provider()方法
为了给service进行配置，我们可以将provider注入到.config()方法里面。
{%codeblock lang:javascript%}
angular.module('myApp')
.config(function(UserProvider) {
  UserProvider.setBackendUrl("http://myApiBackend.com/api");
})
{%endcodeblock%}
这样我们就可以和其他方式一样在应用里面使用这个service了。
{%codeblock lang:javascript%}
angular.module('myApp')
.controller('MainCtrl', function($scope, User) {
  $scope.saveUser = User.save;
});
{%endcodeblock%}
### 什么时候使用provider()方法
1. 当我们希望在应用开始前对service进行配置的时候就需要使用到provider()。比如，我们需要配置services在不同的部署环境里面（开发，演示，生产）使用不同的后端处理的时候就可以使用到了。
2. 当我们打算发布开源provider()也是首选创建service的方法，这样就可以使用配置的方式来配置services而不是将配置数据硬编码写到代码里面。

[转载自http://my.oschina.net/tanweijie/blog/295067](http://my.oschina.net/tanweijie/blog/295067)