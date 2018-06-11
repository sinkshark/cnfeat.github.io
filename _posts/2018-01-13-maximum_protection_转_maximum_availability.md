---
layout: post
title:  maximum protection 转 maximum availability
date: 2018-01-13
tag: oracle
---

**PS：该实验建立在之前DG博客的基础之上的**

### 主库之前的状态，执行转换语句，主库之后的状态
```
15:00:01 SYS @ slow >select name,database_role,protection_mode from v$database;

NAME     DATABASE_ROLE PROTECTION_MODE
--------- ---------------- --------------------
SLOW     PRIMARY     MAXIMUM PROTECTION

1 row selected.

Elapsed: 00:00:00.00
15:00:35 SYS @ slow >select status from v$instance;

STATUS
------------
OPEN

1 row selected.

Elapsed: 00:00:00.00
18:57:24 SYS @ slow >startup mount force;
ORACLE instance started.

Total System Global Area 521936896 bytes
Fixed Size         2254824 bytes
Variable Size         377489432 bytes
Database Buffers     138412032 bytes
Redo Buffers         3780608 bytes
Database mounted.
18:57:44 SYS @ slow >select name,database_role,protection_mode from v$database;

NAME     DATABASE_ROLE PROTECTION_MODE
--------- ---------------- --------------------
SLOW     PRIMARY     MAXIMUM PROTECTION

1 row selected.

Elapsed: 00:00:00.01
18:58:20 SYS @ slow >select status from v$instance;

STATUS
------------
MOUNTED

1 row selected.

Elapsed: 00:00:00.01
18:58:39 SYS @ slow >alter database set standby database to maximize availability; 

Database altered.

Elapsed: 00:00:00.00
19:00:09 SYS @ slow >alter database open;

Database altered.

Elapsed: 00:00:03.03
19:00:25 SYS @ slow >select name,database_role,protection_mode from v$database;

NAME     DATABASE_ROLE PROTECTION_MODE
--------- ---------------- --------------------
SLOW     PRIMARY     MAXIMUM AVAILABILITY

1 row selected.

Elapsed: 00:00:00.00
19:01:20 SYS @ slow >
```
### 备库之前的状态，主库执行转换语句，备库之后的状态  ，，，一切正常，转换成功！！！
```
14:56:33 SYS @ gotime >select name,database_role,protection_mode from v$database;

NAME     DATABASE_ROLE PROTECTION_MODE
--------- ---------------- --------------------
SLOW     PHYSICAL STANDBY MAXIMUM PROTECTION

1 row selected.

Elapsed: 00:00:00.00
14:57:21 SYS @ gotime >select status from v$instance;

STATUS
------------
OPEN

1 row selected.

Elapsed: 00:00:00.00
18:56:45 SYS @ gotime >startup mount force;
ORACLE instance started.

Total System Global Area 521936896 bytes
Fixed Size         2254824 bytes
Variable Size         377489432 bytes
Database Buffers     138412032 bytes
Redo Buffers         3780608 bytes
Database mounted.
18:57:07 SYS @ gotime >select name,database_role,protection_mode from v$database;

NAME     DATABASE_ROLE PROTECTION_MODE
--------- ---------------- --------------------
SLOW     PHYSICAL STANDBY MAXIMUM AVAILABILITY

1 row selected.

Elapsed: 00:00:00.04
19:02:08 SYS @ gotime >
```

<br>

转载请注明:[sinkshark的博客](http://www.sinkshark.com/)