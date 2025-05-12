---
tags:
  - IT/DevOps
---

工作区 ----> 暂存区 ----> 仓库
- 工作区：项目文件夹
- 暂存区（index/stage）和本地仓库可理解为一个抽象文件夹，其具体实现则在 `.git` 中。由于不直接操作这个文件夹，所以用命令。

![[77ea67d8acf8d0bf1a2155b675537d2c.png]]


## git内部目录

> https://jvns.ca/blog/2024/01/26/inside-git/

- `.git/config`：本地设置。全局设置在 `~/.gitconfig`。