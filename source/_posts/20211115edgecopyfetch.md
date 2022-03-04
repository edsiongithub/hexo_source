---
title: 使用Chrome/Edge浏览器复制请求内容
date: 2021-11-15 21:31:44
layout: 2021-11-15 21:31:44
id: 0028
tags:
 - 前端
 - 浏览器
categories:
 - 前端
---

现代浏览器提供了大量开发工具，F12按下后，会有许多前端、前后端交互的信息展示，同时越来越多的调试工具可供开发者使用。这篇文章就记录一下Chrome浏览器（实际使用Edge截图）复制请求功能。

<!--more-->
## 缘起

在开发过程中，需要与第三方系统进行对接。其中，第三方平台提供的webapi需要接收一个超长json串，本地需要构造这个复杂的json参数。并且不提供接口文档（令人很沮丧的事情），只能在第三方平台上实际发起请求，查看请求参数。

后来自己构造的参数开始尝试提交，一直失败。于是需要将参数进一步复制出来进行比对调试。同时，还希望能在postman等工具上模拟提交。

## 解决方案

使用浏览器的F12工具箱，按照如下步骤来复制请求。
* F12，选择“网络”标签
* 筛选出Fetch/XHR类型请求（剔除js、css、图片等）
* 找到你需要的请求方法名
* 右键该方法，选择Copy，弹出子菜单
* 你会看到有一系列你想复制出来的内容以及格式
![浏览器复制Fetch内容](https://raw.githubusercontent.com/edsiongithub/blogimages/master/202111/copyfetch.png)

浏览器允许你只复制该请求的头部，或只复制链接、响应等。也可以将整个请求命令复制成命令行，支持powershell、bash等。

## 扩展--PostMan导入
将请求复制下来后，按照复制的格式可以到指定的命令工具中模拟提交。比如复制成powershell，则可以在windows中使用powershell来调试；复制成bash，可以在linux中使用shell来调试。

webapi调试的另一大图形界面调试工具：postman，此时也可以派上用场。如果你喜欢使用图形界面的调试工具，也可以将复制来的请求导入到postman中。
打开postman，在首页中点击Import ，弹出导入框。
![Import](https://raw.githubusercontent.com/edsiongithub/blogimages/master/202111/homeimport.png)

你也可以在首页中打开Create New，进入创建请求界面后，在你保存的请求会话列表右上角有一个Import，点击弹出导入框，效果同上。
![CreateNewImport](https://raw.githubusercontent.com/edsiongithub/blogimages/master/202111/createnewimport.png)

在弹出的导入框中，选择Raw text，在出现的文本框中输入之前复制的命令。
<strong><font color="red">注意:</font></strong>
按照输入框中提示命令格式，上一步中复制命令得选复制到bash中的curl命令格式。
![Fetch](https://raw.githubusercontent.com/edsiongithub/blogimages/master/202111/import.png)