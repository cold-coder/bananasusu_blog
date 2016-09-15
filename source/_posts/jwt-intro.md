---
title: JWT浅析
tags:
  - 技术分享
  - JWT
date: 2016-09-15 21:11:55
---


![jwt](/images/jwt-intro/logo.svg  "jwt-logo")

可能你在很都场合都听过JWT，他的全称叫JSON Web Tokens，说到Token，那一般就是登录认证用的。

<!-- more -->

## 登录控制

登录控制在很多应用都会用到，尤其是安全性较高的应用，对登录的把控尤其严密。

#### SESSION时代

在说JWT之前，可以先回顾一下以前的应用里是着怎么控制登录的，一般都是用户提供用户名密码，服务器在认证后生成一个字符串，称之为Session返回给客户端，同时自己保存（通常放Cache或者Redis之类的缓存数据库里），以后的每次请求客户端都会把Session带上，服务器对Session进行认证，如果该Session在缓存里，则正常返回数据，不在的话就让用户重新登录。

这样做的问题是当用户数量大了，缓存的开销就大，特别是当认证服务器均衡负载时问题就更加严重。

#### TOKEN来了

这时Token就应运而生了，Token翻译过来是令牌的意思，这个翻译很贴切。它的流程和Session差不多，唯一的不同是服务器不会保存生成的Token，每次请求客户端同样需要把Token带上，但是服务器做的是验证（Verify）这个Token，而不是去内存里找有没有。这就是所谓的Stateless。

![jwt-diagram](/images/jwt-intro/jwt-diagram.png  "jwt-diagram")

打个比方，你开车去学校教书，第一次进门保安上来盘问一番，你告诉他们你是老师，他们在确认你的身份后给你发了一张临时通行证，下次进出学校门保安看到这张证就放行了。后来学校改造，所有的门都换了ETC杆，你拿到了一张通行卡，只要把卡放在车里，到校门前就自动生杆。前面的盘问发通行证就是Session的方式，后面的ETC就是Token方式。


## JWT

#### 协议

JWT的协议参照[RFC 7519](https://tools.ietf.org/html/rfc7519), 整个协议分为三个部分
- Header
- Body/Payload
- Signature

![jwt-structure](/images/jwt-intro/JWTStructure.png "jwt-structure")

下面分别来讲讲这几个部分

#### Header

Header的两个字段是固定的，分别是`alg`算法和`typ`类型，算法是签名算法，就是最后计算Signature的算法，类型是JWT

#### Body/Payload

这里面可以放一些登录用户的信息，比如用户ID，用户名什么的，比如
```
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```
但是决定不能放敏感的信息，密码什么的，同时有几个保留的名字，如iss (issuer), exp (expiration time), sub (subject), aud (audience)等是不能当作key用的。

#### Signature

为什么服务器能验证这个Token是自己发出来的全靠这个Signature，它的生成方法如下
```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```
将前两部分编码后拼起来，再加一个自己的secret，然后用签名Header里提到的签名算法进行加密后得来的。secret是存放在服务器上的一串密钥。

最后将这三部分base64编码后拼起来，中间用`.`分割，就形成了一个类似`xxxxxx.yyyyyyyyyyyy.zzzzzzzzzzzz`的字符串，这就是token

![jwt-encode](/images/jwt-intro/jwt-encode.png "jwt-io")


[jwt.io](http://jwt.io) 提供了如上图的编码与协议左右对照，让人一目了然。

## 使用

在服务器返回Token后，客户端需要将Token存起来，一般存在LocalStorage里面，每次请求是放到请求的Header里面
```
Authorization: Bearer <token>
```
服务器拿到Token后就可以进行验证了，这个头其实是可以自定义的，叫什么都可以，只要服务器能解析到Token就行了。

以上就是JWT的基本概念和使用方法，有不清楚的可以给我留言 :)

## 参考链接
- [jwt.io](https://jwt.io/)
