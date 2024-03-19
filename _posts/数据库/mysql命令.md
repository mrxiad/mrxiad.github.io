---
title: mysql命令
date: 2024-3-19 12:33:12
tags: 数据库
categories: 
- 数据库
- mysql
---





# 基础命令



## 查看客户端连接了

```mysql
show processlist;
```



## 查看空闲连接最大空闲时长

```mysql
 show variables like 'wait_timeout';
```



## 手动断开空闲的连接

```mysql
kill connection +6;  -- 6是id
```



## 查看最大连接数

```mysql
show variables like 'max_connections';
```



# 文件相关



## 数据库的文件存放目录

```mysql
mysql> SHOW VARIABLES LIKE 'datadir';

+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| datadir       | /var/lib/mysql/ |
+---------------+-----------------+
1 row in set (0.00 sec)
```





# 锁

## 对数据库加锁

```mysql
flush tables with read lock;
```

> 执行后，**整个数据库就处于只读状态了**，这时其他线程执行以下操作，都会被阻塞：
>
> - 对数据的增删改操作，比如 insert、delete、update等语句；
> - 对表结构的更改操作，比如 alter table、drop table 等语句。



## 释放锁

```mysql
unlock tables;
```





# 备份

```mysql
mysqldump --single-transaction -u username -p database_name > backup_file.sql
```

- `--single-transaction`：这个选项告诉`mysqldump`在备份开始时开启一个新的事务。为了保持一致性视图，事务的隔离级别会被设置为可重复读。这样，备份操作看到的数据是在事务开始时刻的一致性快照，**而备份操作本身不会阻塞后续的写操作**。
- `-u username`：指定用于连接数据库的用户名。
- `-p`：提示输入用于连接数据库的用户密码。
- `database_name`：指定要备份的数据库名。
- `> backup_file.sql`：将备份输出重定向到一个文件中。





# 恢复

```bash
mysql -u username -p database_name < backup_file.sql
```

- `-u username`：指定用于连接数据库的用户名。
- `-p`：在命令执行后，你会被提示输入该用户的密码。出于安全考虑，建议不要在命令行中直接包含密码。
- `database_name`：指定要恢复到的目标数据库名。这个数据库应该已经存在于MySQL服务器上，`mysql`命令不会自动创建数据库。
- `< backup_file.sql`：这部分告诉shell从`backup_file.sql`文件中读取SQL语句并将其作为输入传递给`mysql`命令。