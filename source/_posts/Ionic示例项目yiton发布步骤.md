---
title: Ionic示例项目yiton发布步骤
date: 2016-03-25 22:02:02
categories: Node.js
tags: [Ionic, Android]
---
## 框架简介
[Ionic](http://ionicframework.com/)是一款基于`Node.js`的跨平台移动App开发解决方案，使用`Sass`、`AngularJS`构建App页面，内部提供了成熟的UI组件。得益于AngularJS的双向数据绑定，前台页面的展示与业务逻辑很容易实现和控制。同时[Cordova](http://cordova.apache.org/)通过`ngCordova`插件为其提供了底层API支持，如GPS、相机、文件系统等。对于前端开发者来说，能用自己熟知的JavaScript、HTML、CSS编写移动客户端是件再酷不过的事情。快速敏捷开发以及跨平台是它的主要卖点，正所谓“一次编写，到处运行”。但是从客观角度来说，相比于Native，性能是它跨不过去的坎。不过随着Node.js和HTML5的快速发展，对于业务逻辑较简单的App，这也不失为一种不错的解决方案。

## 项目简介
该系统主要试用于高校环境，为师生提供交流平台。系统设立了三种基础角色：主任、教师、学生。实现的主要功能有：
- 主任可以向指定老师和学生发送通知，并可以看到通知确认反馈情况，附带特殊情况
- 老师同样可以向指定学生发送通知，收到反馈情况；上课时可以发布签到，查看签到情况
- 学生主要是上课签到，通过拍照和GPS定位确定指定时间在指定的教室上指定的课程(定位精度尚有待斟酌)
- 以上三种角色均可在线聊天

目前该项目仅实现了Android端App，由于各种原因，服务端也并没有上线，且项目中的一些具体实现还有待考虑，所以是一个demo版本，不过基本的业务逻辑是有了。
<!--more-->
## 环境说明
系统要求：Windows 64位系统，建议用Win7 64位(说明书以win10 1511为例)。涉及的软件版本信息如下：
- JavaScript引擎 Node.js 4.4.0
- 代码管理工具 Git 2.8.0
- 开发工具 WebStrom 11.0.3
- 浏览器 Chrome 49.0.2623.110 (64-bit)
- Java运行环境 JDK-8u66 (64-bit)
- 安卓库管理工具 Android SDK tools R24.4.1

## 安装步骤
1.	安装JavaScript引擎Node.js(默认安装)
2.	打开命令提示符，执行命令：`node –v`，出现正确版本号即安装成功
3.	安装git 2.8.0(默认安装)，在命令提示符中执行命令：`git --version`，出现正确版本号即安装成功
4.	安装开发工具WebStrom 11.0.3
5.	安装Chrome浏览器
6.	安装java运行环境JDK-8u66(64-bit)，并手动将jdk的bin目录加入系统Path环境变量中，在命令提示符中执行命令：`java -version`，出现正确版本号即安装成功
7.	安装npm包源控制插件并选择国内镜像：
    1. 在命令提示符中执行命令：`npm install nrm –g`
    2. 继续执行命令：`nrm use taobao` (可执行`nrm ls`列出所有镜像，执行`nrm test`测试延迟)
8.	安装bower模块，执行命令：`npm install bower –g` (执行`bower –v`出现正确版本号即安装成功)
9.	安装构建工具gulp，执行命令：`npm install gulp –g` (执行`gulp –v`出现正确版本号即安装成功)
10.	安装cordova以及ionic框架，执行命令：`npm install cordova ionic –g` (分别执行`cordova –v`、`ionic -v`出现正确版本号即安装成功)
11.	安装android编译环境：
    1. 安装Android SDK tools R24.4.1
    2. 以管理员身份打开安装根目录中的SDK Manager
    3. 由于你懂得原因需要设置代理下载。依次选择Tools-Options，在Proxy Settings填入镜像服务器设置：
        - `Server：mirrors.neusoft.edu.cn`
        - `Port：80`
    4. 返回主界面，依次选择Packages-Reload，重新加载各模块
    5. 安装Android SDK Platform-tools r23.1、Android SDK Build-tools r23.0.2、Android Support Repository、Android Support Library、以及Android SDK Platform的以下版本：
        - Android 6.0 (API 23)
        - Android 5.1.1 (API 22)
        - Android 5.0.1 (API 21)
        - Android 4.4W (API 20)
        - Android 4.4.2 (API 19)
        - Android 4.3 (API 18)
        - Android 4.2.2 (API 17)
    
        一般只需要安装最新版本即可，由于代理网速不错且不在意那一两百兆空间，决议选择全装(强迫症泛滥)。
    6. 编辑环境变量：
        1. 新建用户变量ANDROID_HOME，指向sdk tools根目录
        2. 在用户PATH变量中添加如下值：
            - %ANDROID_HOME%\tools
            - %ANDROID_HOME%\platform-tools
            - %ANDROID_HOME%\build-tools
        3. 在系统Path变量中加入platform-tools、tools的目录
        4. 可通过`adb version`命令检查Android SDK环境是否配好

## 项目配置
1.	下载yiton示例程序并执行以下命令：
    1. 进入项目目录：`cd yiton`
    2. 添加android平台支持：`ionic platform add android`
    3. 编译成apk文件：
        - `ionic build android`
        - 会下载[http://services.gradle.org/distributions](http://services.gradle.org/distributions)编译包，网速可能不佳，需耐心等待
        - 生成的Apk安装包位于`platforms\android\build\outputs\apk`
2.	浏览器调试：
    - `ionic serve`
    - 如果启动顺利则说明ionic环境搭建成功
    - ionic服务启动后，会自动调用默认浏览器访问[http://localhost:8100/](http://localhost:8100/)
    - 按F12进入开发者控制台点击`Toggle device toolbar`，将调试设备改为手机，可自行调节分辨率
3. 模拟器调试：
    - 这里我们使用Genymotion，注意将ADB改为本地安装的Android SDK Tools
    - 使用`ionic run android`即可将App安装到Genymotion模拟器
    - 如果需要实时调试，比如打印出AngularJS中`console.log()`方法的结果，则需要开启log功能，在控制台实时打印。命令如下：`ionic run android -l -c -s` (ionic run android -livereload -consolelogs -serverlogs)
4.	真机调试：
    1. 在安卓手机上打开"usb调试模式"
    2. 通过usb数据线连接至电脑
    3. 执行命令：`adb devices`，确保该机已成功连接至电脑
    4. 执行命令：`ionic run android`，即可在该机上安装并自动运行App

## 运行截图
以下展示App主要界面截图，运行环境为Chrome DevTools的Nexus 5X设备。
![运行效果1](http://7xs1tt.com1.z0.glb.clouddn.com/blog/Ionic%E7%A4%BA%E4%BE%8B%E9%A1%B9%E7%9B%AEyiton%E5%8F%91%E5%B8%83%E6%AD%A5%E9%AA%A4/pic1.PNG)
![运行效果2](http://7xs1tt.com1.z0.glb.clouddn.com/blog/Ionic%E7%A4%BA%E4%BE%8B%E9%A1%B9%E7%9B%AEyiton%E5%8F%91%E5%B8%83%E6%AD%A5%E9%AA%A4/pic2.PNG)
![运行效果3](http://7xs1tt.com1.z0.glb.clouddn.com/blog/Ionic%E7%A4%BA%E4%BE%8B%E9%A1%B9%E7%9B%AEyiton%E5%8F%91%E5%B8%83%E6%AD%A5%E9%AA%A4/pic3.PNG)
![运行效果4](http://7xs1tt.com1.z0.glb.clouddn.com/blog/Ionic%E7%A4%BA%E4%BE%8B%E9%A1%B9%E7%9B%AEyiton%E5%8F%91%E5%B8%83%E6%AD%A5%E9%AA%A4/pic4.PNG)
![运行效果5](http://7xs1tt.com1.z0.glb.clouddn.com/blog/Ionic%E7%A4%BA%E4%BE%8B%E9%A1%B9%E7%9B%AEyiton%E5%8F%91%E5%B8%83%E6%AD%A5%E9%AA%A4/pic5.PNG)