---
title: mysql基础篇
date: 2024-2-27 12:33:12
tags: 数据库
categories: 
- 数据库
- mysql
---



## 登录

```bash
mysql -uroot -p123456
```



## 重置密码步骤(linux)

###  1.停止 MySQL 服务

首先，你需要停止正在运行的 MySQL 服务。这个步骤根据你的操作系统会有所不同。

- **对于 Linux 系统**，可以使用以下命令之一，具体取决于你的系统管理工具：

	```
	bashCopy codesudo systemctl stop mysql
	# 或者
	sudo /etc/init.d/mysql stop
	```

### 2. 启动 MySQL 无密码模式

接下来，你需要以无密码模式启动 MySQL。这允许你以 root 用户身份登录而不需要密码，从而可以重置丢失的密码。

- **对于 Linux**，启动 MySQL 时添加 `--skip-grant-tables` 参数：

	```bash
	sudo mysqld_safe --skip-grant-tables &
	```

### 3. 重置密码

现在，你可以登录到 MySQL 服务器而不需要密码，然后重置密码。

1. 登录到 MySQL：

	```bash
	mysql -u root
	```

2. 选择 MySQL 数据库：

	```mysql
	USE mysql;
	```

3. 更新 root 用户的密码。根据你的 MySQL 版本，命令可能略有不同：

	- **MySQL 5.7 及以前版本**：

		```mysql
		UPDATE user SET password=PASSWORD("新密码") WHERE User='root';
		FLUSH PRIVILEGES;
		```

	- **MySQL 5.7.6 及之后版本**：

		```mysql
		ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';
		FLUSH PRIVILEGES;
		```

	- **MariaDB** 可能也支持 `ALTER USER` 或需要使用 `UPDATE` 语句。

4. 退出 MySQL：

	```mysql
	quit
	```

### 4. 重启 MySQL 服务

最后，停止无密码模式下运行的 MySQL 服务，然后正常启动 MySQL 服务。

- **对于 Linux**：

	```bash
	sudo systemctl stop mysqld
	sudo systemctl start mysql
	# 或者使用 /etc/init.d/mysql stop 和 start
	```



## 更改密码（已知密码）

```mysql
-- 需要先进入数据库

-- 这里把数据库登录密码改为111111
alter user 'root'@'localhost' identified by '111111'; 
  
-- 账号和地址不带引号也可以  
alter user root@localhost identified by '88888888' ;   
```





## 乱码问题

MySQL 数据库乱码问题通常是由于字符集设置不一致导致的。解决这个问题通常涉及检查和设置数据库、表或连接的字符集和排序规则（collation）。

### 检查当前的字符集设置

首先，你需要检查 MySQL 的当前字符集和排序规则设置。可以使用以下命令查看全局、数据库级别和表级别的字符集和排序规则设置：

- **查看全局字符集和排序规则**:

	```mysql
	SHOW VARIABLES LIKE 'character_set_%';
	SHOW VARIABLES LIKE 'collation_%';
	```

- **查看特定数据库的字符集和排序规则**:

	```mysql
	SELECT DEFAULT_CHARACTER_SET_NAME, DEFAULT_COLLATION_NAME
	FROM information_schema.SCHEMATA
	WHERE SCHEMA_NAME = 'your_database_name';
	```

- **查看特定表的字符集和排序规则**:

	```mysql
	SELECT CCSA.character_set_name AS 'Charset', CCSA.collation_name AS 'Collation'
	FROM information_schema.`TABLES` T,
	     information_schema.`COLLATION_CHARACTER_SET_APPLICABILITY` CCSA
	WHERE CCSA.collation_name = T.table_collation
	  AND T.table_schema = "your_database_name"
	  AND T.table_name = "your_table_name";
	```

### 2. 设置字符集

如果你发现字符集设置不正确，你可以通过以下方式进行调整：

- **设置数据库的默认字符集**:

	```mysql
	CREATE DATABASE 数据库名 CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
	```

- **修改现有数据库的字符集**:

	```mysql
	ALTER DATABASE 数据库名 CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
	```

- **设置表的字符集**:

	```mysql
	ALTER TABLE 表名 CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
	```

### 3. 设置连接字符集

当你的应用程序连接到 MySQL 服务器时，确保连接使用的字符集也是正确的。这可以通过在连接字符串中设置字符集来实现，具体方法取决于你使用的编程语言和数据库驱动。

