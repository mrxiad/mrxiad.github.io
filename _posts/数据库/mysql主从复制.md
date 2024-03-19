---
title: mysql主从复制
date: 2024-3-19 12:33:12
tags: 数据库
categories: 
- 数据库
- mysql
---





### 环境准备

- **主服务器IP**: 假设为 `43.138.20.23`（公网IP或局域网IP）
- **从服务器IP**: 假设为从服务器的IP地址
- **MySQL版本**: 保证主从服务器的MySQL版本相同或兼容



### 主服务器配置

1. **配置MySQL**:

	- 编辑MySQL配置文件（通常位于`/etc/mysql/mysql.conf.d/mysqld.cnf`或`/etc/my.cnf`）。
	- 确保设置了唯一的`server-id`。
	- 启用二进制日志（binlog）。
	- 指定要复制的数据库（可选）。

	```config
	server-id=1
	log_bin=mysql-bin
	binlog_do_db=要复制的数据库名（如果复制整个实例，可以省略此行）
	```

	

2. **创建复制用户**:

	- 在MySQL命令行中，创建一个专用的复制用户。

	```mysql
	CREATE USER 'replica'@'%' IDENTIFIED BY 'password';
	GRANT REPLICATION SLAVE ON *.* TO 'replica'@'%';
	FLUSH PRIVILEGES;
	```

	

3. **获取主服务器状态**:

	- 记录二进制日志文件名和位置，这将用于`从服务器`配置。

	```mysql
	SHOW MASTER STATUS;
	```





### 从服务器配置

1. **配置MySQL**:

	- 同样编辑MySQL配置文件。
	- 设置一个与主服务器不同的`server-id`。

	```
	[mysqld]
	server-id=2
	```

2. **配置复制**:

	- 根据主服务器提供的信息，配置从服务器连接到主服务器。

	```mysql
	CHANGE MASTER TO
	MASTER_HOST='主服务器IP',
	MASTER_USER='replica',
	MASTER_PASSWORD='password',
	MASTER_LOG_FILE='记录的文件名',
	MASTER_LOG_POS=记录的位置;
	```

3. **启动复制**:

	- 开始复制进程。

	```mysql
	START SLAVE;
	```

4. **验证复制状态**:

	- 检查复制是否成功启动，关注`Slave_IO_Running`和`Slave_SQL_Running`状态是否都是`Yes`。

	```mysql
	SHOW SLAVE STATUS\G;
	```

	

### 每一步骤的作用

- **配置MySQL**: 设置`server-id`和启用binlog是复制所必需的，因为它们使MySQL服务器能够在复制过程中唯一地标识自己并记录更改。
- **创建复制用户**: 为了安全性和权限管理，复制过程应该使用一个专用账户。这一步创建了这样的账户并授权它进行复制。
- **获取主服务器状态**: 获取当前的binlog文件和位置是为了让从服务器知道从哪里开始复制数据。
- **配置复制**: 这一步实际上是在告诉从服务器如何连接到主服务器，包括使用哪个用户、从哪个binlog文件和位置开始复制。
- **启动复制**: 这一步启动了实际的数据复制过程。
- **验证复制状态**: 最后一步是确认复制是否按预期运行，通过检查`SHOW SLAVE STATUS`的输出来完成。





> **只有主服务器需要显式地创建并配置一个专用的复制用户**