---
title: Ubuntu网络管理
date: 2021-06-02 10:04:00
layout: 2021-06-02
tags: 
 - Linux
 - 网络配置
---


## 网络配置

Hyper-V 虚拟机安装ubuntu20.04系统，默认采用了dchp动态获取IP地址。可以通过手动配置，固定IP，这样再使用ssh工具来连接的时候就方便了，不需要每次修改IP地址。20.04配置与早期版本略有不同。
``` bash
ifconfig    #查看IP地址及网卡接口名称（一般eth0这样子的）

sudo vim /etc/netplan/***.yaml  #打开默认配置文件
```
默认使用dhcp方式的配置文件内容如下：
```bash
network:
  ethernets:
    eth0:
      dchp4: true
  version: 2
```
<!--more-->
修改成如下
``` bash
network:
  ethernets:
    eth0:
      dchp4: false
      address:
        - xx.xx.xx.xx/xx
      gateway4: xx.xx.xx.1
      nameservers:
        addresses: [8.8.8.8,114.114.114.114]
  version: 2
```
配置完毕后，使用下面命令让配置生效。
```
sudo netplan apply      #网络配置生效
ifconfig                #查看网络信息
ip a                    #查看IP信息
```
关于这里的IP地址及网关配置，需要依据Hyper-V默认的虚拟网卡上计算。比如我的虚拟网卡IP地址：172.29.224.1，子网掩码：255.255.240.0。从这里可以看出来，IP地址使用cider表示方式的话，就是172.29.224.1/28,即前面28位用来做网络号，后4位才能用来做主机号。

虚拟机内的网络IP地址配置就根据这个网络号和主机号分配方式来即可。


## 双网卡配置

这里主要以windows主机中，使用Hyper-V创建Linux虚拟机。希望在虚拟机中配置双网卡，一张网卡通外网，另外一张网卡固定IP地址，与物理主机可以互相访问。这样便于ssh连接时固定IP，而不是每次更换IP地址连接。

首先在Hyper-V中添加虚拟交换机，按下图中步骤进行添加。
![添加交换机](https://raw.githubusercontent.com/edsiongithub/blogimages/master/20210831/addswitch.png)

由于Hyper-V有一个默认的虚拟机（Default Switch），已经是内部网络型，我们创建一个新的外部网络类型的虚拟交换机，并选择接口，勾选允许管理操作系统共享此网络适配器。

![选择外部](https://raw.githubusercontent.com/edsiongithub/blogimages/master/20210831/addswitch1.png)

然后选中自己的虚拟机，右键设置，添加硬件，选择网络适配器，添加。

![ubuntu添加网卡](https://raw.githubusercontent.com/edsiongithub/blogimages/master/20210831/ubuntuaddswitch.png)

选中前面创建的虚拟交换机（自己命名，如上图中的Outter）。

![ubuntu选择网卡](https://raw.githubusercontent.com/edsiongithub/blogimages/master/20210831/ubuntuaddswitch1.png)

接下去，启动虚拟机，配置双网卡IP地址情况。首先，我们使用```ip a```命令，查看到所有的网络接口名字，如：eth0、eth1这样的。接下来，类似于前文中提到的IP地址配置，进入/etc/netplan/XXXX.yaml配置文件，添加如下的配置信息。将eth0配置为固定IP地址，eth1配置为通过dhcp动态获取IP地址即可。
``` bash
# This is the network config written by 'subiquity'
network:
  ethernets:
    eth0:
      dhcp4: false
      addresses:
             - 172.29.224.75/24
      gateway4: 172.29.224.1
      nameservers:
           addresses: [8.8.8.8,114.114.114.114]
    eth1:
      dhcp4: true
  version: 2
```
## 防火墙配置
防火墙的作用，可以通过配置策略，控制允许和禁止访问主机内的服务。最常用的，比如关闭或开放80端口，以配置web服务的访问。ubuntu中，底层的iptables来实现，上层用户可以使用ufw来管理策略。下面是一些常用命令

```bash
sudo apt install ufw              #安装
sudo ufw enable                   #启用
sudo ufw default deny             #禁用

sudo ufw allow | deny [service]   #开启或关闭服务
sudo ufw allow 22/tcp             #允许22端口，ssh使用
sudo ufw allow 80/tcp             #允许80端口
sudo ufw allow from 192.168.1.100 #允许该IP访问本机所有服务
sudo ufw deny smtp                #禁用smtp
sudo ufw delete deny smtp         #删除上面的禁用

sudo ufw status                   #查看状态
```

注意：
1. ufw的底层，还是依赖iptables来实现。只是iptables的操作没有ufw这么容易。
2. 使用ufw需要管理员权限，即命令都需要sudo。
3. 配置规则，可以使用命令，也可以到/etc/ufw/user.rules中进行修改，如我们添加80端口允许访问的配置如下：
```bash
-A ufw-user-input -p tcp --dport 80 -j ACCEPT
```
修改完这个配置后，需要```sudo ufw reload```来重新加载配置文件以生效。