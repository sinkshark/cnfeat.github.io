---
layout: post
title: 同一主机下Duplicate DB实验（含GI）
date: 2018-01-05
tag: oracle
---

>说明性文字：
>           oracle版本11g        操作系统：oracle linux 5
>       Main DB（target DB）  ----->连接------>   Duplicate DB


<img src="/images/posts/2018-01-05-同一主机下Duplicate DB实验(含GI)/1.png">
- 根据主库创建Duplicate DB的目录
- 根据主库的参数文件（pfile）创建Duplicate DB的参数文件（pfile），如果在上述路径没有找到的话在主库创建pfile：create pfile from spfile;  
- 然后再到$ORACLE_HOME/dbs路径下ls就能看到了

### 图中的vim /etc/hosts是为了添加IP解析
```
	[root@sink ~]# cat /etc/hosts 

	# Do not remove the following line, or various programs 

	# that require network functionality will fail. 

	127.0.0.1		sink localhost.localdomain localhost 

	::1		localhost6.localdomain6 localhost6 

	192.168.10.6   sink 
```
<img src="/images/posts/2018-01-05-同一主机下Duplicate DB实验(含GI)/2.png">
- 删除 sink.  的行，那是在sink实例下内容，我们这里是在根据sink库创建test库，所以不能存在sink实例的参数信息
- 修改memory_target的大小，最好和原始值有一定的差距，因为那是一个主机运行的内存，如果不修改就会报错，提示无法打开test库
- 如果有db_unique_name需要删掉这一行，不删的话就是一个库，unique是别名的意思，相当于创建辅助库，在手工TSPITR  和 RMAN TSPITR的时候需要设置db_unique_name，这里新设一个库，所以不需要
- 注意control_files的路径，最好设在一个路径下面好修改，当然也可以多路复用，只不过比较麻烦而已，这部分内容我还掌握的不好，不好班门弄斧，这里就设置同一路径吧
- esc 到底行命令模式  :%s/sink/test/g  替换所有sink为test
- 最后 加入 最下面2个 路径转换（数据文件，redo日志文件），最好去主库看一下asm路径，这是只是参考性的路径，您的实际路径肯定和我的不一样
<img src="/images/posts/2018-01-05-同一主机下Duplicate DB实验(含GI)/3.png">
- 配置了GI之后listener.ora监听就由GI接管了，所以在grid用户网络配置中配置监听
- 在duplicate当中test库是必须是nomunt状态的，所以如果要连接监听，就只能是静态注册，所以在默认监听上给test配置静态注册
- 下面的tnsname.ora的信息是我截图多出来了，不用理会
- ORACLE_HOME需要特别注意别写错了，我当时就写错了，然后后面的lsnrctl status的时候看到的状态老是BLOCKED，这里踩了一个大坑，最好在命令行下echo $ORACLE_HOME查看一下实际的路径
<img src="/images/posts/2018-01-05-同一主机下Duplicate DB实验(含GI)/4.png">
- 来到oralce用户，listener.ora信息是截图多出来的不必理会（图片没处理好，请大家见谅，我也只是小白，一知半解，坚持写博客只是为了激发热情，分享感悟）
- 到oracle网络配置路径下，配置tnsname.ora，sink（main DB）test（duplicate DB）
<img src="/images/posts/2018-01-05-同一主机下Duplicate DB实验(含GI)/5.png">
- 默认的是sink实例
- 直接 !sql --->  打开 sink（main DB）
<img src="/images/posts/2018-01-05-同一主机下Duplicate DB实验(含GI)/6.png">
- export到test实例
- 将test库startup nomount
<img src="/images/posts/2018-01-05-同一主机下Duplicate DB实验(含GI)/7.png">
- 到grid用户下重新启动listener监听
- 发现test连接上了，状态信息显示为UNKNOW表示test走的是刚才配置的静态注册，如果为BLOCKED则表示你配置静态注册存在问题，test没有走静态注册
<img src="/images/posts/2018-01-05-同一主机下Duplicate DB实验(含GI)/8.png">
- 开始  main DB （sink） -----连接----->  duplicate DB  （test）
- 提示：error 来自 目标库，target DB 是指 main DB （主库 sink）所以下面重启sink库
<img src="/images/posts/2018-01-05-同一主机下Duplicate DB实验(含GI)/9.png">
- 重启target DB
<img src="/images/posts/2018-01-05-同一主机下Duplicate DB实验(含GI)/10.png">
- 重启之后，正常进入了RMAN环境
- duplicate 开始duplicate target DB 到（to） test 库 
<img src="/images/posts/2018-01-05-同一主机下Duplicate DB实验(含GI)/11.png">
- 这是duplicate命令的结束截图，如图database opened ....表示克隆完成
<img src="/images/posts/2018-01-05-同一主机下Duplicate DB实验(含GI)/12.png">
- 重新来到克隆好的test库，做一下简单查询，查看test的信息
<img src="/images/posts/2018-01-05-同一主机下Duplicate DB实验(含GI)/13.png">
- 好了，感觉成功了

