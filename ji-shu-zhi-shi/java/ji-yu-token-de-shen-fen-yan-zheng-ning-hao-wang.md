# 基于 Token 的身份验证 - 宁皓网

## 基于 Token 的身份验证 - 宁皓网

![](https://github.com/yangbao93/docs/tree/d23f2b2cbc4eb06e62d38114d6a7f5410080c7b5/技术知识/Java/基于%20Token%20的身份验证%20-%20宁皓网/image_1.png)

* [课程](http://ninghao.net/course)
* [博客](http://ninghao.net/blog)
* [社区](http://talk.ninghao.net/)
* [登录](http://ninghao.net/user/login?destination=node/2834)
* [注册](http://ninghao.net/user/register)
* [订阅课程](http://ninghao.net/signup)

## 基于 Token 的身份验证

作者： [王皓](http://ninghao.net/user/6) 发布于：2015-08-07 22:06 更新于：2015-08-07 22:07 最近了解下基于 Token 的身份验证，跟大伙分享下。很多大型网站也都在用，比如 Facebook，Twitter，Google+，Github 等等，比起传统的身份验证方法，Token 扩展性更强，也更安全点，非常适合用在 Web 应用或者移动应用上。Token 的中文有人翻译成 “令牌”，我觉得挺好，意思就是，你拿着这个令牌，才能过一些关卡。

### 传统身份验证的方法

HTTP 是一种没有状态的协议，也就是它并不知道是谁是访问应用。这里我们把用户看成是客户端，客户端使用用户名还有密码通过了身份验证，不过下回这个客户端再发送请求时候，还得再验证一下。

解决的方法就是，当用户请求登录的时候，如果没有问题，我们在服务端生成一条记录，这个记录里可以说明一下登录的用户是谁，然后把这条记录的 ID 号发送给客户端，客户端收到以后把这个 ID 号存储在 Cookie 里，下次这个用户再向服务端发送请求的时候，可以带着这个 Cookie ，这样服务端会验证一个这个 Cookie 里的信息，看看能不能在服务端这里找到对应的记录，如果可以，说明用户已经通过了身份验证，就把用户请求的数据返回给客户端。

上面说的就是 Session，我们需要在服务端存储为登录的用户生成的 Session ，这些 Session 可能会存储在内存，磁盘，或者数据库里。我们可能需要在服务端定期的去清理过期的 Session 。

### 基于 Token 的身份验证方法

使用基于 Token 的身份验证方法，在服务端不需要存储用户的登录记录。大概的流程是这样的：

1. 客户端使用用户名跟密码请求登录
2. 服务端收到请求，去验证用户名与密码
3. 验证成功后，服务端会签发一个 Token，再把这个 Token 发送给客户端
4. 客户端收到 Token 以后可以把它存储起来，比如放在 Cookie 里或者 Local Storage 里
5. 客户端每次向服务端请求资源的时候需要带着服务端签发的 Token
6. 服务端收到请求，然后去验证客户端请求里面带着的 Token，如果验证成功，就向客户端返回请求的数据

### JWT

实施 Token 验证的方法挺多的，还有一些标准方法，比如 JWT，读作：jot ，表示：JSON Web Tokens 。JWT 标准的 Token 有三个部分：

* header
* payload
* signature

中间用点分隔开，并且都会使用 Base64 编码，所以真正的 Token 看起来像这样：

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJuaW5naGFvLm5ldCIsImV4cCI6IjE0Mzg5NTU0NDUiLCJuYW1lIjoid2FuZ2hhbyIsImFkbWluIjp0cnVlfQ.SwyHTEx_RQppr97g4J5lKXtabJecpejuef8AqKYMAJc
```

#### Header

header 部分主要是两部分内容，一个是 Token 的类型，另一个是使用的算法，比如下面类型就是 JWT，使用的算法是 HS256。

```text
{
  "typ": "JWT",
  "alg": "HS256"
}
```

上面的内容要用 Base64 的形式编码一下，所以就变成这样：

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

#### Payload

Payload 里面是 Token 的具体内容，这些内容里面有一些是标准字段，你也可以添加其它需要的内容。下面是标准字段：

* iss：Issuer，发行者
* sub：Subject，主题
* aud：Audience，观众
* exp：Expiration time，过期时间
* nbf：Not before
* iat：Issued at，发行时间
* jti：JWT ID

比如下面这个 Payload ，用到了 iss 发行人，还有 exp 过期时间。另外还有两个自定义的字段，一个是 name ，还有一个是 admin 。

```text
{
 "iss": "ninghao.net",
 "exp": "1438955445",
 "name": "wanghao",
 "admin": true
}
```

使用 Base64 编码以后就变成了这个样子：

```text
eyJpc3MiOiJuaW5naGFvLm5ldCIsImV4cCI6IjE0Mzg5NTU0NDUiLCJuYW1lIjoid2FuZ2hhbyIsImFkbWluIjp0cnVlfQ
```

#### Signature

JWT 的最后一部分是 Signature ，这部分内容有三个部分，先是用 Base64 编码的 header.payload ，再用加密算法加密一下，加密的时候要放进去一个 Secret ，这个相当于是一个密码，这个密码秘密地存储在服务端。

* header
* payload
* secret

```text
var encodedString = base64UrlEncode(header) + "." + base64UrlEncode(payload); 
HMACSHA256(encodedString, 'secret');
```

处理完成以后看起来像这样：

```text
SwyHTEx_RQppr97g4J5lKXtabJecpejuef8AqKYMAJc
```

最后这个在服务端生成并且要发送给客户端的 Token 看起来像这样：

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJuaW5naGFvLm5ldCIsImV4cCI6IjE0Mzg5NTU0NDUiLCJuYW1lIjoid2FuZ2hhbyIsImFkbWluIjp0cnVlfQ.SwyHTEx_RQppr97g4J5lKXtabJecpejuef8AqKYMAJc
```

客户端收到这个 Token 以后把它存储下来，下回向服务端发送请求的时候就带着这个 Token 。服务端收到这个 Token ，然后进行验证，通过以后就会返回给客户端想要的资源。

### 相关链接

* [http://jwt.io/](http://jwt.io/)
* [https://github.com/firebase/php-jwt](https://github.com/firebase/php-jwt)
* [https://scotch.io/tutorials/the-anatomy-of-a-json-web-token](https://scotch.io/tutorials/the-anatomy-of-a-json-web-token)
* [https://github.com/auth0/jwt-decode](https://github.com/auth0/jwt-decode)
* [登录](http://ninghao.net/user/login?destination=node/2834%23comment-form) - 评论

### 评论

[登录](ji-yu-token-de-shen-fen-yan-zheng-ning-hao-wang.md) - 评论 [hangenggeng](http://ninghao.net/user/19836) member 浩哥， 你这边有例子包吗？ 可以发到 1512131351@qq.com 邮箱中吗？ 我是PHP程序员

1 年 4 天 以前

### 统计

6179 分钟

0 你学会了

0% 完成

### 社会化网络

* [ 微博](http://weibo.com/weareninghao)
* [ 优酷](http://i.youku.com/weareninghao)
* [ Github](https://github.com/ninghao)
* [ 53166188](http://user.qzone.qq.com/53166188)
* [ ninghao8080](http://ninghao.net/blog/2070)

### 关于

* [关于](http://ninghao.net/page/211)
* [故事](http://ninghao.net/blog/1121)
* [联系](http://ninghao.net/page/213)
* [购物车](http://ninghao.net/cart)
* [视频](http://ninghao.net/video)

### 微信订阅号

![](https://github.com/yangbao93/docs/tree/d23f2b2cbc4eb06e62d38114d6a7f5410080c7b5/技术知识/Java/基于%20Token%20的身份验证%20-%20宁皓网/image_2.jpeg)

QQ：53166188

© [宁皓网](http://ninghao.net/) \| [隐私条款](http://ninghao.net/privacy) \| [服务条款](http://ninghao.net/terms) \| [浙ICP备12003619号-1](http://www.miibeian.gov.cn/)

[http://ninghao.net/blog/2834](http://ninghao.net/blog/2834)

