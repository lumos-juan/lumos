---
title: "Oauth2.0授权协议"
meta_title: ""
description: "OAuth2.0是一种授权协议，**允许第三方应用在用户授权的情况下，在不暴露用户密码给第三方应用的前提下安全地访问服务器资源**,同时可以提供有限的权限范围，限制第三方应用能够访问的资源，提高了安全性。认证成功之后会返回Acces Token，第三方应用可以用Access Token换取所需要的资源。"
date: 2024-11-21T05:00:00Z
image: "/images/auth/oauth2.0-header.png"
categories: ["授权", "协议"]
author: "Gavain Juan"
tags: ["Oauth2.0", "授权"]
draft: false
---
# 定义

OAuth2.0是一种授权协议，**允许第三方应用在用户授权的情况下，在不暴露用户密码给第三方应用的前提下安全地访问服务器资源**,同时可以提供有限的权限范围，限制第三方应用能够访问的资源，提高了安全性。认证成功之后会返回Acces Token，第三方应用可以用Access Token换取所需要的资源。

# 重点概念

**资源所有者**：**Resource Owner(RO)**： 拥有受保护资源的实体，通常指终端用户。

**资源服务器**：**Resource Server（RS）** 资源的存储位置。

**客户端**：**client** 请求访问受保护资源的第三方应用。

**授权服务器**：**authorization server(AS)** 验证资源身份者，颁发令牌。多数情况下授权服务器需要验证资源拥有者身份，授权服务器也要提供认证功能。

**授权令牌**: **Access Token（AT）** 授权服务器提供给客户端访问受保护资源的第三方应用。

**授权许可**：**Authorization Grant** 是资源拥有者授权给客户端的一个凭据，表明资源拥有者同意客户端代表他访问受保护的资源。授权许可的类型包括授权码、隐式授权、资源拥有者密码凭据和客户端凭据。

## 整体流程

![oauth2.0](/images/auth/2-Oauth2.0%20授权协议-整体流程.png)

1. 当第三方客户端 `client`需要访问资源拥有者 `RO`所拥有的资源的时候，向用户发出授权许可 `Authorization Grant`；
2. 资源拥有者RO批准授权，将授权许可返回给第三方客户端；
3. 第三方客户端拿着用户的授权许可去换取访问令牌 `Access Token`；
4. 授权服务器 `AS`确定第三方客户端client身份和资源拥有者RO身份后返回访问令牌给第三方客户端；
5. 第三方客户端使用访问令牌访问资源；
6. 资源服务器 `RS`验证访问令牌（发送给授权服务器验证等），返回许可访问的资源，

# 四种授权方式

OAuth2.0支持四种授权方式，以适应不同的场景。

## 新增概念

客户端凭据（Client Credentials）授权服务器验证客户端身份的凭证，通常是ClientId&ClientSecret

## 授权码授权

### 概念

授权码流程是最常见的一种。使用的授权许可 `Authorization Grant`是授权码 `Authorization Code`，授权码在前端传递，有效期很短且单次有效，第三方应用后台服务获取到后授权码后，携带着客户端凭据和授权码到授权中心换取访问令牌 `Access Token`，再拿着访问令牌换取资源。

### 流程

![授权码流程](/images/auth/1-Oauth2.0%20授权协议-授权码流程.png)
流程与OAuth2.0整体流程相似，关注点主要在：
第二步重定向到授权服务器地址的时候需要携带的参数：

- **response_type**：必填，其值通常是 `code`，表示客户端希望使用授权码流程。
- **client_id**：必填，表示客户端的ID，这是在授权服务器上注册客户端时获得的。
- **redirect_uri**：必填，指定授权服务器在完成用户认证和授权后应该重定向用户的URI。这个URI必须在客户端注册时提供给授权服务器。
- **scope**：选填，用于指定客户端请求的权限范围。例如，`read`或 `write`等。
- **state**：推荐，用于防止跨站请求伪造（CSRF）攻击。客户端生成一个随机值，并在授权请求中发送。授权服务器在重定向回客户端时应该原样返回这个值，客户端可以验证它以确认响应是来自预期的授权请求。

  **请求示例**

```http
GET /authorize?response_type=code&client_id=CLIENT_ID&redirect_uri=https%3A%2F%2Fclient.example.com%2Fcallback&scope=read%20write&state=xyz HTTP/1.1
Host: authorization-server.com

```

 **返回结果**
