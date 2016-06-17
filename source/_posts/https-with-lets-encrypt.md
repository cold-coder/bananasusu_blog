---
title: 全站HTTPS
tags:
  - 技术分享
  - Linux
  - nginx
  - https
  - let's encrypt
date: 2016-06-16 21:05:38
---


![nginx-letsencrypt](/images/https-with-lets-encrypt/nginx-letsencrypt-min.png "Let's Encrypt secure our website with Nginx")

为了不让博客被电信劫持，注入广告，我决定将我的博客进行HTTPS加密传输。

<!-- more -->

## 背景

互联网刚诞生的时候，所有网站采用的HTTP协议进行传输，而HTTP是明文的，所以只要数据被截取到，里面的内容也就没有秘密了。

那谁会来劫取我们的流量呢？

我们在上网的时候，每次请求服务器都会经过重重路由，这些路由就像是一个森林，而森林里到处都是虎视眈眈的眼睛。比如当你连入某个咖啡厅的Wifi，那么搭Wifi的人就可以截取到你的数据(所以说公共场所的Wifi最好不要连，安全第一)。

但有一双眼睛永远躲不掉，那就是运营商，只要我们通过运营商接入Internet，那我们所有的请求都可以被运营商截获。我家是电信的网络，有些时候在输入一个网址按回车后，会弹出一个电信的页面，上面有我的电信账号和套餐，还有一些乱七八糟的广告，这就是电信注入的广告 >-<

## HTTPS
HTTPS全称是HTTP over TLS，简单说就是加密的HTTP。

有些网站对安全性要求较高，比如银行，他们会使用的HTTPS协议，相比HTTP，HTTPS是经过加密的，即使流量被截取，里面的内容也没有办法被破译，相对比较安全。

