---
layout: post
title: ORACLE ADMIN 13
excerpt: "데이타 이동 1"
categories: [ORACLE ADMINISTRATION]
comments: true
---


# TABLE A CONTENT

> 디렉토리 객체 생성방법

> ORACLE DATA PUMP 사용
   
## 디렉도리 객체 생성 방법   
  
{% highlight css %}

SYS@orcl> startup force
ORACLE instance started.

Total System Global Area  577511424 bytes
Fixed Size                  1338000 bytes
Variable Size             436209008 bytes
Database Buffers          134217728 bytes
Redo Buffers                5746688 bytes
Database mounted.
Database opened.
SYS@orcl> select * from dba_directories;       --startup을 해야지 검색 가능

OWNER                          DIRECTORY_NAME
------------------------------ ------------------------------
DIRECTORY_PATH
--------------------------------------------------------------------------------
SYS                            ORACLE_OCM_CONFIG_DIR
/u01/app/oracle/product/11.2.0/dbhome_1/ccr/state

SYS                            DATA_PUMP_DIR
/u01/app/oracle/admin/orcl/dpdump/

SYS                            MEDIA_DIR
/u01/app/oracle/product/11.2.0/dbhome_1/demo/schema/product_media/

SYS                            XMLDIR
/ade/b/1191423112/oracle/rdbms/xml

SYS                            DATA_FILE_DIR
/u01/app/oracle/product/11.2.0/dbhome_1/demo/schema/sales_history/

SYS                            LOG_FILE_DIR
/u01/app/oracle/product/11.2.0/dbhome_1/demo/schema/log/

SYS                            SS_OE_XMLDIR
/u01/app/oracle/product/11.2.0/dbhome_1/demo/schema/order_entry/

SYS                            SUBDIR
/u01/app/oracle/product/11.2.0/dbhome_1/demo/schema/order_entry//2002/Sep


8 rows selected.

SYS@orcl> create or replace directory data_air     --home/oracle/data경로에 디렉터리 생성
  2  as '/home/oracle/data';

Directory created.

SYS@orcl> create or replace directory dirpump as '/home/oracle/data';

Directory created.

SYS@orcl> select directory_name, directory_path
  2  from dba_directories;

DIRECTORY_NAME
------------------------------
DIRECTORY_PATH
--------------------------------------------------------------------------------
DIRPUMP
/home/oracle/data                 --생성된 디렉터리

DATA_AIR
/home/oracle/data                 --생성된 디렉터리

SUBDIR
/u01/app/oracle/product/11.2.0/dbhome_1/demo/schema/order_entry//2002/Sep

SS_OE_XMLDIR
/u01/app/oracle/product/11.2.0/dbhome_1/demo/schema/order_entry/

LOG_FILE_DIR
/u01/app/oracle/product/11.2.0/dbhome_1/demo/schema/log/

DATA_FILE_DIR
/u01/app/oracle/product/11.2.0/dbhome_1/demo/schema/sales_history/

XMLDIR
/ade/b/1191423112/oracle/rdbms/xml

MEDIA_DIR
/u01/app/oracle/product/11.2.0/dbhome_1/demo/schema/product_media/

DATA_PUMP_DIR
/u01/app/oracle/admin/orcl/dpdump/

ORACLE_OCM_CONFIG_DIR
/u01/app/oracle/product/11.2.0/dbhome_1/ccr/state


10 rows selected.


SYS@orcl> ed
Wrote file afiedt.buf

1* grant read, write on directory data_air to hr,scott
SYS@orcl> /

Grant succeeded.

SYS@orcl> grant read, write on directory dirpump to hr, scott;

Grant succeeded.

SYS@orcl> grant read, write on directory data_air to sh;

Grant succeeded.

SYS@orcl> grant read, write on directory dirpump to sh;

Grant succeeded.
--(만약 계정이 account unlock이 걸렸을 경우 풀고 나서 grant해줘야함)
--select username, account_status from dba_users;
--alter user sh_identified by sh 
--account_unlock;
*/
-----------------------------------------------------------------------------------------------------------------------------------
data.zip
c:\share //data.zip 압출풀기
[orcl:~]$ su -
Password: 
[root@edydr1p0 ~]# cd /media/sf_sh*
[root@edydr1p0 sf_share]# ls
data.zip  mylabs.tar.gz

