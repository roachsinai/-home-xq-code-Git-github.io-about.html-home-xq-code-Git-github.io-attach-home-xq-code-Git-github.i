---
layout:     post
title:      "shadowsocks 梯子使用"
subtitle:   "梯子相关"
date:       2016-08-05 19:30:00 +0800
author:     "Roach"
catalog: true
tags:
    - 工具
---

> 可能是看视频 VPS 被封，幸好搬瓦工可以更换 IP，今天记录一下 ss 搭建的整个过程

## 搬瓦工

这是必备条件，购买一个 `VPS`，不然谁接你扔过墙的绳 T_T...

[搬瓦工](http://banwagong.cn/#)，页面里有优惠码！

## ssh 使用方法


```
[xq@roach ~]$ ssh name@ip_address -p ssh_port
```

上面要填的分别是，要通过 `ssh` 连接主机上的 `用户名`、`主机的IP地址`、`主机上分给ssh连接的端口号`，之后会让输入该用户对应的密码登录即可。

## 安装 ss 服务器端

`ssh` 登录之后添加 `ss-libev` 源：


```
su -c 'dnf copr enable librehat/shadowsocks'
```

安装：

```
su -c 'dnf update'
su -c 'dnf install shadowsocks-libev'
```

## 配置服务器端

在你的用户目录下，编辑配置文件：`vim /etc/shadowsocks.json`

```
# 注释版配置
{
    "server":"servier_ip",   # 服务器IP
    "server_port":65432,     # ss服务器所使用的端口号，建议改到30000-60000
    "password":"password",   # ss服务器密码，轻易不要分享
    "timeout":60,            # 超时时间，建议设置为60
    "method":"rc4-md5"         # 加密方式，需要和客户端配合设置
}

# 复制粘贴版，去掉注释不然可能出错

{
    "server":"servier_ip",
    "server_port":65432,
    "password":"password",
    "timeout":60,
    "method":"rc4-md5"
}
```

上面配置文件中填写内容要和你 `VPS` 中的一样。

#### 运行 server

`ssserver -c /etc/shadowsocks.json -d start`

## Linux 使用

1. [Shadowsocks的图形化客户端](https://www.librehat.com/introduction-to-shadowsocks-graphical-client-shadowsocks-qt5/)

    `sudo dnf copr enable librehat/shadowsocks`
    
    `sudo dnf update`
    
    `sudo dnf install shadowsocks-qt5`

2. [SwitchyOmega](http://switchyomega.com/index.html)，扩展使用方法在[此](http://switchyomega.com/settings.html)，**最新**配置文件在页面**下方**

配置好 `shadowsocks-qt5 和 switchyomega` 之后，梯子就好了。

## Windows 使用

下载安装 `Shadowsocks.exe` 即可。

## 软件下载

链接: http://pan.baidu.com/s/1qXBOVN2 密码: 6427

## References

1. [ShadowSocks（影梭）不完全指南](http://www.auooo.com/2015/06/26/shadowsocks%EF%BC%88%E5%BD%B1%E6%A2%AD%EF%BC%89%E4%B8%8D%E5%AE%8C%E5%85%A8%E6%8C%87%E5%8D%97/)
2. [使用shadowsocks轻松搭建**环境教程](https://blog.phpgao.com/shadowsocks_on_linux.html#浏览器)
3. [选择、获取最新可用 IPv4 hosts - 奶齿 Netsh](https://serve.netsh.org/pub/ipv4-hosts/)

注：参考 [1](http://www.auooo.com/2015/06/26/shadowsocks%EF%BC%88%E5%BD%B1%E6%A2%AD%EF%BC%89%E4%B8%8D%E5%AE%8C%E5%85%A8%E6%8C%87%E5%8D%97/) 中有详细的客户端配置