以前网站搭HTTPS是比较麻烦的一件事，因为需要向[CA](https://en.wikipedia.org/wiki/Certificate_authority)申请证书，直到后来有了[Let's Encrypt](https://letsencrypt.org/)，一切都变得简单了。

#### Let's Encrypt
Let's Encrypt是也一个CA，它是通过[ACME(Automated Certificate Management Environment)](https://github.com/letsencrypt/acme-spec)协议来管理服务器上的证书
>The objective of Let’s Encrypt and the ACME protocol is to make it possible to set up an HTTPS server and have it automatically obtain a browser-trusted certificate, without any human intervention. This is accomplished by running a certificate management agent on the web server.

具体的工作原理在它的官网有[详细的解释](https://letsencrypt.org/how-it-works/)，简单来说就是Let's Encrypt的客户端运行在你的站点服务器上，他会在站点某个地方`.well-known目录`藏一个密码，接着通过访问你的站点去找这个密码，如果找到并验证通过，就颁给你证书。

Here we start~

首先将github上的代码clone到本地

``` bash
$ sudo git clone https://github.com/letsencrypt/letsencrypt /opt/letsencrypt
```

由于Let's Encrypt的验证需要访问网站根目录下面的一个隐藏目录`.well-known`，所以这里要让Nginx放出这个目录的访问权限，编辑`/etc/nginx/sites-available/bananasusu.com`，增加一个location配置项

``` nginx
	location ~ /.well-known {
		allow all;
	}
```

Nginx重新加载配置 `sudo service nginx reload`或者`sudo nginx -s reload`

#### 生成证书
用letsencrypt的脚本生成证书， webroot-path是你网站部署的路径，d是域名
``` bash
$ cd /opt/letsencrypt
$ sudo ./letsencrypt-auto certonly -a webroot --webroot-path=/var/www -d bananasusu.com
```
接着输入找回密码的邮箱并同意条款，稍等片刻就会有Output

```
Output:
IMPORTANT NOTES:
 - If you lose your account credentials, you can recover through
   e-mails sent to sz.yaocheng@gmail.com
 - Congratulations! Your certificate and chain have been saved at
   /etc/letsencrypt/live/bananasusu.com/fullchain.pem. Your
   cert will expire on 2016-09-13. To obtain a new version of the
   certificate in the future, simply run Let's Encrypt again.
 - Your account credentials have been saved in your Let's Encrypt
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Let's
   Encrypt so making regular backups of this folder is ideal.
 - If like Let's Encrypt, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

在`/etc/letsencrypt/live/bananasusu.com/`这个目录下可以找到生成的四个文件，分别是
- cert.pem #域名的证书
- chain.pem #Let's Encrypt chain的证书
- fullchain.pem #前面讲个证书的合并
- privkey.pem #域名证书的私钥



#### 定期更新
Let's Encrypt证书默认的有效期是90天，所以需要在证书过期前进行`续约(renew)`。在Linux上可以通过`crontab`来创建定时job，cron是一个scheduler，通过[特殊的语法](http://www.corntab.com/pages/crontab-gui)定义任务的执行时间，所有的任务都定义在一个文件中，用一下命令编辑这个文件
```bash
$ sudo crontab -e
```
加入以下两行
```bash
30 2 * * 1 /opt/letsencrypt/letsencrypt-auto renew >> /var/log/le-renew.log
35 2 * * 1 /etc/init.d/nginx reload
```
意思就是
1. 在每周一的2:30AM，执行renew脚本，将log保存到/var/log/le-renew.log
2. 五分钟后让Nginx重新加载配置

这样一来，就能在证书过期前renew，保证HTTPS可用。

## 配置Nginx

由于配置比较长，我直接拷贝[h5bp/server-config-nginx](https://github.com/h5bp/server-configs-nginx/blob/master/h5bp/directive-only/)的ssl.conf和ssl-stapling.conf配置块到`/etc/nginx/`，里面有详细的注释说明，然后把证书换成自己的

``` nginx
    ssl_certificate /etc/letsencrypt/live/bananasusu.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/bananasusu.com/privkey.pem;
```

接着打开站点的server配置项，我是`/etc/nginx/sites-available/bananasusu.com`，引入上面下载的两个配置文件
``` nginx
	include ssl.conf;
	include ssl-stapling.conf;
```

然后将之前监听80端口换成443端口，把

``` nginx
listen 80 default_server;
listen [::]:80 default_server ipv6only=on;
```
改为
``` nginx
	listen 443 ssl;
```

让通过http 80端口的访问重定向到443

``` nginx
server {
	listen 80;
	server_name: bananasusu.com;
	return 301 https://$host$request_uri;
}
```

保存并reload `sudo nginx -s reload`

#### 实用小技巧 -> 如果在reload的时候报错，可以用`nginx -t`来debug


#### 验证
登录bananasusu.com，地址栏出现绿色的小锁和https，表明连接是加密的。

![https-lock](/images/https-with-lets-encrypt/https-lock-min.png "出现绿色的小锁")

查看证书

![cert](/images/https-with-lets-encrypt/cert-min.png "Let's Encrypt CA颁发给bananasusu.com的证书")

不服再来[跑个分](https://www.ssllabs.com/ssltest/analyze.html?d=bananasusu.com)

![ssl-report](/images/https-with-lets-encrypt/ssl-report-bananasusu-min.png "A+ 高考能加分吗")

妥妥的A+ 高考能加分吗

## 参考链接
- [How To Secure Nginx with Let's Encrypt on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-14-04)
- [Upgrading Nginx to the latest version on Ubuntu servers](https://leftshift.io/upgrading-nginx-to-the-latest-version-on-ubuntu-servers)
- [Let's Encrypt 给网站加 HTTPS 完全指南](https://ksmx.me/letsencrypt-ssl-https)
- [certbot](https://certbot.eff.org/)
- [Getting a Perfect SSL Labs Score](https://michael.lustfield.net/nginx/getting-a-perfect-ssl-labs-score)