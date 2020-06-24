---
layout: post
title: ORACLE BACKUP 03
excerpt: "ARCHIVE LOG MODE & NOARCHIVE LOG MODE"
categories: [ORACLE BACKUP]
comments: true
---


## DATAFILE_SYSTEM

> 시나리오 : SYSTEM TABLESPACE 장애/복구 (CRITICAL FILE의 손실 -> OFFLINE RECOVERY, COMPLETE RECOVERY)

{% highlight css %}

--1) db 작업 발생
SYS. SQL> create table  sys.test1(id number);
Table created.

SQL> insert into sys.test1(id) values(1);

1 row created.

SQL> commit;
SQL > select group#, sequence#, status,archived
      from v$log;  ==> current 확인,  sequence# 확인
     
SQL > alter system switch logfile;
SQL > alter system switch logfile;
SQL > alter system switch logfile;
SQL > select group#, sequence#, status,archived
      from v$log;  ==> current 확인,  sequence# 확인
	  
SQL > !ls /home/oracle/arch1  ==> archive 확인

--2) 장애 발생

SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.


SQL> ! rm /u01/app/oracle/orcl2/system01.dbf

SQL> startup
ORACLE instance started.

Total System Global Area  849530880 bytes
Fixed Size                  1339824 bytes
Variable Size             553651792 bytes
Database Buffers          289406976 bytes
Redo Buffers                5132288 bytes
Database mounted.
ORA-01157: cannot identify/lock data file 1 - see DBWR trace file
ORA-01110: data file 1: '/u01/app/oracle/orcl2/system01.dbf'

SQL> select status from v$instance;

STATUS
------------
MOUNTED


SQL> select * from v$recover_file; 	--장애가 있는 파일을 조회

     FILE# ONLINE  ONLINE_ ERROR                                                                CHANGE# TIME
---------- ------- ------- ----------------------------------------------------------------- ---------- ---------
         1 ONLINE  ONLINE  FILE NOT FOUND                                                             0


--장애 발생한 데이타 파일이 어느 테이블스페이스에 속한 파일인지 확인

SYS@orcl2> get datafile_tbs
   SELECT FILE#,f.TS#,f.NAME,STATUS, t.name
    FROM V$DATAFILE f  ,v$tablespace t
 where f.ts#=t.ts#;


--3) 복구 
--복원
SQL> !cp -av /home/oracle/hot/system01.dbf /u01/app/oracle/orcl2/system01.dbf

`/home/oracle/backup/arch/hot/system01.dbf' -> `/u01/app/oracle/orcl2/system01.dbf'

SQL> select * from v$recover_file; 

SQL> recover tablespace system; 
     recover database;

-- archive log 화일 적용 메시지 나옴
--리턴 키를 입력

Media recovery complete.

SQL> alter database open;

Database altered.

SQL> select status from v$instance;

STATUS
------------
OPEN

SQL> select * from v$recover_file;

no rows selected

SQL> select * from v$log;

    GROUP#    THREAD#  SEQUENCE#      BYTES  BLOCKSIZE    MEMBERS ARC STATUS           FIRST_CHANGE# FIRST_TIM NEXT_CHANGE# NEXT_TIME
---------- ---------- ---------- ---------- ---------- ---------- --- ---------------- ------------- --------- ------------ ---------
         1          1          7   52428800        512          2 YES INACTIVE                851844 21-AUG-18       851848 21-AUG-18
         2          1          8   52428800        512          2 YES INACTIVE                851848 21-AUG-18       854607 22-AUG-18
         3          1          9   52428800        512          2 NO  CURRENT                 854607 22-AUG-18   2.8147E+14

--4) 복구 확인  & 정리
SYS. SQL> select * from  sys.test1(id number);
SYS. SQL> drop table sys.test1;

{% endhighlight %}

## DATAFILE_USERS_USINGDB  

> DB 운영중에 USERS 테이블스페이스 장애로 부터 복구 -> ONLINE RECOVERY
 
{% highlight css %}

--1. db 작업 발생

SQL>  select a.file#,a.name,a.checkpoint_change#, b.status,b.change#,b.time 
      from v$datafile a, v$backup b where a.file#=b.file#; 
      @dfile

     FILE# NAME                                                                   CHECKPOINT_CHANGE# STATUS                CHANGE# TIME
