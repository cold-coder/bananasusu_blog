---
title: 西海岸的风
tags:
  - 技术分享
  - Linux
  - vps
  - digital ocean
  - shadowsocks
date: 2016-06-12 11:11:42
---


![golden gate bridge](/images/breeze-from-west-coast/golden_gate_bridge.jpg "Golden Gate Bridge")

### Intro
严重怀疑facebook的盗号与VPN提供商有关联，所以一向认为安全第一的我决定自己搭。

## VPS的选择

VPS(virtual private server)虚拟主机服务，就是你花钱租一台服务器，通过网络连上他，可以装软件或者搭服务，说虚拟是因为你不必在意他的物理存在，他可以在世界的任何一个角落。
国内最出名的VPS要数[阿里云](https://www.aliyun.com/)了，国际上的有[Vultr](https://www.vultr.com/), [Digital Ocean](https://www.digitalocean.com/), [Linode](https://www.linode.com/)等等。

### 选择

网上对相关VPS的讨论有很多，在做了一番研究之后我决定购买Digital Ocean的主机。

### 创建实例

创建实例(Droplet)的时候会要选择机房，主机配置和操作系统，机房我选择的旧金山，配置我选的最低配的(512M内存+20GB硬盘)，每月5刀，操作系统我选的Ubuntu 14.04，和我平时使用的一致。
创建的时候还需要导入一个SSH Key， 为了验证登录的信息，一般Linux下的SSH公钥都是保存在~/.ssh/目录下，叫id_xxx.pub(xxx是加密算法，如rsa|dsa|ecdsa...)，还有一个id_xxx的是私钥文件，切不可外漏。
全部信息填完后选择创建，然后就是绑信用卡付款，拿到一个IP地址，心情万分激动。


### 第一次登录

由于之前导入过SSH Key，所以能在自己的机器上就可以直接登录，只要执行

``` bash
$ ssh root@your-server-ip
```
用惯了zsh表示对bash有点不习惯

### 登录之后的配置

按照Linux的哲学，应该为每个用户创建一个登录帐号而不是都用root登录，但我为了偷懒，并没有这么做(不要学我)。

接着配置vim编辑器，按照[我的vim](https://github.com/cold-coder/vim)的来，安装[zsh](http://www.zsh.org/)和[Oh My Zsh](https://github.com/robbyrussell/oh-my-zsh)，安装[tmux](https://tmux.github.io/)。


## 主角登场

[Shadowsocks](https://shadowsocks.org/en/index.html) 是一个socks5代理，有点像VPN，而且是[开源的](https://github.com/shadowsocks).

### 安装服务器端
Shadowsocks提供多种语言的安装包，我选择的Python的，先装Python的包管理工具pip

``` bash
$ apt-get install pip
```
接着就可以用pip去安装ss了。

``` bash
$ pip install shadowsocks
```
shadowsocks的配置文件是/etc/shadowsocks.json，里面的几个参数就照着官网的教程填就可以了。

接着用命令启动 -c 指定配置文件路径
``` bash
$ ssserver -c /etc/shadowsocks.json
```

### 安装客户端
安卓上的shadowsocks客户端相当好用，iOS我装的Shadowrockt，OS X上有GoAgent X， 配置都相当简单。
我现在一般使用Chrome的SwitchyOmega插件，创建一个自动切换模式，用[gfwlist](https://github.com/gfwlist/gfwlist)上的规则过滤，凡是符合规则的都走ss代理，这样国内的网站就不必都去大洋彼岸转一圈了。

每次打开ss都能感觉有风吹来，那是美国西海岸的风，带着淡淡自由的味道。