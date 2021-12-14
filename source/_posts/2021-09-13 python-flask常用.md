---
title: python-flask常用命令及pear-admin项目分析
date: 2021-09-13 10:04:00
layout: 2021-09-13
id: 0018
tags: 
 - flask
 - python
---

python flask框架的学习已经有几个月了，中间断了一段时间后，发现许多命令不记得了。尤其虚拟环境相关的几条命令，总是忘记。另外，最近把开发环境移到一台ubuntu虚拟机上了，于是记录一些常用的命令，会随时增加。第二部分则是对一个PearAdmin的开源项目学习笔记。


<!--more-->

## 常用命令

### 开启虚拟环境
```
py -3 -m venv vevn  #创建虚拟环境
.\venv\Scripts\Activate.ps1 #激活虚拟环境

# Linux中使用下面命令
source venv\bin\activate
```

### 安装需要的包

```
pip install packagename #单个包安装
pip install -r requirements.txt #将所有需要的包放在文件中，从文件中读取安装
```


### pytest测试
```python
pytest          #执行所有测试
pytest xxx.py   #执行指定文件内所有测试
```
以上为一些常用的命令，其中有部分并非flask项目中特有的，学习过程中就一并记录下来了。

---

## PearAdmin项目分析

 该项目来自于gitee，主要拿来学习flask开发web项目。后端采用python flask，前端使用layui，gitee仓库地址：<a href="https://gitee.com/pear-admin/pear-admin-flask">pear-admin-flask</a>。项目主要目录结构及其功能如下：
### applications/view 
字面意思为视图，但我感觉该目录下的功能有点类似```asp.net mvc```中的controller，用来处理路由接收前端请求路径。

### applications/templates
模板，这里就有点类似```asp.net mvc```中的Views视图了，主要放一些html文件，用于向用户展示界面。由于项目前端使用的是LayUI，因此html文件中会有大量js代码夹杂在一起。

### applications/static
静态文件，主要存放项目相关的静态资源文件，如css、js、图片素材等。

### applications/schemas
~~我的理解，这个项目是基于领域设计模式，在这里定义了实体类。~~
这里用到了一个Marshallow的东西，是将复杂的orm对象与python原生数据类型间进行转换。Schema作为一个中间载体，将一个类和json数据进行互相转化（序列化与反序列化），同时也可以在Schema下做数据验证。
创建完schema后，需要在schemas目录下的__init__.py文件中，添加对Schema的引用，这样外部模块才能引用到schemas模块内的新建schema.

### applications/models
模型，处理实体类和数据表的映射关系。
创建完model后，需要在models目录下的__init__.py文件中，添加对Schema的引用，这样外部模块才能引用到models模块内的新建model.

### applications/extensions
扩展，主要作用为初始化一些模块。如初始化sqlalchemy、邮箱、虚拟环境等。其中，```init_sqlalchemy.py```中定义了db、ma变量，分别在schemas和models中被引用。

### applications/configs
配置文件。这一块挺有意思的，dotnet和java中，配置文件大多是独立的xml或json(早期dotnet会有web.config这样)文件作为配置。python直接使用.py文件来配置。

### applications/common
通用方法，一些通用方法存放在此。比如项目中的上传附件、生成验证码、http返回消息格式定义、权限等。其中，该项目提供脚本命令，用于添加一个新的路由。通过命令，可以创建新路由，并自动帮助生成view中相关的文件。该功能在script目录下。

