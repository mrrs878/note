---
title: MySQL-触发器
date: 2018-11-13 22:51:06
tags: MySQL trigger
categories: 数据库 MySQL
---
# MySQL-触发器
## 什么是触发器
- 触发器是一种**特殊的存储过程**，它在插入/删除或修改表中的数据时**触发执行**，它比数据库本身有着更精细和更复杂的数据约束力。

## 触发器的作用
- 用于实现完整性约束
- 通过触发器，用户可以实现MySQL数据库本身并不支持的一些特性，如对于传统CHECK约束的支持，物化视图，高级复制，审计等特性

## 触发器的特性
- 监视地点：一般就是某一张表
- 监视/触发事件：UPDATE/INSERT/DELETE
- 触发时间：AFTER/BEFORE

## 触发器的创建语法
```SQL
CREATE
[DEFINER = { user | CURRENT_USER }]
TRIGGER trigger_name
trigger_time trigger_event
ON tbl_name FOR EACH ROW
trigger_body

trigger_time: { BEFORE | AFTER }
trigger_event: { INSERT | UPDATE | DELETE }
```
- 触发程序是与表有关的数据库对象，当表上出现特定事件时将激活该对象。触发程序与tbl_name相关，tbl_name必须引用永久性表，不能将触发程序与TEMPORARY表或视图关联起来。
- 对于具有相同触发程序动作事件和时间的给定表，不能有两个触发程序。
- trigger_body是当触发程序激活时执行的语句。可以使用BEGIN...END复合语句结构。
- 对于INSERT而言，新插入的行用NEW表示，行中的每一列值用NEW.col_name表示。
- 对于DELETE而言，欲删除的那一行用OLD表示，行中的每一列值用OLD.col_name表示。
- 对于UPDATE而言，修改前的行用OLD表示，修改后的行用NEW表示。

## 触发器的管理
- 查看所有触发器
``` SQL
SHOW TRIGGERS [FROM db_name] [like_or_where]
```
- MySQL中有一个information_schema.TRIGGERS表，存储所有库中的触发器，查看：
``` SQL
DESC information_schema.TRIGGERS
```
- 删除触发器
``` SQL
DROP VIEW [IF EXISTS]
view_name [, view_name] ...
[RESTRICT | CASCADE]
```

## 触发器的使用示例
### 基本使用
``` SQL
DELIMITER //
CREATE TRIGGER user_trg
AFTER INSERT
ON user_info FOR EACH ROW
BEGIN
   INSERT INTO user_salaries SELECT new.id, new.user_name, 0;
END;
//
DELIMITER;

INSERT INTO user_info SELECT 1, '张三'， '123456';
```
![enter image description here](https://t1.picb.cc/uploads/2018/11/13/JS7x7L.png)
### 对于约束的支持
```SQL
CREATE TABLE userCash(id INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY, 
					   cash INT UNSIGNED NOT NULL DEFAULT 0
)ENGINE = INNODB CHARSET = 'UTF8';
CREATE TABLE userCashErr_log (
	id INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
	old_cash INT UNSIGNED NOT NULL,
	new_cash INT UNSIGNED NOT NULL,
	user_name VARCHAR(20),
	time DATETIME
)ENGINE = INNODB CHARSET = 'UTF8';

DELIMITER //
CREATE TRIGGER userCashUpdate_tgr 
BEFORE UPDATE
ON userCash FOR EACH ROW
BEGIN
	IF new.cash - old.cash > 0 THEN
		INSERT INTO userCashErr_log SELECT old.id, old.cash, new.cash, USER(), NOW();
		SET new.cash = old.cash;
	END IF;
END;
//
DELIMITER ;

INSERT INTO userCash SELECT 1, 10000;
SELECT * FROM userCashErr_log;
UPDATE userCash SET cash = cash - (-20) WHERE id = 1;
SELECT * FROM userCashErr_log;
```
![enter image description here](https://t1.picb.cc/uploads/2018/11/13/JS7T9W.png)


