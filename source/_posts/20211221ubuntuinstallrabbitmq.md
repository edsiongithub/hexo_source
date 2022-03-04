---
title: Ubuntu中安装RabbitMQ
date: 2021-12-21 14:28:35
layout: 2021-12-21 14:28:35
id: 0032
tags:
 - linux
 - rabbitmq
categories:
 - 操作系统及网络
---
 
* rabbitmq 简介
* ubuntu 安装rabbitmq
* 一些配置及常用命令

<!--more-->

## 简介
消息队列一般用于系统间对接，能够降低系统耦合度。通过将消息的发送和接收分离来实现应用程序的异步和解耦。你可以使用消息队列实现：
* 数据分发
* 非阻塞操作或推送通知（如邮件、短信提醒等）
* 实现发布/订阅，异步处理
* 工作队列

RabbitMq作为一款消息队列产品，它由Erlang语言开发，实现AMQP（高级消息队列协议）的开源消息中间件。

## 应用场景

1. 异步处理
场景说明：用户注册后，注册信息写入数据库，再发邮件、短信通知。但后两项并非必须，传统做法：数据写入->发送短信->发送邮件->返回给用户。用户等待所有任务完成。
引入rabbitmq：可以将短信通知、邮件通知这两个非必要的任务写入到消息队列，再由专门的任务处理程序去读取消息发送通知。用户等待时间变成写入数据库时间+写入消息队列时间。

2. 应用解耦
网上购物，用户提交订单后，订单系统需要通知库存系统。传统做法，订单系统直接调用库存系统api。当库存系统出现故障，则订单失效。
一如rabbitmq：订单系统完成持久化，将消息写入消息队列。库存系统订阅消息队列中的订单，库存系统故障了，也不会导致订单丢失。恢复后可以从消息队列继读取订单处理。

3. 流量削峰
前端加入消息队列。大批量用户请求到达后，写入消息队列，并设置最大阈值。超过该阈值的请求丢弃返回错误给客户端。系统只需要从消息队列中取正常的请求来进行处理即可。

## 安装
安装环境：Ubuntu 20.04，使用如下命令安装。
```bash
sudo apt install rabbitmq-server
```

```bash
systemctl status rabbitmq-server         #检查状态
service rabbitmq-server status          #检查状态
```
下面是启动后，查看状态的输出。

``` bash
● rabbitmq-server.service - RabbitMQ Messaging Server
     Loaded: loaded (/lib/systemd/system/rabbitmq-server.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2021-12-21 14:28:04 CST; 4min 24s ago
   Main PID: 94216 (beam.smp)
     Status: "Initialized"
      Tasks: 85 (limit: 4611)
     Memory: 56.2M
     CGroup: /system.slice/rabbitmq-server.service
             ├─94212 /bin/sh /usr/sbin/rabbitmq-server
             ├─94216 /usr/lib/erlang/erts-9.2/bin/beam.smp -W w -A 64 -P 1048576 -t 5000000 -stbt db -zdbbl 32000 -K tr>
             ├─94309 /usr/lib/erlang/erts-9.2/bin/epmd -daemon
             ├─94445 erl_child_setup 65536
             ├─94465 inet_gethost 4
             └─94466 inet_gethost 4

Dec 21 14:28:01 pc-ubuntu systemd[1]: Starting RabbitMQ Messaging Server...
Dec 21 14:28:04 pc-ubuntu systemd[1]: rabbitmq-server.service: Supervising process 94216 which is not our child. We'll >
Dec 21 14:28:04 pc-ubuntu systemd[1]: Started RabbitMQ Messaging Server.
Dec 21 14:28:05 pc-ubuntu systemd[1]: rabbitmq-server.service: Supervising process 94216 which is not our child. We'll >
```
下面是一些常用管理命令，包括启用、停用、重启。
```bash
sudo service rabbitmq-server start       #启用
sudo service rabbitmq-server stop        #停用
sudo service rabbitmq-server restart     #重启
```
停用后，再次查看状态，输出如下。重点查看Active后面的内容，此时为inactive（dead).


```bash
● rabbitmq-server.service - RabbitMQ Messaging Server
     Loaded: loaded (/lib/systemd/system/rabbitmq-server.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Tue 2021-12-21 14:33:36 CST; 9s ago
    Process: 95123 ExecStop=/usr/sbin/rabbitmqctl stop (code=exited, status=0/SUCCESS)
   Main PID: 94216
     Status: "Initialized"

Dec 21 14:28:01 pc-ubuntu systemd[1]: Starting RabbitMQ Messaging Server...
Dec 21 14:28:04 pc-ubuntu systemd[1]: rabbitmq-server.service: Supervising process 94216 which is not our child. We'll >
Dec 21 14:28:04 pc-ubuntu systemd[1]: Started RabbitMQ Messaging Server.
Dec 21 14:28:05 pc-ubuntu systemd[1]: rabbitmq-server.service: Supervising process 94216 which is not our child. We'll >
Dec 21 14:33:35 pc-ubuntu systemd[1]: Stopping RabbitMQ Messaging Server...
Dec 21 14:33:36 pc-ubuntu rabbitmq[95131]: Stopping and halting node 'rabbit@pc-ubuntu'
Dec 21 14:33:36 pc-ubuntu systemd[1]: rabbitmq-server.service: Killing process 94216 (beam.smp) with signal SIGKILL.
Dec 21 14:33:36 pc-ubuntu systemd[1]: rabbitmq-server.service: Succeeded.
Dec 21 14:33:36 pc-ubuntu systemd[1]: Stopped RabbitMQ Messaging Server.
```

## 开启Web管理

```bash
sudo rabbitmq-plugins enable rabbitmq_management
```
注意，执行上面的命令时，要确保rabbitmq在运行状态中。
执行完后，在浏览器中输入`http://localhost:15672` 并使用guest/guest 作为用户名密码登录。

![rabbitmqweb管理](https://raw.githubusercontent.com/edsiongithub/blogimages/master/202112/rabbitmqwebmanage.png)

用户管理命令
```bash
sudo rabbitmqctl list_users #列出用户
sudo rabbitmqctl add_user admin adminpassword   #添加用户
sudo rabbitmqctl set_user_tags admin administrator #给用户添加角色
sudo rabbitmqctl delete_user admin  #删除用户
```

给用户配置VirtualHost中的读写权限。使用set_permissions命令来将rabbitadmin这个用户赋予`/`这个虚拟主机的读、写、配置权限。rabbitmq安装好后，会默认有这个`/`虚拟主机的。通过Web管理端可以看见，guest用户对其有所有权限。通过下面命令，将我们刚才创建的rabbitadmin也赋权过去。
```
sudo rabbitmqctl set_permissions -p / rabbitadmin '.*' '.*' '.*'
```


## 参考
1. [rabbitmq中文文档](http://rabbitmq.mr-ping.com/) 
2. [超详细的RabbitMQ入门，看这篇就够了！](https://developer.aliyun.com/article/769883)
3. [ubuntu中安装rabbitmq](https://www.jianshu.com/p/5c8c4495827f)
4. [RabbitMQ官网](https://www.rabbitmq.com/?spm=a2c6h.12873639.0.0.433733dfYmi6jK)