### 注意事项

- **字符集选择**：`utf8mb4`是一个好的通用选择，因为它支持所有Unicode字符，包括表情符号。如果你的应用不需要这么广泛的字符支持，你可以选择其他字符集，但最好保持数据库、表和连接字符集的一致性。
- **备份数据**：在对数据库或表进行字符集更改之前，务必备份你的数据，以避免潜在的数据丢失。
- **测试更改**：在生产环境应用任何更改之前，先在测试环境中进行测试，以确保更改不会导致应用程序中出现问题。







## 数据库的备份和恢复

### 备份



#### 使用 `mysqldump` 工具

`mysqldump` 是 MySQL 自带的一个非常实用的工具，用于创建数据库的一份物理备份。

- **备份整个数据库**:

	```shell
	mysqldump -u [username] -p[password] [database_name] > [backup_file].sql
	```

	这里，`[username]` 是你的数据库用户名，`[password]` 是密码（注意，`-p` 后面没有空格直接跟密码），`[database_name]` 是你要备份的数据库名，`[backup_file].sql` 是备份文件的名称。
	
- **备份多个数据库**:

	```shell
	mysqldump -u [username] -p[password] --databases [database1] [database2] > [backup_file].sql
	```

- **备份所有数据库**:

	```shell
	mysqldump -u [username] -p[password] --all-databases > [backup_file].sql
	```



### 恢复

#### 使用 `mysql` 客户端工具

备份文件可以使用 `mysql` 命令行工具恢复到数据库中。

- 恢复数据库：

	```shell
	mysql -u [username] -p[password] [database_name] < [backup_file].sql
	```

	在这个命令中，你需要指定数据库名称（如果数据库不存在，需要先创建它），然后将备份文件导入到该数据库中。





## 数据类型

### 整数类型

- `int` (integer): 整数类型。

### 小数类型

- `double`: 双精度浮点类型。
- `decimal (m,d)`: 指定整数位与小数位长度的小数类型。例如，`decimal(10,2)` 表示总共10位数字，其中2位为小数。

### 日期类型

- `date`: 日期类型，格式为 `yyyy-MM-dd`，包含年月日，不包含时分秒。例如，`2019-05-06`。
- `datetime`: 日期类型，格式为 `YYYY-MM-DD HH:MM:SS`，包含年月日时分秒。例如，`2019-05-06 09:49:30`。
- `timestamp`: 日期类型，时间戳。

### 文本类型

- `varchar (M)`: 文本类型，`M` 为0-65535之间的整数。





## 数据库基本操作

### 查看所有数据库

```mysql
show databases;
```



### 切换数据库

```mysql
use db1;
```



### 查看当前数据库

```mysql
select database()
```



### 创建数据库

```mysql
create database 数据库名称;
-- 不存在则创建
create database if not exists db1;
-- 创建的时候指定字符集
create database db1 default character set gbk;
```



### 查看某个数据库定义信息

```mysql
show create database db1;
```



### 删除数据库

```mysql
drop database db1;
```









## 数据表基本操作

### 查看所有表

```mysql
show tables;
```



### 查看表的具体信息

```mysql
desc 表名;
```



### 查看如何创建的

```mysql
show create table 表名
```



### 创建表

```mysql
 create table 表名(
         字段1 字段类型,
         字段2 字段类型,
         …
         字段n 字段类型
);
```



### 修改表属性

- 名称

	```mysql
	ALTER TABLE 【old_table_name】 RENAME TO 【new_table_name】;
	```

- 字段

	```mysql
	-- 添加字段
	ALTER TABLE 【table_name】 ADD 【column_name datatype】;
	
	-- 删除字段
	ALTER TABLE 【table_name】 DROP COLUMN 【column_name】;
	
	
	```

	

- 字段类型

	```mysql
	ALTER TABLE 【table_name】 MODIFY 【column_name】 【new_datatype】;
	```



### 删除表

```mysql
drop table 表名;
```





## 数据表的约束

### 表的约束条件

表的约束条件用于限制表中字段的数据，以保证数据的正确性和唯一性。以下是五种常见的约束条件：

- **PRIMARY KEY**: 主键约束用于唯一标识对应的记录。每个表只能有一个主键，主键列中的值必须唯一，且不能为NULL。

