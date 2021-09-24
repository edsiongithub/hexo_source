---
title: conda环境管理
date: 2021-07-01 09:31:14
layout: 2021-07-01 09:31:14
---

## conda 安装
下载安装脚本，<a href="https://repo.anaconda.com/archive/">点这里</a>找到需要的版本。
```
wget https://repo.anaconda.com/archive/Anaconda3-5.0.1-MacOSX-x86_64.sh #随便举个版本的例子

bash Anacondaxxxx.sh    #执行安装脚本
```
期间会有一些询问，一路回车，或者输入yes回车就行了。最后给bashrc添加环境变量
```
export PATH=/home/youruseraccount/anaconda3/bin:$PATH
```

## conda 创建环境并激活
创建虚拟环境，并指定pyhon版本
```
conda create -n yourenvname python=3.9    #创建环境，并指定相关软件的版本
source activate yourenvname               #启动环境
conda activate yourenvname                #启动环境（新命令，优于source那条命令）
```
注意：遇到过一次这样的情况，xshell连接到服务器，中途卡死，直接关闭了shell连接。在此之前并未deactivate环境，当再次连接至服务器时，发现无法使用conda命令了，于是再次执行了一下添加环境变量的命令。但使用```conda activate envname```还是报错，说环境没有配置好conda，但这个时候可以使用```source activate envname```。这个时候，只要再执行一次```source deactivate envname```,退出环境，再执行就可以正常使用了。






