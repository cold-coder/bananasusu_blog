---
title: http2-with-nginx
tags:
- 技术分享
- Linux
- http2
- nginx
---

![http2-multiplexing](/images/http2-with-nginx/multiplexing.png  "HTTP/2 Multiplexing")

HTTP2协议是HTTP1.1的升级版，拥有更快的传输速度和更低的延迟。
<!-- more -->

## Nginx

#### 升级Nginx
Nginx 1.9.5才开始支持HTTP/2，而服务器上的Nginx是1.4.6
```  bash
$ nginx -v
$ nginx version: nginx/1.4.6 (Ubuntu)
```

先备份配置文件
```bash
$ sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.1.4.6.backup
```

停止nginx服务
```bash
$ sudo service nginx stop
```

安装依赖
``` bash
$ sudo aptitude install python-software-properties
```

添加Nginx的repo
```bash
$ sudo add-apt-repository ppa:nginx/stable
```

更新
``` bash
$ sudo aptitude update
$ sudo aptitude safe-upgrade
```

重启Nginx
```bash
$ sudo service nginx restart
```

再次查看版本
```bash
$ nginx -v
$ nginx version: nginx/1.10.1
```

#### 启用http2
Nginx上启用http2非常简单，在开启HTTPS的前提下只要在ssl的后面加上http2这个指令就可以了
``` nginx
	listen 443 ssl http2;
```

重新加载配置文件`nginx -s reload`

#### 测试
在浏览器中我们并不能直接看出网页是否是http2传输，需要打开开发者工具查看协议，如果是`h2`则说明已经成功了。


## 参考链接

- [Upgrade nginx](https://leftshift.io/upgrading-nginx-to-the-latest-version-on-ubuntu-servers)
- [HTTP2 Explained](https://daniel.haxx.se/http2/)
- [How To Set Up Nginx with HTTP/2 Support on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-with-http-2-support-on-ubuntu-16-04)
