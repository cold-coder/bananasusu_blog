---
title: Nginx缓存
tags:
  - 技术分享
  - Linux
  - nginx
  - cache
  - 性能优化
date: 2016-06-15 09:49:51
---


### Intro

博客上线好几天了，发现种种问题，其中之一就是没做缓存，结果就是每次打开或者点后退时都会从服务器重新获取数据和资源，非常低效(慢)。

### 排查

对于这种速度，首先做的当然是打开浏览器的开发者工具的Network，看到底是哪个资源慢了。

![Chrome开发者工具](/images/cache-with-nginx/blog_network_no_cache-min.png "Chrome开发者工具")

这一看问题还不少

1. 图片过于庞大，金门大桥单图达到252kb，在正常的网络条件下需要10s以上的时间才能加载完成

2. 默认来自Google Font CDN上的字体在国内被墙，导致页面假死

3. Nginx没有对静态资源开启缓存

### 方案

1. 对于博客中的图片，先进行尺寸的剪裁，然后进行压缩，确保最终大小在100kb以内。

2. 对于被墙的字体，用国内的[360的CDN](http://libs.useso.com/)代替，寻找到主题配置文件中的font选项
``` yaml
	font:
	enable: true

	# Uri of fonts host. E.g. //fonts.googleapis.com (Default)
	host: //fonts.useso.com #mirror for google stuff by 360
```

3. 开启Nginx的缓存
GitHub上面有一个项目叫做[H5BP](https://h5bp.github.io/)，上面的项目都是当今网站开发从前端到后端的最佳实践，里面有一个[Nginx的配置](https://github.com/h5bp/server-configs-nginx)项目就是一个完整的Nginx配置的范例，每一个配置都有详尽的注释说明。
在这个项目配置中，我找到了关于缓存的配置，具体路径在`server-configs-nginx/h5bp/location/expires.conf`，于是将他wget到自己的Nginx配置路径下。
``` bash
$ sudo wget https://raw.githubusercontent.com/h5bp/server-configs-nginx/master/h5bp/location/expires.conf
```
接着我们看看这里面到底写了些什么
   ``` nginx
   # cache.appcache, your document html and data
   location ~* \.(?:manifest|appcache|html?|xml|json)$ {
     expires -1;
   }
   
   # Feed
   location ~* \.(?:rss|atom)$ {
     expires 1h;
   }
   
   # Media: images, icons, video, audio, HTC
   location ~* \.(?:jpg|jpeg|gif|png|ico|cur|gz|svg|svgz|mp4|ogg|ogv|webm|htc)$ {
     expires 1M;
     access_log off;
     add_header Cache-Control "public";
   }
   
   # CSS and Javascript
   location ~* \.(?:css|js)$ {
     expires 1y;
     access_log off;
   }
   
   # WebFonts
   # If you are NOT using cross-domain-fonts.conf, uncomment the following directive
   # location ~* \.(?:ttf|ttc|otf|eot|woff|woff2)$ {
   #  expires 1M;
   #  access_log off;
   # }
   ```
这个配置大体是对于不同类型的文件采用不用的缓存策略，有的是不缓存(比如html和json)，有的是一个小时(比如rss和atom)，有的一个月(图片音频文件)，还有的一年(js和css)。
下面把这个配置包含进我们的server配置块中
   ``` nginx
	include expires.conf;
   ```
让Nginx重新载入配置文件
   ``` bash
	$ sudo nginx -s reload
   ```

### 测试

刷新我们的bananasusu.com,再次打开Chrome的开发者工具查看网络，我们会发现在服务器返回的资源的response中多了几个头部信息。
比如下图这张图片，可以看到他的头部多了个`Cache-Control`的头，这个就是Nginx加上的缓存头信息，还多了一个Expires头，指定了过期的日期，注意这个日期是用格林尼治标准时间，所以是我们东8区的时间减去8小时。由于我们对图片资源设的有效期是1年，所以Expires是明天的今天。浏览器就是根据这个时间判断是从服务器拿还是直接从本地缓存取。
![加载有缓存的图片资源](/images/cache-with-nginx/blog_image_with_cache-min.png "加载有缓存的图片资源")

其他资源也是类似，都是按照我们Nginx配置的时间来设置有效期。

站点优化之路漫漫其修远兮，吾将上下而求索。

### 参考链接

- [Nginx Caching](https://serversforhackers.com/nginx-caching)