---
layout: post
title: ORACLE BACKUP 01
excerpt: " BACKUP & RECOVERY CONCEPT"
categories: [ORACLE BACKUP]
comments: true
---


## REDO

 * DB에 일어나는 모든 변경      
 * 용도 : RECOVERY     
 * LOG BUFFER : SGA의 REDO LOG를 기록하는 메모리     
 * LGWR : commit시 log buffer의 리두를 redo log file에 기록      
 
 * redo log file(online)  : 복구시 마지막 체크포인트 이후의 리두가 적용  (instance failure -> instance recovery)      
 * archived redo log file (offline) : media failure -> media recovery 시에 사용  
 
 
### INSTANCE RECOVERY TUNING

 * fast_start_mttr_target = 300 (5분)    
  - instance recovery 걸리는 시간을 균질화하기 위한 파라미터      
  - fast_start_mttr_target 설정값 이내로 instance recovery 완료할 수 있도록 자주 체크포인트를 일으키라는 뜻      

## CHECKPOINT  
   
 - 메모리(db buffer cache)와 disk상의 file(datafile)을 동기화 하는 과정    
 - CKPT : control file과 datafile header에 checkpoint info. 을 기록하는 것     
 
 * THREAD CHECKPOINT (database checkpoint), full     
  - redo와 연관된 모든 dirty buffer를 기록   
  - shutdown (abort 외)    
  - log switch로 인한 체크포인트   
  - alter system checkpoint;   
  
 * TABLESPACE, DATAFILE CHECKPOINT 
   - alter tablespace read only; offline;   
   
 * INCREMENTAL CHECKPOINT  
  - aging alg  : aging 알고리즘에 따라서 db buffer cache의 내용을 dbwr가 datafile에 쓰고 checkpoint position을 점진적으로 변경   
  - ckpt가 3초 간격으로 dutty buffer가 있는지를 확인하고 dbwr에 시그날 보냄   
  - target RBA 계산하여 checkpoint queue의 dutty buffer를 내림    
  - RBA(Redo Byte Address) : 리두 로그 파일에서 리두 레코드가 물리적으로 위치한 값   
  - CHECKPOINT POSITION은 움직임 (ckpt가 controlfile에 기록, not datafile header)      
  
 * LRU alg  
  - 서버프로세스가 LRU LIST를 뒤지다가 FREE/CLEAN을 찾지 못하였을 경우 (_db_block_max_scan_pct ) LRU 쪽에서 만난 dirty buffer를 우선적으로 dbwr가 write    
  - CHECKPOINT POSITION은 변경하지 않음 (변경된 순서가 아니기 때문에 나머지 CHECKPOINT 작업은 CHECKPOINT QUEUE 내의 순서대로 기록)   
  
## RECOVERY 가능성을 높이기 위한 설정  
   
  *  MULTIPLEXING : 컨트롤 파일의 다중화, 리두로그 파일의 다중화   
  
{% highlight css %}

-- 컨트롤 파일의 다중화  
/*
File System 인 경우
1) spfile 을 수정한다.
alter system set control_files =    '/u01/app/oracle/orcl2/control01.ctl',  '/u01/app/oracle/orcl2/control02.ctl' 
scope=spfile;
2) db를 종료
3) os 상에서 컨트롤 파일을 카피
cp  /u01/app/oracle/oradata/orcl2/control01.ctl  /u01/app/oracle/oradata/orcl2/control02.ctl
4) startup
*/

--spfile 수정 
SYS@orcl> select name from v$controlfile;

NAME
---------------------------------------------------------------------
+DATA/orcl/controlfile/current.260.796857737
+FRA/orcl/controlfile/current.256.796857739

   alter system set control_files = '+DATA/orcl/controlfile/current.260.796857737','+FRA/orcl/controlfile/current.256.796857739','+DATA'
                           scope=spfile;
--db 종료 
  shutdown immediate
  startup nomount