[root@edydr1p0 sf_share]# cp -R data /home/oracle
	--cp: cannot stat `data': No such file or directory

[root@edydr1p0 sf_share]# ls 
data  data.zip  mylabs.tar.gz

[root@edydr1p0 sf_share]# cp -R data/home/oracle			-- cp -R: 원본이 파일이면 그냥 복사되고 디렉터리라면 디렉터리 전체가 복사된다.
	cp: missing destination file operand after `data/home/oracle'
	Try `cp --help' for more information.
[root@edydr1p0 sf_share]# cp -R data /home/oracle
[root@edydr1p0 sf_share]# ls -al /home/oracle				-- ls -al : 파일 자세히 보는거 
total 57024
drwx------ 31 oracle oinstall     4096 Dec 23 12:12 .
drwxr-xr-x  5 root   root         4096 Dec 13 13:42 ..
-rw-------  1 oracle oinstall     4393 Dec 23 10:40 .ICEauthority
drwx------  2 oracle oinstall     4096 Feb 14  2019 .Trash
drwx------  5 oracle oinstall     4096 Oct 10  2012 .adobe
-rw-------  1 oracle oinstall     7891 Dec 20 18:14 .bash_history
...
[root@edydr1p0 sf_share]# chown -R oracle:oinstall /home/oracle/data				--chown : 해당 디렉토리의 소유자를 변경(=파일의 소유자와 소유 그룹을 변경)
																					--chown -R : 하위파일까지의 모둔 소유자를 oracle/소유그룹을 oinstall로 변경

[root@edydr1p0 sf_share]# ls -al /home/oracle
total 57024
drwx------ 31 oracle oinstall     4096 Dec 23 12:12 .
drwxr-xr-x  5 root   root         4096 Dec 13 13:42 ..
-rw-------  1 oracle oinstall     4393 Dec 23 10:40 .ICEauthority
drwx------  2 oracle oinstall     4096 Feb 14  2019 .Trash
drwx------  5 oracle oinstall     4096 Oct 10  2012 .adobe
...

[root@edydr1p0 sf_share]# exit
logout

[orcl:~]$ ls -al /home/oracle
total 57024
drwx------ 31 oracle oinstall     4096 Dec 23 12:12 .
drwxr-xr-x  5 root   root         4096 Dec 13 13:42 ..
-rw-------  1 oracle oinstall     4393 Dec 23 10:40 .ICEauthority
drwx------  2 oracle oinstall     4096 Feb 14  2019 .Trash
drwx------  5 oracle oinstall     4096 Oct 10  2012 .adobe
-rw-------  1 oracle oinstall     7891 Dec 20 18:14 .bash_history
...
[orcl:~]$ sql

SQL*Plus: Release 11.2.0.1.0 Production on Mon Dec 23 12:14:04 2019

Copyright (c) 1982, 2009, Oracle.  All rights reserved.

Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options

SYS@orcl> select * from dba_directories;

OWNER                          DIRECTORY_NAME
------------------------------ ------------------------------
DIRECTORY_PATH
--------------------------------------------------------------------------------
SYS                            ORACLE_OCM_CONFIG_DIR
/u01/app/oracle/product/11.2.0/dbhome_1/ccr/state

SYS                            DATA_PUMP_DIR
/u01/app/oracle/admin/orcl/dpdump/

SYS                            MEDIA_DIR
/u01/app/oracle/product/11.2.0/dbhome_1/demo/schema/product_media/

SYS                            XMLDIR
/ade/b/1191423112/oracle/rdbms/xml

SYS                            DATA_FILE_DIR
/u01/app/oracle/product/11.2.0/dbhome_1/demo/schema/sales_history/

SYS                            LOG_FILE_DIR
/u01/app/oracle/product/11.2.0/dbhome_1/demo/schema/log/

SYS                            SS_OE_XMLDIR
/u01/app/oracle/product/11.2.0/dbhome_1/demo/schema/order_entry/

SYS                            SUBDIR
/u01/app/oracle/product/11.2.0/dbhome_1/demo/schema/order_entry//2002/Sep

SYS                            DATA_AIR
/home/oracle/data

SYS                            DIRPUMP
/home/oracle/data


10 rows selected.

SYS@orcl> create or replace directory data_dir as '/home/oracle/data';
Directory created.

SYS@orcl> create or replace directory dirpump as '/home/oracle/data';
Directory created.

SYS@orcl> select * from dba_directories;

OWNER                          DIRECTORY_NAME
------------------------------ ------------------------------
DIRECTORY_PATH
--------------------------------------------------------------------------------
SYS                            ORACLE_OCM_CONFIG_DIR
/u01/app/oracle/product/11.2.0/dbhome_1/ccr/state

SYS                            DATA_PUMP_DIR
/u01/app/oracle/admin/orcl/dpdump/

SYS                            MEDIA_DIR
/u01/app/oracle/product/11.2.0/dbhome_1/demo/schema/product_media/

SYS                            XMLDIR
/ade/b/1191423112/oracle/rdbms/xml

SYS                            DATA_FILE_DIR
/u01/app/oracle/product/11.2.0/dbhome_1/demo/schema/sales_history/

SYS                            LOG_FILE_DIR
/u01/app/oracle/product/11.2.0/dbhome_1/demo/schema/log/

SYS                            SS_OE_XMLDIR
/u01/app/oracle/product/11.2.0/dbhome_1/demo/schema/order_entry/

SYS                            SUBDIR
/u01/app/oracle/product/11.2.0/dbhome_1/demo/schema/order_entry//2002/Sep

SYS                            DATA_AIR
/home/oracle/data

SYS                            DIRPUMP
/home/oracle/data

SYS                            DATA_DIR
/home/oracle/data


11 rows selected.

SYS@orcl> grant read, write on directory data_dir to sh,hr,scott;
Grant succeeded.

SYS@orcl> grant read, write on directory dirpump to sh, hr, scott;
Grant succeeded.

SYS@orcl> select username, account_status from dba_users
  2  where username='SH';

USERNAME                       ACCOUNT_STATUS
------------------------------ --------------------------------
SH                             EXPIRED & LOCKED							--expired 상태이면 비밀번호 재설정 하라고 뜸, locked면은 계정 접속 불가능

SYS@orcl> alter user sh identified by sh account unlock;				--계정 접속 가능

User altered.

SYS@orcl> select username, account_status from dba_users
  2  where username = 'SH';

USERNAME                       ACCOUNT_STATUS
------------------------------ --------------------------------
SH                             OPEN										--계정 풀림

SYS@orcl> cd /home/oracle/data
SP2-0734: unknown command beginning "cd /home/o..." - rest of line ignored.
SYS@orcl> exit
Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options

{% endhighlight %}

## ORACLE DATA PUMP 사용  

{% highlight css %}
[orcl:~]$ cd /home/oracle/data

[orcl:data]$ expdp userid=sh/sh directory=dirpump estimate_only=Y logfile=expdp_pump.log tables=SALES
		--dump file 크기 확인 하려고 하는 거

Export: Release 11.2.0.1.0 - Production on Mon Dec 23 13:26:44 2019

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options
 "SH"."SYS_EXPORT_TABLE_01":  userid=sh --******** directory=dirpump estimate_only=Y logfile=expdp_pump.log tables=SALES 
Estimate in progress using BLOCKS method...
Processing object type TABLE_EXPORT/TABLE/TABLE_DATA
.  estimated "SH"."SALES":"SALES_Q4_2001"                    2 MB
.  estimated "SH"."SALES":"SALES_Q1_1999"                 1024 KB
.  estimated "SH"."SALES":"SALES_Q3_2001"                 1024 KB
.  estimated "SH"."SALES":"SALES_Q1_2000"                  960 KB
.  estimated "SH"."SALES":"SALES_Q1_2001"                  960 KB
.  estimated "SH"."SALES":"SALES_Q2_2001"                  960 KB
...
Job "SH"."SYS_EXPORT_TABLE_01" successfully completed at 13:26:53

[orcl:data]$ expdp userid=sh/sh directory=dirpump job_name=datapump logfile=expdp_pump.log dumpfile=expdp_pump%U.dmp filesize=100m tables=SALES
		-- export(추출) 하는거 - SALES라는 테이블을 export 하는거

Export: Release 11.2.0.1.0 - Production on Mon Dec 23 13:48:42 2019

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options
Starting "SH"."DATAPUMP":  userid=sh --******** directory=dirpump job_name=datapump logfile=expdp_pump.log dumpfile=expdp_pump%U.dmp filesize=100m tables=SALES 
Estimate in progress using BLOCKS method...
Processing object type TABLE_EXPORT/TABLE/TABLE_DATA
Total estimation using BLOCKS method: 15.12 MB
Processing object type TABLE_EXPORT/TABLE/TABLE
Processing object type TABLE_EXPORT/TABLE/GRANT/OWNER_GRANT/OBJECT_GRANT
Processing object type TABLE_EXPORT/TABLE/COMMENT
Processing object type TABLE_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
Processing object type TABLE_EXPORT/TABLE/INDEX/FUNCTIONAL_AND_BITMAP/INDEX
Processing object type TABLE_EXPORT/TABLE/INDEX/STATISTICS/FUNCTIONAL_AND_BITMAP/INDEX_STATISTICS
Processing object type TABLE_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
. . exported "SH"."SALES":"SALES_Q4_2001"                2.257 MB   69749 rows
. . exported "SH"."SALES":"SALES_Q1_1999"                2.071 MB   64186 rows
. . exported "SH"."SALES":"SALES_Q3_2001"                2.130 MB   65769 rows
. . exported "SH"."SALES":"SALES_Q1_2000"                2.012 MB   62197 rows
. . exported "SH"."SALES":"SALES_Q1_2001"                1.965 MB   60608 rows
. . exported "SH"."SALES":"SALES_Q2_2001"                2.051 MB   63292 rows
...
******************************************************************************
Dump file set for SH.DATAPUMP is:
  /home/oracle/data/expdp_pump01.dmp
Job "SH"."DATAPUMP" successfully completed at 13:48:49

[orcl:data]$ ls -al *.dmp
-rw-r----- 1 oracle dba 31580160 Dec 23 13:48 expdp_pump01.dmp
[orcl:data]$ ls -lh *.dmp											-- -lh : human : 사람이 보기 쉽게 해주는 거
-rw-r----- 1 oracle dba 31M Dec 23 13:48 expdp_pump01.dmp

[orcl:data]$ expdp userid=sh/sh directory=dirpump  dumpfile=expdp_dump1.dmp logfile=expdp_dump.log tables=sales

Export: Release 11.2.0.1.0 - Production on Mon Dec 23 13:51:32 2019

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options
Starting "SH"."SYS_EXPORT_TABLE_01":  userid=sh--******** directory=dirpump dumpfile=expdp_dump1.dmp logfile=expdp_dump.log tables=sales 
Estimate in progress using BLOCKS method...
Processing object type TABLE_EXPORT/TABLE/TABLE_DATA
Total estimation using BLOCKS method: 15.12 MB
Processing object type TABLE_EXPORT/TABLE/TABLE
Processing object type TABLE_EXPORT/TABLE/GRANT/OWNER_GRANT/OBJECT_GRANT
Processing object type TABLE_EXPORT/TABLE/COMMENT
Processing object type TABLE_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
Processing object type TABLE_EXPORT/TABLE/INDEX/FUNCTIONAL_AND_BITMAP/INDEX
...
******************************************************************************
Dump file set for SH.SYS_EXPORT_TABLE_01 is:
  /home/oracle/data/expdp_dump1.dmp
Job "SH"."SYS_EXPORT_TABLE_01" successfully completed at 13:51:35

[orcl:data]$ ls -al *.dmp
-rw-r----- 1 oracle dba 31580160 Dec 23 13:51 expdp_dump1.dmp
-rw-r----- 1 oracle dba 31580160 Dec 23 13:48 expdp_pump01.dmp
[orcl:data]$ ls -lh *.dmp
-rw-r----- 1 oracle dba 31M Dec 23 13:51 expdp_dump1.dmp
-rw-r----- 1 oracle dba 31M Dec 23 13:48 expdp_pump01.dmp

[orcl:data]$ sql

SQL*Plus: Release 11.2.0.1.0 Production on Mon Dec 23 13:53:26 2019

Copyright (c) 1982, 2009, Oracle.  All rights reserved.

Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options

SYS@orcl> select count(*) from sh.sales;

  COUNT(*)
----------
    918843

SYS@orcl> truncate table sh.sales;


Table truncated.

SYS@orcl> SYS@orcl> select count(*) from sh.sales;

  COUNT(*)
----------
         0

--실습3--------------------------------------------------------------------------
[orcl:data]$ sql

SQL*Plus: Release 11.2.0.1.0 Production on Mon Dec 23 14:06:19 2019

Copyright (c) 1982, 2009, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options

SYS@orcl> select count(*) from sh.sales;

  COUNT(*)
----------
    918843

SYS@orcl> truncate table sh.sales;

Table truncated.

SYS@orcl> exit
Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options
[orcl:data]$ impdp userid=sh/sh directory=data_dir dumpfile=expdp_dump1.dmp logfile=impdp_pump.log tables=sales                           

Import: Release 11.2.0.1.0 - Production on Mon Dec 23 14:08:39 2019

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options
Master table "SH"."SYS_IMPORT_TABLE_01" successfully loaded/unloaded
Starting "SH"."SYS_IMPORT_TABLE_01":  userid=sh --******** directory=data_dir dumpfile=expdp_dump1.dmp logfile=impdp_pump.log tables=sales 
Processing object type TABLE_EXPORT/TABLE/TABLE
ORA-39151: Table "SH"."SALES" exists. All dependent metadata and data will be skipped due to table_exists_action of skip
Processing object type TABLE_EXPORT/TABLE/TABLE_DATA

[orcl:data]$ sql

SQL*Plus: Release 11.2.0.1.0 Production on Mon Dec 23 14:08:51 2019

Copyright (c) 1982, 2009, Oracle.  All rights reserved.

Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options

SYS@orcl> select count(*) from sh.sales;

  COUNT(*)
----------
         0

SYS@orcl> exit
Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options
[orcl:data]$ impdp userid=sh/sh directory=data_dir dumpfile=expdp_dump1.dmp logfile=impdp_pump.log tables=sales table_exits_action=append
LRM-00101: unknown parameter name 'table_exits_action'

[orcl:data]$ impdp userid=sh/sh directory=data_dir dumpfile=expdp_dump1.dmp logfile=impdp_pump.log tables=sales table_exists_action=append

Import: Release 11.2.0.1.0 - Production on Mon Dec 23 14:09:42 2019

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options
Master table "SH"."SYS_IMPORT_TABLE_01" successfully loaded/unloaded
Starting "SH"."SYS_IMPORT_TABLE_01":  userid=sh --******** directory=data_dir dumpfile=expdp_dump1.dmp logfile=impdp_pump.log tables=sales table_exists_action=append 
Processing object type TABLE_EXPORT/TABLE/TABLE
ORA-39152: Table "SH"."SALES" exists. Data will be appended to existing table but all dependent metadata will be skipped due to table_exists_action of append
Processing object type TABLE_EXPORT/TABLE/TABLE_DATA
. . imported "SH"."SALES":"SALES_Q4_2001"                2.257 MB   69749 rows
. . imported "SH"."SALES":"SALES_Q1_1999"                2.071 MB   64186 rows
. . imported "SH"."SALES":"SALES_Q3_2001"                2.130 MB   65769 rows
. . imported "SH"."SALES":"SALES_Q1_2000"                2.012 MB   62197 rows
. . imported "SH"."SALES":"SALES_Q1_2001"                1.965 MB   60608 rows
. . imported "SH"."SALES":"SALES_Q2_2001"                2.051 MB   63292 rows
. . imported "SH"."SALES":"SALES_Q3_1999"                2.166 MB   67138 rows
...

[orcl:data]$ sql

SQL*Plus: Release 11.2.0.1.0 Production on Mon Dec 23 14:10:18 2019

Copyright (c) 1982, 2009, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options

SYS@orcl> select count(*) from sh.sales;

  COUNT(*)
----------
    918843

	--export 한 sh.sales를 import 하는거 
------------------------------------------------------------------------------------------------------------------
--실습4-------------------------------------------------------------------------
--SALES가 들어간 인덱스 빼고 export할거야

[orcl:data]$ cat ~/data/expdb_pump2.par
userid=sh/sh
directory=dirpump
job_name=datapump
logfile=expdp_pump2.log
dumpfile=expdp_pump2%U.dmp
filesize=100m
schemas=SH
Exclude=INDEX:"LIKE 'SALES%'"
[orcl:data]$ expdp parfile=~/data/expdb_pump2.par

Export: Release 11.2.0.1.0 - Production on Mon Dec 23 14:24:00 2019

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options
Starting "SH"."DATAPUMP":  sh --******** parfile=/home/oracle/data/expdb_pump2.par 
Estimate in progress using BLOCKS method...
Processing object type SCHEMA_EXPORT/TABLE/TABLE_DATA
Total estimation using BLOCKS method: 61.18 MB
Processing object type SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
Processing object type SCHEMA_EXPORT/TABLE/TABLE
Processing object type SCHEMA_EXPORT/TABLE/GRANT/OWNER_GRANT/OBJECT_GRANT
Processing object type SCHEMA_EXPORT/TABLE/INDEX/INDEX
. . exported "SH"."SALES":"SALES_Q3_1999"                2.166 MB   67138 rows
...
******************************************************************************
Dump file set for SH.DATAPUMP is:
  /home/oracle/data/expdp_pump201.dmp
Job "SH"."DATAPUMP" successfully completed at 14:24:41

--------------------------------------------------------------------------------
--실습 5
	--sh의 스키마 정보를 export해서 smith한테 import해줄거야

SYS@orcl> create user smith identified by oracle;

User created.

SYS@orcl> grant dba to smith;

Grant succeeded.

SYS@orcl> exit
Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options

[orcl:data]$ impdp userid=system/oracle directory=dirpump job_name=datapump logfile=impdp_pump2.log dumpfile=expdp_pump2%U.dmp REMAP_SCHEMA=SH:SMITH

Import: Release 11.2.0.1.0 - Production on Mon Dec 23 14:53:55 2019

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.

UDI-01017: operation generated ORACLE error 1017
ORA-01017: invalid username/password; logon denied

Username: system
Password: 

Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options
Master table "SYSTEM"."DATAPUMP" successfully loaded/unloaded
Starting "SYSTEM"."DATAPUMP":  userid=system --******** directory=dirpump job_name=datapump logfile=impdp_pump2.log dumpfile=expdp_pump2%U.dmp REMAP_SCHEMA=SH:SMITH 
Processing object type SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
Processing object type SCHEMA_EXPORT/TABLE/TABLE
Processing object type SCHEMA_EXPORT/TABLE/TABLE_DATA
. . imported "SMITH"."CUSTOMERS"                         9.853 MB   55500 rows
. . imported "SMITH"."SUPPLEMENTARY_DEMOGRAPHICS"        697.3 KB    4500 rows
. . imported "SMITH"."SALES":"SALES_Q1_1999"             2.071 MB   64186 rows
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
Processing object type SCHEMA_EXPORT/TABLE/INDEX/FUNCTIONAL_AND_BITMAP/INDEX
Processing object type SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/FUNCTIONAL_AND_BITMAP/INDEX_STATISTICS
Processing object type SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
Processing object type SCHEMA_EXPORT/TABLE/INDEX/DOMAIN_INDEX/INDEX
Processing object type SCHEMA_EXPORT/MATERIALIZED_VIEW
Processing object type SCHEMA_EXPORT/DIMENSION
Job "SYSTEM"."DATAPUMP" successfully completed at 14:54:33

----결과 확인
SYS@orcl> 
 select owner, object_name, object_type
 from dba_objects
 where owner = 'SMITH' order by 3,2;

 --실습 6
 [orcl:data]$ expdp userid=system/oracle_4U directory=dirpump dumpfile=emp.dmp tables=scott.emp

Export: Release 11.2.0.1.0 - Production on Mon Dec 23 15:09:30 2019

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options
Starting "SYSTEM"."SYS_EXPORT_TABLE_01":  userid=system --******** directory=dirpump dumpfile=emp.dmp tables=scott.emp 
Estimate in progress using BLOCKS method...
Processing object type TABLE_EXPORT/TABLE/TABLE_DATA
Total estimation using BLOCKS method: 64 KB
Processing object type TABLE_EXPORT/TABLE/TABLE
Processing object type TABLE_EXPORT/TABLE/GRANT/OWNER_GRANT/OBJECT_GRANT
Processing object type TABLE_EXPORT/TABLE/INDEX/INDEX
Processing object type TABLE_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
Processing object type TABLE_EXPORT/TABLE/AUDIT_OBJ
Processing object type TABLE_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
Processing object type TABLE_EXPORT/TABLE/TRIGGER
. . exported "SCOTT"."EMP"                               8.570 KB      14 rows
Master table "SYSTEM"."SYS_EXPORT_TABLE_01" successfully loaded/unloaded
******************************************************************************
Dump file set for SYSTEM.SYS_EXPORT_TABLE_01 is:
  /home/oracle/data/emp.dmp
Job "SYSTEM"."SYS_EXPORT_TABLE_01" successfully completed at 15:09:35

[orcl:data]$ expdp userid=system/oracle_4U directory=dirpump dumpfile=user_tbs.dmp tablespaces=users

Export: Release 11.2.0.1.0 - Production on Mon Dec 23 15:10:30 2019

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options
Starting "SYSTEM"."SYS_EXPORT_TABLESPACE_01":  userid=system --******** directory=dirpump dumpfile=user_tbs.dmp tablespaces=users 
Estimate in progress using BLOCKS method...
Processing object type TABLE_EXPORT/TABLE/TABLE_DATA
Total estimation using BLOCKS method: 1.062 MB
Processing object type TABLE_EXPORT/TABLE/TABLE
Processing object type TABLE_EXPORT/TABLE/GRANT/OWNER_GRANT/OBJECT_GRANT
Processing object type TABLE_EXPORT/TABLE/INDEX/INDEX
Processing object type TABLE_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
. . exported "SMITH"."DIMENSION_EXCEPTIONS"                  0 KB       0 rows
Master table "SYSTEM"."SYS_EXPORT_TABLESPACE_01" successfully loaded/unloaded
******************************************************************************
Dump file set for SYSTEM.SYS_EXPORT_TABLESPACE_01 is:
  /home/oracle/data/user_tbs.dmp
Job "SYSTEM"."SYS_EXPORT_TABLESPACE_01" successfully completed at 15:10:46

[orcl:data]$ ls -al *.dmp
-rw-r----- 1 oracle dba   135168 Dec 23 15:09 emp.dmp
-rw-r----- 1 oracle dba   835584 Dec 23 15:10 user_tbs.dmp

[orcl:data]$ ls -lh *.dmp
-rw-r----- 1 oracle dba 132K Dec 23 15:09 emp.dmp
-rw-r----- 1 oracle dba 816K Dec 23 15:10 user_tbs.dmp

 scott 사용자의 emp table 삭제 후 dump 화일 이용해서 복구한다.

[orcl:data]$ sqlplus scott/tiger

SQL> select * from emp;
SQL> drop table emp;
SQL> exit

$  impdp userid=scott/tiger directory=dirpump dumpfile=emp.dmp tables=scott.emp

다시 scott 사용자의 emp table 삭제 후 테이블스페이스  dump 화일 이용해서 복구한다.

[orcl:data]$ sqlplus scott/tiger
SQL> select * from emp;
SQL> drop table emp;
SQL> exit

$  impdp userid=system/oracle_4U directory=dirpump dumpfile=users_tbs.dmp tables=scott.emp

복구후 확인 

[orcl:data]$ sqlplus scott/tiger
SQL> select * from emp;
SQL> exit

 
{% endhighlight %}
## Buttons

Make any link standout more when applying the `.btn` class.

{% highlight html %}
<a href="#" class="btn btn-success">Success Button</a>
{% endhighlight %}

<div markdown="0"><a href="#" class="btn btn-info">Info Button</a></div>

## Notices

**Watch out!** You can also add notices by appending `{: .notice}` to a paragraph.
{: .notice}
