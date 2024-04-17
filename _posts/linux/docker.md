---
title: docker
date: 2024-3-19 21:05:30
tags: 技术
categories: 
- 工程
- docker
---





# docker服务器



-  介绍如何在ubuntu环境下创建docker容器，并且将此容器设置成一个“服务器”
-  然后如何ssh进docker容器(root用户)，并且配置免密登录



*步骤如下*



## 1.创建Dockerfile文件

```Docker
# 使用官方Ubuntu基础镜像
FROM ubuntu:latest

# 避免在自动化构建时出现提示
ARG DEBIAN_FRONTEND=noninteractive

# 安装开发所需的包
RUN apt-get update && apt-get install -y \
    vim \
    git \
    curl \
    wget \
    build-essential \
    lsb-release \
    sudo \
    man \
    software-properties-common \
    ca-certificates \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# 如果需要其他语言环境或工具，可以在这里继续安装，比如对于Python开发：
RUN apt-get update && apt-get install -y python3 python3-pip

# 对于Node.js开发：
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash - && apt-get update && apt-get install -y nodejs

# 清理apt缓存以减少镜像大小
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# （可选）设置工作目录，这是容器内的目录，你的项目代码可以放在这里
WORKDIR /root

# 设置默认运行的命令，此命令将保持容器运行
CMD ["tail", "-f", "/dev/null"]
```

------



## 2.执行命令创建镜像

```bash
docker build -t mydocker .
```

将`mydocker`换成你想要的镜像名称

------



## 3.利用镜像创建容器

```bash
docker run -d --name mydocker-container -p 8080:8080 -p 20000:22 mydocker
```

将`mydocker-container`换成你自己的容器名称，`mydocker`是镜像名称

------



此时容器已经创建成功，可以进入容器查看 

```bash
docker exec -it myapp-container /bin/bash
```



## 4.配置并开启docker容器的ssh服务

### (a)确保 SSH 服务已经在容器内安装并正在运行。

可以使用以下命令来安装并启动 SSH 服务：

```bash
docker exec -it xddocker apt-get update
docker exec -it xddocker apt-get install -y openssh-server
docker exec -it xddocker service ssh start
```

如果 `service ssh start` 不工作，您也可以尝试：

```bash
docker exec -it xddocker /etc/init.d/ssh start
```

或者直接使用 systemctl（如果您的容器支持 systemd）：

```bash
docker exec -it xddocker systemctl start ssh
```

### (b)设置 `root` 用户的密码。

你需要使用 `passwd` 命令来为 `root` 用户设置密码：

```bash
docker exec -it xddocker passwd root
```

当系统提示时输入并确认密码。

### (d)配置 SSH 以允许 `root` 用户登录。

编辑容器中的 `/etc/ssh/sshd_config` 文件，将 `PermitRootLogin` 的值更改为 `yes`：

```bash
docker exec -it xddocker sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
```

### (e)重新启动 SSH 服务以使更改生效：

```
docker exec -it xddocker service ssh restart
```

### (f)使用 SSH 连接到容器：

现在您应该可以使用 SSH 连接到容器的 `root` 用户了。使用宿主机的端口 `20000` 来连接：

```bash
ssh root@localhost -p 20000
```





------





**到此为止，docker容器的ssh已经完全配置好了**





------

## 5.配置免密登录

以下是在Windows系统使用Git Bash来完成这个过程的详细步骤

------

### (a)生成SSH密钥对

打开`git bash`,输入命令生成密钥对，一路回车，密钥对在 `/c/Users/<你的用户名>/.ssh/id_rsa`文件中

```bash
ssh-keygen -t rsa -b 4096
```



### (b)复制SSH公钥到远程服务器

本地git bash中查看公钥内容,复制下来

```bash
cat ~/.ssh/id_rsa.pub
```

然后登录远程主机   (端口号20000，这是映射的端口)

```bash
ssh root@远程主机ip -p 20000
```

登录到远程服务器后，执行以下命令来编辑（或创建）`authorized_keys`文件：

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
vim ~/.ssh/authorized_keys
```

将复制的公钥粘贴进文件

```basj
按i键进入编辑模式
按住Shift+Insert进行粘贴
按Esc退出面积模式
输入":wq"保存退出
```

### (c)退出后，即可进行免密登录









# docker网络

## 常用命令

### (1)查看网络帮助

```bash
docker network help
```

```bash
connect 			Connect a container to a network	#将一个容器连接到一个网络

create 				Create a network					#创建一个网络

disconnect 			Disconnect a container from a network #从网络断开一个容器

inspect 			Display detailed information on one or more networks  #在一个或多个网络上显示详细信息

ls 					List networks  					#网络列表

