---
title: SVN服务器及客户端配置
date: 2015-11-16 23:07:36
categories: 版本控制
tags: [SVN]
---
虽然博客使用git做代码托管比较方便也习惯了，但是真正项目中还是倾向于使用svn，它有一个全局的版本号，在服务端保存着相应时间源代码的快照，这也是git相比svn所没有的一个特性，这里不再赘述它们之间的不同点。下面详细介绍在阿里云ECS上svn服务器（ps：云翼计划真便宜）的配置以及客户端的使用。

## 服务端配置
服务端我们使用Visual Svn Server，可以进入visualsvn官网下载最新版（目前最新版为[1.9.2](https://www.visualsvn.com/server/download/)），下载完成后，一路默认安装(选择默认端口时可能需要修改)。
<!--more-->
首先来修改Network设置，在软件主界面右键点击"VisualSVN Server(Local)"，选择"properties"，选择"Network"，在"server name"中填入本机ip地址，可通过ipconfig命令查看提供商分配的公网ip。可以选择性勾选使用https加密链接：
![svn服务器配置1](http://7xs1tt.com1.z0.glb.clouddn.com/blog/SVN%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%8F%8A%E5%AE%A2%E6%88%B7%E7%AB%AF%E9%85%8D%E7%BD%AE/pic1.png)
下面来创建用户，右键点击"Users"，选择"Create User"，在下面填入用户名及密码，用于在提交和下载代码时验证，可添加多个用户：
![svn服务器配置2](http://7xs1tt.com1.z0.glb.clouddn.com/blog/SVN%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%8F%8A%E5%AE%A2%E6%88%B7%E7%AB%AF%E9%85%8D%E7%BD%AE/pic2.png)
同理，我们可以右键点击"Groups"，选择"Create Groups"，输入组名称，点击"Add"为该组添加成员，在我们刚刚添加的成员中选取然后确定即可：
![svn服务器配置3](http://7xs1tt.com1.z0.glb.clouddn.com/blog/SVN%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%8F%8A%E5%AE%A2%E6%88%B7%E7%AB%AF%E9%85%8D%E7%BD%AE/pic3.png)
![svn服务器配置4](http://7xs1tt.com1.z0.glb.clouddn.com/blog/SVN%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%8F%8A%E5%AE%A2%E6%88%B7%E7%AB%AF%E9%85%8D%E7%BD%AE/pic4.png)
下面我们建立项目文件夹，右键点击"Repositories"，选择"Create New Repositories"，type选择默认的FSFS即可，然后填入项目库名称：
![svn服务器配置5](http://7xs1tt.com1.z0.glb.clouddn.com/blog/SVN%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%8F%8A%E5%AE%A2%E6%88%B7%E7%AB%AF%E9%85%8D%E7%BD%AE/pic5.png)
![svn服务器配置6](http://7xs1tt.com1.z0.glb.clouddn.com/blog/SVN%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%8F%8A%E5%AE%A2%E6%88%B7%E7%AB%AF%E9%85%8D%E7%BD%AE/pic6.png)
然后我们在项目库里新建一个trunk文件夹存放项目源代码，右键点击项目库名称，选择"新建"，选择"Folder"，输入trunk：
![svn服务器配置7](http://7xs1tt.com1.z0.glb.clouddn.com/blog/SVN%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%8F%8A%E5%AE%A2%E6%88%B7%E7%AB%AF%E9%85%8D%E7%BD%AE/pic7.png)
右键点击trunk文件夹，选择"Copy URL to Clipboard"，这个url就是我们的项目地址，用于提交下载代码，固定不变可以记下来。

## 客户端配置
下面我们开始客户端配置介绍。客户端使用TortoiseSVN，也就是我们通常所说的小乌龟，可进入官网下载最新版(目前最新版为[1.9.2](http://tortoisesvn.net/downloads.html))。
安装过程默认就好了。我们在本地新建一个"TortoiseSVN"文件夹用于存放svn上传下载源代码，在里面新建项目文件夹，命名要与服务端项目库名称一致，否则它默认又会在该文件下新建一个项目文件夹。下面我们开始上传项目代码，右键该文件夹，选择"SVN Checkout"，在URL一栏填入项目库链接地址，点击ok进行初始化：
![svn服务器配置8](http://7xs1tt.com1.z0.glb.clouddn.com/blog/SVN%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%8F%8A%E5%AE%A2%E6%88%B7%E7%AB%AF%E9%85%8D%E7%BD%AE/pic8.png)
会提示输入用户及密码，我们输入服务端设置的用户及密码即可，勾选记住以便以后验证。初始化完毕后，我们就可以进行上传了，首先将所有项目代码都复制到该目录下，不需要上传的文件或文件夹可以右键选择TorToiseSVN -> Unversion and add to igore list。配置完后开始上传，右键项目文件夹，选择"SVN Commit"进行提交，勾选所有项目文件并点击ok：
![svn服务器配置9](http://7xs1tt.com1.z0.glb.clouddn.com/blog/SVN%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%8F%8A%E5%AE%A2%E6%88%B7%E7%AB%AF%E9%85%8D%E7%BD%AE/pic9.png)
服务端有了源代码我们就可以下载了，新建一个项目文件夹，右键选择"SVN Checkout"，填好项目URL后进行下载，阿里的下载速度还是不错的：
![svn服务器配置10](http://7xs1tt.com1.z0.glb.clouddn.com/blog/SVN%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%8F%8A%E5%AE%A2%E6%88%B7%E7%AB%AF%E9%85%8D%E7%BD%AE/pic10.png)
另外小乌龟对文件状态进行了人性化的标记，通过改变文件夹图标来显示：
![svn服务器配置11](http://7xs1tt.com1.z0.glb.clouddn.com/blog/SVN%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%8F%8A%E5%AE%A2%E6%88%B7%E7%AB%AF%E9%85%8D%E7%BD%AE/pic11.png)