--nomount 상태에서 컨트롤 파일을 카피(RMAN  에서 restore controlfile 명령어 사용)
SQL > startup nomount

  $ rman target /
  RMAN>     restore controlfile from '+FRA/orcl/controlfile/current.256.796857739';

--startup
SYS@orcl > alter database mount;

SYS@orcl> select name from v$controlfile;


SYS@orcl> alter database open;

--리두로그 파일의 다중화  

--file system인 경우 : orcl2 DB
SYS@orcl2> select group#, sequence#, bytes, members, status, archived
  2  from v$log;

    GROUP#  SEQUENCE#      BYTES    MEMBERS STATUS           ARC
---------- ---------- ---------- ---------- ---------------- ---
         1          7   52428800          1 CURRENT          NO
         2          5   52428800          1 INACTIVE         NO
         3          6   52428800          1 INACTIVE         NO

@log   
 select select group#, sequence#, status, archived, members,
               bytes/1024/1024 mb        from v$log;
@logfile 
       select group#, member, status from v$logfile;

-- 리두로그 멤버 추가 
  alter  database add logfile member 
  '/u01/app/oracle/flash_recovery_area/orcl2/redo01b.log' to group 1,
 '/u01/app/oracle/flash_recovery_area/orcl2/redo02b.log' to group 2,
'/u01/app/oracle/flash_recovery_area/orcl2/redo03b.log' to group 3;


alter system switch logfile;	--로그스위치 일으키는 명령어. 새로추가한 리두멤버가 invalid상태였는데 로그스위치를 하게되면 새로만든 멤버를 사용하게되면서 invalid상태가 없어짐
alter system checkpoint;	--체크포인트 명령어

--리두로그 그룹 추가 

alter database add logfile group 4 
    '/u01/app/oracle/orcl2/redo04.log' size 5m ;

 alter  database add logfile member 
  '/u01/app/oracle/flash_recovery_area/orcl2/redo04b.log' to group 4;

 @log

 @logfile

-- 리두로그 그룹 삭제

select * from v$log; -- current group 확인후 
 alter database drop logfile group 4;

-- 리두로그 멤버 삭제

select * from v$log; -- current group 확인후
                        현재 1번 그룹이 current 라면 

  alter  database drop logfile member 
 '/u01/app/oracle/flash_recovery_area/orcl2/redo02b.log' ,
'/u01/app/oracle/flash_recovery_area/orcl2/redo03b.log' ;

 alter system switch logfile;

  alter  database drop logfile member 
 '/u01/app/oracle/flash_recovery_area/orcl2/redo01b.log' ;

--ASM system인 경우 : orcl DB
-- 리두로그 멤버 추가 

sys@ orcl >  ALTER DATABASE ADD LOGFILE MEMBER '+FRA' TO GROUP 2;

--리두로그 그룹 추가 
  alter database add logfile group 4 
    '+DATA' size 5m ;

-- 리두로그 그룹 삭제
  ALTER DATABASE ADD LOGFILE MEMBER '+FRA' TO GROUP 4;
{% endhighlight %} 

 * DB의 LOG MODE의 변경 : NOARCHIVELOG -> ARCHIVELOG    
   - DB의 정상 종료   
   - DB MOUNT     
   - DB 로그 모드의 변경     
   
 * 정기적 백업  
  - archive log mode - cold, hot   
   * hot backup - datafile   
   * control file - online backup   
   * control file trace - 콘트롤 파일을 재생성 할 수 있는 SQL 스크립트   
   
