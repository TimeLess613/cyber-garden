---
tags:
  - IT/容器
  - IT/DevOps
---
## 踩坑

#VScode [[Linux命令#usermod]]
`sudo usermod -aG docker $USER`添加组并重启VScode远程连接主机后，prompt 发生异常（变为极简版本：`hostname$`）。确认目前进程执行的是zsh，但是运行 `exec bash` 后prompt恢复正常。
不确定是否和运行了 `newgrp docker` 有关。多次重启VScode连接无果，且突然发现之前加入的docker组消失了，后来发现 `newgrp docker` 仅为临时设定。

**最终解决：重启主机。**
参考：
> https://stackoverflow.com/questions/56305613/cant-add-user-to-docker-group
> https://stackoverflow.com/questions/56305613/cant-add-user-to-docker-group/66297855#66297855

---

Docker Engine：即 Docker CE（Community Edition）。在某次更新后更名为此（因为以前有的 Docker EE 已经被 Mirantis 接手）。现在 Docker Engine 区别于“Docker Desktop”。

## 安装

> [!NOTE] 仅为过去的项目笔记，最新信息参考[官方文档](https://docs.docker.com/engine/install/)。

```
# yum install -y yum-utils
# yum-config-manager --add-repo
https://download.docker.com/linux/centos/docker-ce.repo
# yum install -y docker-ce docker-ce-cli containerd.io
```

**启动&确认：**
```
# systemctl enable docker
# systemctl start docker

# docker --version
```

## 配置代理

docker是C/S模型，其使用docker命令与dockerd交互。

所以代理有两种。

- 为Docker客户端（Docker CLI）配置代理：使容器中的程序能够使用环境变量中的代理配置
	- [proxy/#configure-the-docker-client](https://docs.docker.com/network/proxy/#configure-the-docker-client)
- 为Docker守护程序（dockerd）本身配置代理服务：其他情况（如pull）
	- [systemd/#httphttps-proxy](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy)

### dockerd

添加配置文件
```
# cat /etc/systemd/system/docker.service.d/http-proxy.conf
[Service]
Environment="HTTP_PROXY=http://proxy:8080"
Environment="HTTPS_PROXY=http://proxy:8080"
```

应用配置
```
sudo systemctl daemon-reload
sudo systemctl restart docker
```

确认是否应用了配置

- `docker info`：会返回所有容器和镜像的基本信息
- `sudo systemctl show --property=Environment docker`



## .dockerenv文件

容器的根目录一般都有这个文件——渗透时可根据这个文件判断自己是否在容器中。



## docker命令

### Docker 运行（docker run）

```bash
docker run [-i -t -d] [--name 容器名] <image> [/bin/bash]
# /bin/bash：如果不指定命令，则默认使用 Dockerfile 中的 `CMD` 指令。
```

#### 运行模式选项

- `-it`（交互模式）：
    - **i**：保持 `stdin` 打开，即可交互输入
    - **t**：分配一个伪[[终端]]（pseudo-TTY），用于创建交互式 shell
    - **注意**：退出 shell 后，容器会停止运行。
- `-d`（后台模式，Detached Mode）：
    - 让容器作为 `dockerd` 的后台进程运行，不需要 `-it` 选项。
    - **注意**：若使用 `-d`，不要指定命令 `bash`，否则容器可能无法正常启动。
	    - 原因：`bash` 运行后没有交互终端，检测到 `stdin` 关闭，会自动终止。

#### 容器创建行为

1. **检查镜像**：
    - 先查找本地是否存在该镜像。
    - 若无，则自动从官方 Registry（Docker Hub）下载。
2. **网络设置**：
    - Docker 默认会为容器创建独立的网络。
    - 以及一个用于和宿主机通信的 **网桥（docker0）**。
	    - `docker0` 本身是 **网关**，容器通过它访问外部网络。
	    - 每个容器会在 `docker0` 上分配一个 **IP 地址**（默认 `172.17.0.0/16`，可修改[[docker#网桥默认池|网桥默认池]]）。
	    - 容器的网络接口以 `veth` 开头，默认连接 `docker0`。

#### 端口映射（Port Mapping）

**手动映射：**`-p <宿主ip:port>:<容器port>`

**自动分配端口：**

- `-p <port>`
    - 只指定容器端口，Docker 会 **随机选择一个宿主机大端口** 进行映射。
- `-P`（大写）
    - **自动公开** Dockerfile 中 `EXPOSE` 指定的所有端口，并映射到宿主机的随机端口。

#### 数据卷（[[docker#volumes方式]]）

```bash
-v <宿主机目录>:<容器目录>
```

- 挂载主机目录到容器内部，使二者数据同步。
- 容器销毁后，宿主机目录仍然存在。



### 删除

**✅ 1. `docker container rm`**

```bash
docker container rm <CONTAINER_ID 或 NAME>
# 等价。可理解为容器经常抛弃所以有简化命令。
docker rm <CONTAINER_ID 或 NAME>
```

- **删除一个或多个容器**，但 **不会影响对应的镜像**。
- **容器必须是已停止状态**，否则不能删除。


**✅ 2. `docker image rm`**

```bash
docker image rm <IMAGE_ID 或 NAME>
```

- **删除一个或多个 Docker 镜像**，但不会影响已运行的容器。
- 如果该镜像仍然被某个容器引用（即使容器已停止），**不能删除**，除非加 `-f` 强制删除。



**清理大师：**
```bash
docker system prune -a
```


### 启动已有容器

- `start <容器>`
- `restart <容器>`


### logs

比如运行容器启动却发现容器立马停止了，于是debug。
```bash
docker logs [-f] backend
```

### 进入容器


> [!NOTE] 容器的运行机制
> Docker容器在后台运行，必须要有一个前台进程（如shell），这里我们让容器有前台程序运行，就可以实现容器的`-d`启动后存活。（如果前台进程不是shell这种，而是运行`ps`的话，容器执行完命令后就会关闭）


#### exec

- `exec -it <容器> <命令>`

在容器内另外启动新进程（对比[[docker#attach|attach]]）。

使用 `docker exec -it <容器> /bin/bash` 进入容器内，**这种方法是最通用的方法**。（ver1.3之后；ver1.2之前用nsenter）


**也可不进入，快速执行命令将结果打印到屏幕：**
```bash
docker exec nginx ls /
```


#### attach

- `attach <容器>`

附着到容器正在执行的命令进程（docker ps看COMMAND）的终端。

> [!NOTE] 这个少用（忘了具体场景……）
> 启动命令不是sh的话进入容器会卡住


> [!NOTE] 请按`Ctrl+P+Q`进行退出容器
> 如果要正常退出不关闭容器（即使-d启动也），请按`Ctrl+P+Q`进行退出容器，这一点很重要，请牢记！


### 网络

> [!NOTE] --link 将被弃用
> 公告： [https://docs.docker.com/network/links/](https://docs.docker.com/network/links/)
> 对策：用[[docker#docker网络|docker网络]]功能（即bridge等那个，基本命令如下）

- `docker network ls`：查看本地docker主机上的全部网络
- 新建网桥：`docker network create my_network`
- 连接：`docker run --net my_network --name app`
	- 于是构建容器命令变为：`docker run --rm -itd --name app1 --expose 80 --net my_network php /bin/sh`


### 镜像

- 搜索：`search  <image>`
- 获取：`pull <image:tag (默认“:latest)”>`
- 查看：`docker image <CONTAINER>`
- 查看镜像被构建时的每一步：`docker history <IMAGE>`

#### 构建镜像

推荐：编写Dockerfile后用docker build命令。

构建环境的目录（即放Dockerfile文件的，称为context）会把context的目录和文件上传到Docker守护进程，使得守护进程能够直接访问你想在镜像中存储的任何代码、文件等。

- -t 命名，建议为镜像设置标签（默认为lateset）。最后的点"."告诉Docker到本地目录找Dockerfile文件。
- 第一条指令都是FROM，指定基础镜像（base image）
- 默认有构建缓存——构建失败的话可以进中途成功的容器继续构建
- （对于CMD？）推荐使用以字符串数组语法来设置要执行的命令。不用的话docker会在指定命令前加上`/bin/sh -c`，可能有意外
- RUN（容器构建时运行）可以覆盖CMD（容器启动时运行）。Dockerfile只指定一条CMD，就算有多条也只有最后一条被使用
- ADD 将context的文件复制到镜像中
- VOLUME 卷目录可以绕过联合文件系统，以正常文件形式存在于宿主机，提供容器间的数据共享和持久化功能。



### 改名

`docker rename UUID/oldname newname`

- UUID（Universally Unique Identifier）

### logs

`logs -f <容器>`

与tail -f相似。


### top

`top <容器>`




## 挂载


### 持久化


> 理解： [https://4sysops.com/archives/introduction-to-docker-bind-mounts-and-volumes/](https://4sysops.com/archives/introduction-to-docker-bind-mounts-and-volumes/)


在 Docker 中，**数据持久化**有两种主要方式：
1. **`volumes`（Docker 管理的数据卷）** ✅ **Docker 推荐**
2. **`bind mounts`（绑定宿主机目录）** ⚠️ **更灵活，但管理成本更高**

> [!note] 就算指定的是文件，也必定被当做目录（忘了具体场景……）

#### bind方式

- **直接绑定宿主机的某个目录到容器内**。
- **宿主机的目录不会被 Docker 管理**，数据改动 **实时同步**。

例：
```
services:
  neo4j:
    image: neo4j
    container_name: neo4j
    volumes:
      - ./neo4j_data:/data  # 绑定宿主机目录到 /data
```
- 这里的 `./neo4j_data:/data` **意味着宿主机的 `neo4j_data/` 目录和容器内的 `/data` 目录实时同步**。


- `-v <宿主机>:<容器>`
- 通过`docker inspect`命令，查看容器“Mounts”那一部分



#### volumes方式

- **数据存储在 Docker 管理的目录**（通常是 `/var/lib/docker/volumes/`）。
- **不会直接映射到宿主机上的可见目录**，但数据可以被多个容器共享。
- ⚠️ 数据存储在 `/var/lib/docker/volumes/`，**宿主机用户看不到**  
- ⚠️ 不能像普通文件夹一样直接访问，需要 `docker volume inspect`

例：
```bash
services:
  neo4j:
    image: neo4j
    container_name: neo4j
    volumes:
      - neo4j_data:/data  # Docker 管理的 volumes
volumes:
  neo4j_data:    # 定义命名卷
```


查看所有卷（匿名卷和命名卷）：
- 匿名卷：当启动容器时没写宿主机目录，仅指定容器目录时，则在宿主机生成匿名卷。
```bash
└─$ docker volume ls
DRIVER    VOLUME NAME
local     0e0be8e918e5d1384a5adf237dbb6d47b8fd46388df1b1d58a3e07971d5739fc
local     a43534676c1c4dfcb21352317132189cf579788677ed1d18cf50977a99b30a99
local     xsoar-visualizer_neo4j_data
```

查看卷被挂载在哪：
```bash
docker volume inspect <VOLUME_NAME>

# Mountpoint 即该卷在宿主机的位置。
└─$ docker volume inspect  a43534676c1c4dfcb21352317132189cf579788677ed1d18cf50977a99b30a99
[
<SNIP>
        "Mountpoint": "/var/lib/docker/volumes/a43534676c1c4dfcb21352317132189cf579788677ed1d18cf50977a99b30a99/_data",
<SNIP>
]
```


删除**所有未被使用的卷**：
```bash
docker volume prune
```




#### 区别

![[Pasted image 20240317223116.png]]
![[Pasted image 20240317223138.png]]


#### **什么时候用 `volumes`，什么时候用 `bind mounts`？**

|**场景**|**推荐方式**|**原因**|
|---|---|---|
|**数据库数据存储（如 Neo4j）**|✅ **`volumes`**|**更安全，防止误删**|
|**开发环境（修改代码自动生效）**|✅ **`bind mounts`**|**代码改动立即同步**|
|**共享数据（多个容器访问相同存储）**|✅ **`volumes`**|**Docker 内部更好管理**|
|**日志文件映射到宿主机**|✅ **`bind mounts`**|**方便本地分析日志**|


### 改映射

> [https://www.cnblogs.com/poloyy/p/13993832.html](https://www.cnblogs.com/poloyy/p/13993832.html)

![[Pasted image 20240317222529.png]]


 > [!NOTE] [但容器应该是短暂的](https://github.com/moby/moby/issues/31452)
 > 所以一般还是停止容器并新建run。
![[Pasted image 20240317222629.png]]




## docker网络

> [[虚拟网卡]]

- 不同网段的容器ping不通——可把容器加入多个网桥解决
- 默认网桥不支持容器名解析，而自定义网桥可以

![[Pasted image 20240317223340.png]]

 > [Docker容器间通信与外网通信](https://blog.csdn.net/vanvan_/article/details/98663379?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.essearch_pc_relevant&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.essearch_pc_relevant)
>  [Docker 容器网络访问原理，SNAT和DNAT](https://blog.csdn.net/shi_hong_fei_hei/article/details/114481494)



### 网桥默认池

> [https://straz.to/2021-09-08-docker-address-pools/](https://straz.to/2021-09-08-docker-address-pools/)

- **base**是整个docker网络
- **size**是从base中划分给每个bridge的大小
```bash
# cat /etc/docker/daemon.json
{
  "default-address-pools": [
    { "base": "10.1.0.0/12", "size": 24 }
  ]
}

# sudo systemctl restart docker
```
- 重启服务后，已有的bridge也会自动变更网段（目前只测试了默认bridge）


docker-compose 也可自定义（需要docker-compose.yml文件定义version2或以上）  
> [https://knowledge.sakura.ad.jp/26522/](https://knowledge.sakura.ad.jp/26522/)



## docker-compose

> [!note] 升级到 Docker Compose v2
> [docker-compose 与 docker compose](https://forums.docker.com/t/docker-compose-vs-docker-compose/137884/7)
> 
> - `sudo apt-get remove docker-compose`
> - `sudo apt-get install docker-compose-plugin`
> - `docker compose version`

```bash
docker-compose up -d --build [service] [service...] [--profile service]
```
- `--profile`：显式指定需要启动的profile。需要如下语法来定义 profiles 标签名：
```docker-compose.yml
service:
	......
	profiles:
		- frontend
```

在大多数情况下，Docker Compose **会自动检测变更并更新容器**（只会 rebuild 有变动的服务），**不需要手动先停止容器**。**但不同情况的行为有所不同（挂载了 volume 使代码同步映射，或者有其他热重载配置）**

🚀 **推荐日常使用方式**

1. **如果只是更新代码（无 Dockerfile 变更），直接运行：**
```bash
docker-compose up -d --build
```

2. **如果修改了 `Dockerfile` 或 `docker-compose.yml`，最好先停掉容器：**
```bash
docker-compose down && docker-compose up -d --build
```

3. **如果需要清理无用容器和 volumes（如数据库存储），使用：**
```bash
# -v：删除所有docker-compose中的相关volumes
# --remove-orphans：清理旧的无用容器
docker-compose down [-v] --remove-orphans
docker-compose up -d --build
```

> [!note] 报错解决：KeyError: 'ContainerConfig'
> 这个就要用上述 2 或 3
 


### 关于 kali上的 Compose 支持

安装时报错：
```bash
└─$ sudo apt-get update && sudo apt-get install docker-compose-plugin
<SNIP>             
E: 无法定位软件包 docker-compose-plugin
```

原因：似乎是 `docker.io` 版本较落后，未支持 `docker compose` v2。

| 安装方式                        | Docker Engine 版本      | Compose 版本                        | 支持 Compose v2 CLI 插件？ |
| --------------------------- | --------------------- | --------------------------------- | --------------------- |
| Kali / Debian (`docker.io`) | 往往是旧版（比如 20.x 或 23.x） | 只有旧版 Python `docker-compose` v1.x | ❌ 不支持                 |
| Docker 官方仓库 (`docker-ce`)   | 最新稳定版（24.x）           | 内置 `docker compose` v2.x 插件       |                       |





---
## 问题管理

### 构建docker的nginx后访问报403错误

解决：[关闭SELinux](onenote:..\20.渗透\环境.one#最小安装&section-id={4F53940D-9292-4939-8ED7-4F9EB86D1E96}&page-id={1F0F8C37-26AF-49F5-8C99-B5CD4E03A395}&object-id={F504FA59-1F0D-4CC7-959B-9BD5D23AE313}&18&base-path=https://d.docs.live.net/d10d79d0adccf558/文档/我的笔记本)。容器需要重开

### docker下overlay2占用空间过大

解决：清理docker占用空间

- [https://www.cnblogs.com/Henry121/p/15627488.html](https://www.cnblogs.com/Henry121/p/15627488.html)
- [https://www.creationline.com/lab/35518](https://www.creationline.com/lab/35518)



