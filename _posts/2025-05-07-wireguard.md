---
title: '[VPN] Wireguard的使用学习'
date: 2025-03-02
permalink: /posts/2025/05/wireguard/
excerpt: '我很羡慕saar安全团队能不断探索新的技术, 机缘巧合下认识了wireguard...'
tags:
  - linux
  - vpn
---

# 目录

- [VPN的基本使用](#VPN的基本使用)
  - [服务端环境设置](#服务端环境设置)
    - [安装](#安装)
    - [配置](#配置)
    - [编辑配置文件](#编辑配置文件)
    - [客户端配置](#客户端配置)
    - [设置开机启动](#设置开机启动)
    - [基础开关功能](#基础开关功能)

# VPN的基本使用

我对VPN一直处于能用层面上的了解, 同样在学习之前我想先把程序用起来. 所以跟着学了一下部署流程.

## 服务端环境设置

### 安装

使用debian安装wireguard:

```bash
sudo apt install wireguard
wg -v
wireguard-tools v1.0.20210914 - https://git.zx2c4.com/wireguard-tools/
```

### 配置

设置流量转发

```bash
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p
```

我认为在组内赋予满权限已经够了, 只给了770权限, 在`/etc/wireguard`生成服务器与客户端的公钥与私钥对:

```bash
cd /etc
chmod 770 wireguard
cd wireguard
umask 077
wg genkey | tee server_privatekey | wg pubkey > server_publickey
wg genkey | tee client_privatekey | wg pubkey > client_publickey
```

### 编辑配置文件

创建配置文件`/etc/wireguard/wg0.conf`:

```ini
echo "[Interface]
Address = # ifconfig:ip/24
SaveConfig = true
PrivateKey = $(cat server_privatekey)
ListenPort = 51820

[Peer]
PublicKey = $(cat client_publickey)
AllowedIPs = # ifconfig:ip + 1/32" > wg0.conf
```

如果设置本地防火墙需要放行对应端口, 这里需要放行`7890/udp`:

```bash
sudo ufw allow 7890/udp
```

这里采用了腾讯云服务器, 直接设置安全组放行对应端口即可.

### 客户端配置

```ini
echo "[Interface]
PrivateKey = $(cat client_privatekey)
Address = # ifconfig:ip+1/24
DNS = 8.8.8.8

[Peer]
PublicKey = $(cat server_publickey)
Endpoint = vycma.xyz:7890
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25" > client.conf
```

### 设置开机启动

```bash
systemctl enable wg-quick@wg0
```


### 基础开关功能

```bash
# 启动WireGuard
wg-quick up wg0
# 停止WireGuard
wg-quick down wg0
```

# 参考文献

[^csdn]: 杨林伟.组网神器WireGuard安装与配置教程（超详细）[EB/OL].CSDN.<a target="_blank" href='https://blog.csdn.net/qq_20042935/article/details/127089626'>https://blog.csdn.net/qq_20042935/article/details/127089626</a>.2022-09-28.
