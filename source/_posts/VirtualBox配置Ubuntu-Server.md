---
title: VirtualBox配置Ubuntu Server
date: 2017-02-18 14:15:36
categories: 服务器
tags: [Ubuntu]
---
自用的server一直都在[HostUS](https://hostus.us/)，因为实在便宜呀，一年12$，流量带宽都很富裕。主要用来跑一些代理，`socket5`、`pptp`等，有时为个人项目跑一些`node.js`服务。在本地搭建一个主要为了学习和各种模拟试验，于是想到了免费开源的VirtualBox。  
安装过程就不多说了，本文主要介绍ubuntu server的网络配置，以及通过xshell连接。

## 系统安装
在VirtualBox中新建虚拟机后，将存储-光盘设置为本地下载好的iso镜像文件，其他设置默认就好。镜像文件可以在[Ubuntu官网](https://www.ubuntu.com/download/server)获取。通过镜像安装很方便，安装过程跟着向导走没毛病。

## 设置root密码
此时安装好的server只有在安装界面设置的一个用户，root用户并没有设置密码，每次开机root密码都是随机的。可通过如下方法手动指定：
{%codeblock lang:bash%}
sudo passwd
{%endcodeblock%}
系统提示输入当前用户密码，输入并回车后，提示输入UNIX密码，此时输入的密码即为root密码。
<!--more-->
## 配置双网卡
VirtualBox提供多种网络方式，NAT、桥接、Host-Only等。为了使server能访问外网，需要有一块NAT网卡；为了可以在外部使用xshell访问，需要有一块Host-Only网卡，用于与主机双向通信。设置很方便，在server关机状态下，在网络设置中将网卡一连接方式选为Host-Only，网卡二选择网络地址转换（NAT）。启动server，以root身份登录。编辑`/etc/network/interfaces`，修改为如下内容：
{%codeblock lang:bash%}
source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto enp0s3
iface enp0s3 inet static
address 192.168.56.101
netmask 255.255.255.0

# The user added network interface
auto enp0s8
iface enp0s8 inet dhcp
{%endcodeblock%}
这里我们指定enp0s3网卡的IP地址，是为了方便xshell连接以及主机其他客户软件连接。保存退出后，重启server生效。

## xshell连接
为什么使用xshell，因为方便快捷（可以复制粘贴），VirtualBox的黑色界面实在是不太友好。有了xshell，我们启动server后就不用管它了，保持窗口开着就好。由于sshd默认不允许远程root用户登录，所以还需更改设置，编辑`/etc/ssh/sshd_config`，修改`PermitRootLogin`：
{%codeblock lang:bash%}
# Authentication:
LoginGraceTime 120
PermitRootLogin yes
StrictModes yes
{%endcodeblock%}
保存退出后重启server生效，在xshell中就可以顺利访问`192.168.56.101`了。

## 替换国内源
系统默认源和你安装时选的位置有关，选择HongKong则为`hk.archive.ubuntu.com`。为了提高访问速度将其改为国内镜像，这里采用aliyun镜像。首先备份默认源：
{%codeblock lang:bash%}
cp /etc/apt/sources.list /etc/apt/sources.list_backup
{%endcodeblock%}
将`/etc/apt/sources.list`改为以下内容：
{%codeblock lang:bash%}
# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://mirrors.aliyun.com/ubuntu xenial main restricted
# deb-src http://mirrors.aliyun.com/ubuntu xenial main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://mirrors.aliyun.com/ubuntu xenial-updates main restricted
# deb-src http://mirrors.aliyun.com/ubuntu xenial-updates main restricted

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://mirrors.aliyun.com/ubuntu xenial universe
# deb-src http://mirrors.aliyun.com/ubuntu xenial universe
deb http://mirrors.aliyun.com/ubuntu xenial-updates universe
# deb-src http://mirrors.aliyun.com/ubuntu xenial-updates universe

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu 
## team, and may not be under a free licence. Please satisfy yourself as to 
## your rights to use the software. Also, please note that software in 
## multiverse WILL NOT receive any review or updates from the Ubuntu
## security team.
deb http://mirrors.aliyun.com/ubuntu xenial multiverse
# deb-src http://mirrors.aliyun.com/ubuntu xenial multiverse
deb http://mirrors.aliyun.com/ubuntu xenial-updates multiverse
# deb-src http://mirrors.aliyun.com/ubuntu xenial-updates multiverse

## N.B. software from this repository may not have been tested as
## extensively as that contained in the main release, although it includes
## newer versions of some applications which may provide useful features.
## Also, please note that software in backports WILL NOT receive any review
## or updates from the Ubuntu security team.
deb http://mirrors.aliyun.com/ubuntu xenial-backports main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu xenial-backports main restricted universe multiverse

## Uncomment the following two lines to add software from Canonical's
## 'partner' repository.
## This software is not part of Ubuntu, but is offered by Canonical and the
## respective vendors as a service to Ubuntu users.
# deb http://archive.canonical.com/ubuntu xenial partner
# deb-src http://archive.canonical.com/ubuntu xenial partner

deb http://mirrors.aliyun.com/ubuntu xenial-security main restricted
# deb-src http://mirrors.aliyun.com/ubuntu xenial-security main restricted
deb http://mirrors.aliyun.com/ubuntu xenial-security universe
# deb-src http://mirrors.aliyun.com/ubuntu xenial-security universe
deb http://mirrors.aliyun.com/ubuntu xenial-security multiverse
# deb-src http://mirrors.aliyun.com/ubuntu xenial-security multiverse
{%endcodeblock%}
deb-src部分默认注释掉了，需要源码或者编译时可以加上。至此，基本配置已经完成，可以作为试验和开发环境了。