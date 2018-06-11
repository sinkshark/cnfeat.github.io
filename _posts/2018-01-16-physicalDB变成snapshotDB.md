---
layout: post
title: "physicalDB转snapshotDB"
date: 2018-01-16
dag: oracle
---

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
- 意思是提示备库没有创建standby redo log所以下面创建4组
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
### 备库上依据redo log的路径建立4组standby  redo log
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
### 打开主库，设置快速恢复区大小，路径，为主库添加4组standby日志，方便之后的主库互换
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
13:26:06 SYS @ slow >edit
Wrote file afiedt.buf

  1* alter database add standby logfile group &a ('/u01/app/oracle/oradata/slow/&b') size 200m
13:27:02 SYS @ slow >r
  1* alter database add standby logfile group &a ('/u01/app/oracle/oradata/slow/&b') size 200m
Enter value for a: 4
Enter value for b: standby04.log
old 1: alter database add standby logfile group &a ('/u01/app/oracle/oradata/slow/&b') size 200m
new 1: alter database add standby logfile group 4 ('/u01/app/oracle/oradata/slow/standby04.log') size 200m

Database altered.

Elapsed: 00:00:02.73
13:27:19 SYS @ slow >r
  1* alter database add standby logfile group &a ('/u01/app/oracle/oradata/slow/&b') size 200m
Enter value for a: 5 
Enter value for b: standby05.log
old 1: alter database add standby logfile group &a ('/u01/app/oracle/oradata/slow/&b') size 200m
new 1: alter database add standby logfile group 5 ('/u01/app/oracle/oradata/slow/standby05.log') size 200m

Database altered.

Elapsed: 00:00:03.50
13:27:44 SYS @ slow >r
  1* alter database add standby logfile group &a ('/u01/app/oracle/oradata/slow/&b') size 200m
Enter value for a: 6
Enter value for b: standby06.log
old 1: alter database add standby logfile group &a ('/u01/app/oracle/oradata/slow/&b') size 200m
new 1: alter database add standby logfile group 6 ('/u01/app/oracle/oradata/slow/standby06.log') size 200m

Database altered.

Elapsed: 00:00:02.04
13:28:03 SYS @ slow >r
  1* alter database add standby logfile group &a ('/u01/app/oracle/oradata/slow/&b') size 200m
Enter value for a: 7 
Enter value for b: standby07.log
old 1: alter database add standby logfile group &a ('/u01/app/oracle/oradata/slow/&b') size 200m
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

