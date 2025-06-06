---
tags:
  - IT/网络
  - IT/蓝队
---
> https://github.com/nicanorflavier/spf-dkim-dmarc-simplified
> - **Phishing Attacks** → SPF
> - **Brand Impersonation** → DKIM
> - **Business Email Compromise (BEC)** → DMARC

例子：
> 如果某银行没有实施 SPF/DKIM/DMARC，攻击者冒充该银行发送钓鱼邮件，而邮件可能会**通过收件服务器的检查并进入用户的收件箱**。用户则认为该电子邮件来自他们的银行。


SPF/DKIM/DMARC 都算是用于**验证发信人的正当性**。

---

### 🔍 共同点

#### ✔️ 都是收件方主动去“查询发件人域名的 DNS”，做出判断。

### 🧱 但它们的侧重点不同

| 验证机制      | 用途                          | 查什么 DNS 记录                          | 验证目标               | 是否对邮件内容签名 |
| --------- | --------------------------- | ----------------------------------- | ------------------ | --------- |
| **SPF**   | 验证发信 IP 是否被域名授权             | 发件人域名的 TXT 记录                       | 发件服务器 IP           | ❌ 不验证内容   |
| **DKIM**  | 验证邮件有没有被篡改                  | `selector._domainkey.发件域名` 的 TXT 记录 | 邮件签名是否有效           | ✅ 有签名     |
| **DMARC** | 统一 SPF/DKIM 的验证标准，并指定失败处理方式 | `_dmarc.发件域名` 的 TXT 记录              | SPF 和 DKIM 的结果是否对齐 | ❌ 自身不签名   |

---

## SPF

> Sender Policy Framework

算是一个授权/许可清单。
指定谁（哪些IP）可以代表我（域名）发送邮件。
别人（收件方）可以来查看发信人域名的清单确认发件人IP是否被授权。

### 🧾 简化流程如下

1. **收到一封邮件**，比如来自 `user@example.com`。
2. **收件人服务器提取发件人域名**（example.com）。
3. **去 DNS 查询 `example.com` 的 SPF 记录**（TXT 类型）。
4. **判断实际发信 IP (`Received 头` 中的 IP) 是否在 SPF 授权范围内**。
5. **如果不在范围内，则 SPF 验证失败**。

> 🧠 补充：为什么用 IP 判断而不是别的？
> 因为 IP 是在 SMTP 层就能直接获得，**不会被邮件头伪造**。相较于邮件正文里写的 From，IP 更可靠。

### 定义格式

