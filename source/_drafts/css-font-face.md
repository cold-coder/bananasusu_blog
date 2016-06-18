---
title: CSS @font-face
tags:
- 技术分享
- Frond-End
- CSS
- font
---

![libre-baskerville-ampersands](/images/css-font-face/libre-baskerville-ampersands-min.png "Ampersand of Baskerville-italic")

@font-face允许我们在网页中使用自定义的字体
<!-- more -->

## 网页字体
在CSS中，我们通过`font-family`属性来指定字体
``` css
font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif;
```
上面代码中定义了一个`font-family`属性，一共四个字体，之间以逗号分开(如果字体中有空格，需要加上引号)。 

浏览器会从左到右，找到第一个本地安装的字体并使用。

## @font-face

但是有时候，我们就是要使用某个电脑上没有安装的字体，这时候，@font-face就可以派上用场了

```css
@font-face {
  font-family: 'Source Sans Pro';
  font-style: normal;
  font-weight: 400;
  src: local('Source Sans Pro'), local('SourceSansPro-Regular'), url('../fonts/SourceSansPro-Regular.woff2') format('woff2');
  unicode-range: U+0102-0103, U+1EA0-1EF9, U+20AB;
}
```

代码中自定义了一个`Source Sans Pro`的字体


## 使用[Google Fonts](https://developers.google.com/fonts/)
Google Fonts是一个云字体库，只要在html`<head>`中插入一个内联CSS的Link，把要使用的字体当作参数传入
```html
<link href='//fonts.googleapis.com/css?family=Source+Sans+Pro' rel='stylesheet' type='text/css'>
```
这样我们也就可以在页面使用`Source Sans Pro`字体了。

在国内需要用[360网站卫士常用前端公共库CDN服务](http://libs.useso.com/)代替，原因你懂的。

## 参考资料
- [CSS Font Stack](http://www.cssfontstack.com/)
- [Unicode-range](http://atozcss.com/intermediate/video/unicode-range-and-at-font-face/)
- [Using @font-face](https://css-tricks.com/snippets/css/using-font-face/)
- [360网站卫士常用前端公共库CDN服务](http://libs.useso.com/)
- [阮一峰-中文字体网页开发指南](http://www.ruanyifeng.com/blog/2014/07/chinese_fonts.html)
- [阮一峰-字体笔记](http://www.ruanyifeng.com/blog/2008/06/typography_notes.html)