---
title: git（github）
date: 2024-2-21 15:14:00
tags: 技术
categories: 
- linux
---







# 配置github的SSH keys



### 步骤



1. ### 查看本机是否有已经生成公钥：

	```
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

	```
	 ssh-keygen -t rsa -b 4096 -C "123@qq.com"
	```

	**一路回车即可**

	

3. ### 查看公钥内容

	```
	 cat ~/.ssh/id_rsa.pub
	```

	

4. ### 复制公钥内容到github上即可





# git常用命令



## git强制拉取远程分支

1. **重置本地分支到远程分支的状态：**

```sh
git fetch origin
git reset --hard origin/master
```

这将会使你的本地分支回到远程分支的状态，丢弃所有本地的更改。

1. **如果需要，清理未跟踪的文件：**

```sh
git clean -fd
```





## git回滚到某个特定版本

如果你想将当前分支回滚到指定的提交，并且不介意放弃该提交之后的所有更改（在本地分支上），你可以使用 `git reset` 命令。这样做会重置你的 HEAD 指针到指定的提交，并且可以选择如何处理工作目录和暂存区的更改。

- **仅重置 HEAD 指针（保留工作目录和暂存区的更改）：**

```sh
git reset --soft 07e95cc2d284e4f040e4c4aba407920cc11237d6
```

- **重置 HEAD 指针，并且重置暂存区（但保留工作目录中的更改作为未提交的更改）：**

```sh
git reset --mixed 07e95cc2d284e4f040e4c4aba407920cc11237d6
```

或者简写为（因为 `--mixed` 是默认选项）：

```sh
git reset 07e95cc2d284e4f040e4c4aba407920cc11237d6
```

- **彻底回滚（重置 HEAD、暂存区和工作目录，放弃所有未提交的更改）：**

```sh
git reset --hard 07e95cc2d284e4f040e4c4aba407920cc11237d6
```

请注意，使用 `git reset --hard` 会丢失所有未提交的更改。确保在使用这个命令之前备份重要更改。







## git强制推送

强制推送到远程仓库的 `master` 分支可以使用以下命令，但请注意，这将覆盖远程分支上的所有更改，可能会导致其他人的工作丢失。因此，只有在你确实知道自己在做什么，并且这是解决问题的唯一方式时，才应该使用强制推送。

```bash
git push origin master --force
```