## BACKUP  
  
   * Redo log file : 오라클이 데이터베이스에서 발생한 모든 변경사항을 기록하는 파일    
   * NoArchive Log Mode (Defult)   
    - ORACLE에서 작업하는 양이 많아지면 리두로그파일에 기록하는 내용도 많아짐   
	그러면 데이터를 기록하기 위해서 리두로그파일을 늘려야하는 일이 발생하지만 오라클 리두로그파일은 파일증가가 아닌     
	몇 개의 리두로그 파일을 생성해놓구 번갈아 가면서 기록하는 구조로 되어있음     
	->이런 구조라면 기록을 하게 되면서 새로운 작업의 내용이 예전의 작업 내용을 덮어쓰는 경우가 발생    
	->그래서 작업 내용을 잃게 된다는 단점이 생기고, 데이터 손실발생(복구X)    
	- NoArchive Log Mode는 체크포인트 발생 후 즉시 재사용가능   
	- online backup 불가능    
	- 전체 데이터가 과거로 돌아감     
  * ARCHIVE LOG MODE   
   - NoArchive Log Mode의 단점을 해결하기 위해서 리두로그파일의 내용을 다른 디렉터리에 자동 복사를 해서 저장할수 있도록 하는 방법   
   - 리두로그 파일이 꽉 차게 되면 체크포인트를 수행->리두로그 파일 아카이브-> 아카이브 수행중이라면 즉시 재사용을 불가능하게함   
   - online backup가능   
   - 특정 시점(번호, 아카이브)으로부터 데이터베이스를 복원할 수 있다는 장점   
   
  * 백업 모드 
   1. offline(cold)  
    - 데이터베이스가 종료되어야만 사용 가능한 백업방식  
	- 백업방식이 가장 빠르지만 데이터베이스를 중단하고 백업해야한다는 단점 
    - 모든 화일들은 동일한 시점의 CHECKPOINT SCN을 갖음  
    - ARCHIVELOG, NOARCHIVELOG MODE 둘다 가능  
	
   2. online(hot)  
    - 데이터파일과 컨트롤 파일만 가능 (로그 파일 불가)   
    - 데이터베이스가 운영중(OPEN)인 상태에서 백업하는 방법   
	- 커맨드상에서 ‘Begin Backup 테이블스페이스’를 입력하면 해당 데이터파일이 물리적으로 복사   
	- Hot Backup이 수행되는 순간 모든 블록을 로그에 남기는 방식   
	- NOARCHIVELOG MODE라면 ONLINE REDO LOG를 재사용하게 되므로 후에 Recovery가 불가능 할 수 있기 때문에 Oracle은 NOARCHIVE LOG MODE에서는 Online Backup을 불가능 하게함    

{% highlight css %}	

SYS@orcl> show parameter mttr

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
fast_start_mttr_target               integer     0

SYS@orcl> exit
Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options
[orcl:~]$ . oraenv
ORACLE_SID = [orcl] ? orcl2
The Oracle base for ORACLE_HOME=/u01/app/oracle/product/11.2.0/dbhome_1 is /u01/app/oracle
[orcl2:~]$ sql

SQL*Plus: Release 11.2.0.1.0 Production on Tue Dec 24 12:16:03 2019

Copyright (c) 1982, 2009, Oracle.  All rights reserved.

Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

SYS@orcl2> show parameter mttr

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
fast_start_mttr_target               integer     0
SYS@orcl2> select name from v$datafile;

NAME
--------------------------------------------------------------------------------
/u01/app/oracle/orcl2/system01.dbf
/u01/app/oracle/orcl2/sysaux01.dbf
/u01/app/oracle/orcl2/undotbs01.dbf
/u01/app/oracle/orcl2/users01.dbf
/u01/app/oracle/orcl2/example01.dbf

SYS@orcl2> select member from v$logfile;

MEMBER
--------------------------------------------------------------------------------
/u01/app/oracle/orcl2/redo03.log
/u01/app/oracle/orcl2/redo02.log
/u01/app/oracle/orcl2/redo01.log

SYS@orcl2> show parameter control_files;

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
control_files                        string      /u01/app/oracle/orcl2/control0
                                                 1.ctl, /u01/app/oracle/flash_r
                                                 ecovery_area/orcl2/control02.c
                                                 tl
SYS@orcl2> show parameter spfile

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
spfile                               string      /u01/app/oracle/product/11.2.0
                                                 /dbhome_1/dbs/spfileorcl2.ora

SYS@orcl2> select name from v$controlfile; 

