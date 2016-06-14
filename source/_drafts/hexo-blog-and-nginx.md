---
title: 使用Hexo搭建博客
tags: 
- 技术分享
- Linux
- godaddy
- digital ocean
- hexo
- nginx
---

## Buy domain from Godaddy

### buy and pay

### configure dns

### test


## Hexo

### hexo intro

### install dep

### config hexo

### start writing

### hexo workflow


## Nginx

### install nginx


### configure


启动nginx
``` bash
$ service nginx start
```

``` bash
$ service nginx status
```

reload configure
``` bash
$ nginx -s reload
```

shutdown nginx
``` bash
$ ps -ax | grep nginx
```
find out the pid and sent a signal to that process
``` bash
$ kill -9 1628
```


配置文件的路径 `/etc/gningx`


find number of cpu cores

``` bash
$ nproc
$ 1
```


### test
