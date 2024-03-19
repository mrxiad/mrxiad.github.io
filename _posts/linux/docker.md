---
title: docker服务器
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





