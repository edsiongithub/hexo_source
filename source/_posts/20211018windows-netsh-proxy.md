---
title: windows上使用netsh映射端口至Hyper-v虚拟机
date: 2021-10-18 08:17:11
layout: 2021-10-18 08:17:11
id: 0022
tags:
 - windowns
 - netsh proxy
categories:
 - 操作系统及网络
---

前段时间折腾Hyper-v虚拟机的双网卡功能，按照某些教程，理论上一个内部网络接口，一个外部网络接口，是能够满足需求了的。


<!--more-->

微软提供了一个<a href="https://docs.microsoft.com/zh-cn/windows-server/networking/technologies/netsh/netsh">Network Shell工具</a>，官方文档上给出的定义如下：
>Network shell (netsh) 是一种命令行实用工具，使用该工具，你可以在运行 Windows Server 的计算机上安装网络通信服务器组件之后配置和显示各种网络通信服务器角色和组件的状态。

>一些客户端技术，例如动态主机配置协议 (DHCP) 客户端和BranchCache，也提供 netsh 命令，通过这些命令，你可以配置运行 Windows 10 的客户端计算机。

>在大多数情况下，netsh 命令提供的功能与你使用每个网络服务器角色或网络功能的 Microsoft 管理控制台 (MMC) 管理单元时提供的功能相同。 例如，可以使用 NPS MMC 管理单元或 netsh nps 上下文中的 netsh 命令来配置网络策略服务器 (NPS) 。


直接进入实际操作。比如我的环境如下：
* 台式主机：windows10 , IP ```202.120.xx.xx```
* Hyper-v虚拟机：Linux Ubuntu20 , IP地址```192.168.137.38```
* 笔记本：windows10, IP ```202.120.xx.xxx```
实际情况是，我只能在台式主机上连接我的Hyper-v虚拟机，用下面的命令，在台式机主机上创建一个代理，这样笔记本通过连接台式机的IP地址，即可通过台式机转发至虚拟机的特定端口上了。

把虚拟机的80端口映射出来，这样访问台式机的80端口（前提台式机的端口不能被占用），就相当于访问虚拟机的80端口了。
```
netsh interface portproxy add v4tov4 listenport=80 connectport=80 connectaddress=192.168.137.38
```
比如我想通过笔记本直接ssh连接Hyper-V上的Linux虚拟机，我只需要添加22端口的映射，在笔记本上直接ssh台式机的IP地址，即可访问到Hyper-v虚拟机了。
```
netsh interface portproxy add v4tov4 listenport=22 connectport=22 connectaddresss=192.168.137.38
```

查看所有映射情况：
```
netsh interface portproxy show all
```

删除某条端口映射
```
netsh interface portproxy delete v4tov4 listenport=5000
```