NAME
--------------------------------------------------------------------------------
/u01/app/oracle/orcl2/control01.ctl
/u01/app/oracle/flash_recovery_area/orcl2/control02.ctl

SYS@orcl2> alter system set control_files='/u01/app/oracle/orcl2/control01.ctl'  
  2  scope=spfile;
System altered.

SYS@orcl2> shutdown immediate 
Database closed.
Database dismounted.
ORACLE instance shut down.
SYS@orcl2> startup 
ORACLE instance started.

Total System Global Area  431038464 bytes
Fixed Size                  1337016 bytes
Variable Size             155191624 bytes
Database Buffers          268435456 bytes
Redo Buffers                6074368 bytes
Database mounted.
Database opened.

SYS@orcl2> show parameter control_files

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
control_files                        string      /u01/app/oracle/orcl2/control0
                                                 1.ctl
SYS@orcl2> select name from v$controlfile;

NAME
--------------------------------------------------------------------------------
/u01/app/oracle/orcl2/control01.ctl

SYS@orcl2> select name from v$datafile;

NAME
--------------------------------------------------------------------------------
/u01/app/oracle/orcl2/system01.dbf
/u01/app/oracle/orcl2/sysaux01.dbf
/u01/app/oracle/orcl2/undotbs01.dbf
/u01/app/oracle/orcl2/users01.dbf
/u01/app/oracle/orcl2/example01.dbf

SYS@orcl2> ed
Wrote file afiedt.buf

  1  select name from v$datafile
  2  union all
  3  select name from v$controlfile
  4  union all
  5* select member from v$logfile
SYS@orcl2> /

NAME
--------------------------------------------------------------------------------
/u01/app/oracle/orcl2/system01.dbf
/u01/app/oracle/orcl2/sysaux01.dbf
/u01/app/oracle/orcl2/undotbs01.dbf
/u01/app/oracle/orcl2/users01.dbf
/u01/app/oracle/orcl2/example01.dbf
/u01/app/oracle/orcl2/control01.ctl
/u01/app/oracle/orcl2/redo03.log
/u01/app/oracle/orcl2/redo02.log
/u01/app/oracle/orcl2/redo01.log

9 rows selected.

SYS@orcl2> exit
Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
[orcl2:~]$ sql

SQL*Plus: Release 11.2.0.1.0 Production on Tue Dec 24 15:53:27 2019

Copyright (c) 1982, 2009, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

SYS@orcl2> select name from v$datafile
  2  union all   
  3  select name from v$controlfile
  4  union all 
  5  select member from v$logfile;

NAME
--------------------------------------------------------------------------------
/u01/app/oracle/orcl2/system01.dbf
/u01/app/oracle/orcl2/sysaux01.dbf
/u01/app/oracle/orcl2/undotbs01.dbf
/u01/app/oracle/orcl2/users01.dbf
/u01/app/oracle/orcl2/example01.dbf
/u01/app/oracle/orcl2/control01.ctl
/u01/app/oracle/orcl2/redo03.log
/u01/app/oracle/orcl2/redo02.log
/u01/app/oracle/orcl2/redo01.log

9 rows selected.

SYS@orcl2> ed
Wrote file afiedt.buf

  1  select name from v$datafile
  2  union all
  3  select name from v$controlfile
  4  union all
  5* select member from v$logfile
SYS@orcl2> save db_list
Created file db_list.sql
SYS@orcl2> ed db_list
Wrote file afiedt.buf

  1  select 'cp'||name||' /home/oracle/noarch' from v$datafile
  2  union all
  3  select 'cp'||name||'/home/oracle/noarch'  from v$controlfile
  4  union all
  5* select 'cp'|| member|| '/home/oracle/noarch' from v$logfile

SYS@orcl2> ed
Wrote file afiedt.buf
  1  set head off feed off
  2  spool backup.sh
  3  select 'cp'||name||' /home/oracle/noarch' from v$datafile
  4  union all
  5  select 'cp'||name||'/home/oracle/noarch'  from v$controlfile
  6  union all
  7  select 'cp'|| member|| '/home/oracle/noarch' from v$logfile
  8  spool off
  9* set head on feed on
 10  
