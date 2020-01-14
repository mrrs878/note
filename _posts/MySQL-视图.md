---
title: MySQL-视图
date: 2018-11-13 16:56:59
tags: MySQL VIEW
categories: 数据库 MySQL
---
# MySQL-视图
## 什么是视图
- 视图是由查询结果形成的一张**虚拟表**，可以当作一张表使用，但与持久表(permanent table)不同的是，视图中的数据**没有实际的物理存储**。

## 视图的作用
- 视图在一定程度上起到一个**安全层**的作用，可以进行**权限控制**
- **简化查询**语句
- 对于某张很大的表，可以将其分为多个视图，从而**加快查询**

## 视图的使用条件
- 如果某个查询结果出现的非常频繁，就是要经常那这个查询结果来做**子查询**，使用视图会更加方便

## 视图的基本操作
- **创建视图**：
``` SQL
CREATE
[OR REPLACE]
[ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
[DEFINER = { user | CURRENT_USER }]
[SQL SECURITY { DEFINER | INVOKER }]
VIEW view_name [(column_list)]
AS select_statement
[WITH [CASCADED | LOCAL] CHECK OPTION]

CREATE OR REPLACE VIEW view_name AS SELECT * FROM table_name
```
- **调取视图**
``` SQL
SELECT * FROM view_name
```
- **修改视图**
``` SQL
ALTER
[ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
[DEFINER = { user | CURRENT_USER }]
[SQL SECURITY { DEFINER | INVOKER }]
VIEW view_name [(column_list)]
AS select_statement
[WITH [CASCADED | LOCAL] CHECK OPTION]

ALTER OR REPLACE VIEW view_name AS SELECT * FROM table_name
```
- **删除视图**
``` SQL
DROP VIEW IF EXISTS view_name
```
- **查看视图**
``` SQL
SHOW TABLES;
```
- **查看视图的定义**
``` SQL
SHOW TABLE STATUS FROM table_name LIKE 'view_name'
```
## 视图的算法--存在两种执行的算法
- MERGE：**合并模式**，先将我们视图的SQL语句与外部查询视图的SQL语句，混合在一起，最终执行，也就是可以**更新**真实表中的数据。
- TEMPTABLE：**临时表模式**，每当查询的时候，将视图所使用的SELECT语句生成一个结果的临时表，再在当前的临时表内进行查询。
- UNDEFINED： 让MySQL选择所要使用的算法。如果可能，它**倾向于MERGE**而不是TEMPTABLE，这是因为MERGE通常更有效，而且如果使用了临时表，视图是不可更新。

## 视图与表的关系
- 虽然视图是基于表的一个虚拟表，但是用户可以对某些视图进行更新操作，其本质就是通过视图的定义来更新基表。一般称进行更新操作的视图为可更新视图(updatable view)。视图定义中的**WITH CHECK OPTION**就是针对于可更新表的。

## WITH CHECK OPTION测试代码
``` SQL
# 创建视图（WITHOUT CHECK OPTION）
CREATE OR REPLACE VIEW user_info_v AS SELECT * FROM user_info WHERE id < 10
# 查看视图（基表 + 视图）
SHOW TABLES
# 只查看基表
SELECT * FROM information_schema.TABLES WHERE table_type = 'BASE TABLE' AND table_schema = database()\G
INSERT INTO user_info_v SELECT 12, 'lisi', '5678'
INSERT INTO user_info_v SELECT 2, 'wangwu', '9012'
SELECT * FROM user_info
SELECT * FROM user_info_v

# 删除视图
DROP VIEW IF EXISTS user_info_v;

# 创建视图（WITH CHECK OPTION）
CREATE OR REPLACE VIEW user_info_v AS SELECT * FROM user_info WHERE id < 10 WITH CHECK OPTION
INSERT INTO user_info_v SELECT 13, 'zhaoliu', '3456'
```
![enter image description here](https://t1.picb.cc/uploads/2018/11/13/JSO1Dr.png)
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-13/37805165.jpg)
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-13/67280010.jpg)
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-13/47500056.jpg)

## 视图的作用测试代码
``` SQL
DELIMITER //
CREATE PROCEDURE view_test(_cnt INT UNSIGNED)
BEGIN
	SET @cnt = 0;
	WHILE @cnt < _cnt DO
		INSERT INTO user_info(user_name, user_pwd) values(REPEAT(CHAR(97+RAND()*26), 20), REPEAT(CHAR(97+RAND()*26), 20));
		SET @cnt = @cnt + 1;
	END WHILE;
END;
//
DELIMITER ;

call view_test(10000);
call view_test(50000);
call view_test(50000);

CREATE OR REPLACE VIEW view_func_test0 AS SELECT * FROM user_info WHERE id % 4 = 0 WITH CHECK OPTION;
CREATE OR REPLACE VIEW view_func_test1 AS SELECT * FROM user_info WHERE id % 4 = 1 WITH CHECK OPTION;
CREATE OR REPLACE VIEW view_func_test2 AS SELECT * FROM user_info WHERE id % 4 = 2 WITH CHECK OPTION;
CREATE OR REPLACE VIEW view_func_test3 AS SELECT * FROM user_info WHERE id % 4 = 3 WITH CHECK OPTION;
```
![enter image description here](https://t1.picb.cc/uploads/2018/11/13/JSWxYv.png)



