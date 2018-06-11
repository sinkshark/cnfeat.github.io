---
layout: post
title: maximum performance 转换 maximum protection
date: 2018-01-13
tag: oracle
---

- 这部分需要添加standby redo log，添加日志相关操作请参照与DG physical DB 转 snapshot DB第一部分，
- 以下内容摘自physical DB 转 snapshot DB第一部分

### 在备库设置快速恢复区，大小，路径，创建4组standby redo log
```
[root@sink ~]# su - oracle
[oracle@sink ~]$ echo $ORACLE_SID
sink
[oracle@sink ~]$ export ORACLE_SID=gotime
[oracle@sink ~]$ echo $ORACLE_SID
gotime
[oracle@sink ~]$ !sql
sqlplus / as sysdba
SQL*Plus: Release 11.2.0.4.0 Production on Fri Jan 12 10:25:51 2018
Copyright (c) 1982, 2013, Oracle. All rights reserved.
Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
10:25:51 SYS @ gotime >select status from v$instance;

STATUS
------------
MOUNTED

1 row selected.

Elapsed: 00:00:00.01
10:26:03 SYS @ gotime >select name,database_role,protection_mode,open_mode from v$database;
NAME     DATABASE_ROLE PROTECTION_MODE    OPEN_MODE
--------- ---------------- -------------------- --------------------
SLOW     PHYSICAL STANDBY MAXIMUM PERFORMANCE    MOUNTED
1 row selected.
Elapsed: 00:00:00.02
10:26:54 SYS @ gotime >show parameter recover
NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
db_recovery_file_dest         string
db_recovery_file_dest_size     big integer 0
db_unrecoverable_scn_tracking     boolean     TRUE
recovery_parallelism         integer     0
10:29:42 SYS @ gotime >recover managed standby database cancel;
ORA-16136: Managed Standby Recovery not active
10:30:31 SYS @ gotime >alter system set db_recovery_file_dest_size=4g;
System altered.
Elapsed: 00:00:00.00
10:32:25 SYS @ gotime >edit
Wrote file afiedt.buf
  1* alter system set db_recovery_file_dest='/dsk1'
10:32:36 SYS @ gotime >r
  1* alter system set db_recovery_file_dest='/dsk1'
System altered.
Elapsed: 00:00:00.00
10:32:37 SYS @ gotime >show parameter recover;
NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
db_recovery_file_dest         string     /dsk1
db_recovery_file_dest_size     big integer 4G
db_unrecoverable_scn_tracking     boolean     TRUE
recovery_parallelism         integer     0
10:39:51 SYS @ gotime >edit
Wrote file afiedt.buf
  1* alter system set db_recovery_file_dest_size=6g
10:40:14 SYS @ gotime >r
  1* alter system set db_recovery_file_dest_size=6g
System altered.
Elapsed: 00:00:00.00
10:40:15 SYS @ gotime >alter database convert to snapshot standby;
alter database convert to snapshot standby
*
ERROR at line 1:
ORA-38784: Cannot create restore point 'SNAPSHOT_STANDBY_REQUIRED_01/12/2018 10:40:26'.
ORA-38788: More standby database recovery is needed
Elapsed: 00:00:00.00
==============意思是提示备库没有创建standby redo log所以下面创建4组=============
13:16:50 SYS @ gotime >alter system set db_recovery_file_dest_size=4g;
System altered.
Elapsed: 00:00:00.01
13:18:31 SYS @ gotime >edit
Wrote file afiedt.buf
  1* select group#,thread#,sequence#,archived,status from v$standby_log
13:18:55 SYS @ gotime >r
  1* select group#,thread#,sequence#,archived,status from v$standby_log
no rows selected
Elapsed: 00:00:00.00
13:18:55 SYS @ gotime >select member from v$logfile;
MEMBER
------------------------
/u01/app/oracle/oradata/gotime/redo01a.log
/u01/app/oracle/oradata/gotime/redo01b.log
/u01/app/oracle/oradata/gotime/redo02a.log
/u01/app/oracle/oradata/gotime/redo02b.log
/u01/app/oracle/oradata/gotime/redo03a.log
/u01/app/oracle/oradata/gotime/redo03b.log
6 rows selected.
Elapsed: 00:00:00.01
```
### 备库上依据redo log的路径建立4组standby redo log
```
13:21:44 SYS @ gotime >edit
Wrote file afiedt.buf
  1 alter database add standby logfile group 4
  2* ('/u01/app/oracle/oradata/gotime/&a') size 200m
13:22:33 SYS @ gotime >r
  1 alter database add standby logfile group 4
  2* ('/u01/app/oracle/oradata/gotime/&a') size 200m
Enter value for a: standby04.log
old 2: ('/u01/app/oracle/oradata/gotime/&a') size 200m
new 2: ('/u01/app/oracle/oradata/gotime/standby04.log') size 200m
Database altered.
Elapsed: 00:00:01.74
13:23:02 SYS @ gotime >edit
Wrote file afiedt.buf
  1* alter database add standby logfile group 4
13:23:12 SYS @ gotime >edit
Wrote file afiedt.buf
  1* alter database add standby logfile group &a ('/u01/app/oracle/oradata/gotime/&b') size 200m
13:23:58 SYS @ gotime >r
  1* alter database add standby logfile group &a ('/u01/app/oracle/oradata/gotime/&b') size 200m
Enter value for a: 5
Enter value for b: standby05.log
old 1: alter database add standby logfile group &a ('/u01/app/oracle/oradata/gotime/&b')size 200m
new 1: alter database add standby logfile group 5 ('/u01/app/oracle/oradata/gotime/standby05.log') size 200m
Database altered.
Elapsed: 00:00:02.02
13:24:16 SYS @ gotime >r
  1* alter database add standby logfile group &a ('/u01/app/oracle/oradata/gotime/&b') size 200m
Enter value for a: 6
Enter value for b: standby06.log
old 1: alter database add standby logfile group &a ('/u01/app/oracle/oradata/gotime/&b')size 200m
new 1: alter database add standby logfile group 6 ('/u01/app/oracle/oradata/gotime/standby06.log') size 200m

Database altered.

Elapsed: 00:00:01.98
13:24:35 SYS @ gotime >r
  1* alter database add standby logfile group &a ('/u01/app/oracle/oradata/gotime/&b') size 200m
Enter value for a: 7
Enter value for b: standby07.log
old 1: alter database add standby logfile group &a ('/u01/app/oracle/oradata/gotime/&b')size 200m
new 1: alter database add standby logfile group 7 ('/u01/app/oracle/oradata/gotime/standby07.log') size 200m

Database altered.

Elapsed: 00:00:01.69
13:24:50 SYS @ gotime >select group#,thread#,sequence#,archived,status from v$standby_log
13:25:15 2 ;

    GROUP# THREAD# SEQUENCE# ARC STATUS
---------- ---------- ---------- --- ----------
     4 0 0 YES UNASSIGNED
     5 0 0 YES UNASSIGNED
     6 0 0 YES UNASSIGNED
     7 0 0 YES UNASSIGNED

4 rows selected.

Elapsed: 00:00:00.00
13:25:16 SYS @ gotime >
```
### 打开主库，设置快速恢复区大小，路径，
```
10:24:42 SYS @ slow >startup
ORACLE instance started.

Total System Global Area 521936896 bytes
Fixed Size         2254824 bytes
Variable Size         377489432 bytes
Database Buffers     138412032 bytes
Redo Buffers         3780608 bytes
Database mounted.
Database opened.
10:24:57 SYS @ slow >select name,database_role,protection_mode,open_mode from v$database;

NAME     DATABASE_ROLE PROTECTION_MODE    OPEN_MODE
--------- ---------------- -------------------- --------------------
SLOW     PRIMARY     MAXIMUM PERFORMANCE    READ WRITE

1 row selected.

Elapsed: 00:00:00.02
10:28:35 SYS @ slow >show parameter recover;

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
db_recovery_file_dest         string
db_recovery_file_dest_size     big integer 0
db_unrecoverable_scn_tracking     boolean     TRUE
recovery_parallelism         integer     0
10:37:21 SYS @ slow >alter system set db_recovery_file_dest_size=4g;

System altered.

Elapsed: 00:00:00.00
10:37:54 SYS @ slow >alter system set db_recovery_file_dest='/dsk1';

System altered.

Elapsed: 00:00:00.01
10:38:19 SYS @ slow >select member from v$logfile;

MEMBER
-----------------------------------------
/u01/app/oracle/oradata/slow/redo01a.log
/u01/app/oracle/oradata/slow/redo01b.log
/u01/app/oracle/oradata/slow/redo02a.log
/u01/app/oracle/oradata/slow/redo02b.log
/u01/app/oracle/oradata/slow/redo03a.log
/u01/app/oracle/oradata/slow/redo03b.log

6 rows selected.

Elapsed: 00:00:00.01
```
### 主库上依据redo log的路径建立4组standby redo log
```
13:26:06 SYS @ slow >edit
Wrote file afiedt.buf

  1* alter database add standby logfile group &a ('/u01/app/oracle/oradata/slow/&b') size 200m
13:27:02 SYS @ slow >r
  1* alter database add standby logfile group &a ('/u01/app/oracle/oradata/slow/&b') size 200m
Enter value for a: 4
Enter value for b: standby04.log
old 1: alter database add standby logfile group &a ('/u01/app/oracle/oradata/slow/&b') size200m
new 1: alter database add standby logfile group 4 ('/u01/app/oracle/oradata/slow/standby04.log') size 200m

Database altered.

Elapsed: 00:00:02.73
13:27:19 SYS @ slow >r
  1* alter database add standby logfile group &a ('/u01/app/oracle/oradata/slow/&b') size 200m
Enter value for a: 5 
Enter value for b: standby05.log
old 1: alter database add standby logfile group &a ('/u01/app/oracle/oradata/slow/&b') size200m
new 1: alter database add standby logfile group 5 ('/u01/app/oracle/oradata/slow/standby05.log') size 200m

Database altered.

Elapsed: 00:00:03.50
13:27:44 SYS @ slow >r
  1* alter database add standby logfile group &a ('/u01/app/oracle/oradata/slow/&b') size 200m
Enter value for a: 6
Enter value for b: standby06.log
old 1: alter database add standby logfile group &a ('/u01/app/oracle/oradata/slow/&b') size200m
new 1: alter database add standby logfile group 6 ('/u01/app/oracle/oradata/slow/standby06.log') size 200m

Database altered.

Elapsed: 00:00:02.04
13:28:03 SYS @ slow >r
  1* alter database add standby logfile group &a ('/u01/app/oracle/oradata/slow/&b') size 200m
Enter value for a: 7 
Enter value for b: standby07.log
old 1: alter database add standby logfile group &a ('/u01/app/oracle/oradata/slow/&b') size200m
new 1: alter database add standby logfile group 7 ('/u01/app/oracle/oradata/slow/standby07.log') size 200m

Database altered.

Elapsed: 00:00:02.33
13:28:19 SYS @ slow >select group#,thread#,sequence#,archived,status from v$standby_log;

    GROUP# THREAD# SEQUENCE# ARC STATUS
---------- ---------- ---------- --- ----------
     4     0     0 YES UNASSIGNED
     5     0     0 YES UNASSIGNED
     6     0     0 YES UNASSIGNED
     7     0     0 YES UNASSIGNED

4 rows selected.

Elapsed: 00:00:00.00
```
### 主库修改传输方式sync affirm ，下面备库也将跟着修改
```
13:55:51 SYS @ slow >r
  1* alter system set log_archive_dest_2='SERVICE=gotime OPTIONAL LGWR SYNC AFFIRM VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=gotime' scope=spfile

System altered.

Elapsed: 00:00:00.00


备库没有设置spfile，是以pfile打开库的，所以创建spifle，以spfile打开，才能修改
13:57:43 SYS @ gotime >edit
Wrote file afiedt.buf

  1* alter system set log_archive_dest_2='SERVICE=slow optional sync AFFIRM VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=slow' scope=spfile
13:58:01 SYS @ gotime >r
  1* alter system set log_archive_dest_2='SERVICE=slow optional sync AFFIRM VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=slow' scope=spfile
alter system set log_archive_dest_2='SERVICE=slow optional sync AFFIRM VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=slow' scope=spfile
*
ERROR at line 1:
ORA-32001: write to SPFILE requested but no SPFILE is in use


Elapsed: 00:00:00.00
13:58:02 SYS @ gotime >create pfile from spfile;
create pfile from spfile
*
ERROR at line 1:
ORA-01565: error in identifying file '?/dbs/spfile@.ora'
ORA-27037: unable to obtain file status
Linux-x86_64 Error: 2: No such file or directory
Additional information: 3


Elapsed: 00:00:00.00
13:59:06 SYS @ gotime >create spfile from pfile;

File created.

Elapsed: 00:00:00.02
14:00:45 SYS @ gotime >alter system set log_archive_dest_2='SERVICE=slow optional sync AFFIRM VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=slow' scope=spfile;
alter system set log_archive_dest_2='SERVICE=slow optional sync AFFIRM VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=slow' scope=spfile
*
ERROR at line 1:
ORA-32001: write to SPFILE requested but no SPFILE is in use


Elapsed: 00:00:00.00
14:01:06 SYS @ gotime >shutdown immediate;
ORA-01109: database not open


Database dismounted.
ORACLE instance shut down.
14:01:37 SYS @ gotime >startup mount;
ORACLE instance started.

Total System Global Area 521936896 bytes
Fixed Size         2254824 bytes
Variable Size         377489432 bytes
Database Buffers     138412032 bytes
Redo Buffers         3780608 bytes
Database mounted.
14:01:57 SYS @ gotime >alter system set log_archive_dest_2='SERVICE=slow optional sync AFFIRM VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=slow' scope=spfile;

System altered.

Elapsed: 00:00:00.01
14:02:09 SYS @ gotime >
```
### 主库开始转换，成功的从maximum performance 到 maximum protection模式
```
13:55:52 SYS @ slow >startup mount force;
ORACLE instance started.

Total System Global Area 521936896 bytes
Fixed Size         2254824 bytes
Variable Size         377489432 bytes
Database Buffers     138412032 bytes
Redo Buffers         3780608 bytes
Database mounted.
14:09:15 SYS @ slow >select status from v$instance;

STATUS
------------
MOUNTED

1 row selected.

Elapsed: 00:00:00.01
14:09:29 SYS @ slow >select name,database_role,protection_mode from v$database;

NAME     DATABASE_ROLE PROTECTION_MODE
--------- ---------------- --------------------
SLOW     PRIMARY     MAXIMUM PERFORMANCE

1 row selected.

Elapsed: 00:00:00.01
14:10:02 SYS @ slow >alter database set standby database to maximize protection;

Database altered.

Elapsed: 00:00:00.01
14:10:52 SYS @ slow >alter database open;

Database altered.

Elapsed: 00:00:03.83
14:11:20 SYS @ slow >select name,database_role,protection_mode from v$database;

NAME     DATABASE_ROLE PROTECTION_MODE
--------- ---------------- --------------------
SLOW     PRIMARY     MAXIMUM PROTECTION

1 row selected.

Elapsed: 00:00:00.00
14:11:45 SYS @ slow >
```
### 主库转换成功之后，备库的状态也随着改变了
```
14:14:09 SYS @ gotime >select status from v$instance;

STATUS
------------
MOUNTED

1 row selected.

Elapsed: 00:00:00.01
14:14:38 SYS @ gotime >select name,database_role,protection_mode from v$database;

NAME     DATABASE_ROLE PROTECTION_MODE
--------- ---------------- --------------------
SLOW     PHYSICAL STANDBY MAXIMUM PROTECTION

1 row selected.

Elapsed: 00:00:00.00
14:15:03 SYS @ gotime >alter database open;

Database altered.

Elapsed: 00:00:00.41
14:15:10 SYS @ gotime >select name,database_role,protection_mode from v$database;

NAME     DATABASE_ROLE PROTECTION_MODE
--------- ---------------- --------------------
SLOW     PHYSICAL STANDBY MAXIMUM PROTECTION

1 row selected.

Elapsed: 00:00:00.01
```
### 主库做一些DML操作，但是最后一步没有提交
```
14:18:31 SYS @ slow >select * from t2017;

    DEPTNO DNAME     LOC
---------- -------------- -------------
    10 ACCOUNTING     NEW YORK
    20 RESEARCH     DALLAS
    30 SALES     CHICAGO
    40 OPERATIONS     BOSTON

4 rows selected.

Elapsed: 00:00:00.00
14:18:46 SYS @ slow >insert into t2017 select * from t2017;

4 rows created.

Elapsed: 00:00:00.00
14:19:29 SYS @ slow >select * from t2017;

    DEPTNO DNAME     LOC
---------- -------------- -------------
    10 ACCOUNTING     NEW YORK
    20 RESEARCH     DALLAS
    30 SALES     CHICAGO
    40 OPERATIONS     BOSTON
    10 ACCOUNTING     NEW YORK
    20 RESEARCH     DALLAS
    30 SALES     CHICAGO
    40 OPERATIONS     BOSTON

8 rows selected.

Elapsed: 00:00:00.00
14:19:43 SYS @ slow >commit;

Commit complete.

Elapsed: 00:00:00.00
14:19:45 SYS @ slow >insert into t2017 select * from t2017; 

8 rows created.

Elapsed: 00:00:00.00
14:20:42 SYS @ slow >select * from t2017;

    DEPTNO DNAME     LOC
---------- -------------- -------------
    10 ACCOUNTING     NEW YORK
    20 RESEARCH     DALLAS
    30 SALES     CHICAGO
    40 OPERATIONS     BOSTON
    10 ACCOUNTING     NEW YORK
    20 RESEARCH     DALLAS
    30 SALES     CHICAGO
    40 OPERATIONS     BOSTON
    10 ACCOUNTING     NEW YORK
    20 RESEARCH     DALLAS
    30 SALES     CHICAGO

    DEPTNO DNAME     LOC
---------- -------------- -------------
    40 OPERATIONS     BOSTON
    10 ACCOUNTING     NEW YORK
    20 RESEARCH     DALLAS
    30 SALES     CHICAGO
    40 OPERATIONS     BOSTON

16 rows selected.

Elapsed: 00:00:00.00
14:21:18 SYS @ slow >alter system switch logfile;

System altered.

Elapsed: 00:00:00.12
14:23:28 SYS @ slow >
```
### 备库重新mount并应用日志（media recover）查询信息，查不到最后没有提交的信息，正常！
```
14:23:38 SYS @ gotime >startup mount force;
ORACLE instance started.

Total System Global Area 521936896 bytes
Fixed Size         2254824 bytes
Variable Size         377489432 bytes
Database Buffers     138412032 bytes
Redo Buffers         3780608 bytes
Database mounted.
14:24:22 SYS @ gotime >recover managed standby database disconnect;
Media recovery complete.
14:24:51 SYS @ gotime >select * from t2017;
select * from t2017
              *
ERROR at line 1:
ORA-01219: database not open: queries allowed on fixed tables/views only


Elapsed: 00:00:00.00
14:25:07 SYS @ gotime >alter database open;
alter database open
*
ERROR at line 1:
ORA-10456: cannot open standby database; media recovery session may be in progress


Elapsed: 00:00:00.00
14:25:18 SYS @ gotime >recover managed standby database cancel;
Media recovery complete.
14:25:42 SYS @ gotime >alter database open;

Database altered.

Elapsed: 00:00:00.21
14:25:47 SYS @ gotime >select * from t2017;

    DEPTNO DNAME     LOC
---------- -------------- -------------
    10 ACCOUNTING     NEW YORK
    20 RESEARCH     DALLAS
    30 SALES     CHICAGO
    40 OPERATIONS     BOSTON
    10 ACCOUNTING     NEW YORK
    20 RESEARCH     DALLAS
    30 SALES     CHICAGO
    40 OPERATIONS     BOSTON

8 rows selected.

Elapsed: 00:00:00.01
14:25:57 SYS @ gotime >
```
### 这个时候我们把备库的网络切断，然后再备库提交试试看...
```
[root@sink ~]# ifconfig
eth0 Link encap:Ethernet HWaddr 08:00:27:03:A5:94 
          inet addr:192.168.10.6 Bcast:192.168.10.255 Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1
          RX packets:29770 errors:0 dropped:0 overruns:0 frame:0
          TX packets:13633 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:32088544 (30.6 MiB) TX bytes:1186858 (1.1 MiB)

lo Link encap:Local Loopback 
          inet addr:127.0.0.1 Mask:255.0.0.0
          UP LOOPBACK RUNNING MTU:16436 Metric:1
          RX packets:4899 errors:0 dropped:0 overruns:0 frame:0
          TX packets:4899 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:3552488 (3.3 MiB) TX bytes:3552488 (3.3 MiB)

[root@sink ~]# ifconfig eth0 down

Connection closed by foreign host.

Disconnected from remote host(sink_root) at 14:26:42.

Type `help
```
### 主库探知备库down，大约5~6分钟之后主库自己也选择自杀了（shutdown abort）
```
14:26:28 SYS @ slow >commit;
commit
*
ERROR at line 1:
ORA-03113: end-of-file on communication channel
Process ID: 4226
Session ID: 1 Serial number: 5


