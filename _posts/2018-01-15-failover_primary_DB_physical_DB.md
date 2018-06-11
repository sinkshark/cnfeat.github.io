
layout: post
title: failover_primary_DB_physical_DB
date: 2018-01-16
tag: oracle
---

### 主库上的准备工作 primary DB---slow
```
[root@slow ~]# su - oracle
[oracle@slow ~]$ !sql
sqlplus '/as sysdba'

SQL*Plus: Release 11.2.0.4.0 Production on Sat Jan 13 20:54:25 2018

Copyright (c) 1982, 2013, Oracle. All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

20:54:25 SYS @ slow >
20:54:25 SYS @ slow >select status from v$instance;

STATUS
------------
OPEN

1 row selected.

Elapsed: 00:00:00.00
20:54:37 SYS @ slow >select name,open_mode,database_role,protection_mode from v$database
20:55:07 2 ;

NAME     OPEN_MODE     DATABASE_ROLE    PROTECTION_MODE
--------- -------------------- ---------------- --------------------
SLOW     READ WRITE     PRIMARY        MAXIMUM AVAILABILITY

1 row selected.

Elapsed: 00:00:00.00
20:55:09 SYS @ slow >show parameter recover

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
db_recovery_file_dest         string
db_recovery_file_dest_size     big integer 0
db_unrecoverable_scn_tracking     boolean     TRUE
recovery_parallelism         integer     0
20:55:37 SYS @ slow >alter system set db_recovery_file_dest_size=4g;

System altered.

Elapsed: 00:00:00.00
20:56:34 SYS @ slow >alter system set db_recovery_file_dest='/dsk1/slow_recover/';

System altered.

Elapsed: 00:00:00.02
20:57:01 SYS @ slow >show parameter recover;

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
db_recovery_file_dest         string     /dsk1/slow_recover/
db_recovery_file_dest_size     big integer 4G
db_unrecoverable_scn_tracking     boolean     TRUE
recovery_parallelism         integer     0
20:57:09 SYS @ slow >show parameter dg_broker_start

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
dg_broker_start          boolean     FALSE
21:03:02 SYS @ slow >alter system set dg_broker_start=true;

System altered.

Elapsed: 00:00:00.02
21:03:40 SYS @ slow >show parameter dg_broker_start

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
dg_broker_start          boolean     TRUE
21:03:56 SYS @ slow >show parameter dg;

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
cell_offloadgroup_name         string
dg_broker_config_file1         string     /u01/app/oracle/product/11.2.0
                         /dbhome_1/dbs/dr1slow.dat
dg_broker_config_file2         string     /u01/app/oracle/product/11.2.0
                         /dbhome_1/dbs/dr2slow.dat
dg_broker_start          boolean     TRUE
21:04:07 SYS @ slow >alter system switch logfile;

System altered.

Elapsed: 00:00:00.02
21:21:09 SYS @ slow >select name,flashback_on from v$database;

NAME     FLASHBACK_ON
--------- ------------------
SLOW     NO

1 row selected.

Elapsed: 00:00:00.01
21:26:08 SYS @ slow >alter database flashback on;

Database altered.

Elapsed: 00:00:01.16
21:26:27 SYS @ slow >select status from v$instance;

STATUS
------------
OPEN

1 row selected.

Elapsed: 00:00:00.01
```