- **FOREIGN KEY**: 外键约束，用于建立两个表之间的链接。外键在当前表中不能有唯一值，其值必须是另一个表中主键或唯一约束的值，或者为NULL。

- **NOT NULL**: 非空约束，确保该字段的每条记录都必须含有值，即该字段不能存储NULL值。

- **UNIQUE**: 唯一性约束，保证所有记录中的该字段的每个值都必须唯一。不同的记录中可以包含NULL值。

- **DEFAULT**: 默认值约束，用于设置字段的默认值。如果在插入记录时没有指定该字段的值，则会使用默认值。

表的约束实际上就是表中数据的限制条件，它们是维护数据完整性的重要工具。





### 具体代码

- 主键约束

	```mysql
	字段名 数据类型 primary key;
	```

	```mysql
	CREATE TABLE persons (
	    firstName varchar(255),
	    lastName varchar(255),
	    address varchar(255),
	    city varchar(255)
	);
	
	ALTER TABLE persons ADD PRIMARY KEY (firstName, lastName);-- 设置主键
	```

	```mysql
	ALTER TABLE persons DROP PRIMARY KEY; -- 撤销主键约束
	```

	

- 外键约束

	```mysql
	-- 在创建数据表时语法如下：
	CONSTRAINT 外键名 FOREIGN KEY (从表外键字段) REFERENCES 主表 (主键字段)
	-- 将创建数据表创号后语法如下：
	ALTER TABLE 从表名 ADD CONSTRAINT 外键名 FOREIGN KEY (从表外键字段) REFERENCES 主表 (主键字段);
	```

	> 主表数据删除，从表数据也要删除

- 非空约束

	```mysql’
	字段名 数据类型 NOT NULL;
	```

	

- 唯一性约束

	```mysql
	字段名 数据类型 UNIQUE;
	```

	

- 默认值约束

	```mysql
	字段名 数据类型 DEFAULT 默认值；
	```

	



### 级联删除（CASCADE）

设置级联删除，如果主表中的一条记录被删除，那么从表中所有引用了该记录的行也会被自动删除。这是为了保持数据库的引用完整性，避免出现“悬挂”的外键引用，即从表中有数据引用了已经不存在的主表数据。



## 数据表内容基本操作

### 插入数据

```mysql
INSERT INTO 表名（字段名1,字段名2,...) VALUES (值 1,值 2,...);

INSERT INTO 表名 VALUES (值1，值2,...)  -- 每个字段都得填写，主键用null

-- 插入多条
INSERT INTO 表名 (字段名1,字段名2,...) VALUES (值 1,值 2,…),(值 1,值 2,…),...;
```



### 更新数据

```mysql
UPDATE 表名 SET 字段名=值, 字段名=值 WHERE 条件表达式;

-- example
update student set age=20,gender='female' where name='tom';

-- 全部更新
update student set age=18;
```





### 删除数据

```mysql
DELETE FROM 表名 WHERE 条件表达式;

-- example

-- 删除表中所有数据
delete from user;

-- 删除 uid 为1的数据
delete from user where uid = 1;
```

```mysql
truncate table 表名;
```

> 区别：
>
> delete :一条一条删除，不清空 ayto_increment 记录数。
>
> truncate :直接将表删除，重新建表，auto_increment 将置为零，从新开始



### 查看auto_increment当前值

```mysql
SELECT AUTO_INCREMENT
FROM information_schema.tables
WHERE table_name = 'your_table_name'
AND table_schema = 'your_database_name';
```



### 修改auto_increment值

```mysql
ALTER TABLE 表名 AUTO_INCREMENT=100
```





### 查询数据

#### 去重查询

```mysql
SELECT DISTINCT 列名 from 表名;-- 去重
```

#### 改字段查询

```mysql
SELECT DISTINCT pname, price + 2 from products;-- price+2 && 去重
```

#### 条件查询

```mysql
-- 条件查询
SELECT * FROM products WHERE price>40 AND price <900;

SELECT * FROM products WHERE price BETWEEN 20 AND 98;

SELECT * FROM products WHERE price NOT IN (20,38,98);
```

#### 模糊查询

```mysql
select * from products where price like '%';
-- %表示任意多个 _表示一个

-- 查找价格中包含数字"5"的产品
SELECT * FROM products WHERE CAST(price AS CHAR) LIKE '%5%'; -- 尽量不要使用，效率不高
```