Elapsed: 00:05:22.15
14:31:54 SYS @ slow >select status from v$instance;
ERROR:
ORA-03114: not connected to ORACLE


Elapsed: 00:00:00.00
14:34:54 SYS @ slow >
```
### 主库最后的告警日志信息
```
***********************************************************************

Fatal NI connect error 12543, connecting to:
 (DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=sink)(PORT=1521)))(CONNECT_DATA=(SERVICE_NAME=gotime)(CID=(PROGRAM=oracle)(HOST=slow)(USER=oracle))))

  VERSION INFORMATION:
    TNS for Linux: Version 11.2.0.4.0 - Production
    TCP/IP NT Protocol Adapter for Linux: Version 11.2.0.4.0 - Production
  Time: 13-JAN-2018 14:31:54
  Tracing not turned on.
  Tns error struct:
    ns main err code: 12543
    
TNS-12543: TNS:destination host unreachable
    ns secondary err code: 12560
    nt main err code: 513
    
TNS-00513: Destination host unreachable
    nt secondary err code: 113
    nt OS err code: 0
Error 12543 received logging on to the standby
Sat Jan 13 14:31:54 2018
LGWR: Error 12543 attaching to RFS for reconnect
Error 16198 for archive log file 3 to 'gotime'
Destination LOG_ARCHIVE_DEST_2 is UNSYNCHRONIZED
LGWR: All standby destinations have failed
******************************************************
WARNING: All standby database destinations have failed
WARNING: Instance shutdown required to protect primary
******************************************************
LGWR (ospid: 4180): terminating the instance due to error 16098
Sat Jan 13 14:31:54 2018
System state dump requested by (instance=1, osid=4180 (LGWR)), summary=[abnormal instance termination].
System State dumped to trace file /u01/app/oracle/diag/rdbms/slow/slow/trace/slow_diag_4160_20180113143154.trc
Dumping diagnostic data in directory=[cdmp_20180113143154], requested by (instance=1, osid=4180 (LGWR)), summary=[abnormal instance termination].
Instance terminated by LGWR, pid = 4180
```
### 最后我们去备库虚拟机的界面，进入后ifconfig eth0 up
<img src="/images/posts/2018-01-13-maximum_performance_转_maximum_protection/1.png">
### 接着通过Xshell连接到备库的虚拟主机，查看状态恢复正常
```
Connecting to 192.168.10.6:22...
Connection established.
To escape to local shell, press 'Ctrl+Alt+]'.