### 这是最后一步执行克隆的输出信息，留给下次研究，因为我也不知道Duplicate DB的过程中到底是什么样的过程，这里暂且搁置，下次一定补好
```
Xshell for Xmanager Enterprise 5 (Build 0738)
Copyright (c) 2002-2015 NetSarang Computer, Inc. All rights reserved.
Type `help' to learn how to use Xshell prompt.
[c:\~]$
Connecting to 192.168.10.6:22...
Connection established.
To escape to local shell, press 'Ctrl+Alt+]'.
Last login: Thu Jan  4 19:33:04 2018 from 192.168.10.1
[root@sink ~]# su - oracle
[oracle@sink ~]$ rman target sys/oracle@sink auxiliary sys/oracle@test
 
Recovery Manager: Release 11.2.0.4.0 - Production on Thu Jan 4 19:33:54 2018
 
Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.
 
RMAN-00571: ===========================================================
RMAN-00569: =============== ERROR MESSAGE STACK FOLLOWS ===============
RMAN-00571: ===========================================================
RMAN-00554: initialization of internal recovery manager package failed
RMAN-04005: error from target database:
ORA-12514: TNS:listener does not currently know of service requested in connect descriptor
[oracle@sink ~]$ rman target sys/oracle@sink auxiliary sys/oracle@test
 
Recovery Manager: Release 11.2.0.4.0 - Production on Thu Jan 4 19:34:33 2018
 
Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.
 
connected to target database: SINK (DBID=207714324)
connected to auxiliary database: TEST (not mounted)
 
RMAN> duplicate target database to "test" from active database nofilenamecheck;
 
Starting Duplicate Db at 04-JAN-18
using target database control file instead of recovery catalog
allocated channel: ORA_AUX_DISK_1
channel ORA_AUX_DISK_1: SID=20 device type=DISK
 
contents of Memory Script:
{
   sql clone "create spfile from memory";
}
executing Memory Script
 
sql statement: create spfile from memory
 
contents of Memory Script:
{
   shutdown clone immediate;
   startup clone nomount;
}
executing Memory Script
 
Oracle instance shut down
 
connected to auxiliary database (not started)
Oracle instance started
 
Total System Global Area     538640384 bytes
 
Fixed Size                     2254992 bytes
Variable Size                415238000 bytes
Database Buffers             117440512 bytes
Redo Buffers                   3706880 bytes
 
contents of Memory Script:
{
   sql clone "alter system set  db_name =
 ''SINK'' comment=
 ''Modified by RMAN duplicate'' scope=spfile";
   sql clone "alter system set  db_unique_name =
 ''TEST'' comment=
 ''Modified by RMAN duplicate'' scope=spfile";
   shutdown clone immediate;
   startup clone force nomount
   backup as copy current controlfile auxiliary format  '/u01/app/oracle/oradata/test/control01.ctl';
   restore clone controlfile to  '/u01/app/oracle/oradata/test/control02.ctl' from
 '/u01/app/oracle/oradata/test/control01.ctl';
   alter clone database mount;
}
executing Memory Script
 