对应第四步授权服务器携带着授权码重定向到第三方应用

```http
HTTP/1.1 302 Found Location:
https://client.example.com/callback?code=SplxlOBeZQQYbYS6WxSbIA&state=xyz
```

 第五步第三方应用携带着授权码请求访问令牌的时候需要的其他参数：

- **grant_type**：必填，这个参数必须设置为 `authorization_code`，以指明客户端正在使用授权码授权流程。
- **code**：必填，之前在授权请求阶段从授权服务器获得的授权码。
- **redirect_uri**：必填，在用户授权时使用的重定向URI，必须与获取授权码时使用的重定向URI相同。
- **client_id**：必填，客户端应用的ID，这是在应用注册授权服务器时获得的。
- **client_secret**：推荐，通常情况客户端应用的密钥，这是在应用注册授权服务器时获得的。某些授权服务器可能要求在请求访问令牌时提供这个参数，以提高安全性。
  **请求示例**

```http
POST /token HTTP/1.1
Host: authorization-server.com
Authorization: Basic base64encode(client_id:client_secret) -- 不同实现可能传入位置不同
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
code=SplxlOBeZQQYbYS6WxSbIA -- 上个接口返回的
redirect_uri=REDIRECT_URI

```

**响应示例**：

```http
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8

{
  "access_token": "ACCESS_TOKEN",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "REFRESH_TOKEN",
  "scope": "read write"
}
```

### 优缺点&使用场景

- **安全性高**:不会暴露Access Token,适合于大多数第三方应用；
- **有服务端**：适用于有服务端的场景；
- **流程略复杂**；

## 隐式授权

隐式（implicit）授权 ,不会生成授权码，**会直接返回access token到前端**,使用较少。

### 流程

![隐式流](/images/auth/Oauth2.0%20授权协议-隐式授权流程.png)
与授权码授权相比，隐式授权不返回授权码，直接返回了Access Token ,少了拿授权码换取Access Token的步骤。
重点关注的还是第二步：
第三方应用将用户浏览器重定向到授权服务器，并在请求中包含以下参数：

- response_type: 必须设置为 `token`，表示客户端请求直接返回访问令牌。
- client_id: 客户端的标识符，注意：此模式无client secret。
- redirect_uri: 用户授权后授权服务器重定向到的URI。
- scope: 客户端请求的权限范围。
- state: 客户端生成的一个随机字符串，用于防止跨站请求伪造（CSRF）攻击。

  **请求示例**

```http
GET /authorize?response_type=token&client_id=CLIENT_ID&redirect_uri=https://client-app.com/callback&scope=read_profile&state=随机字符串 HTTP/1.1
Host: authorization-server.com
```

    **响应示例**

```http
HTTP/1.1 302 Found
Location: https://client-app.com/callback#access_token=ACCESS_TOKEN&token_type=bearer&expires_in=3600&scope=read_profile&state=随机字符串
```

### 优缺点&&适用场景

- **安全性较低**：由于访问令牌直接在浏览器中传输，因此更容易受到中间人攻击。
- **无法刷新令牌**：隐式授权流程不支持刷新令牌，一旦访问令牌过期，用户需要重新进行授权。
- **令牌泄露风险**：由于令牌直接在客户端传递，如果客户端存在安全漏洞，令牌可能会被泄露。
- **无后端参与**：适合纯前端，没有后端的网站。

## 密码授权

密码模式授权，直接将用户名密码交给第三方应用。这种方式不符合OAuth2.0的原则：第三方应用不应该拥有用户的凭证，增加了凭证泄露的风险。

### 流程

![密码模式](/images/auth/Oauth2.0%20授权协议-密码模式.png)
步骤上重点在于第二步，第三方应用需要拿着用户的用户名密码去请求token。

**请求示例**：

```http
POST /token HTTP/1.1
Host: authorization-server.com
Authorization: Basic base64encode(client_id:client_secret) 
Content-Type: application/x-www-form-urlencoded

grant_type=password&username=USER&password=PASSWORD&scope=read
```

**响应示例**：

```http
HTTP/1.1 200 OK 
Content-Type: application/json;charset=UTF-8 

{ 
	"access_token":"2YotnFZFEjr1zCsicMWpAA",
	"token_type":"bearer", 
	"expires_in":3600, 
	"scope":"read" 
}
```

### 优缺点&适用场景