### 备库上的准备工作
```
[oracle@sink ~]$ echo $ORACLE_SID
gotime
[oracle@sink ~]$ !sql
sqlplus / as sysdba

SQL*Plus: Release 11.2.0.4.0 Production on Sat Jan 13 20:59:35 2018

Copyright (c) 1982, 2013, Oracle. All rights reserved.

Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

20:59:35 SYS @ gotime >select status from v$instance;

STATUS
------------
OPEN

1 row selected.

Elapsed: 00:00:00.01
20:59:47 SYS @ gotime >select name,open_mode,database_role,protection_mode from v$database;

NAME     OPEN_MODE     DATABASE_ROLE    PROTECTION_MODE
--------- -------------------- ---------------- --------------------
SLOW     READ ONLY     PHYSICAL STANDBY MAXIMUM AVAILABILITY

1 row selected.

Elapsed: 00:00:00.00
21:01:18 SYS @ gotime >shutdown immediate;
Database closed.
Database dismounted.
ORACLE instance shut down.
21:05:03 SYS @ gotime >startup mount;
ORACLE instance started.

Total System Global Area 521936896 bytes
Fixed Size         2254824 bytes
Variable Size         377489432 bytes
Database Buffers     138412032 bytes
Redo Buffers         3780608 bytes
Database mounted.
21:05:18 SYS @ gotime >show parameter recover;

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
db_recovery_file_dest         string
db_recovery_file_dest_size     big integer 0
db_unrecoverable_scn_tracking     boolean     TRUE
recovery_parallelism         integer     0
21:08:14 SYS @ gotime >alter system set db_recovery_file_dest_size=4g scope=spfile;

System altered.

Elapsed: 00:00:00.01
21:09:40 SYS @ gotime >alter system set db_recovery_file_dest='/dsk1/gotime_recover/' scope=spfile;
j
System altered.

Elapsed: 00:00:00.01
21:10:07 SYS @ gotime show parameter recover;

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
db_recovery_file_dest         string     /dsk1/gotime_recover/
db_recovery_file_dest_size     big integer 4G
db_unrecoverable_scn_tracking     boolean     TRUE
recovery_parallelism         integer     0
21:10:21 SYS @ gotime >show parameter dg_broker_start

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
dg_broker_start          boolean     FALSE
21:18:49 SYS @ gotime >alter system dg_broker_start=true;
alter system dg_broker_start=true
             *
ERROR at line 1:
ORA-02065: illegal option for ALTER SYSTEM


Elapsed: 00:00:00.00
21:19:07 SYS @ gotime >alter database open;

Database altered.

Elapsed: 00:00:00.41
21:19:44 SYS @ gotime >alter system dg_broker_start=true;
alter system dg_broker_start=true
             *
ERROR at line 1:
ORA-02065: illegal option for ALTER SYSTEM


Elapsed: 00:00:00.00
21:20:01 SYS @ gotime >shutdown immediate;
Database closed.
Database dismounted.
ORACLE instance shut down.
21:20:17 SYS @ gotime >startup mount;
ORACLE instance started.

Total System Global Area 521936896 bytes
Fixed Size         2254824 bytes
Variable Size         377489432 bytes
Database Buffers     138412032 bytes
Redo Buffers         3780608 bytes
Database mounted.
21:21:02 SYS @ gotime >recover managed standby database disconnect;
Media recovery complete.
21:21:52 SYS @ gotime >show parameter dg

NAME                 TYPE     VALUE
------------------------------------ ----------- ------------------------------
cell_offloadgroup_name         string
dg_broker_config_file1         string     /u01/app/oracle/product/11.2.0
                         /dbhome_1/dbs/dr1gotime.dat
dg_broker_config_file2         string     /u01/app/oracle/product/11.2.0
                         /dbhome_1/dbs/dr2gotime.dat
dg_broker_start          boolean     FALSE
21:22:56 SYS @ gotime >alter system set dg_broker_start=true;

System altered.

Elapsed: 00:00:00.01
21:23:15 SYS @ gotime >select name,flashback_on from v$database;

NAME     FLASHBACK_ON
--------- ------------------
SLOW     NO

1 row selected.

Elapsed: 00:00:00.01
21:27:06 SYS @ gotime >alter database flashback on;
alter database flashback on
*
ERROR at line 1:
ORA-01153: an incompatible media recovery is active


Elapsed: 00:00:00.01
21:27:48 SYS @ gotime >recover managed standby database cancel;
Media recovery complete.
21:28:24 SYS @ gotime >alter database flashback on;

Database altered.

Elapsed: 00:00:01.93
21:28:42 SYS @ gotime >recover managed standby database using current logfile disconnect;
Media recovery complete.
21:49:18 SYS @ gotime >select name,open_mode,database_role,protection_mode,switchover_status from v$database;

NAME     OPEN_MODE     DATABASE_ROLE    PROTECTION_MODE SWITCHOVER_STATUS
--------- -------------------- ---------------- -------------------- --------------------
SLOW     MOUNTED     PHYSICAL STANDBY MAXIMUM AVAILABILITY NOT ALLOWED

1 row selected.

Elapsed: 00:00:00.00
```
### 主库操作系统上，主库设置dg_broker_start=true后，进程ora_dmon_slow，启动
```
[root@slow ~]# ps -ef|grep ora_|grep -v grep
oracle 5629 1 0 18:57 ? 00:00:00 ora_pmon_slow
oracle 5633 1 0 18:57 ? 00:00:01 ora_psp0_slow
oracle 5637 1 0 18:57 ? 00:01:10 ora_vktm_slow
oracle 5643 1 0 18:57 ? 00:00:00 ora_gen0_slow
oracle 5647 1 0 18:57 ? 00:00:00 ora_diag_slow
oracle 5651 1 0 18:57 ? 00:00:00 ora_dbrm_slow
oracle 5655 1 0 18:57 ? 00:00:02 ora_dia0_slow
oracle 5659 1 0 18:57 ? 00:00:00 ora_mman_slow
oracle 5663 1 0 18:57 ? 00:00:00 ora_dbw0_slow
oracle 5667 1 0 18:57 ? 00:00:02 ora_lgwr_slow
oracle 5671 1 0 18:57 ? 00:00:02 ora_ckpt_slow
oracle 5675 1 0 18:57 ? 00:00:00 ora_smon_slow
oracle 5679 1 0 18:57 ? 00:00:00 ora_reco_slow
oracle 5683 1 0 18:57 ? 00:00:01 ora_mmon_slow
oracle 5687 1 0 18:57 ? 00:00:01 ora_mmnl_slow
oracle 5691 1 0 18:57 ? 00:00:00 ora_d000_slow
oracle 5695 1 0 18:57 ? 00:00:00 ora_s000_slow
oracle 5726 1 0 19:00 ? 00:00:00 ora_arc0_slow
oracle 5730 1 0 19:00 ? 00:00:00 ora_arc1_slow
oracle 5734 1 0 19:00 ? 00:00:00 ora_arc2_slow
oracle 5738 1 0 19:00 ? 00:00:00 ora_arc3_slow
oracle 5742 1 0 19:00 ? 00:00:00 ora_qmnc_slow
oracle 5770 1 0 19:00 ? 00:00:01 ora_cjq0_slow
oracle 5774 1 0 19:00 ? 00:00:09 ora_vkrm_slow
oracle 5778 1 0 19:00 ? 00:00:00 ora_q000_slow
oracle 5782 1 0 19:00 ? 00:00:00 ora_q001_slow
oracle 5809 1 0 19:05 ? 00:00:00 ora_smco_slow
oracle 6338 1 0 20:55 ? 00:00:00 ora_w000_slow
oracle 6344 1 0 20:56 ? 00:00:00 ora_nss2_slow
oracle 6411 1 0 21:03 ? 00:00:00 ora_dmon_slow
oracle 6415 1 0 21:03 ? 00:00:00 ora_insv_slow
oracle 6421 1 0 21:05 ? 00:00:00 ora_w001_slow
oracle 6426 1 0 21:05 ? 00:00:00 ora_w002_slow
[root@slow ~]#
```
### 备库操作系统上，备库设置dg_broker_start=true后，进程ora_dmon_gotime，启动
```
[root@sink ~]# ps -ef|grep ora_|grep -v grep
oracle 10913 1 0 21:20 ? 00:00:00 ora_pmon_gotime
oracle 10917 1 0 21:20 ? 00:00:00 ora_psp0_gotime
oracle 10922 1 0 21:20 ? 00:00:01 ora_vktm_gotime
oracle 10928 1 0 21:20 ? 00:00:00 ora_gen0_gotime
oracle 10932 1 0 21:20 ? 00:00:00 ora_diag_gotime
oracle 10936 1 0 21:20 ? 00:00:00 ora_dbrm_gotime
oracle 10940 1 0 21:20 ? 00:00:00 ora_dia0_gotime
oracle 10944 1 0 21:20 ? 00:00:00 ora_mman_gotime
oracle 10948 1 0 21:20 ? 00:00:00 ora_dbw0_gotime
oracle 10952 1 0 21:20 ? 00:00:00 ora_lgwr_gotime
oracle 10956 1 0 21:20 ? 00:00:00 ora_ckpt_gotime
oracle 10960 1 0 21:20 ? 00:00:00 ora_smon_gotime
oracle 10964 1 0 21:20 ? 00:00:00 ora_reco_gotime
oracle 10968 1 0 21:20 ? 00:00:00 ora_mmon_gotime
oracle 10972 1 0 21:20 ? 00:00:00 ora_mmnl_gotime
oracle 10976 1 0 21:20 ? 00:00:00 ora_d000_gotime
oracle 10980 1 0 21:20 ? 00:00:00 ora_s000_gotime
oracle 10995 1 0 21:20 ? 00:00:00 ora_nss2_gotime
oracle 11003 1 0 21:21 ? 00:00:00 ora_arc0_gotime
oracle 11007 1 0 21:21 ? 00:00:00 ora_arc1_gotime
oracle 11011 1 0 21:21 ? 00:00:00 ora_arc2_gotime
oracle 11016 1 0 21:21 ? 00:00:00 ora_arc3_gotime
oracle 11044 1 0 21:21 ? 00:00:00 ora_mrp0_gotime
oracle 11064 1 0 21:23 ? 00:00:00 ora_dmon_gotime
[root@sink ~]#
```
### 主库上的listener.ora
```
[oracle@slow ~]$ vim /u01/app/oracle/product/11.2.0/dbhome_1/network/admin/listener.ora 

# listener.ora Network Configuration File: /u01/app/oracle/product/11.2.0/dbhome_1/network/admin/listener.ora
# Generated by Oracle configuration tools.

LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = slow)(PORT = 1521))
    )
  )

ADR_BASE_LISTENER = /u01/app/oracle


SID_LIST_LISTENER=
   (SID_DESC=
    (SID_NAME=slow)
    (SDU=32767)
        (ORACLE_HOME=/u01/app/oracle/product/11.2.0/dbhome_1)
        (GLOBAL_DBNAME=slow_dgmgrl)
   )
```
### 主库上的tnsname.ora
```
[oracle@slow ~]$ vim /u01/app/oracle/product/11.2.0/dbhome_1/network/admin/tnsnames.ora 

# tnsnames.ora Network Configuration File: /u01/app/oracle/product/11.2.0/dbhome_1/network/admin/tnsnames.ora
# Generated by Oracle configuration tools.

GOTIME =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = sink)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVICE_NAME = gotime)
    )
  )

SLOW =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = slow)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVICE_NAME = slow)
    )
  )
```
### 备库上的listener.ora
```
[grid@sink ~]$ vim /u01/11.2.0/grid/network/admin/listener.ora

# listener.ora Network Configuration File: /u01/11.2.0/grid/network/admin/listener.ora
# Generated by Oracle configuration tools.

LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = sink)(PORT = 1521))
    )
  )

ADR_BASE_LISTENER = /u01/app/grid

ENABLE_GLOBAL_DYNAMIC_ENDPOINT_LISTENER=ON # line added by Agent

SID_LIST_LISTENER=
 (SID_LIST=
  (SID_DESC=
    (SID_NAME=gotime)
    (SDU=32767)
      (ORACLE_HOME=/u01/app/oracle/product/11.2.0/dbhome_1)
      (GLOBAL_DBNAME=gotime_dgmgrl)
   )
 )
```
### 备库上的tnsname.ora
```
[grid@sink ~]$ su - oracle
Password: 
[oracle@sink ~]$ vim /u01/app/oracle/product/11.2.0/dbhome_1/network/admin/tnsnames.ora 

# tnsnames.ora Network Configuration File: /u01/app/oracle/product/11.2.0/dbhome_1/network/admin/tnsnames.ora
# Generated by Oracle configuration tools.
SLOW =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = slow)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVICE_NAME = slow)
    )
  )

GOTIME =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = sink)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVICE_NAME = gotime)
    )
  )
```
### 主库上的网络状态
```
[oracle@slow ~]$ lsnrctl stop

LSNRCTL for Linux: Version 11.2.0.4.0 - Production on 13-JAN-2018 21:15:40

Copyright (c) 1991, 2013, Oracle. All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=slow)(PORT=1521)))
TNS-12541: TNS:no listener
 TNS-12560: TNS:protocol adapter error
  TNS-00511: No listener
   Linux Error: 111: Connection refused
[oracle@slow ~]$ lsnrctl start

LSNRCTL for Linux: Version 11.2.0.4.0 - Production on 13-JAN-2018 21:15:43

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
Start Date 13-JAN-2018 21:15:43
Uptime 0 days 0 hr. 0 min. 0 sec
Trace Level off
Security ON: Local OS Authentication
SNMP OFF
Listener Parameter File /u01/app/oracle/product/11.2.0/dbhome_1/network/admin/listener.ora
Listener Log File /u01/app/oracle/diag/tnslsnr/slow/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=slow)(PORT=1521)))
Services Summary...
Service "slow_dgmgrl" has 1 instance(s).
  Instance "slow", status UNKNOWN, has 1 handler(s) for this service...
The command completed successfully
```
### 备库上的网络状态
```
[grid@sink ~]$ lsnrctl stop

LSNRCTL for Linux: Version 11.2.0.4.0 - Production on 13-JAN-2018 21:14:56

Copyright (c) 1991, 2013, Oracle. All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=sink)(PORT=1521)))
The command completed successfully
[grid@sink ~]$ lsnrctl start

LSNRCTL for Linux: Version 11.2.0.4.0 - Production on 13-JAN-2018 21:14:59

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
Start Date 13-JAN-2018 21:14:59
Uptime 0 days 0 hr. 0 min. 0 sec
Trace Level off
Security ON: Local OS Authentication
SNMP OFF
Listener Parameter File /u01/11.2.0/grid/network/admin/listener.ora
Listener Log File /u01/app/grid/diag/tnslsnr/sink/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=sink)(PORT=1521)))
Services Summary...
Service "gotime_dgmgrl" has 1 instance(s).
  Instance "gotime", status UNKNOWN, has 1 handler(s) for this service...
The command completed successfully
[grid@sink ~]$ 
[grid@sink ~]$ lsnrctl status

LSNRCTL for Linux: Version 11.2.0.4.0 - Production on 13-JAN-2018 21:15:03

Copyright (c) 1991, 2013, Oracle. All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=sink)(PORT=1521)))
STATUS of the LISTENER
------------------------
Alias LISTENER
Version TNSLSNR for Linux: Version 11.2.0.4.0 - Production
Start Date 13-JAN-2018 21:14:59
Uptime 0 days 0 hr. 0 min. 4 sec
Trace Level off
Security ON: Local OS Authentication
SNMP OFF
Listener Parameter File /u01/11.2.0/grid/network/admin/listener.ora
Listener Log File /u01/app/grid/diag/tnslsnr/sink/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=sink)(PORT=1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=127.0.0.1)(PORT=1522)))
Services Summary...
Service "gotime_dgmgrl" has 1 instance(s).
  Instance "gotime", status UNKNOWN, has 1 handler(s) for this service...
The command completed successfully
```
================== 配置FSF（fast_start failover）==============================
### 在主库主机上配置configuration
```
[root@slow ~]# su - oracle
[oracle@slow ~]$ dgmgrl
DGMGRL for Linux: Version 11.2.0.4.0 - 64bit Production

Copyright (c) 2000, 2009, Oracle. All rights reserved.

Welcome to DGMGRL, type "help" for information.
DGMGRL> connect
Username: sys
Password:
Connected.
DGMGRL> create configuration 'slow_cfg1' as primary database is 'slow' connect identifier is 'slow';
```
======================最后面这个slow是tnsname.ora里面的名称===================
```
Configuration "slow_cfg1" created with primary database "slow"
DGMGRL> show configuration

Configuration - slow_cfg1

  Protection Mode: MaxAvailability
  Databases:
    slow - Primary database

Fast-Start Failover: DISABLED

Configuration Status:
DISABLED

DGMGRL> add database 'gotime' as connect identifier is 'gotime' maintained as physical;
```
===================最后面这个gotime是tnsname.ora里面的名称================
```
Database "gotime" added
DGMGRL> show configuration;

Configuration - slow_cfg1

  Protection Mode: MaxAvailability
  Databases:
    slow - Primary database
    gotime - Physical standby database

Fast-Start Failover: DISABLED

Configuration Status:
DISABLED

DGMGRL> enable configuration;
Enabled.
DGMGRL> show configuration;

Configuration - slow_cfg1

  Protection Mode: MaxAvailability
  Databases:
    slow - Primary database
    gotime - Physical standby database

Fast-Start Failover: DISABLED

Configuration Status:
SUCCESS

DGMGRL> start observer;
Observer started
...
至此不能继续进行，需要新建一个连接窗口继续配置
```
### 此为新建窗口，继续配置...
```
[oracle@slow ~]$ export ORACLE_SID=gotime
[oracle@slow ~]$ echo $ORACLE_SID
gotime
[oracle@slow ~]$ dgmgrl
DGMGRL for Linux: Version 11.2.0.4.0 - 64bit Production

Copyright (c) 2000, 2009, Oracle. All rights reserved.

Welcome to DGMGRL, type "help" for information.
DGMGRL> connect 
Username: sys
Password:
Connected.
DGMGRL> show database verbose slow
ORA-01034: ORACLE not available
Process ID: 0
Session ID: 0 Serial number: 0

Configuration details cannot be determined by DGMGRL
DGMGRL> exit 
[oracle@slow ~]$ export ORACLE_SID=slow
[oracle@slow ~]$ echo $ORACLE_SID
slow
[oracle@slow ~]$ dgmgrl
DGMGRL for Linux: Version 11.2.0.4.0 - 64bit Production

Copyright (c) 2000, 2009, Oracle. All rights reserved.

Welcome to DGMGRL, type "help" for information.
DGMGRL> connect
Username: sys
Password:
Connected.
DGMGRL> show database verbose slow

Database - slow

  Role: PRIMARY
  Intended State: TRANSPORT-ON
  Instance(s):
    slow

  Properties:
    DGConnectIdentifier = 'slow'
    ObserverConnectIdentifier = ''
    LogXptMode = 'SYNC'
    DelayMins = '0'
    Binding = 'optional'
    MaxFailure = '0'
    MaxConnections = '1'
    ReopenSecs = '300'
    NetTimeout = '30'
    RedoCompression = 'DISABLE'
    LogShipping = 'ON'
    PreferredApplyInstance = ''
    ApplyInstanceTimeout = '0'
    ApplyParallel = 'AUTO'
    StandbyFileManagement = 'AUTO'
    ArchiveLagTarget = '0'
    LogArchiveMaxProcesses = '4'
    LogArchiveMinSucceedDest = '1'
    DbFileNameConvert = '/u01/app/oracle/oradata/slow/, /u01/app/oracle/oradata/gotime/'
    LogFileNameConvert = '/u01/app/oracle/oradata/slow/, /u01/app/oracle/oradata/gotime/'
    FastStartFailoverTarget = ''
    InconsistentProperties = '(monitor)'
    InconsistentLogXptProps = '(monitor)'
    SendQEntries = '(monitor)'
    LogXptStatus = '(monitor)'
    RecvQEntries = '(monitor)'
    ApplyLagThreshold = '0'
    TransportLagThreshold = '0'
    TransportDisconnectedThreshold = '30'
    SidName = 'slow'
    StaticConnectIdentifier = '(DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=slow)(PORT=1521))(CONNECT_DATA=(SERVICE_NAME=slow_DGMGRL)(INSTANCE_NAME=slow)(SERVER=DEDICATED)))'
    StandbyArchiveLocation = '/dsk1/arch_slow/'
    AlternateLocation = ''
    LogArchiveTrace = '0'
    LogArchiveFormat = '%t_%s_%r.arc'
    TopWaitEvents = '(monitor)'

Database Status:
SUCCESS

DGMGRL> show database verbose gotime;

Database - gotime

  Role: PHYSICAL STANDBY
  Intended State: APPLY-ON
  Transport Lag: 0 seconds (computed 1 second ago)
  Apply Lag: 0 seconds (computed 1 second ago)
  Apply Rate: 0 Byte/s
  Real Time Query: OFF
  Instance(s):
    gotime

  Properties:
    DGConnectIdentifier = 'gotime'
    ObserverConnectIdentifier = ''
    LogXptMode = 'SYNC'
    DelayMins = '0'
    Binding = 'OPTIONAL'
    MaxFailure = '0'
    MaxConnections = '1'
    ReopenSecs = '300'
    NetTimeout = '30'
    RedoCompression = 'DISABLE'
    LogShipping = 'ON'
    PreferredApplyInstance = ''
    ApplyInstanceTimeout = '0'
    ApplyParallel = 'AUTO'
    StandbyFileManagement = 'AUTO'
    ArchiveLagTarget = '0'
    LogArchiveMaxProcesses = '4'
    LogArchiveMinSucceedDest = '1'
    DbFileNameConvert = '/u01/app/oracle/oradata/slow/, /u01/app/oracle/oradata/gotime/'
    LogFileNameConvert = '/u01/app/oracle/oradata/slow/, /u01/app/oracle/oradata/gotime/'
    FastStartFailoverTarget = ''
    InconsistentProperties = '(monitor)'
    InconsistentLogXptProps = '(monitor)'
    SendQEntries = '(monitor)'
    LogXptStatus = '(monitor)'
    RecvQEntries = '(monitor)'
    ApplyLagThreshold = '0'
    TransportLagThreshold = '0'
    TransportDisconnectedThreshold = '30'
    SidName = 'gotime'
    StaticConnectIdentifier = '(DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=sink)(PORT=1521))(CONNECT_DATA=(SERVICE_NAME=gotime_DGMGRL)(INSTANCE_NAME=gotime)(SERVER=DEDICATED)))'
    StandbyArchiveLocation = '/dsk1/arch_gotime/'
    AlternateLocation = ''
    LogArchiveTrace = '0'
    LogArchiveFormat = '%t_%s_%r.arc'
    TopWaitEvents = '(monitor)'

Database Status:
SUCCESS

DGMGRL> show configuration;

Configuration - slow_cfg1

  Protection Mode: MaxAvailability
  Databases:
    slow - Primary database
    gotime - Physical standby database

Fast-Start Failover: DISABLED

Configuration Status:
SUCCESS

DGMGRL> enable fast_start failover;
Enabled.
DGMGRL> show configuration;

Configuration - slow_cfg1

  Protection Mode: MaxAvailability
  Databases:
    slow - Primary database
    gotime - (*) Physical standby database

Fast-Start Failover: ENABLED

Configuration Status:
SUCCESS

DGMGRL>
```
===============开始测试========== 前面是准备工作很重要，否则失败 =================
### 此时主库的状态，是，primary
```
21:26:27 SYS @ slow >select status from v$instance;

STATUS
------------
OPEN

1 row selected.

Elapsed: 00:00:00.01
21:41:31 SYS @ slow >select name,open_mode,database_role,protection_mode,switchover_status from v$database;

NAME     OPEN_MODE     DATABASE_ROLE    PROTECTION_MODE SWITCHOVER_STATUS
--------- -------------------- ---------------- -------------------- --------------------
SLOW     READ WRITE     PRIMARY        MAXIMUM AVAILABILITY SESSIONS ACTIVE

1 row selected.

Elapsed: 00:00:00.00
```
### 此时备库的状态，是，physical standby
```
21:50:41 SYS @ gotime >select status from v$instance;

STATUS
------------
MOUNTED

1 row selected.

Elapsed: 00:00:00.01
21:53:15 SYS @ gotime >alter database open;

Database altered.

Elapsed: 00:00:01.26
21:53:37 SYS @ gotime >select name,open_mode,database_role,protection_mode,switchover_status from v$database;

NAME     OPEN_MODE     DATABASE_ROLE    PROTECTION_MODE SWITCHOVER_STATUS
--------- -------------------- ---------------- -------------------- --------------------
SLOW     READ ONLY WITH APPLY PHYSICAL STANDBY MAXIMUM AVAILABILITY NOT ALLOWED

1 row selected.

Elapsed: 00:00:00.03
```
### 主库意外宕机。。。
```
21:48:16 SYS @ slow >shutdown abort;
ORACLE instance shut down.
21:56:45 SYS @ slow >
```
### 几秒钟后。。。之前配置configuration时候，不能继续进行的窗口，弹出，显示failover successful！！！
```
DGMGRL> start observer;
Observer started
...

21:57:17.41 Saturday, January 13, 2018
Initiating Fast-Start Failover to database "gotime"...
Performing failover NOW, please wait...
Failover succeeded, new primary is "gotime"
21:57:22.68 Saturday, January 13, 2018
```
### 那么此时备库（physical standby）应该成主库（primary）了，瞅一下，对了，那么成功！！！
```
21:53:51 SYS @ gotime >select status from v$instance;
select status from v$instance
*
ERROR at line 1:
ORA-03135: connection lost contact
Process ID: 11015
Session ID: 18 Serial number: 7


ERROR:
ORA-03114: not connected to ORACLE


Elapsed: 00:00:00.00
21:57:25 SYS @ gotime >conn / as sysdba
Connected.
21:57:42 SYS @ gotime >select status from v$instance;

STATUS
------------
OPEN

1 row selected.

Elapsed: 00:00:00.01
21:57:50 SYS @ gotime >select name,database_role,protection_mode,switchover_status from v$database;

NAME     DATABASE_ROLE PROTECTION_MODE    SWITCHOVER_STATUS
--------- ---------------- -------------------- --------------------
SLOW     PRIMARY     MAXIMUM AVAILABILITY NOT ALLOWED

1 row selected.

Elapsed: 00:00:00.01
21:58:27 SYS @ gotime >
```
<br>
转载请注明:[sinkshark的博客](http://www.sinkshark.com/)