sql statement: alter system set  db_name =  ''SINK'' comment= ''Modified by RMAN duplicate'' scope=spfile
 
sql statement: alter system set  db_unique_name =  ''TEST'' comment= ''Modified by RMAN duplicate'' scope=spfile
 
Oracle instance shut down
 
Oracle instance started
 
Total System Global Area     538640384 bytes
 
Fixed Size                     2254992 bytes
Variable Size                415238000 bytes
Database Buffers             117440512 bytes
Redo Buffers                   3706880 bytes
 
Starting backup at 04-JAN-18
allocated channel: ORA_DISK_1
channel ORA_DISK_1: SID=40 device type=DISK
channel ORA_DISK_1: starting datafile copy
copying current control file
output file name=/u01/app/oracle/product/11.2.0/dbhome_1/dbs/snapcf_sink.f tag=TAG20180104T193452 RECID=2 STAMP=964553692
channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:01
Finished backup at 04-JAN-18
 
Starting restore at 04-JAN-18
allocated channel: ORA_AUX_DISK_1
channel ORA_AUX_DISK_1: SID=18 device type=DISK
 
channel ORA_AUX_DISK_1: copied control file copy
Finished restore at 04-JAN-18
 
database mounted
 
contents of Memory Script:
{
   set newname for datafile  1 to
 "/u01/app/oracle/oradata/test/system01.dbf";
   set newname for datafile  2 to
 "/u01/app/oracle/oradata/test/sysaux01.dbf";
   set newname for datafile  3 to
 "/u01/app/oracle/oradata/test/undotbs01.dbf";
   set newname for datafile  4 to
 "/u01/app/oracle/oradata/test/users01.dbf";
   set newname for datafile  5 to
 "/u01/app/oracle/oradata/test/example01.dbf";
   set newname for datafile  6 to
 "/u01/app/oracle/oradata/test/tbssss.256.963504823";
   backup as copy reuse
   datafile  1 auxiliary format
 "/u01/app/oracle/oradata/test/system01.dbf"   datafile
 2 auxiliary format
 "/u01/app/oracle/oradata/test/sysaux01.dbf"   datafile
 3 auxiliary format
 "/u01/app/oracle/oradata/test/undotbs01.dbf"   datafile
 4 auxiliary format
 "/u01/app/oracle/oradata/test/users01.dbf"   datafile
 5 auxiliary format
 "/u01/app/oracle/oradata/test/example01.dbf"   datafile
 6 auxiliary format
 "/u01/app/oracle/oradata/test/tbssss.256.963504823"   ;
   sql 'alter system archive log current';
}
executing Memory Script
 
executing command: SET NEWNAME
 
executing command: SET NEWNAME
 
executing command: SET NEWNAME
 
executing command: SET NEWNAME
 
executing command: SET NEWNAME
 
executing command: SET NEWNAME
 
