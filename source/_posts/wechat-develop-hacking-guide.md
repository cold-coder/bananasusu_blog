---
title: 微信开发填坑指南
date: 2017-12-25 10:13:01
tags:
    - 技术分享
    - 微信开发
---

![wechat-development](/images/wechat-develop-hacking-guide/wechat-development.png  "wechat-development")

微信浏览器是移动时代的IE6

<!-- more -->

### 起因

虽然之前也做过一阶段的微信开发，但大多都是基于前人搭好的框架，这次从零开始，踩了不少的坑，在这篇一一记下。

### SPA支付目录的配置问题

支付目录指的是发起指微信支付请求的目录。微信后台允许配置`三`个支付目录。理论上这个目录应该就是你的微信站点发布的目录。比如，`http://bananasusu.com/wx/`，在这个目录下所有的网页发起的微信支付都应该是合法的。然而，当使用SPA时，例如在`http://bananasusu.com/wx/#/order/434874`下发起支付，会遇到`当前页面的URL未注册`的错误，而当我们把支付目录设为`http://bananasusu.com/wx/#/order`才可以完成支付。显然微信不能正确识别hash路由下的目录结构。

怎样才能让微信识别我们的SPA的目录呢（我猜测是微信不支持pushState的跳转方式，所以尝试用location.href的方式跳转到支付页面，但是并没有效果。）后来我看到了[这篇文章](https://www.tuicool.com/articles/mQ7RRfb)。他用了`?#`代替`#`，这个hack首先保证我们的程序可以正常工作，而`?`的使用可以让微信把后面的路由认为是参数，最终识别到正确的路径。可以说是一个四两拨千斤的方案。

在感叹群众的智慧的同时，还要吐槽一下微信，在SPA当道的今天，对其支持居然如此之差，真让人心寒。

### JS-SDK getLocalImgData的问题

微信中如果要上传图片，可以用原生的`<input tppe='file'>`的上传方式，不过这样做有一个明显的缺点，就是图片不能压缩，现在手机照片一张都要3M左右，上传时间也就会很长。

微信的JS-SKD提供一个`wx.chooseImage`的方法，而且可以配置上传前对图片进行压缩处理。那就用它吧，真是个愉快的决定 :)

选完照片后需要显示出来，之前的做法是使用`FileReader.readAsDataURL()`的方法将图片转成base64编码喂给img标签的src，而如果用JS-SDK提供了的`wx.getLocalImgData`的方法，只需要将之前`wx.chooseImage`返回的`localId`当参数传入，就能返回图片的base64编码，嗯哼，so easy，不需要写多余代码了。然而我还是太年轻了，这个方法有两个愚蠢的bug，是两个！

1. 在iOS和Android上返回的base64编码格式不统一，安卓上没有`data:image/jpeg;base64,`前缀
2. iOS上的`data:image/jpeg;base64,`前缀`jpeg`会变成`jgp`。。。**jgp** 什。么。鬼。

针对第一个问题，我们需要在前面补上前缀，但是到底是jpeg，还是png格式，这个还是要从base64的chunk里面找线索，幸好我找到了[一个gist](https://gist.github.com/nazoking/2822127)

``` javascript
function guessImageMime(data){
  if(data.charAt(0)=='/'){
    return "image/jpeg";
  }else if(data.charAt(0)=='R'){
    return "image/gif";
  }else if(data.charAt(0)=='i'){
    return "image/png";
  }
}
```
简单粗暴，有时候没必要考虑那么复杂，短短几行就能解决问题。

第二个问题相对简单，只要把jgp替换成jpeg就行了，`localData.replace('jgp', 'jpeg')`，但万一base64的chunk里也有`jgp`就完蛋了，虽然这个概率微乎其微。

[微信的JS-SDK开发文档](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115)上对这两个问题只字未提，就静静地望着开发者掉入坑里，心寒啊！

### 未完待续

微信开发的坑肯定还有很多很多，填都填不完，等遇到再来更新吧~

## 参考链接
- [使用微信的 JS SDK 选取手机照片并进行上传](https://segmentfault.com/a/1190000008656542)
- [开发单页应用(SPA)时候遇到的微信支付授权目录的坑](https://www.tuicool.com/articles/mQ7RRfb)
- [FileReader.readAsDataURL()](https://developer.mozilla.org/en-US/docs/Web/API/FileReader/readAsDataURL)
- [get image mime type from base64](https://gist.github.com/nazoking/2822127)