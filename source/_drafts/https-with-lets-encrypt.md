---
title: 全站HTTPS
tags: 
- 技术分享
- Linux
- nginx
- https
- let's encrypt
---

![nginx-letsencrypt](/images/https-with-lets-encrypt/nginx-letsencrypt.png "Let's Encrypt secure our website with Nginx")

### Intro
在家里电信的网络，每次用微信打开博客就会被强制重排，我的推测是网页被电信劫持，注入了广告。

最早的网站采用的HTTP协议是明文传输，所以只要传输的数据被截取到，里面的内容也就没有秘密了。那谁会来劫取我们的流量呢？其实我们在浏览网页时，每次请求服务器和服务器的回复中间，都会经过重重的路由，这些路由就像是一个森林，而森林里到处都是虎视眈眈的眼睛。比如使用了公共Wifi，那么搭Wifi的人就可以截取到你的数据(所以说公共场所的Wifi最好不要连，安全第一)。

但有一双眼睛我们永远躲不掉，那就是运营商，只要我们通过运营商接入Internet，那我们所有的请求都可以被运营商截获。我家是电信的网络，有些时候在输入一个网址按回车后，会弹出一个电信的页面，上面有我的电信账号和套餐，还有一些乱七八糟的广告，这就是电信注入的广告 >-<

## HTTPS
HTTPS全称是HTTP over TLS，简单说就是加密的HTTP。

有些网站对安全性要求较高，比如银行，他们会使用的HTTPS协议，相比HTTP，HTTPS是经过加密的，即使流量被截取，里面的内容也没有办法被破译，相对比较安全。

以前网站搭HTTPS是比较麻烦的一件事，因为需要向[CA](https://en.wikipedia.org/wiki/Certificate_authority)申请证书，直到后来有了[Let's Encrypt](https://letsencrypt.org/)，一切都变得简单了。

### Let's Encrypt
Let's Encrypt是一个CA，它是通过[ACME(Automated Certificate Management Environment)](https://github.com/letsencrypt/acme-spec)协议来管理服务器上的证书
>The objective of Let’s Encrypt and the ACME protocol is to make it possible to set up an HTTPS server and have it automatically obtain a browser-trusted certificate, without any human intervention. This is accomplished by running a certificate management agent on the web server.

具体的工作原理在它的官网有[详细的解释](https://letsencrypt.org/how-it-works/)

### 生成证书

### 定期更新

## 配置Nginx

### 升级
[Upgrade nginx](https://leftshift.io/upgrading-nginx-to-the-latest-version-on-ubuntu-servers)

### 配置


## 参考链接
- [How To Secure Nginx with Let's Encrypt on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-14-04)
- [Upgrading Nginx to the latest version on Ubuntu servers](https://leftshift.io/upgrading-nginx-to-the-latest-version-on-ubuntu-servers)
- [Let's Encrypt 给网站加 HTTPS 完全指南](https://ksmx.me/letsencrypt-ssl-https)
- [certbot](https://certbot.eff.org/)