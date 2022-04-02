---
title: 20220402EnityFramework知识集
date: 2022-04-02 09:42:15
layout: 2022-04-02 09:42:15
id: 0043
tags:
 - EntityFramework
 - 翻译
categories:
 - EntityFramework
 - 翻译
---
 
 连续肝完LINQ教程的翻译总结工作（这样总结似乎不太对，没有完整翻译，也没太多总结）。接下来一鼓作气再把Entity Framework肝完，毕竟Entity Framework中LINQ还是可以大展拳脚的。
 原文连接：https://dotnettutorials.net/lesson/entity-framework-architecture/


<!--more-->
EntityFramework 是一个开源的对象-关系映射框架（ORM:Object-Relational Mapping Framework)。它可以让.net开发人员像使用领域对象一样使用关系数据，而不需要关心底层数据库表结构。从下图可以看到Entity Framework在我们的应用中扮演的作用。
 
![](https://dotnettutorials.net/wp-content/uploads/2018/09/Where-Entity-Framework-used-in-our-application.png)


## EntityFramework架构
从下面图中可以看出，EntityFramework由以下几个部分构成：
1. The Entity Data Model (实体数据模型)
2. LINQ to Entities
3. Entity SQL
4. The Object Services Layer
5. Entity Client Data Provider
6. ADO.NET Data Provider

![架构图](https://dotnettutorials.net/wp-content/uploads/2018/09/Entity-Framework-Architecture.png)



## 上下文类（Context）
