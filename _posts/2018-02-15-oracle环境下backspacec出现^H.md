---
layout: post
title: oracle环境下backspacec出现^H
date: 2018-02-15
tag: oracle
---

### 提出问题
```
Connecting to 192.168.56.6:22...
Connection established.
To escape to local shell, press 'Ctrl+Alt+]'.

Last login: Thu Feb 15 13:28:49 2018 from 192.168.56.1
[root@shark ~]# su - oracle
[oracle@shark ~]$ !sql
sqlplus / as sysdba

SQL*Plus: Release 11.2.0.4.0 Production on Thu Feb 15 13:56:08 2018

Copyright (c) 1982, 2013, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

13:56:08 SYS@shark>stty erase ^H
SP2-0734: unknown command beginning "stty erase..." - rest of line ignored.
13:56:08 SYS@shark>select name,^H^H^H^H
```
### 解决问题
添加stty erase '^H'
```
[root@shark ~]# su - oracle
[oracle@shark ~]$ vim .bash_profile 

# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin

export PATH

export ORACLE_BASE=/u01/app/oracle

export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/dbhome_1

export ORACLE_SID=shark

export PATH=$PATH:$HOME/bin:$ORACLE_HOME/bin

stty erase '^H'
```

### exit退出oracle用户，重新su - oracle进入oracle用户，sqlplus / as sysdba到sql则不会出现^H
### 如果还出现^H,则直接在sql环境下面执行host stty erase '^H'即可！

转载请注明 : [sinkshark的博客](http://sinkshark.com/)