在DNS的[[DNS#TXT|TXT记录]]中定义。格式通常如下：
```
v=spf1 ip4:123.123.123.123 ~all
```

> Here's the command I usually run to fetch that:
```
dig TXT example.com
```

### SPF定义与认证结果

- pass: `+all`
- softfail: `~all`。（常用这个定义）
- fail: `-all`
- ……

> [2. What's the difference between ~all, -all, ?all, and +all in an SPF record?](https://github.com/nicanorflavier/spf-dkim-dmarc-simplified?tab=readme-ov-file#faqs-with-spf-dkim-and-dmarc)

## DKIM

> Domain Keys Identified Mail

还是应用公钥基盘——在TXT记录中发布公钥，发信时在里面添加一段摘要（私钥签名）。收件方可以查找送信方DNS的公钥确认邮件是否被篡改。


### 定义格式

[[DNS#TXT|TXT记录]]中的格式通常如下：
```
v=DKIM1; k=rsa; p=NICfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDBolTXCqbxwoRBffyg2efs+Dtlc+CjxKz9grZGBaISRvN7EOZNoGDTyjbDIG8CnEK479niIL4rPAVriT54MhUZfC5UU4OFXTvOW8FWzk6++a0JzYu+FAwYnOQE9R8npKNOl2iDK/kheneVcD4IKCK7IhuWf8w4lnR6QEW3hpTsawIDAQ0B"
```
> Here's the command I usually run to fetch that:
```
dig TXT selector1._domainkey.example.com
```


> [https://milestone-of-se.nesuke.com/l7protocol/smtp/dkim-spf-senderid/](https://milestone-of-se.nesuke.com/l7protocol/smtp/dkim-spf-senderid/)



## DMARC标准

> Domain-based Message Authentication Reporting and Conformance

**DMARC** 是一种用于**电子邮件身份验证**的协议，目的是防止 **伪造发件人域名的邮件**（如钓鱼邮件或垃圾邮件）成功送达用户邮箱。

它基于两种技术：
1. **SPF（Sender Policy Framework）**
2. **DKIM（DomainKeys Identified Mail）**

DMARC 在此基础上再加一层机制：  
📌 **告诉收件服务器“如何处理未通过 SPF 或 DKIM 验证的邮件”**，并且  
📌 **向域名所有者报告验证结果**。


### 🛠️ DMARC 的工作原理（简要流程）

1. 发件人设置好 SPF 和/或 DKIM 策略。
2. 收件人服务器收到邮件后，验证 SPF 和 DKIM 是否通过。
3. 收件人查找发件人域名的 **DMARC 记录**（在 DNS 中）。
4. 按照 DMARC 规定：
    - 如果 SPF/DKIM 有通过，且“对齐”（即域名一致），则通过；
    - 如果验证失败，则根据 DMARC 策略进行处理：
        - **none**（什么都不做，仅报告）
        - **quarantine**（隔离，比如放入垃圾箱）
        - **reject**（直接拒收）
5. 收件人服务器可将验证结果报告给域名所有者，便于分析。

---

#### 📌 SPF 验证步骤

1. **解析发件人邮箱的域名**（如 `user@example.com` → `example.com`）。
2. **查询该域名 DNS 的 SPF 记录**，例如：
```
example.com → v=spf1 ip4:192.0.2.0/24 include:_spf.google.com ~all
```
3. **检查发信 IP** 是否在 SPF 记录中授权范围内。
4. **判断发件人域名与 Envelope From（MAIL FROM）地址是否对齐**（DMARC 所要求的“对齐”）。

#### 📌 DKIM 验证步骤

1. **从邮件头部读取 DKIM 签名**（`DKIM-Signature:` 字段）。
2. **根据签名中的 selector 和 d=参数定位 DNS 公钥**，例如：
```
s=selector1; d=example.com;
→ 查询 DNS 记录：selector1._domainkey.example.com
```
3. **用公钥验证邮件的哈希是否匹配**（即签名是否有效）。
4. **判断 DKIM 中的 d=example.com 是否与 From 地址域名对齐**（DMARC 所要求的“对齐”）。

#### 🔄 DMARC 中的“对齐”（Alignment）

DMARC 要求 SPF 和 DKIM **至少一个通过且“对齐”** 才算验证成功。

|验证机制|对齐对象|对齐要求说明|
|---|---|---|
|SPF|MAIL FROM vs From|两者域名一致或子域名|
|DKIM|d= vs From|DKIM 的 d= 域 与 From 域名一致或子域名|

---

### 定义格式

[[DNS#TXT|TXT记录]]中的格式通常如下：
```
v=DMARC1; p=none; rua=mailto:postmaster@example.com
```
> Here's the command I usually run to fetch that:
```
dig TXT _dmarc.example.com
```



---


## 🧪 如何实际验证 SPF 和 DKIM？

### ✅ 方法一：使用在线工具

- **MxToolbox SPF/DKIM Lookup**  
    https://mxtoolbox.com
- **Google Admin Toolbox - Check MX**  
    https://toolbox.googleapps.com/apps/checkmx/
- **Mail-tester.com**  
    发送测试邮件后，它会分析 SPF、DKIM、DMARC、Spam 分数等。

### ✅ 方法二：查看收到邮件的原始邮件头（在 Gmail 中为例）

1. 点击邮件右上角菜单 → "查看原始邮件（Show original）"
2. 页面会显示：
    - **SPF：PASS/FAIL**
    - **DKIM：PASS/FAIL**
    - **DMARC：PASS/FAIL**