SYS@orcl2> save db_list

SYS@orcl2> @db_list

NAME
--------------------------------------------------------------------------------
/u01/app/oracle/orcl2/system01.dbf
/u01/app/oracle/orcl2/sysaux01.dbf
/u01/app/oracle/orcl2/undotbs01.dbf
/u01/app/oracle/orcl2/users01.dbf
/u01/app/oracle/orcl2/example01.dbf
/u01/app/oracle/orcl2/control01.ctl
/u01/app/oracle/orcl2/redo03.log
/u01/app/oracle/orcl2/redo02.log
/u01/app/oracle/orcl2/redo01.log

9 rows selected.

SYS@orcl2> ed
Wrote file afiedt.buf

  1  select name from v$datafile
  2  union all
  3  select name from v$controlfile
  4  union all
  5* select member from v$logfile
  6  

SYS@orcl2> @db_list;

cp /u01/app/oracle/orcl2/system01.dbf /home/oracle/noarch
cp /u01/app/oracle/orcl2/sysaux01.dbf /home/oracle/noarch
cp /u01/app/oracle/orcl2/undotbs01.dbf /home/oracle/noarch
cp /u01/app/oracle/orcl2/users01.dbf /home/oracle/noarch
cp /u01/app/oracle/orcl2/example01.dbf /home/oracle/noarch
cp /u01/app/oracle/orcl2/control01.ctl /home/oracle/noarch
cp /u01/app/oracle/orcl2/redo03.log /home/oracle/noarch
cp /u01/app/oracle/orcl2/redo02.log /home/oracle/noarch
cp /u01/app/oracle/orcl2/redo01.log /home/oracle/noarch

SYS@orcl2> !ls *.sh
backup.sh  orcl.sh

SYS@orcl2> exit
Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

[orcl2:~]$ cp db_list.sql /home/oracle/test
[orcl2:~]$ ls *.sql
CloneRmanRestore.sql  emp_list.sql     log.sql             postScripts.sql
cloneDBCreation.sql   lockAccount.sql  orcl.sql            rmanRestoreDatafiles.sql
db_list.sql           lock_info.sql    postDBCreation.sql  sal.sql
[orcl2:~]$ rm db_list.sql
[orcl2:~]$ cd test
[orcl2:test]$ ls
ORCL  datafile.sql  db_list.sql  log.sql
[orcl2:test]$ sql

SQL*Plus: Release 11.2.0.1.0 Production on Tue Dec 24 16:07:32 2019

Copyright (c) 1982, 2009, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

SYS@orcl2> ed db_list

create pfile from spfile;
set head off feed off
spool backup.sh
select 'cp -av $ORACLE_HOME/dbs/initorcl2.ora/home/oracle/noarch'
from dual
union all
select 'cp -av ' ||name||' /home/oracle/noarch'  from v$datafile
union all
select 'cp -av ' ||name||' /home/oracle/noarch'  from v$controlfile
union all
select 'cp -av ' || member||' /home/oracle/noarch' from v$logfile;
spool off
set head on feed on

SYS@orcl2> @db_list

File created.

cp -av $ORACLE_HOME/dbs/initorcl2.ora /home/oracle/noarch
cp -av /u01/app/oracle/orcl2/system01.dbf /home/oracle/noarch
cp -av /u01/app/oracle/orcl2/sysaux01.dbf /home/oracle/noarch
cp -av /u01/app/oracle/orcl2/undotbs01.dbf /home/oracle/noarch
cp -av /u01/app/oracle/orcl2/users01.dbf /home/oracle/noarch
cp -av /u01/app/oracle/orcl2/example01.dbf /home/oracle/noarch
cp -av /u01/app/oracle/orcl2/control01.ctl /home/oracle/noarch
cp -av /u01/app/oracle/orcl2/redo03.log /home/oracle/noarch
cp -av /u01/app/oracle/orcl2/redo02.log /home/oracle/noarch
cp -av /u01/app/oracle/orcl2/redo01.log /home/oracle/noarch

