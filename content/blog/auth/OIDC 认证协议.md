---
title: "OIDC认证协议"
meta_title: ""
description: "OpenID Connect协议，允许第三方应用通过统一的认证中心进行身份认证，认证成功之后会返回ID token, 第三方应用可以据此建立会话。OIDC在OAuth2.0的基础上进行了身份认证的扩展，Oauth2.0拥有的授权功能，通常OIDC也可以支持。"
date: 2024-11-21T05:00:00Z
image: "/images/auth/OIDC.png"
categories: ["认证", "协议"]
author: "Gavain Juan"
tags: ["OIDC", "认证"]
draft: false
---
# 介绍

OIDC是基于OAuth2.0的身份认证协议.以下简单介绍下Oauth2.0:
[Oauth2.0 授权协议](/blog/auth/oauth2.0-授权协议/)

## OIDC

**OpenID Connect** 协议，**允许第三方应用通过统一的认证中心进行身份认证，认证成功之后会返回ID token, 第三方应用可以据此建立会话**。OIDC在OAuth2.0的基础上进行了身份认证的扩展，Oauth2.0拥有的授权功能，通常OIDC也可以支持。

## 关于认证和授权的区别

**认证**，Authentication，认证的目标是让第三方应用知道当前登录的用户是谁，在OIDC中第三方应用通过ID token能够直接解析出当前登录的用户信息。
**授权**，Authorization, 授权的目标是知道当前用户能干啥，可以不知道这个用户是谁，在OAuth2.0中用户在认证服务器中认证成功之后，会返回当前用户的Access Token,第三方应用可以拿着Access Token 来换取当前用户所需要的用户资源。

> 如果当前用户的Access Token 允许访问的资源中包含了用户信息，也可以根据Access Token换取用户信息，但是这种获取用户信息的方式是根据授予的权限访问资源，偏向于授权的概念。

# OIDC中新增的概念

**ID token**：OIDC认证成功后返回的[JWT](https://www.jianshu.com/p/576dbf44b2ae),里面包含了用户信息
**身份提供者（Identity Provider, IdP）**:负责用户身份认证并颁发ID token, access token。

> 相当于OAuth2.0的授权服务器，只是在OAuth 2.0的框架中，授权服务器的主要职责是处理授权请求和颁发访问令牌。它并不直接负责身份认证的过程,但它在实际应用中常常与身份认证过程相结合，与OIDC中的IdP相差不大

**第三方应用（Relying Party, RP）**：通过IdP验证身份的应用，对应OAuth2.0的client
**声明（Claims）**：ID token中包含的字段信息，例如用户的用户名、邮箱、登录时间、令牌的有效期等。这些信息由身份提供者（Identity Provider，IdP）在身份验证过程中生成，并传递给客户端（Relying Party，RP）。

> Claims和Scopes
> Scope是客户端在发起授权请求时定义的一系列权限，决定了客户端能够访问用户的哪些信息或资源。OIDC定义了一些标准的Scope，如openid、profile、email等。Claims需要的信息也需要包含在scopes中才能获取到。

# 流程

OIDC的流程与OAuth2.0基本一致，支持OAuth2.0中的授权码模式，隐式授权模式，密码模式和隐式授权（OIDC1.1弃用，因为会隐式授权id token access token直接返回到前端并不安全）
![OIDC授权码流程](/images/auth/OIDC%20认证协议-OIDC授权码流程.png)
主要区别在于第六步中的响应结果中，相对于OAuth2.0多了ID token

```json
{
  "access_token": "ACCESS_TOKEN",
  "token_type": "Bearer",
  "expires_in": 3600,
  "id_token": "ID_TOKEN",
  "refresh_token": "REFRESH_TOKEN"
}
```

# ID Token

```shell
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0.POstGetfAytaZS82wHcjoTyoqhMyxXiWdR7Nn7A29DNSl0EiXLdwJ6xC6AfgZWF1bOsS_TuYI3OG85AmiExREkrS6tDfTQ2B3WXlrr-wp5AokiRbz3_oB4OxG-W9KcEEbDRcZc0nH3L7LzYptiy1PtAylQGxHTWZXtGz4ht0bAecBgmpdgXMguEIcoqPJ1n3pIWk_dUZegpqx0Lka21H6XxUTxiy8OcaarA8zdnPUnV6AmNP3ecFawIFYdvJB_cm-GvpCSbr8G8y_Mllj8f4x9nBH8pQux89_6gUY618iYv7tuPWBFfEbLxtF2pZS6YC1aSfLQxeNe8djT9YjpvRZAQ
```

这个ID Token由三部分组成，分别是：

1. **Header**: 描述了JWT的签名算法和令牌类型。
2. **Payload**: 包含了关于用户身份和认证状态的实际信息。
3. **Signature**: 用于验证令牌完整性的签名。
   以下是上述ID Token的解码后的Header和Payload部分：
   **Header**

```json
{
  "alg": "RS256",
  "typ": "JWT"
}
```

**Payload**

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true,
  "iat": 1516239022
}
```

以下是Payload中各个字段的含义：

- `sub` (Subject)：主题，通常是用户的唯一标识符，例如用户ID。
- `name`：用户的姓名。
- `admin`：表示用户是否是管理员。
- `iat` (Issued At)：令牌的发行时间戳。

# 参考

[OIDC协议的概述和工作流程](https://www.ctyun.cn/developer/article/598965336391749)

[OAuth2.0 vs OIDC](https://zhuanlan.zhihu.com/p/620872500)

[Oauth2.0四种模式的场景](https://zhuanlan.zhihu.com/p/375154660)

[Oauth2.0](https://juejin.cn/post/7276330110835458103?searchId=20241017095434C6D32D1D51DBDA55F5B5)

[关于OIDC，一种现代身份验证协议](https://cloud.tencent.com/developer/article/2416477)