#### 聚合函数

```mysql
-- 聚合查询（对单列进行分析）

select count(*) from products;
select max(price) as 商品价格最大值 from products;
select min(price) as 商品价格最小值 from products;
select sum(price) as 商品总价 from products;
select avg(price) as 平均价格 from products;
```

#### 分组查询

> 注意事项：
>
> ### 分组查询的注意事项：
>
> 1. **选择的非聚合列必须在GROUP BY子句中声明**：在`SELECT`语句中，如果你使用了聚合函数，那么除了聚合函数外的其他所有列都应该包含在`GROUP BY`子句中。这是因为非聚合列的值在分组聚合中是不明确的。
> 2. **HAVING子句的使用**：`HAVING`子句是在分组后对聚合结果应用条件的方式。它类似于`WHERE`子句，但`WHERE`子句在数据分组前过滤行，而`HAVING`子句在数据分组和聚合之后过滤组。
> 3. **性能考虑**：大量数据的分组查询可能会导致性能下降，特别是如果分组依据的列没有被索引。优化分组查询通常涉及确保适当的索引存在，以及尽量减少需要处理的数据量。
> 4. **聚合函数和NULL值**：大多数聚合函数在计算时会忽略NULL值。因此，如果分组列中包含NULL值，这些值会被视为一个单独的组。
> 5. **分组的结果**：使用`GROUP BY`时，可以把它想象成对于每个唯一的分组键（本例中的`sid`），SQL都会创建一个“虚拟的子表”（虽然这不是在物理上发生的），然后对每个这样的“子表”应用聚合函数。`GROUP BY sid`后，查询就像是对每个由`sid`定义的分组单独进行了。
> 6. **排序分组结果**：可以使用`ORDER BY`子句对分组查询的结果进行排序，但需要注意的是，排序是在所有分组完成后进行的。

```mysql
-- 分组查询
select sid as 商品分类,count(*) as 商品个数 from products group by sid;
select sid as 商品分类,avg(price) as 平均价格 from products group by sid having avg(price)>=60;
```



#### 分页查询

```mysql
SELECT * FROM 表名 LIMIT 起始位置, 记录数;   -- 一般结合排序

-- example
SELECT product_id, name, price, added_on
FROM products
ORDER BY added_on DESC              
LIMIT 0, 5;
```







## 多表操作

### 内连接查询

```mysql
SELECT 查询字段1,查询字段2, ... FROM 表1 [INNER] JOIN 表2 ON 表1.关系字段=表2.关系字段
```



### 外连接查询

```mysql
SELECT 查询字段1,查询字段2, ... FROM 表1 LEFT | RIGHT [OUTER] JOIN 表2 ON 表1.关系字段=表2.关系字段 WHERE 条件
```

**1、LEFT [OUTER] JOIN 左(外)连接：返回包括左表中的所有记录和右表中符合连接条件的记录。
2、RIGHT [OUTER] JOIN 右(外)连接：返回包括右表中的所有记录和左表中符合连接条件的记录**



## 子查询



### 带比较运算符的子查询

```mysql
select * from class where cid=(select classid from student where sname='张三');
```

### 带EXISTS关键字的子查询

EXISTS关键字后面的参数可以是任意一个子查询， 它不产生任何数据只返回TRUE或FALSE。当返回值为TRUE时外层查询才会 执行

> 假如王五同学在学生表中则从班级表查询所有班级信息 MySQL命令：

```mysql
select * from class where exists (select * from student where sname='王五');
```



### 带ANY关键字的子查询

ANY关键字表示满足其中任意一个条件就返回一个结果作为外层查询条件。

```mysql
select * from class where cid > any (select classid from student);
```



### 带ALL关键字的子查询

ALL关键字与ANY有点类似，只不过带ALL关键字的子査询返回的结果需同时满足所有内层査询条件

```mysql
select * from class where cid > all (select classid from student);
```





## 各个指令的书写顺序【重要】

1. **SELECT**
	- 指定要从数据集中选择哪些列或进行哪些计算。
2. **DISTINCT**
	- 用于返回唯一不同的值。如果使用，它紧跟在 SELECT 关键字后面。
3. **FROM**
	- 指定要从哪个表中检索数据。这是定义数据源的地方。
4. **JOIN**
	- 如果涉及多个表，使用 JOIN 来说明如何将这些表根据逻辑关系连接起来。
