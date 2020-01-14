---
title: MySQL高级-函数
date: 2018-11-12 20:12:37
tags: MySQL function
categories: 数据库 MySQL
---
# MySQL高级-函数

@(mysql)[procedure]

## **查看数据库是否支持函数** ：
``` SQL
SHOW VARIABLES LIKE '%FUN%';
```
- **若为off，则需要开启**
``` SQL
SET GLOBAL log_bin_trust_function_creators = TRUE;
```

### **创建函数的语法** ：
``` SQL
CREATE FUNCTION func_name(variable1, variable2 ...)
RETURNS return_type
BEGIN
	...
END;

DELIMITER //
CREATE FUNCTION func_add(a INT, b INT)
RETURNS INT
BEGIN
	RETURN a + b;
END;
//
```
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-12/68702030.jpg)

##  **函数的管理** ：
- **函数的管理类似于存储过程**
### **查看函数**
``` SQL
SELECT * FROM mysql.proc;
SHOW FUNCTION STATUS WHERE db = 'test';
```
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-12/68702030.jpg)
### **删除函数**
``` SQL
DROP FUNCTION IF EXISTS func_add;
```
## 实例：

``` SQL
CREATE TABLE user_info (id INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
						user_name VARCHAR(20) NOT NULL DEFAULT '',
						user_pwd  VARCHAR(20) NOT NULL DEFAULT ''
						)ENGINE = INNODB CHARSET = utf8;
						 
CREATE FUNCTION func_login(name VARCHAR(20), pwd VARCHAR(20))
RETURNS VARCHAR(30)
BEGIN
	DECLARE cnt INT DEFAULT 0;
	SELECT count(*) INTO cnt FROM user_info WHERE user_name = name;
	IF cnt = 0 THEN
		RETURN 'user not exists';
	END IF;
	SELECT count(*) INTO cnt FROM user_info WHERE user_name = name AND user_pwd = pwd;
	IF cnt = 0 THEN
		RETURN 'pwd input error';
	ELSE
		RETURN 'user login success';
	END IF;
END;
//
``` 
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-12/35546849.jpg)

