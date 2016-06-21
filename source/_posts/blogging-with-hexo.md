---
title: 使用Hexo搭建博客
tags:
  - 技术分享
  - Linux
  - godaddy
  - digital ocean
  - hexo
date: 2016-06-21 09:42:19
---

![zhihu zhuanlan](/images/blogging-with-hexo/zhihu-zhuanlan-min.png "知乎专栏")

如果只是想写写文章，也不用个人建站，有很多现成的博客平台就能满足，比如[简书](http://www.jianshu.com/)，[知乎专栏](https://zhuanlan.zhihu.com/)等等。

而这里我选择的是从无到有自己搭一个，这样的话自由度更高一点。

<!-- more -->

## 购买域名
如果没有域名，那访问只能通过IP，这显然是反人类的。

世界上有很多卖域名的，比如阿里云旗下的[万网](https://wanwang.aliyun.com/)，国际上的[GoDaddy](https://sg.godaddy.com/)等。

之所以选择GoDaddy，是因为好像在国内买的域名是要备案的。

选择域名的原则是简单好记的，难记的话也就失去了意义。但是当今好记的域名几乎都被注册了，记得在互联网刚火起来的时候，就有人靠抢注域名发家。

`bananasusu.com`这个域名其实我很久以前就想好了，它与我的[微博ID](http://weibo.com/bananasusu)一致，前面的`巴娜娜`加后面的`susu`，基本上可以过目不忘吧。

#### 购买并付款
就像其他剁手网站一样，选择你要购买的域名后加入购物车，填写域名持有人信息，然后就可以付款了，付款支持支付宝。

这里有个小插曲，在支付宝扫码付款后，网页跳转挂了，所以支付宝的钱已经扣了，域名还在购物车。幸好页面上客服的电话还算醒目，只能打过去了哇。

![godaddy-support](/images/blogging-with-hexo/godaddy-support-min.png "GoDaddy的金牌客服")

虽然是端午小长假，不过客服还是有人的，在一番交涉后，客服妹子又给我发了个付款码，并告诉我之前的支付失败并已经退款，钱会在30天内退回到支付宝(结果第二天就到了)，扫了妹子的付款码，域名就买好了。

> 哥也是有域名的人了。

### 配置域名的DNS

关于DNS的工作原理，最近阮一峰老师的[这篇文章](http://www.ruanyifeng.com/blog/2016/06/dns.html)已经说得很清楚了，我这里就不科普了。

由于我的VPS是在Degital Ocean买的，因此需要把Digital Ocean的DNS服务器填到GoDaddy的域名配置项里面，让DNS能指向Digital Ocean的主机

![godaddy-dns](/images/blogging-with-hexo/gd-dns-min.png "将Digital Ocean提供的DNS服务器地址填写到GoDaddy域名的DNS管理页")

### Whois测试
这时就可以用`whois bananasusu.com`命令来测试了
``` bash
$ whois bananasusu.com
```
不用命令的话很多域名网站也是可以查到的

![whois-bananasusu](/images/blogging-with-hexo/whois-bananasusu-min.png "whois bananasusu")

#### 配置Droplet
还差最后一步，配置VPS的DNS记录，增加一条A记录(IPv4)，一条AAAA记录(IPv6)，一条CNAME记录(子域名跳转)
![digital-ocean-dns](/images/blogging-with-hexo/do-dns-min.png "配置Digital Ocean的DNS")

#### ping测试
最后，`ping`一下
``` bash
$ ping bananasusu.com
```
通了，说明域名已经跟IP成功绑定了。


## Hexo

#### 博客平台的选择
现在市面上有很多博客平台，最出名的要数WordPress，这是一个老牌的基于PHP的博客平台，前几年风靡一时。此外还有基于Ruby的[Jekyll](https://jekyllrb.com/)，基于NodeJS的[Ghost](https://github.com/TryGhost/Ghost)，这些都是静态博客平台，即最终文件都会生成静态html文件。

平台的选择应该基于个人的技术栈，比如我对JS比较熟悉，就会现在基于NodeJS的平台，所以看似Ghost是一个不错的选择。

但是，安装Ghost居然要装MySQL。。。这有点让我惊讶，原来Ghost是一个重量级的博客平台，那就算了吧。

这时[Hexo](https://hexo.io/)出现了 
> A fast, simple & powerful blog framework

好，就是它了。

#### 安装
安装NodeJS和Git
``` bash
$ sudo install nodejs git
```
``` bash
$ node -v
$ v4.4.5
$ git --version
$ git version 1.9.1
```

安装hexo
``` bash
$ sudo npm install hexo-cli -g
```
安装完毕就可以使用hexo命令了
```bash
$ hexo init bananasusu_blog
```
进入bananasusu_blog目录，安装依赖
```bash
$ cd bananasusu_blog
$ npm install
```
然后就可以开始写博客了，新建一篇文章
```bash
$ hexo new breeze-from-west-cost
```
hexo内置一个运行静态服务器命令
```bash
$ hexo server
```
访问localhost:4000，整个站点就展现在你眼前了。

更多命令请访问 [Hexo Commands](https://hexo.io/docs/commands.html)。

#### 配置
Hexo用过一个`_config.yml`来配置站点，[官方文档](https://hexo.io/docs/configuration.html)也有详细的说明。

#### 主题
[Hexo提供了相当多的主题](https://hexo.io/themes/)，安装也相当简单，只要把主题代码clone到themes目录下，然后在`_config.yml`中启用就可以了。

#### 生成静态文件
执行
```bash
$ hexo generate
```
生成静态站点，生成目录为`public`，生成好之后可以进去用`anywhere`打开看一下，应该是和之前`hexo server`的一模一样。

#### 部署
部署就是把上一步生成的静态站点目录推送的服务器上，Hexo通过插件支持多个部署平台，如[GitHub Pages](https://pages.github.com/)，[Heroku](https://www.heroku.com/)，而我是个人VPS，所以用的rsync。

先安装`hexo-deployer-rsync`
```bash
$ npm install hexo-deployer-rsync --save
```
在配置`_config-yml`里的deploy配置项
```yml
deploy:
  type: rsync
  host: bananasusu.com
  user: yaocheng
  root: /var/www
  port: 22
```

<div class="tip">
	这里有个坑，虽然文档上说端口默认是22，但是上面的port端口不能省略，必须显式指定22，不然会导致部署失败。
</div>

然后执行`hexo deploy`就可以一键部署了，这是访问bananasusu.com就可以看到发布的站点了。



## 参考链接
- [How To Set Up a Host Name with DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-host-name-with-digitalocean)
- [趣谈个人建站](http://macshuo.com/?p=547)