Last login: Sat Jan 13 14:37:06 2018
[root@sink ~]# su - oracle
[oracle@sink ~]$ echo $ORACLE_SID
sink
[oracle@sink ~]$ export ORACLE_SID=gotime
[oracle@sink ~]$ echo $ORACLE_SID
gotime
[oracle@sink ~]$ !sql
sqlplus / as sysdba

SQL*Plus: Release 11.2.0.4.0 Production on Sat Jan 13 14:56:19 2018

Copyright (c) 1982, 2013, Oracle. All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

14:56:19 SYS @ gotime >select status from v$instance;

STATUS
------------
OPEN

1 row selected.

Elapsed: 00:00:00.01
14:56:33 SYS @ gotime >select name,database_role,protection_mode from v$database;

NAME     DATABASE_ROLE PROTECTION_MODE
--------- ---------------- --------------------
SLOW     PHYSICAL STANDBY MAXIMUM PROTECTION

1 row selected.

Elapsed: 00:00:00.00
14:57:21 SYS @ gotime >
```
### 主库再启动到open，因为是自杀（shutdown abort）所以会比较就一点，好了，一切正常！成功！
```
14:34:54 SYS @ slow >select status form v$instance;
ERROR:
ORA-03114: not connected to ORACLE


Elapsed: 00:00:00.00
14:58:55 SYS @ slow >startup
ORA-24324: service handle not initialized
ORA-01041: internal error. hostdef extension doesn't exist
14:58:59 SYS @ slow >exit
Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
[oracle@slow ~]$ !sql
sqlplus '/as sysdba
```

<br>
转载请注明:[sinkshark的博客](http://www.sinkshark.com/)