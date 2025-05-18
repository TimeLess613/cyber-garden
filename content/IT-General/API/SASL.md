---
tags:
  - IT/认证
---
> Simple Authentication and Security Layer

## 背景

Web（http）、电子邮件（SMTP、POP3、IMAP）等往往需要身份验证，但对于应用程序开发人员来说每次都从头开始创建身份验证模块是无意义的——一个中间层诞生了。

## 解决的问题

- 消除了应用程序开发人员开发身份验证的需要
- **每个协议**都必须支持 SASL（[IANA管理的 SASL 机制清单](https://www.iana.org/assignments/sasl-mechanisms/sasl-mechanisms.xhtml)）
- 但 SASL 并没有规定具体的实现方法

## SASL 的工作方式

在客户端和服务器之间建立通信通道后，双方协商选择一个合适的身份验证机制，然后进行身份验证过程——选择用什么认证机制。

例（使用 GSSAPI 时）：
```
┌──────────────────────────┐
│  应用协议帧 (SMTP / LDAP)  │ ① 顶层：传业务、也运输认证负载
├──────────────────────────┤
│  SASL 挑战 ↔ 应答         │ ② 第二层：定义流程；可选多种机制
│    ├─机制 PLAIN          │
│    ├─机制 SCRAM-SHA-1    │
│    └─机制 GSSAPI/GS2 ★  │  ← 这里会把 GSS-API 产生的 token 塞进来
├──────────────────────────┤
│  GSS-API ▶ token          │ ③ 第三层：系统级 API，生成/解析 token
│      ├─Kerberos 机制     │
│      └─NTLM 机制         │
└──────────────────────────┘
```