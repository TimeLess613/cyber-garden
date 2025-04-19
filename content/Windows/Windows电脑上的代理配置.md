---
tags:
  - IT/Windows
---

| 代理类型                 | 设置方式             | 生效范围                       |
| -------------------- | ---------------- | -------------------------- |
| WinINet（系统代理）        | IE设置/注册表         | 浏览器/Explorer/Office/WebDAV |
| PAC 文件               | IE设置中引入          | 同上                         |
| WinHTTP 代理           | `netsh winhttp`  | PowerShell/更新服务            |
| 浏览器独立代理              | 浏览器内设置           | 当前浏览器                      |
| 环境变量代理               | HTTP_PROXY/脚本参数等 | curl/pip/git 等             |
| Zscaler 代理（agent 应用） | ZCC客户端安装         | 全局接管（根据配置）                 |

> [!note] PowerShell默认（Invoke-WebRequest）用 WinHTTP 网络栈；还可以调用 WinINet 网络栈（如 .NET 的 `System.Net.WebClient` 或 COM 组件）
> 但注意并不是所有 `System.Net.WebClient` 都调用 WinINet 网络栈，其取决于运行时环境（.NET Framework vs .NET Core/5+/6+）与系统配置。
> - 一般来说 .NET Framework（PowerShell 5.x 及之前）会使用 WinINet 网络栈
> - 而 .NET Core / .NET 5/6+ / PowerShell Core（6.0+）开始为了跨平台兼容性，微软就剥离了 WinINet 支持，`HttpClient` 和所有网络库都默认使用 `SocketsHttpHandler`（基于 sockets）
> #GPT摘录 

![[FileTransfers-Windows#Net.WebClient download cradl]]