5. **WHERE**
	- 对从 FROM 子句指定的表中检索到的数据进行条件过滤。
6. **GROUP BY**
	- 如果需要按某一列或多列对结果集进行分组，使用 GROUP BY 子句。
7. **HAVING**
	- 用于对 GROUP BY 分组后的结果集进行条件过滤。这是唯一可以在聚合函数上应用条件的地方。
8. **ORDER BY**
	- 指定如何对查询结果进行排序（例如，按某列升序或降序）。
9. **LIMIT** / **OFFSET**
	- 用于限制返回的结果数量，或指定从哪一条记录开始返回。这在进行分页时特别有用。



## 各个指令的执行顺序【重要】

1. **FROM**子句
	- 这是第一步，数据库系统从指定的表中读取数据。如果有多个表，会进行相应的连接操作。
2. **JOIN**（如果有）
	- 如果查询涉及多个表，会在这一步根据 JOIN 条件进行表的连接。
3. **WHERE**子句
	- 接下来，数据库系统会根据 WHERE 子句中的条件过滤数据。这一步发生在任何分组（GROUP BY）之前，意味着 WHERE 条件是应用于原始数据上的。
4. **GROUP BY**子句
	- 如果查询指定了 GROUP BY，那么在 WHERE 过滤之后，下一步就是根据指定的列或表达式对结果集进行分组。
5. **HAVING**子句
	- HAVING 子句用于过滤经过 GROUP BY 分组后的数据集。这意味着，与 WHERE 不同，HAVING 是在分组后对组应用的条件。
6. **SELECT**子句
	- 这一步选择从上一步结果中需要获取的列。如果有聚合函数（如 SUM、COUNT 等），它们也会在这一步计算。
7. **DISTINCT**（如果有）
	- 如果查询指定了 DISTINCT，那么在 SELECT 后，会去除重复的行。
8. **ORDER BY**子句
	- 接着，根据 ORDER BY 子句指定的列或表达式对结果集进行排序。注意，这是在所有的选择、聚合和过滤操作完成后进行的。
9. **LIMIT**子句（或 OFFSET）
	- 最后，如果指定了 LIMIT 或 OFFSET，数据库系统会根据这些限制来限定或跳过结果集中的一部分行。





## 事务

[参考文章](https://blog.csdn.net/qq_56880706/article/details/122653735?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522170900723016800182152977%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=170900723016800182152977&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-2-122653735-null-null.142^v99^pc_search_result_base3&utm_term=mysql%E4%BA%8B%E5%8A%A1&spm=1018.2226.3001.4187)

- 原子性（Atomicity）：指事务是一个不可分割的最小工作单位，事务中的操作只有都发生和都不发生两种情况
- 一致性（Consistency）：事务必须使数据库从一个一致状态变换到另外一个一致状态，举一个栗子，李二给王五转账50元，其事务就是让李二账户上减去50元，王五账户上加上50元；一致性是指其他事务看到的情况是要么李二还没有给王五转账的状态，要么王五已经成功接收到李二的50元转账。而对于李二少了50元，王五还没加上50元这个中间状态是不可见的。
- 隔离性（Isolation）：一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰。
- 持久性（Durability）：一个事务一旦提交成功，它对数据库中数据的改变将是永久性的，接下来的其他操作或故障不应对其有任何影响。



### 事务分类

- 隐式事务：该事务没有明显的开启和结束标记，它们都具有自动提交事务的功能；不妨思考一下，update语句修改数据时，是不是对表中数据进行改变了，它的本质其实就相当于一个事务
- 显示事务：该事务具有明显的开启和结束标记；也是本文重点要讲的东西。使用显式事务的前提是你得先把自动提交事务的功能给禁用。禁用自动提交功能就是设置`autocommit`变量值为0（0:禁用 1:开启）



### 查看是否自动提交

```mysql
select @@autocommit;  -- 为0为禁用
```



### 开启事务

```mysql
#步骤一：开启事务（可选）
start transaction;
#步骤二：编写事务中的sql语句（insert、update、delete）
#这里实现一下"李二给王五转账"的事务过程
update t_account set balance = 50 where vname = "李二";
update t_account set balance = 130 where vname = "王五";
#步骤三：结束事务
commit; #提交事务
# rollback; #回滚事务:就是事务不执行，回滚到事务执行前的状态
```



