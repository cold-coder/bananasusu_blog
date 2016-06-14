---
title: 开启缓存
tags: 
- 技术分享
- Linux
- nginx
- cache
---

### Intro

博客上线好几天了，发现种种问题，其中之一就是没做缓存，结果就是每次打开或者点后退时都会从服务器重新获取数据和资源，非常低效(慢)。

### 排查

对于这种速度，首先做的当然是打开浏览器的开发者工具的Network，看到底是哪个资源慢了。

![Chrome开发者工具](/images/cache-with-nginx/blog_network_no_cache-min.png "Chrome开发者工具")

这一看问题还不少

1. 图片过于庞大，金门大桥单图达到252kb，在正常的网络条件下需要10s以上的时间才能加载完成

2. 默认来自Google Font CDN上的字体在国内被墙，导致页面假死

3. Nginx没有开启缓存

下面是我的解决方案

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
这个配置大体是对于不同类型的文件采用不用的缓存策略，有的是不缓存(比如html和json)，有的是一个小时(比如rss和atom)，有的一个月(图片音频文件)还有的一年(js和css)。
下面把这个配置包含进我们的server配置块中
   ``` nginx
	include expires.conf;
   ```
让Nginx重新载入配置文件
   ``` bash
	$ sudo nginx -s reload
   ```
刷新我们的bananasusu.com,再次打开Chrome的开发者工具查看网络，我们会发现在每个资源的request中多了几个头部信息。