---------- ---------------------------------------------------------------------- ------------------ ------------------ ---------- ---------
         1 /u01/app/oracle/orcl2/system01.dbf                                          854999 NOT ACTIVE             854464 22-AUG-18
         2 /u01/app/oracle/orcl2/sysaux01.dbf                                          854999 NOT ACTIVE             854464 22-AUG-18
         3 /u01/app/oracle/orcl2/undotbs01.dbf                                         854999 NOT ACTIVE             854464 22-AUG-18
         4 /u01/app/oracle/orcl2/users01.dbf                                           854999 NOT ACTIVE             854464 22-AUG-18
         5 /u01/app/oracle/orcl2/example01.dbf                                         854999 NOT ACTIVE             854464 22-AUG-18

SQL> select * from v$log;

    GROUP#    THREAD#  SEQUENCE#      BYTES  BLOCKSIZE    MEMBERS ARC STATUS           FIRST_CHANGE# FIRST_TIM NEXT_CHANGE# NEXT_TIME
---------- ---------- ---------- ---------- ---------- ---------- --- ---------------- ------------- --------- ------------ ---------
         1          1          7   52428800        512          2 YES INACTIVE                851844 21-AUG-18       851848 21-AUG-18
         2          1          8   52428800        512          2 YES INACTIVE                851848 21-AUG-18       854607 22-AUG-18
         3          1          9   52428800        512          2 NO  CURRENT                 854607 22-AUG-18   2.8147E+14


SQL>  select sequence#, name from v$archived_log;
 SEQUENCE# NAME
---------- ----------------------------------------------------------------------
         8 /home/oracle/arch1/1_8_984672345.arc
         8 /home/oracle/arch2/1_8_984672345.arc


SQL> !ls /home/oracle/arch1/
1_8_984672345.arc

SQL> !ls /home/oracle/arch2/
1_8_984672345.arc

SQL> create table hr.test1(id number) tablespace users;

Table created.

SQL> insert into hr.test1(id) values(1);

1 row created.

SQL> commit;

Commit complete.

SQL> select * from v$log;

    GROUP#    THREAD#  SEQUENCE#      BYTES  BLOCKSIZE    MEMBERS ARC STATUS           FIRST_CHANGE# FIRST_TIM NEXT_CHANGE# NEXT_TIME
---------- ---------- ---------- ---------- ---------- ---------- --- ---------------- ------------- --------- ------------ ---------
         1          1          7   52428800        512          2 YES INACTIVE                851844 21-AUG-18       851848 21-AUG-18
         2          1          8   52428800        512          2 YES INACTIVE                851848 21-AUG-18       854607 22-AUG-18
         3          1          9   52428800        512          2 NO  CURRENT                 854607 22-AUG-18   2.8147E+14
		 

SQL > alter system swtich logfile ; 3번 이상 발생 시킨 후 

SQL > select * from v$log; 


SQL> select sequence#, name, first_change#, next_change# from v$archived_log;


--2) 장애 발생

SQL> select f.file_name from dba_extents e, dba_data_files f where e.file_id = f.file_id and e.segment_name = 'TEST1';

FILE_NAME
------------------------------------------------------------
/u01/app/oracle/orcl2/users01.dbf

SQL> exit

$ rm -f /u01/app/oracle/orcl2/users01.dbf

$ sqlplus / as sysdba

SQL> select * from hr.test1;

        ID
----------
         1

SQL> alter system flush buffer_cache;
--참고 ) alter system flush shared_pool;

SQL> create table hr.test2 tablespace users as select * from hr.employees;
create table hr.test2 tablespace users as select * from hr.employees
                                                           *
ERROR at line 1:
ORA-01116: error in opening database file 4
ORA-01110: data file 4: '/u01/app/oracle/orcl2/users01.dbf'
ORA-27041: unable to open file
Linux Error: 2: No such file or directory
Additional information: 3

SQL> select a.file#,b.name,a.status,a.checkpoint_change# from v$datafile a, v$tablespace b where a.ts#=b.ts#;
SQL> save dfile  rep

      FILE# NAME                                                                   STATUS  CHECKPOINT_CHANGE#
---------- ---------------------------------------------------------------------- ------- ------------------
         1 SYSTEM                                                                 SYSTEM              854999
         2 SYSAUX                                                                 ONLINE              854999
         3 UNDOTBS1                                                               ONLINE              854999
         4 USERS                                                                  ONLINE              854999
         5 EXAMPLE                                                                ONLINE              854999


