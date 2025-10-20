---
title: '[VPN] Wireguard的基本使用方法'
date: 2025-03-02
permalink: /posts/2025/05/wireguard/
excerpt: 'VTF开发过程中我们想引入插件概念, 其中vpn实现内网功能是我们很在意其中一种插件, 于是我们决定从wireguard开始学习...'
tags:
  - linux
  - vpn
---

# 目录

- [vpn的基本使用](#vpn的基本使用)
  - [服务端环境设置](#服务端环境设置)
    - [安装](#安装)
    - [配置](#配置)
    - [编辑配置文件](#编辑配置文件)
    - [基础开关功能](#基础开关功能)
  - [客户端配置](#客户端配置)
    - [连接](#连接)
    - [检测](#检测)
- [进阶部署](#进阶部署)
  - [全隧道](#全隧道)
  - [流量转发](#流量转发)
  - [net规则验证](#net规则验证)
  - [开机启动](#开机启动)
- [参考文献](#参考文献)

# VPN的基本使用

VTF开发过程中我们想引入插件概念, 其中vpn实现内网功能是我们很在意其中一种插件, 于是我们决定从wireguard开始学习, 在学习之前更重要的是能把程序先用起来. 于是跟着学了一下部署流程. WireGuard与其他VPN项目相比，代码量小、性能高、配置简单, 是基于udp传输的高效vpn。

## 服务端环境设置

### 安装

以debian为例安装wireguard:

```bash
sudo apt install wireguard
wg -v
wireguard-tools v1.0.20210914 - https://git.zx2c4.com/wireguard-tools/
```

除此以外还需要安装以下配置工具:

```bash
sudo apt install iptables
```

### 配置

我认为在组内赋予满权限已经够了, 只给了770权限, 有时间慢慢研究一下最小权限分配应该怎么做好一点[^csdn], 在`/etc/wireguard`生成服务器的公私钥对:

```bash
cd /etc
chmod 770 wireguard
cd wireguard
umask 077
mkdir key
wg genkey | tee server_privatekey | wg pubkey > key/server_publickey
```

除此之外进行通信还需要配置对应的客户端公私钥对发起访问:

```bash
wg genkey | tee client[编号]_privatekey | wg pubkey > key/client[编号]_publickey
```

### 编辑配置文件

创建配置文件`/etc/wireguard/wg0.conf`:

```ini
echo "[Interface]
Address = # ifconfig:ip/24 服务端虚拟网段
SaveConfig = true
PrivateKey = $(cat server_privatekey)
ListenPort = 7890

[Peer]
PublicKey = $(cat client_publickey)
AllowedIPs = # ifconfig:ip + 1/32 client1虚拟网段

[Peer]
PublicKey = $(cat client_publickey)
AllowedIPs = # ifconfig:ip + 1/32 client2虚拟网段
" > wg0.conf
```

如果设置本地防火墙需要放行对应端口, 如上文配置需要放行`7890/udp`:

```bash
sudo ufw allow 7890/udp
```

这里采用了腾讯云服务器, 直接设置安全组放行对应端口即可.


### 基础开关功能

```bash
# 启动WireGuard
wg-quick up wg0
# 停止WireGuard
wg-quick down wg0
```

可以通过以下代码快速重启服务:

```bash
wg-quick down wg0 && wg-quick up wg0
```

## 客户端配置

### 连接

客户端可以分别配置规则如下的配置文件:

```ini
echo "[Interface]
PrivateKey = $(cat client_privatekey)
Address = # ifconfig:ip+1/24 客户端对应预设的虚拟网段
ListenPort = 8080
DNS = 8.8.8.8

[Peer]
PublicKey = $(cat server_publickey)
Endpoint = [服务器IP地址]:7890
AllowedIPs = [VPN虚拟网段]/24, [内网资源网段]/24
PersistentKeepalive = 25" > client.conf
```

### 检测

我们可以通过访问内网资源的ip测试连接是否成功:

```bash
ping [内网IP]
```

如果发包成功则说明基础的VPN搭建成功.

# 进阶部署

## 全隧道

在VPN部署中存在让客户端的所有网络流量都经由服务器的公网出口进行访问的手段, 这种功能被称为全隧道模式, 全隧道模式需要做到虚拟网段搭建、流量转发、NET地址伪造3个部分, 为了使流量通经由服务器访问, 需要设置客户端配置:

```bash
AllowedIPs = 0.0.0.0/0
```

## 流量转发

在服务器中可以通过以下方式设置流量转发:

```bash
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p
```

## net规则验证

在配置项可以对配置文件`/etc/wireguard/wg0.conf`进行以下预设:

```bash
PostUp = iptables -t nat -A POSTROUTING ! -o wg0 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING ! -o wg0 -j MASQUERADE
```

用于在开启或结束时自动配置NET中MASQUERADE的转发规则.

有时候反复设置往往会使规则出现问题, 以至于无法有效转发流量, 可以通过以下脚本验证net规则:

```bash
sudo iptables -t nat -L -n -v
```

可以检查POSTROUTING段是否有一条固定的规则在正常转发, 如果出现冲突需要进行重设, 以下提供一个最基础的方法清理所有POSTROUTING规则:

```bash
sudo iptables -t nat -F POSTROUTING
```

## 开机启动

```bash
systemctl enable wg-quick@wg0
```


# 参考文献

[^csdn]: 杨林伟.组网神器WireGuard安装与配置教程（超详细）[EB/OL].CSDN.<a target="_blank" href='https://blog.csdn.net/qq_20042935/article/details/127089626'>https://blog.csdn.net/qq_20042935/article/details/127089626</a>.2022-09-28.
