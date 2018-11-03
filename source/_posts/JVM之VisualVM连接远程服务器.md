---
title: JVM之VisualVM连接远程服务器
date: 2018-11-02 14:42:14
tags:JVM VisualVM
---

# 如何连接到远程服务器

# 常用的方法(不推荐)

## JStatD

> jstatd is a daemon that is distributed with JDK. You start it from the command line (it’s likely necessary to run it as the user running the target JVM or as root) on the target machine and VisualVM will contact it to fetch information about the remote JVMs.
>
> - Advantages: Can connect to a running JVM, no need to start it with special parameters
> - Disadvantages: Much more limited monitoring capabilities (f.ex. no CPU usage monitoring, not possible to run the Sampler and/or take thread dumps).

我们可以在JDK自带的工具里面找到`JStatD`, 然后使用`VisualVM`连接`JstatD`. 

优点:

1. JVM不必重启, 可以直接监控运行中的JVM

缺点:

1. 由于没有侵入运行中的JVM, 所以可用的功能有限
2. 需要服务器开放端口给`JstatD` 使用.



## JMX

JVM的远程管理接口

优点:

1. 可以使用`VisualVM`的全部功能

缺点:

1. 需要重启JVM, 使用以下参数

   ```
   yourJavaCommand... 
   -Dcom.sun.management.jmxremote 
   -Dcom.sun.management.jmxremote.ssl=false 
   -Dcom.sun.management.jmxremote.authenticate=false 
   -Dcom.sun.management.jmxremote.port=1098
   ```

2. 需要服务器开放`com.sun.management.jmxremote.port` 端口



上述两种方法的共通特点就是需要开放服务器端口和重启JVM, 对于开发人员来说, 这两个都是比较困难的, 下面介绍的方法可以完全避开这两个动作.



# 前置条件

1. 你可以SSH到你需要监控的服务器中
2. 你有相应的权限去安装一些软件和重启SSH服务
3. 你需要有一款Windows的X Server 软件



# 原理

## SSH隧道

当你和服务器建立SSH连接时, 你就可以借助这个SSH连接转发任意的TCP流量. 关于SSH隧道的原理可以参考这篇文章: 

[SSH TUNNEL](https://www.ssh.com/ssh/tunneling/)

![](http://liimg.oss-cn-shenzhen.aliyuncs.com/18-11-2/75189828.jpg)



建立SSH隧道之后, 流量可以进行两个方向的流动:

1. SSH Client -> SSH Server

   比如说: 你有一台WEB服务器, 上面有一个MYSQL实例在运行, 服务器开放了80端口对外进行服务, 现在你需要连接到MYSQL中去进行一些操作. 正常情况下这是不可能的, 因为服务器没有对外开放3306端口, 但是如果你可以SSH到这台服务器, 那么你就可以把你的MYSQL数据包借助SSH隧道发送到服务器. 常用的MYSQL客户端Navicat就提供这个功能.

   ![](http://liimg.oss-cn-shenzhen.aliyuncs.com/18-11-2/11888416.jpg)

2. SSH Server -> SSH Client

   这个就是我们今天的主角, 借助SSH隧道, 我们可以把服务器的流量转发的客户端. 要讲这个我们首先需要了解Linux的GUI的工作原理.


## X Window System

X 常用于Linux的GUI, 而VisualVM就是一个GUI程序, 要在远程服务器启动VisualVM, 我们必须了解X的运行原理.

首先X是一个CS架构的程序, 由服务器和客户端组成.

![](http://liimg.oss-cn-shenzhen.aliyuncs.com/18-11-2/70863260.jpg)

用户输入事件由操作系统传递给X Server, 而GUI程序从X Server中获取这些事件,并把UI变化传递给X Server, 最后X Server把这些UI展示给用户.

需要注意的有两点:

1. X Server 负责渲染UI, 也就是说如果XServer在我们本地服务器, 那么我们就可以在本地服务器中显示UI
2. X Client 负责响应事件, 并发送请求给X Server 进行UI更新





有上面的两个技术, 那么我们就可以在本地运行远程服务器中的UI程序, 比如说VisualVM, 整体的架构如下:



![](http://liimg.oss-cn-shenzhen.aliyuncs.com/18-11-2/60369001.jpg)



# 实现 



## 服务器开启 X Client流量转发

1. 编辑`/etc/ssh/sshd_config`, 修改配置项为:

   ```
   ##启用X11 Forwarding
   X11Forwarding yes
   ```

   修改之后重启sshd服务

   CentOS6  ` service sshd restart`

   CentOS7 `systemctl restart sshd`

   重启之后可以看到sshd会监听本地的6010端口, 也就是X Server的端口, 当这个端口收到流量时会转发给SSH的客户端

   ```
   [root@VM_45_207_centos ~]# netstat -anltp | grep ssh
   tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      22375/sshd          
   tcp        0      0 127.0.0.1:6010          0.0.0.0:*               LISTEN      22487/sshd: root@pt 
   tcp        0     52 10.104.45.207:22        140.240.17.92:35490     ESTABLISHED 22487/sshd: root@pt
   ```

## 服务器安装 X Client 依赖

```
yum install xorg-x11-apps # 这个包有我们需要的依赖

	Dep-Install libXaw-1.0.13-4.el7.x86_64     @os
    Dep-Install libXmu-1.1.2-2.el7.x86_64      @os
    Dep-Install libXt-1.1.5-3.el7.x86_64       @os
    Dep-Install libxkbfile-1.0.9-3.el7.x86_64  @os
    Install     xorg-x11-apps-7.7-7.el7.x86_64 @os

```

```
yum install xorg-x11-xauth # x client 和 server 的会话管理
```



## 客户端启动 X Server服务器

X Server我使用的是X Manager 自带的.



## 客户端转发 X Client 流量到 X Server服务器



![1541172879942](D:\blog\source\_posts\assets\1541172879942.png)



## 连接测试

进行上述操作之后, 理论上你就可以在本地环境中运行远程服务器的X Client 程序了, 下面是测试环节



### Cookie 测试

如果你成功的建立的X11 转发隧道, 那么登录服务器之后可以在服务器端查看到当前的会话Cookie

```
[root@VM_45_207_centos ~]# xauth list
VM_45_207_centos/unix:10  MIT-MAGIC-COOKIE-1  895715dfbd457651d243e3744d3cf52b
VM_45_207_centos/unix:11  MIT-MAGIC-COOKIE-1  892a667a2b427e12de87a75449ca73d8
```

### X app 测试

可以使用X 自带的一些小程序来检测隧道是否建立, 比如说: `xclock`

![](http://liimg.oss-cn-shenzhen.aliyuncs.com/18-11-2/28797689.jpg)





进行上述操作之后, 理论上来说你可以在本机中运行服务器的任何GUI程序, 包括Firefox, VisualVM.



# 关于VisualVM的一些坑

## 运行没有响应

运行VisualVM之后命令自动退出, 没有报错, 也没有日志. 遇到这种情况首先应该把日志打开, 然后根据日志的报错信息去排查. 命令如下:

```
./jvisualvm -J-Dnetbeans.logger.console=true
```



## 中文乱码

这个就需要安装中文字体了

```
yum groupinstall "fonts"
```

当然这种安装方式可能会安装很多不必要的包, 你可以在网上找到只安装特定字体的教程, 这里就不再累述了.