---
tags:
  - IT/Linux
  - IT/网络
---

- 五表（filter / nat / mangle / raw / security）
- 五链（PREROUTING / INPUT / FORWARD / OUTPUT / POSTROUTING）
- 链定义时机，表定义功能。


## 5链

| 链名              | 触发时机                   | 处理对象                              | 关键点解释                              |
| --------------- | ---------------------- | --------------------------------- | ---------------------------------- |
| **PREROUTING**  | 数据包刚进入内核、路由判定之前        | 所有进入系统的包（不论目标是不是本机）               | 这是第一个钩子，最早能改目的地址（DNAT）、透明代理重定向。    |
| **INPUT**       | 路由判定后，确认目标是本机          | **发往本机的入站包**                      | 内核路由表判断目标 IP 属于本机接口，就进 INPUT。      |
| **FORWARD**     | 路由判定后，目标不是本机，而要转发给别的主机 | **经过本机转发的包**                      | 例如 Linux 做路由器、NAT、桥接、防火墙时，流量走这里。   |
| **OUTPUT**      | 本机上应用进程产生的出站包          | **由本机发送的包**                       | 本地主机发起的请求（如 curl、ping）在这里触发。       |
| **POSTROUTING** | 路由判定后、即将发往物理网卡前        | **所有出站包**（无论来自 OUTPUT 还是 FORWARD） | 在这一步可做源地址转换（SNAT、MASQUERADE）、修改头部。 |

### 五条链的宏观顺序

```
数据包进入内核
   ↓
PREROUTING   ← 路由前拦截
   ↓
路由判断 (查路由表)
   ↓
   ├─ 目标是本机 → INPUT → 本地应用
   │
   └─ 要转发的包 → FORWARD → POSTROUTING → 出接口
   ↓
本地主机发出的包：OUTPUT → POSTROUTING
```


## 5表

**表（table）决定「在每个链上干什么」**
每个链可能挂载若干表（table），每个表在该链上有各自的职责。  
在一个链内部，不同表按内核固定顺序执行：
```
raw → mangle → nat → filter → security
```

| 表            | 注册的链（hook）                    | 功能范围              |
| ------------ | ----------------------------- | ----------------- |
| **raw**      | PREROUTING、OUTPUT             | 控制是否启用 conntrack  |
| **mangle**   | 五条链都有                         | 改包头（TOS、TTL、mark） |
| **nat**      | PREROUTING、OUTPUT、POSTROUTING | 地址转换（DNAT / SNAT） |
| **filter**   | INPUT、FORWARD、OUTPUT          | 防火墙过滤（是否放行）       |
| **security** | INPUT、FORWARD、OUTPUT          | 安全模块接口（SELinux）   |


## 具体实例对比说明

假设有这样几条规则：
```
# NAT 转换
iptables -t nat -A PREROUTING -d 1.1.1.1 -j DNAT --to 10.0.0.10

# 修改标记
iptables -t mangle -A PREROUTING -j MARK --set-mark 1

# 防火墙过滤
iptables -t filter -A INPUT -p tcp --dport 22 -j DROP
```

数据包经过的顺序是：

1️⃣ **进入内核 → 触发 PREROUTING 链**

- 依次执行：
    - `raw (如果有)`
    - `mangle`（打标）
    - `nat`（执行 DNAT）  
        → 然后进行路由判定。

2️⃣ **路由结果：目标是本机 → 进入 INPUT 链**

- 依次执行：
    - `mangle`（可能改 TTL）
    - `filter`（DROP TCP 22）
    - `security`（SELinux）

3️⃣ **若没被 DROP → 交给应用程序。**

> 🔹 这里可以看到：先走哪条链是由包的路由方向决定的（链顺序），  
> 而在链里做哪些事情由表的挂载顺序决定（表顺序）。


---


> [!note] OCI 公开服务器端口时，除了配置 VCN 子网入站规则，还需要在服务器中配置 FW 放行

理解：
![[Pasted image 20240311165707.png]]