SQL> ! ls /u01/app/oracle/orcl2/users01.dbf
ls: /u01/app/oracle/orcl2/users01.dbf: No such file or directory

--3) 복구
SQL> alter tablespace users offline immediate;  -- 체크포인트 안함
                            offline temporary;  -- 일부 파일만 체크포인트 함

Tablespace altered.

-- 복원
SQL> !cp -av /home/oracle/cold/users01.dbf /u01/app/oracle/orcl2/users01.dbf 

SQL> recover tablespace users;
Media recovery complete.

SQL> alter tablespace users online;

Tablespace altered.

--4) 복구 확인

SQL> select a.file#,b.name,a.status,a.checkpoint_change# from v$datafile a, v$tablespace b where a.ts#=b.ts#;

     FILE# NAME                                                                   STATUS  CHECKPOINT_CHANGE#
---------- ---------------------------------------------------------------------- ------- ------------------
         1 SYSTEM                                                                 SYSTEM              854999
         2 SYSAUX                                                                 ONLINE              854999
         3 UNDOTBS1                                                               ONLINE              854999
         4 USERS                                                                  ONLINE              855709
         5 EXAMPLE                                                                ONLINE              854999

SQL> select * from hr.test1;

        ID
----------
         1

SQL> create table hr.test2 tablespace users as select * from hr.employees;

Table created.

SQL> select count(*) from hr.test2;

  COUNT(*)
----------
       107

SQL> exit;

SQL> drop table hr.test1;
        drop table hr.test2;

