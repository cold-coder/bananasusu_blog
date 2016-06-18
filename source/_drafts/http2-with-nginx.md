---
title: http2-with-nginx
tags:
- 技术分享
- Linux
- http2
- nginx
---

<!-- more -->

## Nginx

#### 升级Nginx
Nginx 1.9.5才开始支持HTTP/2，而服务器上的Nginx是1.4.6
```  bash
$ nginx -v
$ nginx version: nginx/1.4.6 (Ubuntu)
```
backup
```bash
$ sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.1.4.6.backup
```

stop service
```bash
$ sudo service nginx stop
```
install dependencies
``` bash
$ sudo aptitude install python-software-properties
```

add repo
```bash
$ sudo add-apt-repository ppa:nginx/stable
```

update and upgrade
``` bash
$ sudo aptitude update
$ sudo aptitude safe-upgrade
```

restart nginx
```bash
$ sudo service nginx restart
```

verify
```bash
$ nginx -v
$ nginx version: nginx/1.10.1
```

#### Config Nginx



## Reference
- [Upgrade nginx](https://leftshift.io/upgrading-nginx-to-the-latest-version-on-ubuntu-servers)
