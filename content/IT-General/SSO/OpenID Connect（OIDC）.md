---
tags:
  - IT/认证
---

是 [[OAuth 2.0]] 的扩展。在 OAuth 基础上补齐认证功能。

近年来，集成了 OpenID 和 OAuth2 的 OpenID Connect 标准与 [[SAML]] 一起成为实现 SSO 的主流。
优点是比 SAML 更容易实现和操作，并且正在企业服务中获得认可，ADFS 可以同时使用 SAML 和 OIDC。

> OpenID Connect 是建立在 OAuth 2.0 之上的身份验证标准。
> 它添加了一个称为 ID令牌（ID token）的附加令牌。为此，它使用简单的 JSON Web 令牌 (JWT)。
> OAuth 2.0 是关于资源访问和共享的，而 OIDC 则是关于用户身份验证的。


> [!note] 与 SAML 的核心区别——传递给 IdP 的 身份认证信息的格式不同（在用户输入账号密码后生成）。OIDC 用 JWT，SAML 用 SAML Assertion（XML格式）。
> 登陆流程都是先访问服务，如果为登陆，则跳转到指定 IdP 的登陆界面要求用户输入账号密码，之后生成认证信息发给服务。（可参考 [[SAML#SP initiated Flow]]）