### 确认备库dest1和dest2
```
17:02:29 SYS @ gotime >show parameter dest;

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
audit_file_dest          string     /u01/app/oracle/admin/gotime/a
                         dump
background_dump_dest         string     /u01/app/oracle/diag/rdbms/got
                         ime/gotime/trace
core_dump_dest             string     /u01/app/oracle/diag/rdbms/got
                         ime/gotime/cdump
cursor_bind_capture_destination string     memory+disk
db_create_file_dest         string
db_create_online_log_dest_1     string
db_create_online_log_dest_2     string
db_create_online_log_dest_3     string

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
db_create_online_log_dest_4     string
db_create_online_log_dest_5     string
db_recovery_file_dest         string     /dsk1/gotime_recover/
db_recovery_file_dest_size     big integer 4G
diagnostic_dest          string     /u01/app/oracle
log_archive_dest         string
log_archive_dest_1         string     LOCATION=/dsk1/arch_gotime/
                         VALID_FOR=(ALL_LOGFILES,ALL_
                         ROLES)
                         DB_UNIQUE_NAME=gotime
log_archive_dest_10         string

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
log_archive_dest_11         string
log_archive_dest_12         string
log_archive_dest_13         string
log_archive_dest_14         string
log_archive_dest_15         string
log_archive_dest_16         string
log_archive_dest_17         string
log_archive_dest_18         string
log_archive_dest_19         string
log_archive_dest_2         string     SERVICE=slow ASYNC
                         VALID_FOR=(ONLINE_LOGFILES,P

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
                         RIMARY_ROLE)
                         DB_UNIQUE_NAME=slow
log_archive_dest_20         string
log_archive_dest_21         string
log_archive_dest_22         string
log_archive_dest_23         string
log_archive_dest_24         string
log_archive_dest_25         string
log_archive_dest_26         string
log_archive_dest_27         string
log_archive_dest_28         string

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
log_archive_dest_29         string
log_archive_dest_3         string
log_archive_dest_30         string
log_archive_dest_31         string
log_archive_dest_4         string
log_archive_dest_5         string
log_archive_dest_6         string
log_archive_dest_7         string
log_archive_dest_8         string
log_archive_dest_9         string
log_archive_dest_state_1     string     ENABLE

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
log_archive_dest_state_10     string     enable
log_archive_dest_state_11     string     enable
log_archive_dest_state_12     string     enable
log_archive_dest_state_13     string     enable
log_archive_dest_state_14     string     enable
log_archive_dest_state_15     string     enable
log_archive_dest_state_16     string     enable
log_archive_dest_state_17     string     enable
log_archive_dest_state_18     string     enable
log_archive_dest_state_19     string     enable
log_archive_dest_state_2     string     ENABLE

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
log_archive_dest_state_20     string     enable
log_archive_dest_state_21     string     enable
log_archive_dest_state_22     string     enable
log_archive_dest_state_23     string     enable
log_archive_dest_state_24     string     enable
log_archive_dest_state_25     string     enable
log_archive_dest_state_26     string     enable
log_archive_dest_state_27     string     enable
log_archive_dest_state_28     string     enable
log_archive_dest_state_29     string     enable
log_archive_dest_state_3     string     enable

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
log_archive_dest_state_30     string     enable
log_archive_dest_state_31     string     enable
log_archive_dest_state_4     string     enable
log_archive_dest_state_5     string     enable
log_archive_dest_state_6     string     enable
log_archive_dest_state_7     string     enable
log_archive_dest_state_8     string     enable
log_archive_dest_state_9     string     enable
log_archive_duplex_dest      string
log_archive_min_succeed_dest     integer     1
standby_archive_dest         string     ?/dbs/arch

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
user_dump_dest             string     /u01/app/oracle/diag/rdbms/got
                         ime/gotime/trace
17:06:51 SYS @ gotime >
确认主库dest1和dest2
17:34:18 SYS @ slow >show parameter dest;

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
audit_file_dest          string     /u01/app/oracle/admin/slow/adu
                         mp
background_dump_dest         string     /u01/app/oracle/diag/rdbms/slo
                         w/slow/trace
core_dump_dest             string     /u01/app/oracle/diag/rdbms/slo
                         w/slow/cdump
cursor_bind_capture_destination string     memory+disk
db_create_file_dest         string
db_create_online_log_dest_1     string
db_create_online_log_dest_2     string
db_create_online_log_dest_3     string

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
db_create_online_log_dest_4     string
db_create_online_log_dest_5     string
db_recovery_file_dest         string
db_recovery_file_dest_size     big integer 0
diagnostic_dest          string     /u01/app/oracle
log_archive_dest         string
log_archive_dest_1         string     LOCATION=/dsk1/arch_slow/
                         VALID_FOR=(ALL_LOGFILES,ALL_
                         ROLES)
                         DB_UNIQUE_NAME=slow
log_archive_dest_10         string

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
log_archive_dest_11         string
log_archive_dest_12         string
log_archive_dest_13         string
log_archive_dest_14         string
log_archive_dest_15         string
log_archive_dest_16         string
log_archive_dest_17         string
log_archive_dest_18         string
log_archive_dest_19         string
log_archive_dest_2         string     SERVICE=gotime ASYNC
                         VALID_FOR=(ONLINE_LOGFILES,P

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
                         RIMARY_ROLE)
                         DB_UNIQUE_NAME=gotime
log_archive_dest_20         string
log_archive_dest_21         string
log_archive_dest_22         string
log_archive_dest_23         string
log_archive_dest_24         string
log_archive_dest_25         string
log_archive_dest_26         string
log_archive_dest_27         string
log_archive_dest_28         string

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
log_archive_dest_29         string
log_archive_dest_3         string
log_archive_dest_30         string
log_archive_dest_31         string
log_archive_dest_4         string
log_archive_dest_5         string
log_archive_dest_6         string
log_archive_dest_7         string
log_archive_dest_8         string
log_archive_dest_9         string
log_archive_dest_state_1     string     ENABLE

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
log_archive_dest_state_10     string     enable
log_archive_dest_state_11     string     enable
log_archive_dest_state_12     string     enable
log_archive_dest_state_13     string     enable
log_archive_dest_state_14     string     enable
log_archive_dest_state_15     string     enable
log_archive_dest_state_16     string     enable
log_archive_dest_state_17     string     enable
log_archive_dest_state_18     string     enable
log_archive_dest_state_19     string     enable
log_archive_dest_state_2     string     ENABLE

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
log_archive_dest_state_20     string     enable
log_archive_dest_state_21     string     enable
log_archive_dest_state_22     string     enable
log_archive_dest_state_23     string     enable
log_archive_dest_state_24     string     enable
log_archive_dest_state_25     string     enable
log_archive_dest_state_26     string     enable
log_archive_dest_state_27     string     enable
log_archive_dest_state_28     string     enable
log_archive_dest_state_29     string     enable
log_archive_dest_state_3     string     enable

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
log_archive_dest_state_30     string     enable
log_archive_dest_state_31     string     enable
log_archive_dest_state_4     string     enable
log_archive_dest_state_5     string     enable
log_archive_dest_state_6     string     enable
log_archive_dest_state_7     string     enable
log_archive_dest_state_8     string     enable
log_archive_dest_state_9     string     enable
log_archive_duplex_dest      string
log_archive_min_succeed_dest     integer     1
standby_archive_dest         string     ?/dbs/arch

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
user_dump_dest             string     /u01/app/oracle/diag/rdbms/slo
                         w/slow/trace
```
### 主库做一些DML操作，提交，并切换日志处罚ARCH归档日志
```
17:36:32 SYS @ slow >create table t2 as select * from scott.emp;

Table created.

Elapsed: 00:00:00.10
17:38:43 SYS @ slow >commit;

Commit complete.

Elapsed: 00:00:00.00
17:38:47 SYS @ slow >alter system switch logfile;

System altered.

Elapsed: 00:00:00.01
17:38:58 SYS @ slow >
```
### 查看备库的告警日志，是否接收到日志，此时已处罚RFS（remote file server）故主备连接没有问题
```
RFS[1]: Assigned to RFS process 4339
RFS[1]: Selected log 4 for thread 1 sequence 16 dbid 1147482511 branch 963616975
Fri Jan 12 17:33:36 2018
RFS[2]: Assigned to RFS process 4335
RFS[2]: Opened log for thread 1 sequence 10 dbid 1147482511 branch 963616975
Archived Log entry 1 added for thread 1 sequence 10 rlc 963616975 ID 0x4465718f dest 2:
RFS[1]: Opened log for thread 1 sequence 12 dbid 1147482511 branch 963616975
RFS[2]: Opened log for thread 1 sequence 13 dbid 1147482511 branch 963616975
Fri Jan 12 17:33:36 2018
Archived Log entry 2 added for thread 1 sequence 16 ID 0x4465718f dest 1:
Fri Jan 12 17:33:36 2018
RFS[3]: Assigned to RFS process 4343
RFS[3]: Opened log for thread 1 sequence 11 dbid 1147482511 branch 963616975
Archived Log entry 3 added for thread 1 sequence 13 rlc 963616975 ID 0x4465718f dest 2:
RFS[2]: Opened log for thread 1 sequence 14 dbid 1147482511 branch 963616975
Archived Log entry 4 added for thread 1 sequence 14 rlc 963616975 ID 0x4465718f dest 2:
RFS[2]: Opened log for thread 1 sequence 15 dbid 1147482511 branch 963616975
Archived Log entry 5 added for thread 1 sequence 15 rlc 963616975 ID 0x4465718f dest 2:
Archived Log entry 6 added for thread 1 sequence 11 rlc 963616975 ID 0x4465718f dest 2:
我上一次搭建DG的时候主库是startup pfile=xxxx 所以隔天我来做主备互换的时候是startup，主备无法连接，折
腾一下午终于找出原因了，通过以上的一些验证证明没有问题之后我们继续下面的操作
```
### 执行此命令将备库开启到snapshot mode
```
17:42:10 SYS @ gotime >r
  1* alter database convert to snapshot standby

Database altered.

```
Elapsed: 00:00:02.37
### 查看备库告警日志
```
Created guaranteed restore point SNAPSHOT_STANDBY_REQUIRED_01/12/2018 17:42:11
Killing 4 processes with pids 4406,4385,4389,4393 (all RFS) in order to disallow current and futions. Requested by OS process 3844
Begin: Standby Redo Logfile archival
End: Standby Redo Logfile archival
RESETLOGS after incomplete recovery UNTIL CHANGE 336029
Resetting resetlogs activation ID 1147498895 (0x4465718f)
Online log /u01/app/oracle/oradata/gotime/redo01a.log: Thread 1 Group 1 was previously cleared
Online log /u01/app/oracle/oradata/gotime/redo01b.log: Thread 1 Group 1 was previously cleared
Online log /u01/app/oracle/oradata/gotime/redo02a.log: Thread 1 Group 2 was previously cleared
Online log /u01/app/oracle/oradata/gotime/redo02b.log: Thread 1 Group 2 was previously cleared
Online log /u01/app/oracle/oradata/gotime/redo03a.log: Thread 1 Group 3 was previously cleared
Online log /u01/app/oracle/oradata/gotime/redo03b.log: Thread 1 Group 3 was previously cleared
Standby became primary SCN: 336027
Fri Jan 12 17:42:13 2018
Setting recovery target incarnation to 2
CONVERT TO SNAPSHOT STANDBY: Complete - Database mounted as snapshot standby
Completed: alter database convert to snapshot standby
Fri Jan 12 17:43:05 2018
ARC1: Becoming the 'no SRL' ARCH
此时备库的 状态是mounted，我们下面可以将他转换成open read write 状态了
17:56:17 SYS @ gotime >select status from v$instance;

STATUS
------------
MOUNTED

1 row selected.

Elapsed: 00:00:00.00
```
### 现在可以以read write的方式打开备库了！！！因为转换成功了
```
17:56:25 SYS @ gotime >alter database open read write;

Database altered.

Elapsed: 00:00:01.01
17:58:12 SYS @ gotime >
```
### 查看转换后的database_role是snapshot standby ！！！
```
18:00:55 SYS @ gotime >select name,database_role,protection_mode,open_mode fromv$database;

NAME     DATABASE_ROLE PROTECTION_MODE    OPEN_MODE
--------- ---------------- -------------------- --------------------
SLOW     SNAPSHOT STANDBY MAXIMUM PERFORMANCE    READ WRITE

1 row selected.

Elapsed: 00:00:00.01
18:02:03 SYS @ gotime >
```
### 做一些DML操作，提交，好等备库回到snapshot DB之前做对比
```
18:00:30 SYS @ gotime >r
  1* create table t2018 as select * from scott.emp

Table created.

Elapsed: 00:00:00.04
18:00:30 SYS @ gotime >select * from t2018;

     EMPNO ENAME JOB     MGR HIREDATE     SAL COMM DEPTNO
---------- ---------- --------- ---------- --------- ---------- ---------- ----------
      7369 SMITH CLERK     7902 17-DEC-80     800          20
      7499 ALLEN SALESMAN     7698 20-FEB-81     1600 300     30
      7521 WARD SALESMAN     7698 22-FEB-81     1250 500     30
      7566 JONES MANAGER     7839 02-APR-81     2975          20
      7654 MARTIN SALESMAN     7698 28-SEP-81     1250 1400     30
      7698 BLAKE MANAGER     7839 01-MAY-81     2850          30
      7782 CLARK MANAGER     7839 09-JUN-81     2450          10
      7788 SCOTT ANALYST     7566 19-APR-87     3000          20
      7839 KING PRESIDENT      17-NOV-81     5000          10
      7844 TURNER SALESMAN     7698 08-SEP-81     1500      0     30
      7876 ADAMS CLERK     7788 23-MAY-87     1100          20

     EMPNO ENAME JOB     MGR HIREDATE     SAL COMM DEPTNO
---------- ---------- --------- ---------- --------- ---------- ---------- ----------
      7900 JAMES CLERK     7698 03-DEC-81     950          30
      7902 FORD ANALYST     7566 03-DEC-81     3000          20
      7934 MILLER CLERK     7782 23-JAN-82     1300          10

14 rows selected.

Elapsed: 00:00:00.00
18:00:42 SYS @ gotime >commit;

Commit complete.

Elapsed: 00:00:00.00
18:00:55 SYS @ gotime >
```
### 这时候刚才创建的standby redo log被使用了
```
18:02:03 SYS @ gotime >select group#,thread#,sequence#,archived,status from v$standby_log;

    GROUP# THREAD# SEQUENCE# ARC STATUS
---------- ---------- ---------- --- ----------
     4     1     22 YES ACTIVE
     5     1     0 NO UNASSIGNED
     6     0     0 YES UNASSIGNED
     7     0     0 YES UNASSIGNED

4 rows selected.

Elapsed: 00:00:00.00
18:03:39 SYS @ gotime >
```
### 主库做一些DML操作，提交，好等备库回到snapshot DB之前做对比
```
17:38:58 SYS @ slow >create table t2017 as select * from scott.dept;

Table created.

Elapsed: 00:00:00.06
18:08:04 SYS @ slow >select * from t2017;

    DEPTNO DNAME     LOC
---------- -------------- -------------
    10 ACCOUNTING     NEW YORK
    20 RESEARCH     DALLAS
    30 SALES     CHICAGO
    40 OPERATIONS     BOSTON

4 rows selected.

Elapsed: 00:00:00.01
18:08:13 SYS @ slow >commit;

Commit complete.

Elapsed: 00:00:00.00
18:08:19 SYS @ slow >alter system switch logfile;

System altered.

Elapsed: 00:00:00.01
18:10:28 SYS @ slow >
```
### 关闭备库，重新启动到mounted然后转换回physical standby DB
```
18:11:34 SYS @ gotime >shutdown immediate;
Database closed.
Database dismounted.
ORACLE instance shut down.
18:12:42 SYS @ gotime >startup mount;
ORACLE instance started.

Total System Global Area 521936896 bytes
Fixed Size         2254824 bytes
Variable Size         377489432 bytes
Database Buffers     138412032 bytes
Redo Buffers         3780608 bytes
Database mounted.
18:12:57 SYS @ gotime >alter database convert to physical standby;

Database altered.

Elapsed: 00:00:02.23
18:13:18 SYS @ gotime >
```
### 转换回physical standby DB告警日志的情况
```
alter database convert to physical standby
ALTER DATABASE CONVERT TO PHYSICAL STANDBY (gotime)
Killing 3 processes with pids 4994,4998,5002 (all RFS) in order to disallow current and future RFS connections. Requested by OS process 4988
Flashback Restore Start
Flashback Restore Complete
Drop guaranteed restore point 
Stopping background process RVWR
Deleted Oracle managed file/dsk1/gotime_recover/GOTIME/flashback/o1_mf_f5k0qmlb_.flb
Deleted Oracle managed file/dsk1/gotime_recover/GOTIME/flashback/o1_mf_f5k0qoco_.flb
Guaranteed restore point dropped
Clearing standby activation ID 1149107987 (0x447dff13)
The primary database controlfile was created using the
'MAXLOGFILES 10' clause.
There is space for up to 7 standby redo logfiles
Use the following SQL commands on the standby database to create
standby redo logfiles that match the primary database:
ALTER DATABASE ADD STANDBY LOGFILE 'srl1.f' SIZE 104857600;
ALTER DATABASE ADD STANDBY LOGFILE 'srl2.f' SIZE 104857600;
ALTER DATABASE ADD STANDBY LOGFILE 'srl3.f' SIZE 104857600;
ALTER DATABASE ADD STANDBY LOGFILE 'srl4.f' SIZE 104857600;
Shutting down archive processes
Archiving is disabled
Fri Jan 12 18:13:17 2018
ARCH shutting down
ARC3: Archival stopped
Fri Jan 12 18:13:17 2018
ARCH shutting down
ARC2: Archival stopped
Fri Jan 12 18:13:17 2018
ARCH shutting down
ARC1: Archival stopped
Fri Jan 12 18:13:17 2018
ARCH shutting down
ARC0: Archival stopped
Completed: alter database convert to physical standby
```
### 打开备库查看数据库模式role和状态
```
18:13:18 SYS @ gotime >select name,database_role,protection_mode,open_mode fromv$database; select name,database_role,protection_mode,open_mode from v$database
                                                         *
ERROR at line 1:
ORA-01507: database not mounted


Elapsed: 00:00:00.01
18:15:42 SYS @ gotime >select name,database_role,protection_mode,open_mode fromv$database;
select name,database_role,protection_mode,open_mode from v$database
                                                         *
ERROR at line 1:
ORA-01507: database not mounted


Elapsed: 00:00:00.00
18:15:51 SYS @ gotime >recover managed standby database disconnect;
ORA-01507: database not mounted


18:16:37 SYS @ gotime >select status from v$instance;

STATUS
------------
STARTED

1 row selected.

Elapsed: 00:00:00.01
18:17:00 SYS @ gotime >alter database mount; 
alter database mount
*
ERROR at line 1:
ORA-00750: database has been previously mounted and dismounted


Elapsed: 00:00:00.00
18:17:13 SYS @ gotime >startup mount;
ORA-01081: cannot start already-running ORACLE - shut it down first
18:17:26 SYS @ gotime >shutdown immediate;
ORA-01507: database not mounted


ORACLE instance shut down.
18:18:58 SYS @ gotime >startup mount;
ORACLE instance started.

Total System Global Area 521936896 bytes
Fixed Size         2254824 bytes
Variable Size         377489432 bytes
Database Buffers     138412032 bytes
Redo Buffers         3780608 bytes
Database mounted.
18:19:10 SYS @ gotime >select name,database_role,protection_mode,open_mode fromv$database;

NAME     DATABASE_ROLE PROTECTION_MODE    OPEN_MODE
--------- ---------------- -------------------- --------------------
SLOW     PHYSICAL STANDBY MAXIMUM PERFORMANCE    MOUNTED

1 row selected.

Elapsed: 00:00:00.02
18:19:17 SYS @ gotime >
```
### 重新恢复到physical DB之后，对积压的主库传过来的日志进行media recover 应用
```
18:19:17 SYS @ gotime >recover managed standby database disconnect;
Media recovery complete.
```
### 此时备库告警日志信息
```
MRP0 started with pid=27, OS id=5221 
MRP0: Background Managed Standby Recovery process started (gotime)
Serial Media Recovery started
Managed Standby Recovery not using Real Time Apply
Waiting for all non-current ORLs to be archived...
All non-current ORLs have been archived.
Clearing online redo logfile 1 /u01/app/oracle/oradata/gotime/redo01a.log
Clearing online log 1 of thread 1 sequence number 25
Completed: ALTER DATABASE RECOVER managed standby database disconnect 
Clearing online redo logfile 1 complete
Clearing online redo logfile 2 /u01/app/oracle/oradata/gotime/redo02a.log
Clearing online log 2 of thread 1 sequence number 2
Clearing online redo logfile 2 complete
Clearing online redo logfile 3 /u01/app/oracle/oradata/gotime/redo03a.log
Clearing online log 3 of thread 1 sequence number 24
Fri Jan 12 18:21:56 2018
Clearing online redo logfile 3 complete
Media Recovery Log /dsk1/arch_gotime/1_21_963616975.arc
Media Recovery Log /dsk1/arch_gotime/1_22_963616975.arc
Media Recovery Log /dsk1/arch_gotime/1_23_963616975.arc
Media Recovery Log /dsk1/arch_gotime/1_24_963616975.arc
Media Recovery Waiting for thread 1 sequence 25 (in transit)
```
### 以只读的方式打开数据库，此时备库snapshot DB期间主库所做的操作都应用到备库了
```
18:21:51 SYS @ gotime >alter database open;
alter database open
*
ERROR at line 1:
ORA-10456: cannot open standby database; media recovery session may be in progress


Elapsed: 00:00:00.00
18:24:11 SYS @ gotime >recover managed standby database cancel;
Media recovery complete.
18:24:34 SYS @ gotime >alter database open ;

Database altered.

Elapsed: 00:00:00.28
18:24:41 SYS @ gotime >select name,database_role,protection_mode,open_mode fromv$database;

NAME     DATABASE_ROLE PROTECTION_MODE    OPEN_MODE
--------- ---------------- -------------------- --------------------
SLOW     PHYSICAL STANDBY MAXIMUM PERFORMANCE    READ ONLY

1 row selected.

Elapsed: 00:00:00.01
18:25:15 SYS @ gotime >
```
### 在备库上对比查询，t2017（主库DML）存在，t2018（备库DML）消失，snapshot DB转换成功！！！
```
18:25:15 SYS @ gotime >select * from t2017;

    DEPTNO DNAME     LOC
---------- -------------- -------------
    10 ACCOUNTING     NEW YORK
    20 RESEARCH     DALLAS
    30 SALES     CHICAGO
    40 OPERATIONS     BOSTON

4 rows selected.

Elapsed: 00:00:00.01
18:26:51 SYS @ gotime >select * from t2018;
select * from t2018
              *
ERROR at line 1:
ORA-00942: table or view does not exist


Elapsed: 00:00:00.00
18:27:01 SYS @ gotime >
```
2017悄然逝去已成历史，我们无法改变，2018还像一张白纸，等我们认真填写，我那时候站在2019回望2018又会不会呆着遗憾呢，期待2018。。。

<br>

转载请注明:[sinkshark的博客](http://www.sinkshark.com/)