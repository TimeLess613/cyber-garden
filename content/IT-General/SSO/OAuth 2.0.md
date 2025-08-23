---
tags:
  - IT/认证
---
## 概念

> OAuth 2.0 是应用程序用来为**客户端应用程序**提供访问权限的标准。
- 即主要用于授权。

OAuth 2.0 **使用访问令牌（Access Token）来代表用户的授权**。访问令牌通常是简短的字符串，由认证服务器签发，并且用于访问资源服务器上的受保护资源。


### 4个角色

- Client：一般是**某个应用**，来代替用户访问他的资源。
- Resource Owner：资源的所有者，**一般是用户**。决定是否**授权某个应用**可以访问自己的资源。
- Resource Server：（用户的）资源存放的地方，**一般是云服务厂商**。
- Authorization Server：如 Azure、Google等。

**作用方式/应用场景：**  
- 用户想授权/委派某个应用，让它代替用户去获取某个资源，但是不应该直接把访问某个资源的账号密码给这个应用——所以需要传递某个授权码。
- 而为了避免用户在多个界面（多个应用&授权服务器）之间来回跳转，OAuth 将跳转流程自动化了（所以配置的时候需要重定向 URL——对于授权码工作流）。


## 3种认证方式

> ServiceNow的参考连接： [OAuth Grant Types: Explained](https://support.servicenow.com/kb?id=kb_article_view&sysparm_article=KB1647747)


### 【废弃】Resource Owner Password Credentials (ROPC) Flow

```
[Client App] <── [User] 用户输入账号密码（直接给客户端）
    |
    └─> [Authorization Server] ──> Access Token
```

- 客户端直接处理账号密码，如果客户端被攻破凭据有泄漏风险。
- 不支持 MFA。

> 可联系 [[委派#非约束委派]]？Client App 即前端，Authorization Server 即 KDC。


### Authorization Code Flow

关键：凭据→授权码→token。
- 似乎也要提前注册 Client App。
- 为了防止授权码被盗，获取授权码后，Client 还得带上 `client_id` 和 `client_secret` 以获取令牌（Access Token），之后用令牌才能访问资源。
	- 本质：为了简单实现重定向，用了 GET 请求传输授权码。之后才用 POST 请求获取 token。

```
# 用户在 Client 中打算访问另一应用的资源。Client 请求用户授权，转到另一应用的登陆界面，用户登陆并提供授权范围：

 +---------+                               +---------------+
 |         |--(A)- Authorization Request ->|   Resource    |
 |         |                               |    Owner      |
 |         |<-(B)-- Authorization Grant ---|    (User)     |
 |         |     (redirect_uri?code=...)   +---------------+
 |         |                                     |
 | Client  |                                     | (B) login & get code
 |         |                                     |
 |         |                               +---------------+
 |         |--(C)-- Authorization Code --->| Authorization |
 |         |    (client_id&client_secret)  |   Server      |
 |         |                               |               |
 |         |<-(D)----- Access Token -------|               |
 |         |                               +---------------+
 |         |
 |         |                               +---------------+
 |         |--(E)----- Access Token ------>|    Resource   |
 |         |                               |     Server    |
 |         |<-(F)---- Protected Resource --|               |
 +---------+                               +---------------+
 
 A: 客户端向资源所有者（用户）请求授权，并被重定向到授权服务器。
 B: 用户登陆并同意授权，客户端获得授权码。（之后都是在后端发生）
 C: 客户端使用授权码向授权服务器请求访问令牌。
 D: 授权服务器验证授权码并返回访问令牌。
 E: 客户端使用访问令牌向资源服务器请求受保护的资源。
 F: 资源服务器验证访问令牌并返回受保护的资源数据。
```
- **有用户交互**——用户是在授权服务器的页面输入凭据，客户端应用看不到。
- 支持 MFA、SSO。

```
[User] ──> [Client App (browser)]
			└─> Redirect to Authorization Server (login + consent)
	            └─> Redirect back with code
		           └─> [Client Backend] sends code to Authorization Server
		                    └─> Get Access Token
		                        └─> Call Resource Server with token
```
- 更贴近实际开发中的跳转与回调逻辑
- Resource Owner 与 Authorization Server 合并为同一系统入口


安全措施：关于授权码的防护——state 和 PKCE，可看 b站 BV195Yfz9Ebw。

#### 理解流程的 CTF

> [!note] 根据 [[kazkiti memo#OAUTH-004]] 的经验，请求时有几个编码问题需要注意。





### Client Credentials Flow

- **纯后端调用、无用户参与**（如对比 Authorization Code Flow）
- 在 Client Credentials Flow 中，`client_id` 和 `client_secret` 是从授权服务器注册应用时获取的

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

> 服务/应用自己访问资源。在 IdP（如 Azure AD、Entra ID）中注册应用可类比 AD 环境中注册一个 [[SPN]]？

#### 🔑 对比 Authorization Code Flow

|对比点|Authorization Code Flow|Client Credentials Flow|
|---|---|---|
|**是否有用户参与**|✅ 有（用户要登录、同意授权）|❌ 无（纯后台服务调用）|
|**Access Token 代表谁**|代表 **用户**（Delegated 权限）|代表 **应用**（Application 权限）|
|**获取 Token 的凭据**|授权码（code）+ client_id/secret|client_id + client_secret|
|**适用场景**|Web 应用、桌面应用、移动应用，需要用户数据|后台任务、服务对服务（machine-to-machine）|
|**权限范围**|取决于用户 + 应用申请的 scope|取决于管理员授予的应用权限|
|**支持刷新 token**|✅ 支持（Refresh Token）|❌ 一般不支持，直接再申请新 token|
|**安全风险**|客户端不会拿到用户密码，安全|如果 secret 泄露，应用身份被冒充|


---



