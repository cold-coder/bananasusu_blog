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

个人建站是每一个程序员都应该掌握的基本技能。

如果只是想写写文章，也不用个人建站，有很多现成的博客平台就能满足，比如[简书](http://www.jianshu.com/)，[知乎专栏](https://zhuanlan.zhihu.com/)等等。

<!-- more -->

## 购买域名
如果没有域名，那访问只能通过IP，这显然是反人类的。

世界上有很多卖域名的，比如阿里云旗下的[万网](https://wanwang.aliyun.com/)，国际上的[GoDaddy](https://sg.godaddy.com/)等。

之所以选择GoDaddy，是因为好像在国内买是要备案的。

选择域名的原则是简单好记的，难记的话也就失去了意义。但是当今好记的域名几乎都被注册了，记得在互联网刚火起来的时候，就有人靠抢注域名发家。

`bananasusu.com`这个域名其实我很久以前就想好了，它与我的[微博ID](http://weibo.com/bananasusu)一致，前面的banana加后面的susu，翻译过来就是我微博的第一个呢称-`巴娜娜叔叔`。

### 购买并付款
就像其他剁手网站一样，选择你要购买的域名后加入购物车，填写域名持有人信息，然后就可以付款了，付款支持支付宝。

这里有个小插曲，在支付宝扫码付款后，网页跳转挂了，所以支付宝的钱已经扣了，域名还在购物车。无奈只能打电话找客服。

![godaddy-support](/images/blogging-with-hexo/godaddy-support-min.png "GoDaddy的金牌客服")

虽然是端午小长假，不过客服还是有人的，在一番交涉后，客服妹子又给我发了个付款码，并告诉我之前的支付失败并已经退款，钱会在30天内退回到支付宝(结果第二天就到了)，扫了妹子的付款码，域名就买好了。

### 配置域名的DNS
由于我的VPS是在Degital Ocean买的，因此需要把Digital Ocean的DNS服务器填到GoDaddy的域名配置项里面，让DNS能指向DO的主机

![godaddy-dns](/images/blogging-with-hexo/gd-dns-min.png "将Digital Ocean提供的DNS服务器地址填写到GoDaddy域名的DNS管理页")

### Whois测试
这时就可以用`whois bananasusu.com`命令来测试了
``` bash
$ whois bananasusu.com
$
```

### 配置Droplet
还差最后一步，配置DO Droplet的DNS记录，增加一条A记录(IPv4)，一条AAAA记录(IPv6)，一条CNAME记录(子域名跳转)
![digital-ocean-dns](/images/blogging-with-hexo/do-dns-min.png "配置Digital Ocean的DNS")

### ping测试
最后，`ping`一下
``` bash
$ ping bananasusu.com

```
通了，说明域名已经跟IP成功绑定了。


## Hexo

### 博客平台的选择
现在市面上有很多博客平台，最出名的要数WordPress，这是一个老牌的基于PHP的博客平台，前几年风靡一时。此外还有基于Ruby的[Jekyll](https://jekyllrb.com/)，基于NodeJS的[Ghost](https://github.com/TryGhost/Ghost)，这些都是静态博客平台，即最终文件都会生成静态html文件。

平台的选择应该基于个人的技术栈，比如我对JS比较熟悉，就会现在基于NodeJS的平台，所以看似Ghost是一个不错的选择。

但是，安装Ghost居然要装MySQL。。。这有点让惊讶，原来Ghost是一个重量级的博客平台，那就算了吧。

这是[Hexo](https://hexo.io/)出现了 
> A fast, simple & powerful blog framework

好，就是它了。

### 安装
#### 先安装NodeJS和Git
``` bash
$ sudo install nodejs git
```
``` bash
$ node -v
$ v4.4.5
$ git --version
$ git version 1.9.1
```

#### 安装hexo
``` bash
$ sudo npm install hexo-cli -g
```

### 配置hexo

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

## 参考链接
- [How To Set Up a Host Name with DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-host-name-with-digitalocean)