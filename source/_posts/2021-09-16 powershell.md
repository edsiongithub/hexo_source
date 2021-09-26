---
title: windows powershell 常用功能
date: 2021-09-17 10:04:00
layout: 2021-09-17
tags: 
 - powershell
---

使用windowns操作系统的，对cmd命令提示符不会陌生。powershell可以看作cmd的升级版，可以通过powershell命令做许多自动化管理。这里记录在powershell使用过程中遇到的一些问题解决，以及一些常用的使用命令。

<!--more-->

### powershell执行策略错误
win10中执行powershell脚本出现如下错误：
```
Activate.ps1，因为在此系统上禁止运行脚本。有关详细信息，请参阅 https:/go.microsoft.com/fwlink/?LinkID=135170 中的 about_Execution_Policies。
```
powershell的执行策略问题，使用get-executionpolicy查看策略

```
Restricted 执行策略不允许任何脚本运行。  

AllSigned 和 RemoteSigned 执行策略可防止 Windows PowerShell 运行没有数字签名的脚本。

 本主题说明如何运行所选未签名脚本（即使在执行策略为 RemoteSigned 的情况下），还说明如何对  脚本进行签名以便您自己使用。

有关 Windows PowerShell 执行策略的详细信息，请参阅 about_Execution_Policy。
```
使用set-executionpolicy设置策略，以管理员身份启动powershell，执行以下命令： ``` set-executionpolicy RemoteSigned```


### 通过powershell使用ssh连接Linxu

安装ssh工具，然后连接服务器即可。
``` bash
iwr https://chocolatey.org/install.ps1 -UseBasicParsing | iex

choco install openssh

#ssh连接，输入下面命令回车，再输入密码即可
ssh username@hostname(IP)
```

### 修改powershell背景颜色
powershell默认背景色为蓝色，连接到Linux后，由于linux中的目录字体颜色也是蓝色，会导致文字很难看清。进入```C:\Users\username\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Windows PowerShell```目录，右键powershell的快捷方式，弹出的属性框中选择颜色选项卡，屏幕背景色选择黑色即可。

![修改powershell背景色](https://gitee.com/gxwang/blogimages/blob/master/20210916/backgroundcolor.png)

