---
title: C#异步编程
date: 2021-11-08 20:57:14
layout: 2021-11-08 20:57:14
tags:
 - C#
 - 异步编程
---

* 异步编程场景
* async与await
* 返回类型
<!--more-->

### 异步编程场景
解决耗时任务占用主线程执行的问题。因此耗时高的任务适用于异步编程，C#中有以下场景：
1. 
![异步执行步骤](https://raw.githubusercontent.com/edsiongithub/blogimages/master/202111/navigation-trace-async-program.png)