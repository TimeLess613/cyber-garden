---
tags:
  - IT/Linux
---


## usr

> [!NOTE] User/Unix System Resource

- `/usr/bin`随系统更新
- `/usr/bin/local`的不会被更新覆盖

## passwd

- /etc/passwd ： 所有用户
	- 第二字段是密码：`x`表示存在shadow，`*` 表示无密码、即新用户、或停用该账号
- 而【/usr/bin/passwd】是改密需要执行的二进制程序文件。




## bashrc

> [!NOTE] rc：run command

- `.bash_profile`：只会在用户登录时运行。（联系：[[docker#^52e30c|exec bash -l]]）
- `.bashrc`：是专门为 **interactive non-login shell（非登录交互式 shell）** 设计的，每次启动新 shell 时都会运行（如执行 `bash` 或 `exec bash` 命令）。可运行 `source .bashrc` 立即应用其中的配置。
	- 用户级（~/.bashrc）覆盖系统级（/etc/bashrc）

当以 **login shell**（比如 `bash -l` 或用户登录）启动时，Bash 依次查找并执行以下文件中的第一个存在的文件：
```bash
/etc/profile
~/.bash_profile
~/.bash_login
~/.profile
```
✅ 只会执行找到的第一个用户配置文件（比如你有 `.bash_profile`，就不会继续找 `.bash_login`）



## bin

- sbin：管理员命令、daemons
- /bin 被软连接到/usr/bin。相当于Windows的System32




## tmp

这个目录较特殊，其他用户对该目录的文件有可执行权限


## dev

- /dev 硬件驱动程序