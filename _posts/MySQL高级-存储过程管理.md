---
title: MySQL高级-存储过程管理
date: 2018-11-12 17:07:28
tags: MySQL procedure
categories: 数据库 MySQL
---
# MySQL高级-存储过程管理

@(mysql)[procedure]

- **查看数据库的存储过程** ：
``` SQL
show procedure status where db='test'\G
```
- ![enter image description here](https://i.loli.net/2018/11/12/5be94f5043e82.png)

- **查看当前数据库下的存储过程** ：
``` SQL
select specific_name from mysql.proc;\G
```
- ![enter image description here](https://i.loli.net/2018/11/12/5be94ee471819.png)

- **查看存储过程的内容** ：
``` SQL
select specific_name, body from mysql.proc\G
```
- ![enter image description here](https://i.loli.net/2018/11/12/5be94f5050259.png)

- **查看某个存储过程的内容**：
``` SQL
select specific_name, body from mysql.proc where specific_name = 'load_t'\G
```
- ![enter image description here](https://i.loli.net/2018/11/12/5be94f504e883.png)
- **删除某个存储过程**
``` SQL
drop procedure if exists load_t;
```
- ![enter image description here](https://i.loli.net/2018/11/12/5be94f504cd4b.png)