--5) 백업 받기
SQL >  @db_list
SQL > exit
$ .  cold_backup.sh  - 전체 db 백업
rm /home/oracle/arch1/* /home/oracle/arch2/*  -- 불필요한 archive log 삭제

{% endhighlight %}

## DATAFILE_USERS_STARTUP 

> DB STARTUP 시에 USERS TABLESPACE 장애로 부터 복구 (MEDIA FAILURE -> MEDIA RECOVERY)

{% highlight css %}

--1) DB 작업 발생

SQL> select file#, name, status , checkpoint_change#
from v$datafile;

SQL> save dfile

SQL> select * from v$log;

SQL> select sequence#, name from v$archived_log;

SQL> !ls /home/oracle/cold/
control01.ctl  example01.dbf  sysaux01.dbf  system01.dbf  temp01.dbf  undotbs01.dbf  users01.dbf

SQL> !ls /home/oracle/arch1/
1_8_984672345.dbf

-- DB 작업을 일으켜 본다.

SQL> create table hr.tab1 tablespace users 
as select * from hr.employees;

SQL> select count(*) from hr.tab1;

-- 107 건

SQL> @log

-- current 로그 번호? 

SQL > alter system switch logfile;

SQL > alter system switch logfile;

SQL > alter system switch logfile;

SQL > alter system switch logfile;

SQL > @log  -- current log번호?


--2. 장애발생

SQL> shutdown immediate


SQL> ! rm -f /u01/app/oracle/orcl2/users01.dbf

SQL> startup
ORACLE instance started.

Total System Global Area  849530880 bytes
Fixed Size                  1339824 bytes
Variable Size             507514448 bytes
Database Buffers          335544320 bytes
Redo Buffers                5132288 bytes
Database mounted.
ORA-01157: cannot identify/lock data file 4 - see DBWR trace file
ORA-01110: data file 4: '/u01/app/oracle/orcl2/users01.dbf'

SQL> select * from v$recover_file;

     FILE# ONLINE  ONLINE_ ERROR                                                                CHANGE# TIME
---------- ------- ------- ----------------------------------------------------------------- ---------- ---------
         4 ONLINE  ONLINE  FILE NOT FOUND                                                             0



SQL>  select a.file#,b.name,a.status,a.checkpoint_change# from v$datafile a, v$tablespace b where a.ts#=b.ts#;

     FILE# NAME                                                                   STATUS  CHECKPOINT_CHANGE#
---------- ---------------------------------------------------------------------- ------- ------------------
..... 
         4 USERS                                                                  ONLINE              856106


--3. 복구 수행

SQL> alter database datafile '/u01/app/oracle/orcl2/users01.dbf' offline;
*****************************************************************************
Database altered.

SQL> @dfile

SQL> alter database open;

Database altered.

SQL> select * from v$recover_file;

     FILE# ONLINE  ONLINE_ ERROR                                                                CHANGE# TIME
---------- ------- ------- ----------------------------------------------------------------- ---------- ---------
         4 OFFLINE OFFLINE FILE NOT FOUND                                                             0


SQL> select a.file#,b.name,a.status,a.checkpoint_change# from v$datafile a, v$tablespace b where a.ts#=b.ts#;

     FILE# NAME                                                                   STATUS  CHECKPOINT_CHANGE#
---------- ---------------------------------------------------------------------- ------- ------------------
.....
         4 USERS                                                                  OFFLINE             856106
		 
SQL> select tablesapce_name, status
from dba_tablespaces
where tablespace_name='USERS'; -- 만약 ONLINE 이라면 alter tablespace users offline immediate; 수행
                             
SQL> !cp -av /home/oracle/backup/arch/cold/users01.dbf /u01/app/oracle/orcl2/users01.dbf

SQL> recover tablespace users;
Media recovery complete.

SQL> alter tablespace users online;

Tablespace altered.

--4. 복구확인

SQL> select count(*) from hr.tab1; ==> 복구 확인


SQL>  select a.file#,b.name,a.status,a.checkpoint_change# from v$datafile a, v$tablespace b where a.ts#=b.ts#;

     FILE# NAME                                                                   STATUS  CHECKPOINT_CHANGE#
---------- ---------------------------------------------------------------------- ------- ------------------
...                
         4 USERS                                                                  ONLINE              856257

SQL> select * from v$log;
SQL> exit

--%   전체 DB cold backup 수행
--ARCHIVE  LOG  : 전체 DB 백업

SQL > @db_list
SQL > shutdown immediate

. backup.sh             -- 전체 DB 백업
rm /home/oracle/arch1/* /home/oracle/arch2/*  -- 불필요한 archive log 삭제
SQL> startup 

{% endhighlight %}

## DATAFILE_USERS_USINGDB  

> COMPLETE - ONLINE - 백업이 없는 상태에서의 RECOVERY

{% highlight css %}
1. DBA SESSION

SQL> select a.file#,b.name,a.status,a.checkpoint_change# from v$datafile a, v$tablespace b where a.ts#=b.ts#;

     FILE# NAME                           STATUS  CHECKPOINT_CHANGE#
---------- ------------------------------ ------- ------------------
         1 SYSTEM                         SYSTEM             1123825
         2 SYSAUX                         ONLINE             1123825
         3 UNDOTBS1                       ONLINE             1123825
         4 USERS                          ONLINE             1123825
         5 EXAMPLE                        ONLINE             1123825
  
SQL> save tablespace_status

SQL> ed datafile_backup.sql
col name for a35
select a.file#,a.name,b.status,b.change# from v$datafile a, v$backup b where a.file#=b.file#
/
SYS@orcl2> @datafile_backup

     FILE# NAME                                STATUS                CHANGE#
---------- ----------------------------------- ------------------ ----------
         1 /u01/app/oracle/orcl2/system01.dbf  NOT ACTIVE             858562
         2 /u01/app/oracle/orcl2/sysaux01.dbf  NOT ACTIVE             858562
         3 /u01/app/oracle/orcl2/undotbs01.dbf NOT ACTIVE             858562
         4 /u01/app/oracle/orcl2/users01.dbf   NOT ACTIVE             859626
         5 /u01/app/oracle/orcl2/example01.dbf NOT ACTIVE             858562
 

SYS@orcl2> ed log
  1  select group#, sequence#, status , archived, first_change#, next_change#
  2* from v$log
SYS@orcl2> @log

    GROUP#  SEQUENCE# STATUS           ARC FIRST_CHANGE# NEXT_CHANGE#
---------- ---------- ---------------- --- ------------- ------------
         1         19 INACTIVE         YES       1101871      1123824
         2         20 CURRENT          NO        1123824   2.8147E+14
         3         18 INACTIVE         YES       1080871      1101871

SYS@orcl2> 

SQL> select sequence#, name, first_change#, first_time, next_change#, next_time from v$archived_log;  -- archived log 확인

SQL> save archived_log


SQL> create tablespace data01 
datafile '/u01/app/oracle/oradata/data01.dbf' size 5m 
extent management local uniform size 64k 
segment space management auto;

Tablespace created.

SQL> select a.file#,a.name,b.status,b.change#,b.time from v$datafile a, v$backup b where a.file#=b.file#;
SYS@orcl2> @datafile_backup
     FILE# NAME                                STATUS                CHANGE#
---------- ----------------------------------- ------------------ ----------
         1 /u01/app/oracle/orcl2/system01.dbf  NOT ACTIVE             858562
         2 /u01/app/oracle/orcl2/sysaux01.dbf  NOT ACTIVE             858562
         3 /u01/app/oracle/orcl2/undotbs01.dbf NOT ACTIVE             858562
         4 /u01/app/oracle/orcl2/users01.dbf   NOT ACTIVE             859626
         5 /u01/app/oracle/orcl2/example01.dbf NOT ACTIVE             858562
         6 /u01/app/oracle/oradata/data01.dbf  NOT ACTIVE                  0

6 rows selected.

SYS@orcl2> create table hr.dept_temp tablespace data01 as select * from hr.departments;

Table created.

SQL> select count(*) from hr.dept_temp;

  COUNT(*)
----------
        27

SYS@orcl2> @log

    GROUP#  SEQUENCE# STATUS           ARC
---------- ---------- ---------------- ---
         1         13 CURRENT          NO
         2         11 INACTIVE         YES
         3         12 INACTIVE         YES

SYS@orcl2> alter system switch logfile;

SYS@orcl2> alter system switch logfile;
SYS@orcl2> alter system switch logfile;


SYS@orcl2> @log


    GROUP#  SEQUENCE# STATUS           ARC FIRST_CHANGE# NEXT_CHANGE#
---------- ---------- ---------------- --- ------------- ------------
         1         19 INACTIVE         YES       1101871      1123824
         2         20 CURRENT          NO        1123824   2.8147E+14
         3         18 INACTIVE         YES       1080871      1101871


SYS@orcl2> select f.file_name from dba_extents e, dba_data_files f where e.file_id = f.file_id and e.segment_name = 'DEPT_TEMP';

FILE_NAME
--------------------------------------------------------------------------------
/u01/app/oracle/oradata/data01.dbf

SQL> ! rm /u01/app/oracle/oradata/data01.dbf

SQL> ! ls /u01/app/oracle/oradata/data01.dbf
ls: /u01/app/oracle/oradata/data01.dbf: No such file or directory

SQL> select * from hr.dept_temp; 

SQL> alter system flush buffer_cache;

SQL> exit

$ sqlplus / as sysdba

SQL> select * from hr.dept_temp; 
select * from hr.dept_temp
*
ERROR at line 1:
ORA-01116: error in opening database file 6
ORA-01110: data file 6: '/u01/app/oracle/oradata/data01.dbf'
ORA-27041: unable to open file
Linux Error: 2: No such file or directory
Additional information: 3
SYS@orcl2> select * from hr.dept_temp;
select * from hr.dept_temp
                 *
				 
				 or
ERROR at line 1:
ORA-00376: file 6 cannot be read at this time
ORA-01110: data file 6: '/u01/app/oracle/oradata/data01.dbf'

SQL>  v$datafile 의 상태 확인 : select name, file#, status from v$datafile;  --ONLINE or RECOVER

SQL> alter tablespace data01 offline immediate;  -- online이라면  offline 한다

Tablespace altered.

SQL> v$datafile 의 상태 확인 : select name, file#, status from v$datafile; -- RECOVER

-- 백업이 없는 데이타파일을 생성해준다.
SQL> alter database create datafile '/u01/app/oracle/oradata/data01.dbf';

Database altered.

SQL> ! ls /u01/app/oracle/oradata/data01.dbf
/u01/app/oracle/oradata/data01.dbf


SQL> recover tablespace data01; or recover datafile '/u01/app/oracle/oradata/data01.dbf';

Media recovery complete.

SQL>  v$datafile 의 상태 확인 : select name, file#, status from v$datafile;  -- OFFLINE

SQL> alter tablespace data01 online;

Tablespace altered.

SQL> select f.file_name from dba_extents e, dba_data_files f where e.file_id = f.file_id and e.segment_name = 'DEPT_TEMP';

FILE_NAME
------------------------------------------------
/u01/app/oracle/oradata/data01.dbf

SQL>  v$datafile 의 상태 확인 : select name, file#, status from v$datafile; -- ONLINE


--%   전체 DB cold backup 수행
--ARCHIVE  LOG  : 전체 DB 백업

SQL > @db_list
SQL > shutdown immediate

. cold_backup.sh             -- 전체 DB 백업

rm /home/oracle/arch1/* /home/oracle/arch2/*  -- 불필요한 archive log 삭제

SQL> startup 

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