Starting backup at 04-JAN-18
using channel ORA_DISK_1
channel ORA_DISK_1: starting datafile copy
input datafile file number=00001 name=/u01/app/oracle/oradata/sink/system01.dbf
output file name=/u01/app/oracle/oradata/test/system01.dbf tag=TAG20180104T193459
channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:25
channel ORA_DISK_1: starting datafile copy
input datafile file number=00002 name=/u01/app/oracle/oradata/sink/sysaux01.dbf
output file name=/u01/app/oracle/oradata/test/sysaux01.dbf tag=TAG20180104T193459
channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:15
channel ORA_DISK_1: starting datafile copy
input datafile file number=00005 name=/u01/app/oracle/oradata/sink/example01.dbf
output file name=/u01/app/oracle/oradata/test/example01.dbf tag=TAG20180104T193459
channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:07
channel ORA_DISK_1: starting datafile copy
input datafile file number=00006 name=+DATA/sink/datafile/tbssss.256.963504823
output file name=/u01/app/oracle/oradata/test/tbssss.256.963504823 tag=TAG20180104T193459
channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:03
channel ORA_DISK_1: starting datafile copy
input datafile file number=00003 name=/u01/app/oracle/oradata/sink/undotbs01.dbf
output file name=/u01/app/oracle/oradata/test/undotbs01.dbf tag=TAG20180104T193459
channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:03
channel ORA_DISK_1: starting datafile copy
input datafile file number=00004 name=/u01/app/oracle/oradata/sink/users01.dbf
output file name=/u01/app/oracle/oradata/test/users01.dbf tag=TAG20180104T193459
channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:01
Finished backup at 04-JAN-18
 
sql statement: alter system archive log current
 
contents of Memory Script:
{
   backup as copy reuse
   archivelog like  "/dsk1/arch_sink/1_12_963448790.dbf" auxiliary format
 "/dsk1/arch_test/1_12_963448790.dbf"   ;
   catalog clone archivelog  "/dsk1/arch_test/1_12_963448790.dbf";
   switch clone datafile all;
}
executing Memory Script
 
Starting backup at 04-JAN-18
using channel ORA_DISK_1
channel ORA_DISK_1: starting archived log copy
input archived log thread=1 sequence=12 RECID=8 STAMP=964553753
output file name=/dsk1/arch_test/1_12_963448790.dbf RECID=0 STAMP=0
channel ORA_DISK_1: archived log copy complete, elapsed time: 00:00:01
Finished backup at 04-JAN-18
 
cataloged archived log
archived log file name=/dsk1/arch_test/1_12_963448790.dbf RECID=8 STAMP=964553755
 
datafile 1 switched to datafile copy
input datafile copy RECID=2 STAMP=964553755 file name=/u01/app/oracle/oradata/test/system01.dbf
datafile 2 switched to datafile copy
input datafile copy RECID=3 STAMP=964553755 file name=/u01/app/oracle/oradata/test/sysaux01.dbf
datafile 3 switched to datafile copy
input datafile copy RECID=4 STAMP=964553755 file name=/u01/app/oracle/oradata/test/undotbs01.dbf
datafile 4 switched to datafile copy
input datafile copy RECID=5 STAMP=964553755 file name=/u01/app/oracle/oradata/test/users01.dbf
datafile 5 switched to datafile copy
input datafile copy RECID=6 STAMP=964553755 file name=/u01/app/oracle/oradata/test/example01.dbf
datafile 6 switched to datafile copy
input datafile copy RECID=7 STAMP=964553755 file name=/u01/app/oracle/oradata/test/tbssss.256.963504823
 
contents of Memory Script:
{
   set until scn  1158304;
   recover
   clone database
    delete archivelog
   ;
}
executing Memory Script
 
executing command: SET until clause
 
Starting recover at 04-JAN-18
using channel ORA_AUX_DISK_1
 
starting media recovery
 
archived log for thread 1 with sequence 12 is already on disk as file /dsk1/arch_test/1_12_963448790.dbf
archived log file name=/dsk1/arch_test/1_12_963448790.dbf thread=1 sequence=12
media recovery complete, elapsed time: 00:00:00
Finished recover at 04-JAN-18
Oracle instance started
 
Total System Global Area     538640384 bytes
 
Fixed Size                     2254992 bytes
Variable Size                415238000 bytes
Database Buffers             117440512 bytes
Redo Buffers                   3706880 bytes
 
contents of Memory Script:
{
   sql clone "alter system set  db_name =
 ''TEST'' comment=
 ''Reset to original value by RMAN'' scope=spfile";
   sql clone "alter system reset  db_unique_name scope=spfile";
   shutdown clone immediate;
   startup clone nomount;
}
executing Memory Script
 
