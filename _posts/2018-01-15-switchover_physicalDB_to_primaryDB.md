---
layout: post
title: "switchover_physicalDB_TO_primaryDB"
date: 2018-01-15 
description: "DG"
tag: oracle
---

> 这一些实验是建立在我之前博客实验的基础上的
>为什么要先alter 先切主库  再切备库，
>因为如果先切备库的话，那么主库的一些日志可能备库收不到，导致不一致的情况发生

### 先切主库------>standby DB 切换之后主库是断开close的，重新open后查看其状态
```
20:53:33 SYS @ slow >select name,database_role,protection_mode,switchover_status from v$database;

NAME     DATABASE_ROLE PROTECTION_MODE    SWITCHOVER_STATUS
--------- ---------------- -------------------- --------------------
SLOW     PRIMARY     MAXIMUM AVAILABILITY TO STANDBY

1 row selected.

Elapsed: 00:00:00.00
20:55:15 SYS @ slow >edit
Wrote file afiedt.buf

  1* select username,sid,serial# from v$session where username is not null
20:55:24 SYS @ slow >r
  1* select username,sid,serial# from v$session where username is not null

USERNAME             SID SERIAL#
------------------------------ ---------- ----------
SYS                    1     5

1 row selected.

Elapsed: 00:00:00.01
20:55:25 SYS @ slow >r 
  1* select username,sid,serial# from v$session where username is not null

USERNAME             SID SERIAL#
------------------------------ ---------- ----------
SYS                    1     5
SCOTT                 41     47

2 rows selected.

Elapsed: 00:00:00.00
20:56:25 SYS @ slow >alter database commit to switchover to standby;

Database altered.

Elapsed: 00:00:01.89
20:57:05 SYS @ slow >select name,database_role,protection_mode,switchover_status from v$database;
select name,database_role,protection_mode,switchover_status from v$database
*
ERROR at line 1:
ORA-01034: ORACLE not available
Process ID: 5713
Session ID: 1 Serial number: 5


Elapsed: 00:00:00.00
20:58:08 SYS @ slow >select status from v$instance;
select status from v$instance
*
ERROR at line 1:
ORA-01034: ORACLE not available
Process ID: 5713
Session ID: 1 Serial number: 5


Elapsed: 00:00:00.00
20:58:46 SYS @ slow >startup 
ORACLE instance started.

Total System Global Area 521936896 bytes
Fixed Size         2254824 bytes
Variable Size         377489432 bytes
Database Buffers     138412032 bytes
Redo Buffers         3780608 bytes
Database mounted.
Database opened.
21:07:19 SYS @ slow >select name,open_mode,database_role,protection_mode,switchover_status from v$database;

NAME     OPEN_MODE     DATABASE_ROLE    PROTECTION_MODE SWITCHOVER_STATUS
--------- -------------------- ---------------- -------------------- --------------------
SLOW     READ ONLY     PHYSICAL STANDBY MAXIMUM AVAILABILITY TO PRIMARY

1 row selected.

Elapsed: 00:00:00.01
21:07:55 SYS @ slow >recover managed standby database disconnect;
Media recovery complete.
21:08:35 SYS @ slow >select name,open_mode,database_role,protection_mode,switchover_status from v$database;

NAME     OPEN_MODE     DATABASE_ROLE    PROTECTION_MODE SWITCHOVER_STATUS
--------- -------------------- ---------------- -------------------- --------------------
SLOW     READ ONLY WITH APPLY PHYSICAL STANDBY MAXIMUM AVAILABILITY TO PRIMARY

1 row selected.

Elapsed: 00:00:00.00
21:09:02 SYS @ slow >
```
### 再切换备库----->primary DB  备库mount状态下操作  成功！！！
```
20:52:21 SYS @ gotime >select name,database_role,protection_mode,switchover_status from v$database;

NAME     DATABASE_ROLE PROTECTION_MODE    SWITCHOVER_STATUS
--------- ---------------- -------------------- --------------------
SLOW     PHYSICAL STANDBY MAXIMUM AVAILABILITY NOT ALLOWED

1 row selected.

Elapsed: 00:00:00.01
20:53:12 SYS @ gotime >r
  1* select name,database_role,protection_mode,switchover_status from v$database

NAME     DATABASE_ROLE PROTECTION_MODE    SWITCHOVER_STATUS
--------- ---------------- -------------------- --------------------
SLOW     PHYSICAL STANDBY MAXIMUM AVAILABILITY NOT ALLOWED

1 row selected.

Elapsed: 00:00:00.00
20:59:16 SYS @ gotime >select username,sid from v$session where username is not null;

USERNAME             SID
------------------------------ ----------
SYS                 24

1 row selected.

Elapsed: 00:00:00.01
21:02:26 SYS @ gotime >alter database commit to switchover to primary with session shutdown;
alter database commit to switchover to primary with session shutdown
*
ERROR at line 1:
ORA-16139: media recovery required


Elapsed: 00:00:00.01
21:03:18 SYS @ gotime >recover managed standby database disconnect;
Media recovery complete.
21:03:51 SYS @ gotime >alter database commit to switchover to primary with session shutdown;

Database altered.

Elapsed: 00:00:02.04
21:05:00 SYS @ gotime >select name,database_role,protection_mode,switchover_status from v$database;

NAME     DATABASE_ROLE PROTECTION_MODE    SWITCHOVER_STATUS
--------- ---------------- -------------------- --------------------
SLOW     PRIMARY     MAXIMUM AVAILABILITY NOT ALLOWED

1 row selected.

Elapsed: 00:00:00.01
21:05:12 SYS @ gotime >alter database open;

Database altered.

Elapsed: 00:00:00.58
21:05:45 SYS @ gotime >select name,database_role,protection_mode,switchover_status from v$database;

NAME     DATABASE_ROLE PROTECTION_MODE    SWITCHOVER_STATUS
--------- ---------------- -------------------- --------------------
SLOW     PRIMARY     MAXIMUM AVAILABILITY FAILED DESTINATION

1 row selected.

Elapsed: 00:00:00.01
21:06:13 SYS @ gotime >
```
==但是，switchover_status是failed destination 状态==

