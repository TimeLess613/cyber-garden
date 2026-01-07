---
tags:
  - IT/网络
  - IT/加密
---
> SSL 那家公司后来不做了 → TLS


> https://milestone-of-se.nesuke.com/sv-advanced/digicert/digital-certification-summary/


## SSL握手

SSL在TCP和应用层之间，保证了传输层的安全。


> [!note] 总结：Hello/协商、证书交换、密钥交换


## 举例：HTTPS 通信步骤

### 一、协商加密组件&提供公钥证书

1. 客户端发送 **ClientHello**，提供可选[[加密组件]]和随机数（Client Random）。
2. 服务器返回 **ServerHello**，回应加密组件和随机数（Server Random），并提供[[公钥证书]]。

### 二、验证公钥证书&共享 pre-master secret

3. **客户端验证公钥证书**。验证通过后客户端生成 pre-master secret，并用服务器的公钥加密然后发送给服务器。
4. **双方计算 session keys**（pre-master secret → master secret → session keys），并发送 Change Cipher Spec（更改密码规格）报文（客户端先发服务器后发）。
5. **TLS 握手完成**。之后用 session key 的对称加密进行通信。


### 三、（校验）SSL建立完毕之后，可发送HTTP请求/响应

### 四、断开连接时进行TCP四次挥手

