---
layout: post
title: ORACLE ADMIN 01
excerpt: "There are three major structures in Oracle Database server architecture: memory structures, process structures, and storage structures."
categories: [ORACLE ADMINISTRATION]
comments: true
---

### Explanation

> ORACLE 11GWS1 오라클 인스턴스 정보와 디비 정보, 서버 기동 확인 해보기, install하기

### Instance information
echo $ORACLE_SID
select * from v$sga;
select * from v$sgastat;      -- detail info
select spid, pname from v$process;  -- BG process, server process
select * from v$instance;  

### Server maneuver 
ps  -ef |  grep orcl

### DB information
show parameter db_name
show parameter control_files
select * from v$controlfile;
select * from v$logfile;
select * from v$datafile;

## Code Snippets

{% highlight css %}
[orcl:~]$ echo $ORACLE_SID
orcl
[orcl:~]$ echo $ORACLE_HOME
/u01/app/oracle/product/11.2.0/dbhome_1

[orcl:~]$ cd $ORACLE_HOME
[orcl:dbhome_1]$ pwd
/u01/app/oracle/product/11.2.0/dbhome_1
[orcl:dbhome_1]$ ls dbs --다른 목록들 중 확인 했을 때 dbs가 중요함!
hc_DBUA0.dat  init.ora      lkORCL     peshm_DBUA0_0  init.ora --parameter conf
hc_orcl.dat   initorcl.ora  orapworcl  peshm_orcl_0

/*
다른 목록들
rdbms -> 모든 스크립트 실행
sqlplus -> sqlplus와 관련된 내용
*/

[orcl:dbhome_1]$ ls sqlplus
admin  bin  doc  lib  mesg
[orcl:dbhome_1]$ cd sqlplus
[orcl:sqlplus]$ cd admin
[orcl:admin]$ ls
glogin.sql  help  libsqlplus.def  plustrce.sql  pupbld.sql
[orcl:admin]$ vi glogin.sql

set pages 100
set sqlp "user'@'_connect_identifier> "  
define_editor=vi
:wq

[orcl:admin]$ cd 
[orcl:~]$ sqlplus / as sysdba
user@orcl>

{% endhighlight %}