sql statement: alter system set  db_name =  ''TEST'' comment= ''Reset to original value by RMAN'' scope=spfile
 
sql statement: alter system reset  db_unique_name scope=spfile
 
Oracle instance shut down
 
connected to auxiliary database (not started)
Oracle instance started
 
Total System Global Area     538640384 bytes
 
Fixed Size                     2254992 bytes
Variable Size                415238000 bytes
Database Buffers             117440512 bytes
Redo Buffers                   3706880 bytes
sql statement: CREATE CONTROLFILE REUSE SET DATABASE "TEST" RESETLOGS ARCHIVELOG
  MAXLOGFILES     16
  MAXLOGMEMBERS      3
  MAXDATAFILES      200
  MAXINSTANCES     8
  MAXLOGHISTORY      292
 LOGFILE
  GROUP   1 ( '/u01/app/oracle/oradata/test/redo01.log' ) SIZE 50 M  REUSE,
  GROUP   2 ( '/u01/app/oracle/oradata/test/redo02.log' ) SIZE 50 M  REUSE,
  GROUP   3 ( '/u01/app/oracle/oradata/test/redo03.log' ) SIZE 50 M  REUSE
 DATAFILE
  '/u01/app/oracle/oradata/test/system01.dbf'
 CHARACTER SET ZHS16GBK
 
 
contents of Memory Script:
{
   set newname for tempfile  1 to
 "/u01/app/oracle/oradata/test/temp01.dbf";
   switch clone tempfile all;
   catalog clone datafilecopy  "/u01/app/oracle/oradata/test/sysaux01.dbf",
 "/u01/app/oracle/oradata/test/undotbs01.dbf",
 "/u01/app/oracle/oradata/test/users01.dbf",
 "/u01/app/oracle/oradata/test/example01.dbf",
 "/u01/app/oracle/oradata/test/tbssss.256.963504823";
   switch clone datafile all;
}
executing Memory Script
 
executing command: SET NEWNAME
 
renamed tempfile 1 to /u01/app/oracle/oradata/test/temp01.dbf in control file
 
cataloged datafile copy
datafile copy file name=/u01/app/oracle/oradata/test/sysaux01.dbf RECID=1 STAMP=964553762
cataloged datafile copy
datafile copy file name=/u01/app/oracle/oradata/test/undotbs01.dbf RECID=2 STAMP=964553762
cataloged datafile copy
datafile copy file name=/u01/app/oracle/oradata/test/users01.dbf RECID=3 STAMP=964553762
cataloged datafile copy
datafile copy file name=/u01/app/oracle/oradata/test/example01.dbf RECID=4 STAMP=964553762
cataloged datafile copy
datafile copy file name=/u01/app/oracle/oradata/test/tbssss.256.963504823 RECID=5 STAMP=964553762
 
datafile 2 switched to datafile copy
input datafile copy RECID=1 STAMP=964553762 file name=/u01/app/oracle/oradata/test/sysaux01.dbf
datafile 3 switched to datafile copy
input datafile copy RECID=2 STAMP=964553762 file name=/u01/app/oracle/oradata/test/undotbs01.dbf
datafile 4 switched to datafile copy
input datafile copy RECID=3 STAMP=964553762 file name=/u01/app/oracle/oradata/test/users01.dbf
datafile 5 switched to datafile copy
input datafile copy RECID=4 STAMP=964553762 file name=/u01/app/oracle/oradata/test/example01.dbf
datafile 6 switched to datafile copy
input datafile copy RECID=5 STAMP=964553762 file name=/u01/app/oracle/oradata/test/tbssss.256.963504823
 
contents of Memory Script:
{
   Alter clone database open resetlogs;
}
executing Memory Script
 
database opened
Finished Duplicate Db at 04-JAN-18
 
RMAN> 
```
<br>
转载请注明:[sinkshark的博客](http://www.sinkshark.com/)