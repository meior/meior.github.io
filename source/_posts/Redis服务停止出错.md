---
title: Redis服务停止出错
date: 2017-02-24 12:31:17
categories: 数据库
tags: [Redis]
---
在本地配置了一台`Ubuntu Server 16.04`，发现关闭或者重启服务器时总是卡在stop redis_6379 service。之前自用`Redis`和`MongoDB`服务都在[HostUS](https://hostus.us/)，跑着CentOS 6，VPS从未手动关机或重启过，自然也就没有注意到Redis会出问题。既然出现了不和谐现象，就要追根溯源，彻底解决。既然`service redis_6379 stop`无效，那么只能手动通过redis-cli关闭服务了：
{%codeblock lang:bash%}
root@ubuntu:~# redis-cli -a yourpassword
127.0.0.1:6379> shutdown save
{%endcodeblock%}
但是这样关闭服务太过麻烦，并且设置了redis开机自启动后，每次关机都要先手动关闭服务，很是繁琐。那么如果将服务默认关闭方法改为调用redis-cli来完成，问题不就解决了吗？于是马上在`/etc/init.d`中找到了redis_6379，打开一看立马懵圈。原来系统默认调用的就是redis-cli，而且还是未认证方式，redis服务设了密码，当然无法关闭了。将`stop`方法改为以下内容即可：
<!--more-->
{%codeblock lang:bash%}
stop)
        if [ ! -f $PIDFILE ]
        then
            echo "$PIDFILE does not exist, process is not running"
        else
            PID=$(cat $PIDFILE)
            echo "Stopping ..."
            $CLIEXEC -a "yourpassword" -p $REDISPORT shutdown
            while [ -x /proc/${PID} ]
            do
                echo "Waiting for Redis_$REDISPORT to shutdown ..."
                sleep 1
            done
            echo "Redis_$REDISPORT stopped"
        fi
        ;;
{%endcodeblock%}
其实只是在调用`$CLIEXEC`时，使用`-a "yourpassword"`认证。OK，大功告成！服务关闭重启再无卡顿。另外将redis加入开机启动更是美滋滋：
{%codeblock lang:bash%}
root@ubuntu:~# update-rc.d redis_6379 defaults
{%endcodeblock%}