SYS@orcl2> ed db_list

create pfile from spfile;
set head off feed off
spool backup.sh
select 'cp -av $ORACLE_HOME/dbs/initorcl2.ora/home/oracle/noarch'
from dual
union all
select 'cp -av ' ||name||' /home/oracle/noarch'  from v$datafile
union all
select 'cp -av ' ||name||' /home/oracle/noarch'  from v$controlfile
union all
select 'cp -av ' || member||' /home/oracle/noarch' from v$logfile;
spool off
set head on feed on
shutdown immediate

SYS@orcl2> @db_list

File created.
cp -av $ORACLE_HOME/dbs/initorcl2.ora/home/oracle/noarch
cp -av /u01/app/oracle/orcl2/system01.dbf /home/oracle/noarch
cp -av /u01/app/oracle/orcl2/sysaux01.dbf /home/oracle/noarch
cp -av /u01/app/oracle/orcl2/undotbs01.dbf /home/oracle/noarch
cp -av /u01/app/oracle/orcl2/users01.dbf /home/oracle/noarch
cp -av /u01/app/oracle/orcl2/example01.dbf /home/oracle/noarch
cp -av /u01/app/oracle/orcl2/control01.ctl /home/oracle/noarch
cp -av /u01/app/oracle/orcl2/redo03.log /home/oracle/noarch
cp -av /u01/app/oracle/orcl2/redo02.log /home/oracle/noarch
cp -av /u01/app/oracle/orcl2/redo01.log /home/oracle/noarch

Database closed.
Database dismounted.
ORACLE instance shut down.
SYS@orcl2> exit
Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
[orcl2:test]$ cd
[orcl2:~]$ mkdir naorch
[orcl2:~]$ cd test
[orcl2:test]$ . backup.sh
`/u01/app/oracle/product/11.2.0/dbhome_1/dbs/initorcl2.ora' -> `/home/oracle/noarch'
`/u01/app/oracle/orcl2/system01.dbf' -> `/home/oracle/noarch'
`/u01/app/oracle/orcl2/sysaux01.dbf' -> `/home/oracle/noarch'
`/u01/app/oracle/orcl2/undotbs01.dbf' -> `/home/oracle/noarch'
`/u01/app/oracle/orcl2/users01.dbf' -> `/home/oracle/noarch'
`/u01/app/oracle/orcl2/example01.dbf' -> `/home/oracle/noarch'
`/u01/app/oracle/orcl2/control01.ctl' -> `/home/oracle/noarch'
`/u01/app/oracle/orcl2/redo03.log' -> `/home/oracle/noarch'
`/u01/app/oracle/orcl2/redo02.log' -> `/home/oracle/noarch'
`/u01/app/oracle/orcl2/redo01.log' -> `/home/oracle/noarch'          --백업 완료!

{% endhighlight %} 	
 
## RECOVERY 종류   

 * COMPLETE RECOVERY : DATA의 유실 없이 현재까지 모두 복구     
 * INCOMPLETE RECOVERY : 과거의 특정 시점으로 복구    
 
### 백업과 복원과 복구의 차이점 

 * 백업 -> 원본파일 COPY  
 * 복원 -> 원본 파일이 손상되어서 백업받은 파일 가져오는 작업   
 * 복구 -> 복원한 파일에 로그를 적용해서 최신 파일로 만들어주는 작업   
 
## Buttons

Make any link standout more when applying the `.btn` class.

{% highlight html %}
<a href="#" class="btn btn-success">Success Button</a>
{% endhighlight %}

<div markdown="0"><a href="#" class="btn btn-info">Info Button</a></div>

## Notices

**Watch out!** You can also add notices by appending `{: .notice}` to a paragraph.
{: .notice}
