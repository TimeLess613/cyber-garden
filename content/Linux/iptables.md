---
tags:
  - IT/Linux
  - 渗透/后渗透
---

### 背景

2023/07/08

玩 TryHackMe 时需要用 Windows，然而在Windows上使用 openvpn 不知道为何无法连接目标网络。于是尝试依旧用 kali 连接 THM，然后将 Windows 访问 THM 的流量转发到 kali，由 kali 转发至 THM。相当于让 kali 充当一个路由器/防火墙/跳板。

拓扑：
```
  Windows           kali(eth0)          kali(enumad)         THM目标网络
(10.10.10.20)      (10.10.10.10)       (10.50.47.202)       (10.200.49.0/24)
    | ------------------|--------------------|---------------------|
```

### 启用IP转发功能

命令行写入：`sysctl [-w] net.ipv4.ip_forward=1`

- 使用 `-w` 即和修改 `/etc/sysctl.conf` 文件里的 `net.ipv4.ip_forward=1` 一样
- 若是修改配置文件，执行此命令使更改生效：`sysctl -p`

#### 确认配置

确认是否为 `1`：`sysctl net.ipv4.ip_forward`

### 配置转发规则

#### 🧠 SNAT vs Masquerade 的区别

| 项目      | SNAT          | Masquerade                 |
| ------- | ------------- | -------------------------- |
| 适用场景    | 出口 IP 是固定的    | 出口 IP 是动态获取的（如 DHCP、PPPoE） |
| IP 地址配置 | 需要手动写出转换用的 IP | 不需要手动指定 IP，会自动获取当前接口 IP    |
| 性能      | 更高（转换表可缓存）    | 略低（每个连接需重新查接口 IP）          |
- PAT（Port Address Translation）：即 NAPT，动态 SNAT 的一种特殊形式，适用于接口 IP 动态变化的情况（如 DHCP 拨号）。在 Linux 中由 `MASQUERADE` 实现。


#### 方法1： MASQUERADE + DNAT

`tracert` 的第一跳直接目标，且“显示”无延迟。

```bash
sudo iptables -t nat -A PREROUTING -i eth0 -s 10.10.10.20 -j DNAT --to-destination 10.50.47.202
sudo iptables -t nat -A POSTROUTING -o enumad -s 10.10.10.20 -j MASQUERADE
```

📌 **说明**：
- 把源为 `10.10.10.20` 的数据包，目的地址转换为 `10.50.47.202`
- 同时做 MASQUERADE，让外网返回包可以正确回到 NAT 设备
- **由于开启了 IP 转发所以会将数据包发送到原本的目标地址。其他方法相同。**

📌 **特点**：
- 双向做了地址转换（源 + 目的）
- 适用场景：**流量中转代理 / 显示跳板**
- `MASQUERADE` 适用于出口 IP 不固定场景


#### 方法2： SNAT + DNAT

```bash
sudo iptables -t nat -A PREROUTING -s 10.10.10.20 -d 10.10.10.10 -j DNAT --to-destination 10.50.47.202
sudo iptables -t nat -A POSTROUTING -s 10.10.10.20 -j SNAT --to-source 10.50.47.202
```

📌 **说明**：
- 手动指定转换 IP：`10.50.47.202`
- 与方法 1 类似，但使用的是 **SNAT 而不是 Masquerade**

📌 **特点**：
- 更适合出口 IP 是**固定的**情况
- 性能高于 Masquerade（因为可以缓存）

#### 方法3： 纯转发 + MASQUERADE

```bash
sudo iptables -A FORWARD -i eth0 -o enumad -s 10.10.10.20 -j ACCEPT
sudo iptables -A FORWARD -i enumad -o eth0 -d 10.10.10.20 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -t nat -A POSTROUTING -o enumad -s 10.10.10.20 -j MASQUERADE
```

> - 这个方法是最详细的，包括了防火墙规则和 NAT 规则。
> - 前两条规则用于允许从 `10.10.10.20` 到 `10.50.47.202` 的流量通过防火墙。第一条规则允许出站流量，第二条规则允许相关或已建立的流量返回。
> - 第三条规则使用MASQUERADE对流量进行NAT（同方法1的第二条规则），以便从 `10.10.10.20` 发送的流量经过enumad接口时具有伪装的源IP地址。

📌 **说明**：
- 不改写目标地址，允许从内网到外网的自由访问
- 使用连接跟踪机制处理回程数据包
- 只做 MASQUERADE，不做 DNAT

📌 **特点**：
- **最常见的 NAT 用法**（家庭路由器就是这样）
- 源地址转换，不修改目的地址
- 配置最简单，通用性最强

### 扩展知识

#### ✅ Linux 转发流程核心链路图

```
数据包进入
    |
    v
[ PREROUTING ]  ← 做 DNAT（目标地址转换）
    |
    v
[ 路由判断 ]
    |
    +--> 是发给本机 → [ INPUT ] → 本地服务
    |
    +--> 是转发流量 → [ FORWARD ]
							|
							v
					   [ POSTROUTING ]  ← 做 SNAT/MASQUERADE（源地址转换）
							|
							v
	                    发出系统
```


#### ✅ 如何知道原始的目标地址？

Linux NAT 系统依赖 **conntrack（连接跟踪）机制**，会记录原始的会话信息：
`<源IP>:<源端口> → <原始目标IP>:<目标端口>`

当 DNAT 改变目标地址时，它会：
- 把**原目标地址**保存进 conntrack 表
- 当响应回来时，还会根据这个表进行 **逆 NAT（Reverse DNAT）**，确保回包能正确送回 Windows
所以 Kali 虽然把包目标地址改成了自己，但 Linux 其实知道这是一个中转连接！
- 确认 conntrack 表：`sudo conntrack -L`

### 其他命令

- 确认配置的NAT表：`sudo iptables -t nat -L`
- 确认filter表：`sudo iptables -L` *（因为iptables默认filter表所以不用`-t`选项）*
- 清除配置：`sudo iptables -F [-t nat]`

### 注意

- 需要在Windows添加路由：`route add 10.200.49.0 mask 255.255.255.0 10.10.10.10`
- 这些规则将在当前会话中生效，但不会在系统重启后持久生效。如果你希望在系统重启后仍然生效，你需要将这些命令添加到适当的启动脚本中（如/etc/rc.local）。*（也有其他方式）*