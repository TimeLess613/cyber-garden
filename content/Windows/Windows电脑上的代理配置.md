---
tags:
  - IT/Windows
---

| 代理类型                         | 设置方式                               | 生效范围                                                                          |
| ---------------------------- | ---------------------------------- | ----------------------------------------------------------------------------- |
| WinINet<br>（用户态，“系统代理”，功能丰富） | IE设置<br>（配置保存在注册表 HKCU）            | 浏览器/Explorer/Office/WebDAV等。<br>支持 PAC 文件、静态代理、WPAD。                          |
| WinHTTP 代理<br>（服务态，轻量级 API）  | `netsh winhttp`<br>（配置保存在注册表 HKLM） | 系统服务和后台程序用的网络栈。PowerShell/更新服务等。<br>API 支持 PAC/WPAD，但 `netsh winhttp` 常配静态代理。 |
| 浏览器独立代理                      | 浏览器内设置                             | 当前浏览器                                                                         |
| 环境变量代理                       | HTTP_PROXY/脚本参数等                   | curl/pip/git 等                                                                |
| Zscaler 代理（agent 应用）         | ZCC客户端安装                           | 全局接管（根据配置）                                                                    |


> [!note] PowerShell 的网络栈
> - 5.1版本之前都用 **WinINET**。
> 	- 在 Windows PowerShell 3.0 ~ 5.1（基于 .NET Framework 4.x），`Invoke-WebRequest`/`Invoke-RestMethod` 默认使用 **WinINET** 网络栈。
> 	- PowerShell 1.0/2.0 没有 `Invoke-WebRequest`，用 .NET 框架类库（如 `System.Net.WebClient`）或调用 COM 对象来发送网络请求。都依赖 **WinINET**。
> - 从 PowerShell Core（6.0+）开始（基于 .NET Core/.NET 5+），微软为了跨平台兼容性，移除了对 WinINET 的依赖。`HttpClient`/`Invoke-WebRequest` 等都默认使用 `SocketsHttpHandler`（**基于 sockets**），不再自动遵循 WinINET/WinHTTP 的系统代理，需要**显式配置**。

> 本质上就是 **.NET 框架/运行时的发展** 引起了 PowerShell 网络请求方式和代理行为的变化。
> 📌 演进脉络
> 1. **PowerShell 1.0 / 2.0**（基于 .NET Framework 2.0/3.5）
> 	- 没有内置 HTTP cmdlet。
> 	- 需要用 `.NET Framework` 提供的类：`System.Net.WebClient` / `System.Net.HttpWebRequest`。
> 	- 它们底层 → **WinINET** → 自动继承 IE/系统代理（支持 PAC）。
> 2. **PowerShell 3.0 ~ 5.1**（基于 .NET Framework 4.x）
>     - 引入 `Invoke-WebRequest` / `Invoke-RestMethod`，但其实就是对 `.NET HttpWebRequest` 的封装。
>     - 依旧走 **WinINET** → 遵循系统代理设置。
> 3. **PowerShell Core 6.0+**（基于 .NET Core 2.0+）
>     - `.NET Core` 为跨平台重写了网络栈，默认用 `HttpClient` + `SocketsHttpHandler`。
>     - 不再依赖 WinINET / WinHTTP，改为跨平台的代理机制（环境变量 + 显式配置）。
>     - 因此 PAC、IE 设置等完全不再生效。
> 4. **PowerShell 7+**（基于 .NET 5/6+）
>     - 在 `.NET (Core)` 的延长线上达成统一，继续使用 `SocketsHttpHandler`。





************

![[FileTransfers-Windows#Net.WebClient download cradl]]
