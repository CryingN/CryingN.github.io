---
title: '[virtualbox]arch linux增强功能布置中面临的一些问题'
date: 2022-08-17
permalink: /posts/2022/08/arch_virtualbox_transfer/
excerpt: 'virtual box中Archlinux增强功能配置与文件传输...'
tags:
  - arch
  - virtualbox
---

virtualbox是一款开源虚拟机，现已更新到7.0.6版本，但是在一些电脑上不能安装，比如我的电脑（喵了个咪），还好6.1版本依旧能用，以下是在6.1版本中面对的问题与临时解决方法。

# 问题描述

在virtualbox的linux虚拟机中不能方便地与主机交换文件

# 临时解决方法

没看懂archwiki与大佬们一笔带过的思路，尝试使用virtual-guest-utils包时竟然默认安装vmware的安装包，甲骨文用户大受震撼，不知道怎么使用arch自带增强功能。 以下为个人解决思路：

**布置增强功能**

右ctrl+f选择 **设备>安装增强功能** 将增强功能安装到`/mnt`目录下，这里我多创建了一个文件夹用于挂载:

```bash
sudo mkdir /mnt
sudo mount /dev/cdrom /mnt/cdrom
cd /mnt/cdrom
```

会被警告挂载文件仅为可读文件，无法编辑，但是问题不大，linux通用增强工具为 **VBoxLinuxAdditions.run** ，运行一下。

```bash
sudo ./VBoxLinuxAdditions.run
```

<img src='/images/arch_virtualbox_transfer1.webp'>

<img src='/images/arch_virtualbox_transfer2.webp'>

这个应该没有什么问题，如下勾选完以后进入虚拟机进行查看，会发现神奇的一幕：“我的mnt呢？怎么打不开了？”

```bash
ls -all /
```

通过以上命令可以发现，在一堆root属组中mnt一行被更变为vboxsf了。

**解决思路**

属主依然是最强大的root用户，那问题就好办了，先调用我们的管理员权限：

```bash
sudo su
```

在root用户下可以正常打开/mnt文件，现在以学习资料为案例，用root权限将文件复制到/home文件夹下：

```bash
cp -r /mnt/学习资料 /home
ls -all /home
```

可以看到文件的属主与属组为root，但是如果想尝试直接进入会发现没有权限，个人感觉是因为root可访问root_cn(我的个人用户)，vboxsf，但root_cn与vboxsf互不能访问，文件是从vboxsf复制过来的，所以不能访问。 于是直接将属主与属组直接更改为root_cn：

```bash
sudo chown -R root_cn:root_cn 学习资料
```

现在可以正常打开了。

<img src='/images/arch_virtualbox_transfer3.webp'>

一直没好好理清楚权限管理，属主，属组等这些概念与linux关系，应用起来感觉总是在吃亏，甚至到现在也没搞懂，那么万能的sudo后面为什么不能接cd，装了增强功能后系统会出现一些变化，设置之前最好先备份一遍作个保险。
