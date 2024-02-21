---
title: github的ssh配置
date: 2024-2-21 15:14:00
tags: 技术
categories: 
- linux
---





# 本文介绍linux服务器如何配置github的SSH keys



## 步骤



1. ### 查看本机是否有已经生成公钥：

	```bash
	ls -al ~/.ssh
	
	#输出如下：
	total 20
	drwx------  2 xiadong xiadong 4096 Feb 21 15:08 .
	drwxr-xr-x 24 xiadong xiadong 4096 Feb 21 15:06 ..
	-rw-------  1 xiadong xiadong 3239 Feb 21 15:08 id_rsa      
	-rw-r--r--  1 xiadong xiadong  743 Feb 21 15:08 id_rsa.pub
	-rw-r--r--  1 xiadong xiadong  666 Jan 27 19:50 known_hosts
	```

	

2. ## 如果没有`id_rsa`和`id_rsa.pub`，则生成公钥

	```bash
	ssh-keygen -t rsa -b 4096 -C "123@qq.com"
	```

	**一路回车即可**

	

3. ### 查看公钥内容

	```bash
	cat ~/.ssh/id_rsa.pub
	```

	

4. ### 复制公钥内容到github上即可





