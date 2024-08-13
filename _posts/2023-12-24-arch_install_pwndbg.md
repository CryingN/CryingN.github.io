---
title: '在Archlinux上配置pwndbg'
date: 2023-12-24
permalink: /posts/2023/12/arch_install_pwndbg/
excerpt: '如何有效在archlinux中安装pwndbg(适用于wsl1/wsl2/虚拟机)...'
tags:
  - arch
  - pwn
  - pwndbg
---

[pwndbg地址](https://link.zhihu.com/?target=https%3A//gitee.com/sakana_ctf/pwndbg)

# archlinux配置问题

> 现在archlinux已经支持直接使用pacman包直接安装pwndbg, 当然安装过程并没有因此变得像其他包一样简单, 以下为我个人遇到的一些问题与解决方式. by: sudopacman

# 包安装
如果不是从源码进行安装, 结束后发现调用gdb并不会变成pwndbg的环境, 查询发现需要将`source /usr/share/pwndbg/gdbinit.py`保存至`~/.gdbinit`目录, 当然, 再次打开后发现还是普通的gdb.通过排查可以发现gdbinit.py文件无法调用一个`/usr/share/pwndbg/.venv`的文件夹, 如果查看源码可以发现所谓的.venv文件夹为python的虚拟环境, 得出结论: 

**archlinux一键安装pwndbg失败来源于python的虚拟环境构建.**  

小知识, 不同于其他linux发行版, archlinux安装python库不需要(官方似乎也禁用了)pip包, 尝试使用pip进行包安装会出现报错, 提醒使用pacman进行安装, 所以我们需要重新构建一个虚拟环境:

```bash
# 安装虚拟环境库
sudo pacman -S python-virtualenv
# 设置虚拟环境
cd /usr/share/pwndbg
python -m venv .venv
source .venv/bin/activate
# 成功进入虚拟环境后命令行开头会增加(.venv),安装所需包(备注: 可根据需要添加镜像源)
sudo pip install --upgrade pip
sudo pip install pwntools typing_extensions tabulate
```

完成后尝试运行gdb发现已经出现了`pwndbg>`的标注, 当然有可能会出现一些警告, 我这里报告需要多安装一个QEMU, 直接使用pacman包安装:

```bash
sudo pacman -S qemu
```

直接打开后显示如下:

```bash
[root@archlinux ~]$ gdb
GNU gdb (GDB) 13.2
Copyright (C) 2023 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-pc-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word".
pwndbg: loaded 142 pwndbg commands and 44 shell commands. Type pwndbg [--shell | --all] [filter] for a list.
pwndbg: created $rebase, $ida GDB functions (can be used with print/break)
------- tip of the day (disable with set show-tips off) -------
Pwndbg sets the SIGLARM, SIGBUS, SIGPIPE and SIGSEGV signals so they are not passed to the app; see info signals for full GDB signals configuration
pwndbg>
```

# 从源码安装

尽管现在setup.sh文件已经支持了pacman包, 不过个人实际测试时出现了一些严重错误, 我们在尝试优化setup.sh在arch上的配置, 成功后将会尝试合并到主分支.
