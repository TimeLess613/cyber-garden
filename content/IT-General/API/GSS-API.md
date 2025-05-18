---
tags:
  - IT/认证
---
## 前提

![[SASL#SASL 的工作方式]]


- 属于 [[SASL]] 框架中的一种


## 概念

> Generic Security Service Application Program Interface

一种统一模式，旨在
- 为使用者提供一种无关平台环境、机制、程序语言的可移植的安全服务。
- 为应用提供**统一的身份验证、完整性与机密性**功能。
- 开发者只需调用标准 API **生成/校验令牌**，即可在不同协议和操作系统之间重用同一套安全逻辑，从而轻松实现 SSO 等跨协议场景。

- GSS-API 也只是一个框架，所以需要具体实现。而使用 GSS-API 最有名的实现是：微软 Windows AD 域的 [[SPNEGO]] (SSO)。
- [[NTLM]] 和 [[Kerberos]] 都支持 GSS-API，区别在于 token。一个是 NTLM token，一个是 Kerberos ticket。
	- SSPI 是 GSS-API 的一个变体，进行了扩展并具有许多特定于 Windows 的数据类型。
	![[Pasted image 20240816121836.png]]