- **安全性较低**：客户端需要存储用户的用户名和密码，这增加了安全风险，尤其是在客户端遭受攻击的情况下。
- **信任问题**：需要用户高度信任客户端应用，因为用户需要将自己的凭证信息提供给客户端。客户端应用高度信任，例如第一方应用。
- **不一定支持刷新令牌**：虽然密码授权流程可以返回刷新令牌，但不是所有实现都支持，这意味着当访问令牌过期时，用户可能需要重新提供用户名和密码。
- **不适用于第三方应用**：由于安全问题，密码授权不适用于第三方应用，因为第三方应用通常不被用户高度信任。

## 凭证授权

凭证（Client Credentials）授权主要用于服务器间通信的场景，适用于后台服务需要访问受保护的资源，不涉及到具体用户的资源访问，这部分资源应该是归属于后台服务的。

### 流程

![凭证授权](/images/auth/Oauth2.0%20授权协议-凭证授权.png)
重点关注第二步，客户端使用其 client_id 和 client_secret 向授权服务器发送请求，请求一个访问令牌。这个请求通常不需要用户参与。

**请求示例**

```http
POST /token HTTP/1.1
Host: authorization-server.com
Authorization: Basic base64encode(client_id:client_secret)
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials

```

**响应示例**

```http
HTTP/1.1 200 OK 
Content-Type: application/json;charset=UTF-8 
Cache-Control: no-store 
Pragma: no-cache 

{ 
	"access_token":"2YotnFZFEjr1zCsicMWpAA", 
	"token_type":"Bearer", 
	"expires_in":3600, 
	"scope":"read write" 
}
```

### 优缺点&适用场景

- **无用户参与**：这种模式不需要用户交互，因此适用于没有用户参与的服务间通信。
- **访问令牌不包含用户信息**：由于不涉及用户，因此访问令牌通常不包含用户信息。
- **有限的权限**：通常，使用 Client Credentials 授权模式获得的访问令牌只能访问有限的资源或执行有限的操作，因为这些操作与最终用户无关。
- 适用于服务器到服务器的通信的有限数据访问的授权，不需要用户授权数据的场景。

# refresh token

它用于在访问令牌（access token）过期后获取新的访问令牌，而无需用户再次进行身份验证。Refresh token 是一种长期有效的令牌，通常在用户同意应用程序进行授权后发放。

与访问令牌不同，refresh token 不用于访问资源。它只在后台使用，用于获取新的访问令牌。当访问令牌过期时，客户端应用程序可以使用 refresh token 向授权服务器请求新的访问令牌，而无需用户参与。

# 参考

[Oauth2.0](https://juejin.cn/post/7195762258962219069)

[OAuth2.0 vs OIDC](https://zhuanlan.zhihu.com/p/620872500)

[Oauth2.0四种模式的场景](https://zhuanlan.zhihu.com/p/375154660)

[Oauth2.0](https://juejin.cn/post/7276330110835458103?searchId=20241017095434C6D32D1D51DBDA55F5B5)

[开放平台鉴权以及OAuth2.0介绍 - duanxz - 博客园 (cnblogs.com)](https://www.cnblogs.com/duanxz/p/4369738.html)

[帮你深入理解OAuth2.0协议_SecCloud的专栏-CSDN博客_oauth2.0协议](https://blog.csdn.net/seccloud/article/details/8192707)

[OAuth2.0协议 - 简书 (jianshu.com)](https://www.jianshu.com/p/2f9d9014fbb6)

[Oauth2.0 协议到底是干什么的？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/49424214)

[OAuth 2.0 的一个简单解释 - 阮一峰的网络日志 (ruanyifeng.com)](https://www.ruanyifeng.com/blog/2019/04/oauth_design.html)

[OAuth 2.0 的四种方式 - 阮一峰的网络日志 (ruanyifeng.com)](https://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html)

[GitHub OAuth 第三方登录示例教程 - 阮一峰的网络日志 (ruanyifeng.com)](https://www.ruanyifeng.com/blog/2019/04/github-oauth.html)

[深入理解OAuth2.0 - duanxz - 博客园 (cnblogs.com)](https://www.cnblogs.com/duanxz/p/4022459.html)

[OAuth 的权限问题与信息隐忧 - duanxz - 博客园 (cnblogs.com)](https://www.cnblogs.com/duanxz/p/4022752.html)
