---
title: Linux上创建开发asp.net core应用
date: 2021-06-03 10:04:00
reward: true
id: 0012
tags: 
 - linux
 - asp.net core
 - vs code
 - 远程开发
---

前面的一篇文章，已经将windows中使用Visual Studio 2019创建并开发的 ```asp.net core```应用部署至一台Linux服务器中，一睹dotnet core的跨平台威力。接下来，我们直接在Linux主机上创建项目，并编译执行。同时，使用VS Code在windows平台上实现远程开发。
## 目标

* Linux主机上使用dotnet cli创建应用
* 使用dotnet命令执行
* 通过Windows平台上VS Code连接到Linux上的项目进行远程开发

<!--more-->
---
## 步骤

### 创建项目
使用dotnet cli工具创建项目，控制台中使用如下命令
``` bash
cd netcoreapp
dotnet new mvc 
```
如果不知道需要创建的项目类型缩写是什么，可以直接使用 ```dotnet new``` 回车后，会显示所有提供的模板类型。注意，创建项目时，不需要项目名称，你在哪个目录下创建，就会生成和目录名相同的项目。所以提前规划好你的项目目录。

你可以在目录中使用ls命令，查看创建的项目结构，和Windows下使用Visual Studio创建的项目结构是一样的。

![创建项目](https://raw.githubusercontent.com/edsiongithub/blogimages/master/dotnetnew.png)

### 编译生成并运行

接下来，我们使用命令工具来编译并执行项目。为了有点效果，我们先用Vim编辑器修改HomeController和对应的View，让控制器传递一条消息给视图（ViewBag）。

![修改代码](https://raw.githubusercontent.com/edsiongithub/blogimages/master/changecodes.png)

执行```dotnet run``` 命令，运行项目，注意观察输出信息。 由于我本机已经有一个dotnet core应用在运行中，且占用了5000端口，因此这个项目运行失败了，错误信息中提示了。我们可以修改一下配置文件，让项目启动的默认地址换一个端口。
![错误](https://raw.githubusercontent.com/edsiongithub/blogimages/master/error.png)

进入项目中Properties目录，下面有一个launchSettings.json文件，使用Vim打开，修改dotnetcore节点下的applicationUrl中的地址，保存退出。再次执行```dotnet run```命令，就可以使用浏览器访问http://localhost:5002 查看效果了。

```
 "dotnetcore": {
      "commandName": "Project",
      "dotnetRunMessages": "true",
      "launchBrowser": true,
      "applicationUrl": "http://localhost:5002",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }

```
![执行结果](https://raw.githubusercontent.com/edsiongithub/blogimages/master/result.png)


至此，Linux上创建、开发、运行项目已经走通了。部署的步骤，可以参考上一篇。关于dotnet core的命令，记不住的话就使用``` dotnet --help```来提示。

### VS Code远程开发

为什么要远程开发？好像也没有标准答案为什么要这么做。VS Code也有Linux版本了，如果你日常工作的机器就是Linux为主，那直接在上面做开发也没问题。但大多数人工作机器都是windows系统，很多时候代码放在一个集中服务器中，Linux作为服务器又有它的优势。所以，多一种方式总归不是坏事。

VS Code是一款强大的编辑器，可以使用插件让它成为你熟悉的IDE。要想连接到远端的Linux服务器上代码，在插件管理中搜索remote-ssh，安装。

![安装插件](https://raw.githubusercontent.com/edsiongithub/blogimages/master/installremotessh.png)

接下来，添加远程主机信息。输入远程主机IP地址和用户，回车。

![添加远程主机](https://raw.githubusercontent.com/edsiongithub/blogimages/master/addremoteserver.png)

接下来选择第一个作为本地配置文件，选中后回车。如下图，

![本地配置](https://raw.githubusercontent.com/edsiongithub/blogimages/master/configremoteserver.png)

接下来选择打开远程目录，选择Linux上项目存放的路径。
![选择目录](https://raw.githubusercontent.com/edsiongithub/blogimages/master/selectpath.png)

接下来VS Code加载项目文件，你就可以在VS Code上写代码了。整个步骤中，注意观察VS Code顶部出现的输入框，会提示你输入Linux主机用户密码，输入密码后回车即可。

![远程写代码](https://raw.githubusercontent.com/edsiongithub/blogimages/master/remotecoding.png)

---
## 复盘

没有复盘，过程比较简单。后面部署的步骤可以参考上一篇，只是开发方式不同。

