---
title: 全站HTTPS
tags: 
- 技术分享
- Linux
- nginx
- https
- let's encrypt
---

![nginx-letsencrypt](/images/https-with-lets-encrypt/nginx-letsencrypt-min.png "Let's Encrypt secure our website with Nginx")

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

跟着官网上的步骤我们开始

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

### 生成证书
用letsencrypt的脚本生成证书， webroot-path是你网站部署的路径，d是域名
``` bash
$ cd /opt/letsencrypt
$ sudo ./letsencrypt-auto certonly -a webroot --webroot-path=/var/www -d bananasusu.com
```
接着会要你输入找回密码的邮箱，同意条款，稍等片刻就会有Output

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

在`/etc/letsencrypt/live/bananasusu.com/`这个目录下我们可以找到生成的四个文件，分别是
- cert.pem
- chain.pem
- fullchain.pem
- privkey.pem



### 定期更新
Let's Encrypt证书默认的有效期是90天，所以有必要在过期前就行'续约(renew)'。在Linux上我们可以通过`crontab`来创建定时job

## 配置Nginx

### 配置
由于配置比较长，我推荐直接拷贝[h5bp/server-config-nginx](https://github.com/h5bp/server-configs-nginx/blob/master/h5bp/directive-only/)的ssl.conf和ssl-stapling.conf配置块到`/etc/nginx/`，里面有详细的注释说明，记得把证书换成自己的

``` nginx
    ssl_certificate /etc/letsencrypt/live/bananasusu.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/bananasusu.com/privkey.pem;
```

接着打开站点的server配置项，我是`/etc/nginx/sites-available/bananasusu.com`，引入上面下载的两个配置文件
``` nginx
	include ssl.conf;
	include ssl-stapling.conf;
```

然后将之前监听80端口换成443端口

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


### 验证
登录bananasusu.com，地址栏出现绿色的小锁和https，表明连接是加密的。

![https-lock](/images/https-with-lets-encrypt/https-lock-min.png "出现绿色的小锁")

查看证书

![cert](/images/https-with-lets-encrypt/cert-min.png "Let's Encrypt CA颁发给bananasusu.com的证书")

不服再来[跑个分](https://www.ssllabs.com/ssltest/analyze.html?d=bananasusu.com)

![ssl-report](/images/https-with-lets-encrypt/ssl-report-bananasusu-min.png "A+ 高考能加分吗")


## 参考链接
- [How To Secure Nginx with Let's Encrypt on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-14-04)
- [Upgrading Nginx to the latest version on Ubuntu servers](https://leftshift.io/upgrading-nginx-to-the-latest-version-on-ubuntu-servers)
- [Let's Encrypt 给网站加 HTTPS 完全指南](https://ksmx.me/letsencrypt-ssl-https)
- [certbot](https://certbot.eff.org/)
- [Getting a Perfect SSL Labs Score](https://michael.lustfield.net/nginx/getting-a-perfect-ssl-labs-score)