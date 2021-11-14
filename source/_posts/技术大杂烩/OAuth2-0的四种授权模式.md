---
title: OAuth2.0的四种授权模式
categories:
  - 技术大杂烩
tags:
  - OAuth2.0

date: 2020-04-08 10:40:53
---





# **OAuth2.0 为何物**

`OAuth` 简单理解就是一种授权机制，它是在客户端和资源所有者之间的授权层，用来分离两种不同的角色。在资源所有者同意并向客户端颁发令牌后，客户端携带令牌可以访问资源所有者的资源。
`OAuth2.0` 是`OAuth` 协议的一个版本，有`2.0`版本那就有`1.0`版本，有意思的是`OAuth2.0` 却不向下兼容`OAuth1.0` ，相当于废弃了`1.0`版本。



**举个小栗子解释一下什么是 OAuth 授权？**
在家肝文章饿了定了一个外卖，外卖小哥30秒火速到达了我家楼下，奈何有门禁进不来，可以输入密码进入，但出于安全的考虑我并不想告诉他密码。
此时外卖小哥看到门禁有一个高级按钮“`一键获取授权`”，只要我这边同意，他会获取到一个有效期 2小时的令牌（`token`）正常出入。



令牌（`token`）和 `密码` 的作用虽然相似都可以进入系统，但还有点不同。`token` 拥有权限范围，有时效性的，到期自动失效，而且无效修改

`OAuth2.0` 的授权简单理解其实就是获取令牌（`token`）的过程，`OAuth` 协议定义了四种获得令牌的授权方式（`authorization grant` ）如下：

- 隐式授权模式（Implicit Grant）
- 授权码授权模式（Authorization code Grant）
- 密码模式（Resource Owner Password Credentials Grant）
- 客户端凭证模式（Client Credentials Grant）



但值得注意的是，不管我们使用哪一种授权方式，在三方应用申请令牌之前，都必须在系统中去申请身份唯一标识：客户端 ID（`client ID`）和 客户端密钥（`client secret`）。这样做可以保证 `token` 不被恶意使用。
下面我们会分析每种授权方式的原理，在进入正题前，先了解 `OAuth2.0` 授权过程中几个重要的参数：



- `response_type`：code 表示要求返回授权码，token 表示直接返回令牌
- `client_id`：客户端身份标识
- `client_secret`：客户端密钥
- `redirect_uri`：重定向地址
- `scope`：表示授权的范围，`read`只读权限，`all`读写权限
- `grant_type`：表示授权的方式，`AUTHORIZATION_CODE`（授权码）、`password`（密码）、`client_credentials`（凭证式）、`refresh_token` 更新令牌
- `state`：应用程序传递的一个随机数，用来防止`CSRF`攻击。

---

# 1.隐式授权模式（Implicit Grant）

![](1.png)

- 第一步：用户访问页面时，重定向到认证服务器。
- 第二步：认证服务器给用户一个认证页面，等待用户授权。
- 第三步：用户授权，认证服务器想应用页面返回Token
- 第四步：验证Token，访问真正的资源页面

![](2.png)


# 2.授权码授权模式（Authorization code Grant）

![](3.png)

- 第一步：用户访问页面
- 第二步：访问的页面将请求重定向到认证服务器
- 第三步：认证服务器向用户展示授权页面，等待用户授权
- 第四步：用户授权，认证服务器生成一个code和带上client_id发送给应用服务器,然后，应用服务器拿到code，并用client_id去后台查询对应的client_secret
- 第五步：将code、client_id、client_secret传给认证服务器换取access_token和refresh_token
- 第六步：将access_token和refresh_token传给应用服务器
- 第七步：验证token，访问真正的资源页面

![](4.png)


# 3.密码模式（Resource Owner Password Credentials Grant）


![](5.png)

- 第一步：用户访问用页面时，输入第三方认证所需要的信息(QQ/微信账号密码)
- 第二步：应用页面那种这个信息去认证服务器授权
- 第三步：认证服务器授权通过，拿到token，访问真正的资源页面

`优点`：不需要多次请求转发，额外开销，同时可以获取更多的用户信息。(都拿到账号密码了)

`缺点`：局限性，认证服务器和应用方必须有超高的信赖。(比如亲兄弟？)

`应用场景`：自家公司搭建的认证服务器


# 4.客户端凭证模式（Client Credentials Grant）

![](6.png)


- 第一步：用户访问应用客户端
- 第二步：通过客户端定义的验证方法，拿到token，无需授权
- 第三步：访问资源服务器A
- 第四步：拿到一次token就可以畅通无阻的访问其他的资源页面。

**这是一种最简单的模式，只要client请求，我们就将AccessToken发送给它。这种模式是最方便但最不安全的模式。因此这就要求我们对client完全的信任，而client本身也是安全的。**

**因此这种模式一般用来提供给我们完全信任的服务器端服务。在这个过程中不需要用户的参与。**