### switchover之后的备库slow，重新启动listener
```
[oracle@slow ~]$ lsnrctl stop

LSNRCTL for Linux: Version 11.2.0.4.0 - Production on 13-JAN-2018 21:31:12

Copyright (c) 1991, 2013, Oracle. All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=slow)(PORT=1521)))
TNS-12541: TNS:no listener
 TNS-12560: TNS:protocol adapter error
  TNS-00511: No listener
   Linux Error: 111: Connection refused
[oracle@slow ~]$ lsnrctl start

LSNRCTL for Linux: Version 11.2.0.4.0 - Production on 13-JAN-2018 21:31:20

Copyright (c) 1991, 2013, Oracle. All rights reserved.

Starting /u01/app/oracle/product/11.2.0/dbhome_1/bin/tnslsnr: please wait...

TNSLSNR for Linux: Version 11.2.0.4.0 - Production
System parameter file is /u01/app/oracle/product/11.2.0/dbhome_1/network/admin/listener.ora
Log messages written to /u01/app/oracle/diag/tnslsnr/slow/listener/alert/log.xml
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=slow)(PORT=1521)))

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=slow)(PORT=1521)))
STATUS of the LISTENER
------------------------
Alias LISTENER
Version TNSLSNR for Linux: Version 11.2.0.4.0 - Production
Start Date 13-JAN-2018 21:31:20
Uptime 0 days 0 hr. 0 min. 0 sec
Trace Level off
Security ON: Local OS Authentication
SNMP OFF
Listener Parameter File /u01/app/oracle/product/11.2.0/dbhome_1/network/admin/listener.ora
Listener Log File /u01/app/oracle/diag/tnslsnr/slow/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=slow)(PORT=1521)))
The listener supports no services
The command completed successfully
[oracle@slow ~]$ lsnrctl status

LSNRCTL for Linux: Version 11.2.0.4.0 - Production on 13-JAN-2018 21:31:23

Copyright (c) 1991, 2013, Oracle. All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=slow)(PORT=1521)))
STATUS of the LISTENER
------------------------
Alias LISTENER
Version TNSLSNR for Linux: Version 11.2.0.4.0 - Production
Start Date 13-JAN-2018 21:31:20
Uptime 0 days 0 hr. 0 min. 2 sec
Trace Level off
Security ON: Local OS Authentication
SNMP OFF
Listener Parameter File /u01/app/oracle/product/11.2.0/dbhome_1/network/admin/listener.ora
Listener Log File /u01/app/oracle/diag/tnslsnr/slow/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=slow)(PORT=1521)))
The listener supports no services
The command completed successfully
```
### 关闭slow备库，再打开mount状态
```
21:28:36 SYS @ slow >shutdown immediate;
Database closed.
Database dismounted.
ORACLE instance shut down.
21:28:53 SYS @ slow >startup mount;
ORACLE instance started.

Total System Global Area 521936896 bytes
Fixed Size         2254824 bytes
Variable Size         377489432 bytes
Database Buffers     138412032 bytes
Redo Buffers         3780608 bytes
Database mounted.
21:29:08 SYS @ slow >
```
### 等slow备库重新启动之后，状态为ready了
```
[oracle@slow ~]$ lsnrctl status

LSNRCTL for Linux: Version 11.2.0.4.0 - Production on 13-JAN-2018 21:32:05

Copyright (c) 1991, 2013, Oracle. All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=slow)(PORT=1521)))
STATUS of the LISTENER
------------------------
Alias LISTENER
Version TNSLSNR for Linux: Version 11.2.0.4.0 - Production
Start Date 13-JAN-2018 21:31:20
Uptime 0 days 0 hr. 0 min. 44 sec
Trace Level off
Security ON: Local OS Authentication
SNMP OFF
Listener Parameter File /u01/app/oracle/product/11.2.0/dbhome_1/network/admin/listener.ora
Listener Log File /u01/app/oracle/diag/tnslsnr/slow/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=slow)(PORT=1521)))
Services Summary...
Service "slow" has 1 instance(s).
  Instance "slow", status READY, has 1 handler(s) for this service...
The command completed successfully
[oracle@slow ~]$
```
### 现在的主库gotime这边重启listener
```
[grid@sink ~]$ lsnrctl stop

LSNRCTL for Linux: Version 11.2.0.4.0 - Production on 13-JAN-2018 21:31:41

Copyright (c) 1991, 2013, Oracle. All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=sink)(PORT=1521)))
The command completed successfully
[grid@sink ~]$ lsnrctl start

LSNRCTL for Linux: Version 11.2.0.4.0 - Production on 13-JAN-2018 21:31:46

Copyright (c) 1991, 2013, Oracle. All rights reserved.

Starting /u01/11.2.0/grid/bin/tnslsnr: please wait...

TNSLSNR for Linux: Version 11.2.0.4.0 - Production
System parameter file is /u01/11.2.0/grid/network/admin/listener.ora
Log messages written to /u01/app/grid/diag/tnslsnr/sink/listener/alert/log.xml
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=sink)(PORT=1521)))

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=sink)(PORT=1521)))
STATUS of the LISTENER
------------------------
Alias LISTENER
Version TNSLSNR for Linux: Version 11.2.0.4.0 - Production
Start Date 13-JAN-2018 21:31:46
Uptime 0 days 0 hr. 0 min. 0 sec
Trace Level off
Security ON: Local OS Authentication
SNMP OFF
Listener Parameter File /u01/11.2.0/grid/network/admin/listener.ora
Listener Log File /u01/app/grid/diag/tnslsnr/sink/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=sink)(PORT=1521)))
Services Summary...
Service "gotime" has 1 instance(s).
  Instance "gotime", status UNKNOWN, has 1 handler(s) for this service...
The command completed successfully
[grid@sink ~]$
```
### gotime主库重新启动，到open
```
21:32:12 SYS @ gotime >startup force;
ORACLE instance started.

Total System Global Area 521936896 bytes
Fixed Size         2254824 bytes
Variable Size         377489432 bytes
Database Buffers     138412032 bytes
Redo Buffers         3780608 bytes
Database mounted.
Database opened.
21:32:35 SYS @ gotime >
```
### switchover之后的备库slow，查询其状态，not allowed，正常了！！
```
21:29:08 SYS @ slow >recover managed standby database disconnect;
Media recovery complete.
21:29:32 SYS @ slow >select name,open_mode,database_role,protection_mode,switchover_status from v$database;

NAME     OPEN_MODE     DATABASE_ROLE    PROTECTION_MODE SWITCHOVER_STATUS
--------- -------------------- ---------------- -------------------- --------------------
SLOW     MOUNTED     PHYSICAL STANDBY MAXIMUM AVAILABILITY NOT ALLOWED

1 row selected.

Elapsed: 00:00:00.01
21:33:16 SYS @ slow >
```
### 在查询状态，为to standby了，正常了！！
```
21:32:35 SYS @ gotime >select name,open_mode,database_role,protection_mode,switchover_status from v$database
21:32:45 2 ;

NAME     OPEN_MODE     DATABASE_ROLE    PROTECTION_MODE SWITCHOVER_STATUS
--------- -------------------- ---------------- -------------------- --------------------
SLOW     READ WRITE     PRIMARY        MAXIMUM AVAILABILITY TO STANDBY

1 row selected.

Elapsed: 00:00:00.02
21:32:47 SYS @ gotime >
```
==到这里就成功了！！！==

<br>
转载请注明:[sinkshark的博客](https://sinkshark.com/)