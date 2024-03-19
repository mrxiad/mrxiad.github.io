---
title: linux常用指令
date: 2024-1-17 23:28:00
tags: 技术
categories: 
- linux
---







# 已知进程杀死进程

- ps aux：查看所有进程
- ps aux | grep nginx : 查看nginx进程
- kill -9 pid：杀死pid进程







# 已知端口杀死进程



## 方案一

- 查看哪些端口被占用（tcp，udp）

```bash
netstat -tulnp
```

- 杀死端口进程

```bash
sudo fuser -k [端口]

sudo fuser -k 8080
```


	




## 方案二

- 查看端口的进程

  ```bash
  lsof -i :[端口]
  
  #例如
  lsof -i :8080
  
  #输出
  COMMAND     PID    USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
  httpServe 24009 xiadong    3u  IPv4 1496040      0t0  TCP *:http-alt (LISTEN)
  ```

- 杀死这个进程

```bash
kill -9 [pid]
```



# 查看公网ip

```bash
curl ifconfig.me
```



# 查看内网ip

```bash
ip addr
```