prune 				Remove all unused networks 		#删除所有未使用的网络(不要使用)

rm 					Remove one or more networks 	#删除一个或多个网络。
```



### (2)查看容器信息

```bash
docker port 【容器id】
docker inspect 【容器id】
```

### (3)查看容器详细信息

```bash
docker inspect 【容器id】
```







## 常用的网络模式

### (1)bridge模式(默认)

Docker的**默认模式**，它会在docker容器启动时候，自动配置好自己的网络信息，同一宿主机的所有容器都在一个网络下，彼此间可以通信。利用宿主机的网卡进行通信，因为涉及到网络转换，所以会造成资源消耗，网络效率会低。



### (2)host模式

容器使用宿主机的ip地址进行通信。特点：容器和宿主机共享网络



### (3)container模式

新创建的容器间使用，使用已创建的容器网络，类似一个局域网。 特点：容器和容器共享网络



### (4)none模式

这种模式最纯粹，不会帮你做任何网络的配置，可以最大限度的定制化。 不提供网络服务，容器启动后无网络连接。



### (5)overlay模式

容器彼此不在同一网络，而且能互相通行。







# dockerfile（构建镜像）

- **基础镜像**：从一个已存在的镜像开始构建新镜像（例如`FROM ubuntu:18.04`）。
- **维护者信息**：镜像创建者的信息（例如`MAINTAINER`）。
- **环境配置**：设置环境变量（例如`ENV`）。
- **安装软件**：使用包管理器安装所需软件（例如`RUN apt-get install -y nginx`）。
- **添加文件/目录**：将本地文件或目录添加到镜像中（例如`ADD`或`COPY`）。
- **配置工作目录**：设置工作目录的路径（例如`WORKDIR /app`）。
- **配置启动命令**：设置容器启动时执行的命令（例如`CMD ["nginx", "-g", "daemon off;"]`）。
- **端口暴露**：声明容器运行时监听的端口（例如`EXPOSE 80`）。



## 构建命令

```bash
docker build -t [镜像名]:[版本号][Dockerfile所在目录]
```





## 多阶段构建



example:

```dockerfile
# 第一阶段：构建应用
FROM golang:1.16 AS builder
WORKDIR /app       #相当于cd

# 复制源代码到工作目录
COPY . .

# 选择以下两种方式之一：

# 方式1: 如果你的项目使用Go模块，请确保你的项目目录中有go.mod和go.sum文件，然后使用此行：
# RUN CGO_ENABLED=0 go build -o myapp .

# 方式2: 如果你的项目不使用Go模块，取消以下行的注释：
RUN GO111MODULE=off CGO_ENABLED=0 go build -o myapp .

# 第二阶段：构建最终镜像
FROM alpine:latest
WORKDIR /root/

# 从builder阶段复制构建好的应用
COPY --from=builder /app/myapp .

# 设置运行时的命令
CMD ["sh", "-c", "./myapp || tail -f /dev/null"]
```



创建并启动容器

```bash
docker run -d --name mygo-app mygo:1.0
```



查看日志

```bash
docker logs mygo-app
```









# docker-compose（创建容器）



## 概念



### 服务（Services）

- **image**：指定服务使用的镜像。
- **build**：**指定一个目录的路径，该目录中包含了一个Dockerfile，Docker Compose将用它来构建镜像。**
- **command**：覆盖容器启动后默认执行的命令。
- **environment**：设置环境变量。
- **volumes**：挂载卷。
- **ports**：端口映射。
- **depends_on**：设置服务间的依赖关系。
- **networks**：指定网络设置。

### 卷（Volumes）

- 定义用于数据持久化或共享数据的卷。

### 网络（Networks）

- 定义容器间如何相互通信的网络。





> build  &  image 都是指定镜像



## 常用命令

### `docker compose`命令（Docker CLI的一部分）

> 需要在docker-compose.yml目录下运行

- **启动服务**：

  ```bash
  docker compose up
  ```

  ```bash
  docker compose up -d  #
  ```

- **停止删除服务**：

	```bash
	docker compose down
	```

- **停止服务**

  ```bash
  docker compose stop
  ```

- **构建或重建服务**：

	```bash
	docker compose build
	```

- **查看运行中的服务**：

	```bash
	docker compose ps
	```

- **查看服务的日志**：

	```bash
	docker compose logs
	```

- **执行一个服务容器中的命令**：

	```bash
	docker compose exec service_name command
	```





```yaml
version: '3'
services:
  web:
    image: "nginx:alpine"
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
  db:
    image: "postgres:9.6"
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:

```







# 技巧



## 创建一个容器用于调试

```bash
docker run -it --rm busybox
```







# 可进行的配置

- 内存限制
- 网络
- 数据卷