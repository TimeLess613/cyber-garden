---
tags:
  - IT/Windows
  - 渗透
  - OSEP
---
## LSA Protection

原本，Windows 将其进程分为四个不同的完整性级别。从 Windows 8 开始引入了一个额外的缓解级别，**Protected Processes Light (PPL)**，它可以叠加在当前完整性级别之上。即：SYSTEM 完整性下运行的进程无法访问或修改启用了 PPL 的 SYSTEM 完整性下运行的进程的内存空间。（导致 mimikatz 转储失败，但其实依然可以绕过）

LSASS 支持启用 PPL，可通过注册表配置：`This is done through the RunAsPPL DWORD value in HKLM\SYSTEM\CurrentControlSet\Control\Lsa with a value of 1.`
由于第三方兼容性问题，此保护机制可能默认被禁用。

## mimikatz 绕过 PPL

- PPL 在 `EPROCESS` 结构中以一个位（bit）进行控制    
- 如果能获取 **内核级执行权限（Ring 0）**，可以修改这个位，从而**禁用 LSA 进程的保护**    
- Mimikatz 带有一个名为 `mimidrv.sys` 的内核驱动，可以实现这个操作。


❗ 但现在的情况是（截至 2025 年）：
- 🔐 **Windows 10/11 + Defender + Credential Guard 启用时：**    
    - 驱动签名更加严格
    - 默认开启 HVCI（Hypervisor-Protected Code Integrity），会**阻止加载未签名或非微软驱动**
    - 即使尝试加载 `mimidrv.sys`，也会被 Defender/EDR 拦截。
- 🔧 除非满足以下条件之一，才可能有效：    
    -  在 **禁用 Defender / HVCI 的测试环境**中
    - 使用某些漏洞（如驱动漏洞）获得内核执行权限，然后修改 `EPROCESS`
    - 自行绕过签名机制（使用 Bring Your Own Vulnerable Driver 技术，如利用RTCore、ASRock驱动等）后加载 mimidrv 功能。