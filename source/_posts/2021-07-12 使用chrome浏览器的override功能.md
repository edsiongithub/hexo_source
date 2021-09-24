---
title: 使用chrome浏览器的overrides功能
date: 2021-07-12 10:50:27
layout: 2021-07-12 10:50:27
tags:
 - 前端
 - 浏览器
---

## 背景

上半年接到的一个任务，是解决目前在用的几个老系统的浏览器兼容性问题。这几个系统存在以下一些问题：
1. 建设时期早，使用技术陈旧
2. 只支持IE浏览器兼容模式下使用
3. 无开发环境、测试环境，只有在用的正式环境
4. 运行时间长久，无乙方提供后端技术支持
5. 无业务部门牵头，做业务梳理，走重新开发的道路

于是，为了能凑合继续使用，经讨论，由开发部采用前端改造的方法，让这几个老旧系统能够适应现代浏览器，继续提供服务。


## Chrome的local overrides
chrome 65更新的一个功能，该功能允许在浏览器中修改请求的返回数据。你可以在本地，直接重写一些请求的资源，而不用等待服务器的返回。这就给前端调试JS带来极大的方便。接下来看看这个厉害的功能。

<!--more-->

打开chrome浏览器，去访问你需要的url。F12打开开发者工具，选择Sources标签，点FileSystem后面的>>按钮，选择下面Overrides菜单。
![选择Overrides](https://raw.githubusercontent.com/edsiongithub/blogimages/master/20210712/override.png)

此时，点下面加号那一行，选择一个本地目录来存储从服务器上拉下来的文件。保存的路径会按照你访问的url创建目录树。


![选择目录](https://raw.githubusercontent.com/edsiongithub/blogimages/master/20210712/selectfolder1.png)

选择好本地存储路径后，浏览器顶部会弹出一个提示框，要求获得本地路径的完整访问权限。这时候选择允许。

![安全提示](https://raw.githubusercontent.com/edsiongithub/blogimages/master/20210712/selectfolder.png)

接下来，在Page标签下，找到你需要修改的资源（比如某个请求的页面），右键选中，并点击最下面的Save for override。再返回到Overrides标签下，找到刚刚保存下来的本地资源。
![保存本地](https://raw.githubusercontent.com/edsiongithub/blogimages/master/20210712/selectpageforoverride.png)

这个时候，你就可以再这个本地资源中进行任意修改。保存本地文件后，刷新浏览器重新请求指定url，此时浏览器就会直接使用本地保存的这个资源，而不需要等待服务器返回该资源。

![修改本地文件](https://raw.githubusercontent.com/edsiongithub/blogimages/master/20210712/modifylocalpage.png)


## 项目修改

具体到我们这次需要修改的项目，主要问题集中在数据表格的渲染上。老的表格渲染方式，在现代浏览器中无法显示表格，导致无法操作。因此，我们在页面上引入jquery、jquery-ui等js框架，重新渲染数据。再重写页面中按钮的事件方法，本地测试通过后，部署到服务器中。

这样的修改，不涉及后端代码改动，对系统影响最小。项目修改过程中，也不影响系统正常（变态IE浏览器兼容模式）使用。