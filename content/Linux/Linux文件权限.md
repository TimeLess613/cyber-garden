---
tags:
  - IT/Linux
  - 渗透
---
## 概览

### 权限

一般看3位八进制，底层是4位八进制。
- special bits：
	- setuid (`4xxx`)
	- setgid (`2xxx`)
	- sticky bit (`1xxx`)
	- 不需要的话可以设为 `0xxx`
- “读”位：用户可以列出文件
- “写”位：用户可以删除和创建新文件
- “执行”位：用户可以“cd”进入该文件夹

```
[special][owner][group][other]
```


### 两种表示法

- bit方式则对整个文件各角色的设置
- ugo（user，group，other）的方式一般针对特定角色

![[Pasted image 20240310160451.png]]

#### 特殊位的表现

![[Pasted image 20240310160947.png]]


## umask

#踩坑 rsyslog 即使配置了文件创建权限，还是要和 umask 计算后得出最终权限。所以还要配置 umask 参数。或者直接配置文件所属。

*具体计算过程略*


## SUID

背景： [https://www.jianshu.com/p/2c374f9522bd](https://www.jianshu.com/p/2c374f9522bd)

- RUID（即我们登陆时的ID）
- EUID（如给root的某文件设置了SUID，那么我们-RUID-运行这个文件时，进程的EUID就是root）
- SUID（设置给文件的）

默认情况下用户发起一个进程，进程的属主是进程的发起者（RUID），也就是说这个进程是以发起者的身份运行。**但如果该程序有SUID权限，那么程序运行为进程时，进程的属主不是发起者，而是该程序文件的属主（EUID）。**

好处：对某些文件临时赋予普通用户以root权限，避免切换到root这么危险的权限来执行操作（比如普通用户要改密码），方便管理。

> [!NOTE] 大写的S
> 表示虽然设置了SUID权限，但是没有赋予“执行(x)”权限。




### Sticky 目录权限

属性flag为：`t`

- 只能给目录，如 `/tmp` 就有
- 任何用户都可以在 sticky 目录里增改文件，但只有创建者和 root 可删。

**例：**

- `chmod 4777 file`：设置 SUID 以及 full 权限
- `chmod +rwx file` 或 `chmod a+rwx file`：给file的"所有用户"增加读、写、执行权限
- `chmod u+srw file`：给 file 的 "user" 增加 SUID、读、写权限


### ![[Linux命令#找设置了SUID的文件]]









