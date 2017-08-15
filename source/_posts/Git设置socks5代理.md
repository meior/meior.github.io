---
title: Git设置socks5代理
date: 2017-03-05 16:24:39
categories: 版本控制
tags: [Git, GitHub, Proxy]
---
[GitHub](https://github.com/)在国内访问一直都很慢，稍大一点的项目就无法clone，严重时甚至无法push。平时一直在用ss，正好就用它给`Git`设置代理。一般步骤很简单，使用git命令配置：
{%codeblock lang:bash%}
# ss服务本地默认端口为1080
git config --global http.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080
# 也可将http/https改为socks5协议，如：socks5://127.0.0.1:1080

# 若http代理需要认证，需加上用户名:密码
git config –global http.proxy http://user:password@127.0.0.1:1080
git config –global http.proxy https://user:password@127.0.0.1:1080

# 删除http代理
git config --global --unset http.proxy
git config --global --unset https.proxy
{%endcodeblock%}
事实上，代理设置好后下载速度直接达到带宽满速，但是仅仅只适用于`http`或者`https`协议的Git地址，对于`git@`开头的`ssh`协议地址无效。通过ssh key与GitHub通信更加便捷，不用每次push都输入账号和密码，并且ssh协议也更加安全。显然无法直接在不同协议之间代理，那么就需要转换数据包。
<!--more-->
幸好ssh提供了`ProxyCommand`选项，它指定一个可执行程序作为媒介，用于ssh客户端与服务器之间的隧道通信，即将连接的两个端口各自的出入口截获, 就可以用其它的信道来传输数据，从而达到代理的效果。Git提供了这样的媒介（`connect.exe`），它位于Git安装路径/mingw64/bin下。

ss客户端除了http协议也提供了socks5协议服务，地址和端口不变，并且会话层的socks5相比于应用层的http效率更高，因此socks5是个不错的选择。只需在`~/.ssh/`目录下添加`config`文件，无扩展名，内容如下：
{%codeblock lang:bash%}
# 在Git环境中可以直接使用connect命令，不必指定其路径
# -H：使用http代理，-S：使用socks5代理
ProxyCommand connect -S 127.0.0.1:1080 %h %p

# 需要指定用于身份认证的私钥，即id_rsa
Host github.com
  User git
  Port 22
  Hostname github.com
  IdentityFile "C:\Users\winusername\.ssh\id_rsa"
  TCPKeepAlive yes
  IdentitiesOnly yes

Host ssh.github.com
  User git
  Port 443
  Hostname ssh.github.com
  IdentityFile "C:\Users\winusername\.ssh\id_rsa"
  TCPKeepAlive yes
  IdentitiesOnly yes
{%endcodeblock%}
配置完成后，使用`ssh git@github.com`命令测试效果。上述方法以`Windows`为例，其他平台只需替换和指定可执行程序connect以及ssh私钥的路径。