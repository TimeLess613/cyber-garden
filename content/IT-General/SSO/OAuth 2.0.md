---
tags:
  - IT/认证
---

> OAuth 2.0 是应用程序用来为客户端**应用程序**提供访问权限的标准。
- 即主要用于授权。

OAuth 2.0 使用访问令牌（Access Token）来**代表用户**的授权。访问令牌通常是简短的字符串，由认证服务器签发，并且用于访问资源服务器上的受保护资源。

- Resource Owner：资源的所有者，一般是用户。决定是否让某个应用可以访问自己的资源。
- Resource Server：资源存放的地方。
- Client：一般是某个应用，来代替用户访问他的资源。



## 3种认证方式

> ServiceNow的参考连接： [OAuth Grant Types: Explained](https://support.servicenow.com/kb?id=kb_article_view&sysparm_article=KB1647747)

### Authorization Code Flow

```
 +--------+                               +---------------+
 |        |--(A)- Authorization Request ->|   Resource    |
 |        |                               |     Owner     |
 |        |<-(B)-- Authorization Grant ---|               |
 |        |     (redirect_uri?code=...)   +---------------+
 |        |
 |        |                               +---------------+
 |        |--(C)-- Authorization Code --->| Authorization |
 | Client |                               |     Server    |
 |        |<-(D)----- Access Token -------|               |
 |        |                               +---------------+
 |        |
 |        |                               +---------------+
 |        |--(E)----- Access Token ------>|    Resource   |
 |        |                               |     Server    |
 |        |<-(F)---- Protected Resource --|               |
 +--------+                               +---------------+
 
 A: 客户端向资源所有者（用户）请求授权，并被重定向到授权服务器。
 B: 资源所有者同意授权，并将授权授予客户端（通常通过授权码）。
 C: 客户端使用授权码向授权服务器请求访问令牌。
 D: 授权服务器验证授权码并返回访问令牌。
 E: 客户端使用访问令牌向资源服务器请求受保护的资源。
 F: 资源服务器验证访问令牌并返回受保护的资源数据。
```


### Client Credentials Flow

```
+--------+                                  +---------------+
|        |--(A)--- Token Request ---------> |               |
|        |   (client_id + client_secret     | Authorization |
| Client |     + grant_type=client_creds)   |    Server     |
|        |                                  |               |
|        |<--(B)-- Access Token ------------|               |
+--------+                                  +---------------+
    |
    | 使用 Access Token 访问受保护资源
    |
    v
+--------+                                  +---------------+
|        |--(C)--- Resource Request ------->|               |
| Client |   Authorization: Bearer token    |   Resource    |
|        |<--(D)-- Protected Resource ------|    Server     |
+--------+                                  +---------------+
```


---




> [!note] 根据 [[kazkiti memo#OAUTH-004]] 的经验，请求时有几个编码问题需要注意。

