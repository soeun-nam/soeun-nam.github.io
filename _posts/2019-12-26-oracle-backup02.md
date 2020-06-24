---
layout: post
title: ORACLE BACKUP 02
excerpt: " ARCHIVE LOG MODE & NOARCHIVE LOG MODE "
categories: [ORACLE BACKUP]
comments: true
---


## ARCHIVE LOGMODE 변경

> 정상종료 : SHUTDOWN IMMEDIATE   

> MOUNT 모드로 STARTUP : STARTUP MOUNT  

> LOG MODE의 변경 : ALTER DATABASE ARCHIVELOG  

{% highlight css %}

[orcl:~]$ sqlplus / as sysdba

SQL> archive log list

Database log mode              No Archive Mode
Automatic archival             Disabled
Archive destination            USE_DB_RECOVERY_FILE_DEST
Oldest online log sequence     3
Current log sequence           5

SQL> !
[orcl:~]$ pwd
/home/oracle
[orcl:~]$ mkdir arch1
[orcl:~]$ mkdir arch2
[orcl:~]$ ls

[orcl:~]$ exit
exit

SQL> alter system set log_archive_dest_1="location=/home/oracle/arch1 mandatory"
	 scope=spfile;  

SQL> alter system set log_archive_dest_2="location=/home/oracle/arch2 optional" 
	scope=spfile; 

SQL> alter system set log_archive_format='arch_%t_%s_%r.arc' scope=spfile;  
						%t thread 번호
						%s log sequence#
						%r resetlogs 번호 


SQL> shutdown immediate		/아카이브 모드 변경시 반드시 정상종료해야함

SQL> startup mount 

SQL> alter database archivelog;

SQL> alter database open;  


SQL> archive log list
Database log mode              Archive Mode
Automatic archival             Enabled
Archive destination            /home/oracle/arch2
Oldest online log sequence     3
Next log sequence to archive   5
Current log sequence           5


SQL> set linesize 300
SQL> col member format a50
SQL> select * from v$log;

    GROUP#    THREAD#  SEQUENCE#      BYTES  BLOCKSIZE    MEMBERS ARC STATUS           FIRST_CHANGE# FIRST_TIM NEXT_CHANGE# NEXT_TIME
---------- ---------- ---------- ---------- ---------- ---------- --- ---------------- ------------- --------- ------------ ---------
         1          1          4   52428800        512          2 YES INACTIVE                810193 18-AUG-18       850428 20-AUG-18
         2          1          5   52428800        512          2 NO  CURRENT                 850428 20-AUG-18   2.8147E+14
         3          1          3   52428800        512          2 YES INACTIVE                797102 18-AUG-18       810193 18-AUG-18


SQL> 

SQL> col destination format a40
SQL> select destination, binding, status from v$archive_dest;

DESTINATION                              BINDING   STATUS
---------------------------------------- --------- ---------
/home/oracle/arch1                       MANDATORY VALID
/home/oracle/arch2                       OPTIONAL  VALID
.
.
.
.


SQL> alter system archive log current;   -- log switch + archive 도 수행

System altered.



SQL> select * from v$log;

    GROUP#    THREAD#  SEQUENCE#      BYTES  BLOCKSIZE    MEMBERS ARC STATUS           FIRST_CHANGE# FIRST_TIM NEXT_CHANGE# NEXT_TIME
---------- ---------- ---------- ---------- ---------- ---------- --- ---------------- ------------- --------- ------------ ---------
         1          1          4   52428800        512          2 YES INACTIVE                810193 18-AUG-18       850428 20-AUG-18
         2          1          5   52428800        512          2 YES ACTIVE                  850428 20-AUG-18       853286 20-AUG-18
         3          1          6   52428800        512          2 NO  CURRENT                 853286 20-AUG-18   2.8147E+14

SQL> 


SQL> col name format a60

SQL> select sequence#, name from v$archived_log;

 SEQUENCE# NAME
---------- ------------------------------------------------------------
         5 /home/oracle/arch1/arch_1_5_984497831.arc
         5 /home/oracle/arch2/arch_1_5_984497831.arc


SQL> select * from v$archive_processes; 

   PROCESS STATUS     LOG_SEQUENCE STAT
---------- ---------- ------------ ----
         0 ACTIVE                0 IDLE
         1 ACTIVE                0 IDLE
         2 ACTIVE                0 IDLE
         3 ACTIVE                0 IDLE
         4 STOPPED               0 IDLE
         5 STOPPED               0 IDLE
....

30 rows selected.



SQL> archive log list
Database log mode              Archive Mode
Automatic archival             Enabled
Archive destination            /home/oracle/arch2
Oldest online log sequence     4
Next log sequence to archive   6
Current log sequence           6

SQL> select * from v$log_history;



     RECID      STAMP    THREAD#  SEQUENCE# FIRST_CHANGE# FIRST_TIM NEXT_CHANGE# RESETLOGS_CHANGE# RESETLOGS
---------- ---------- ---------- ---------- ------------- --------- ------------ ----------------- ---------
         1  984497945          1          1        754488 18-AUG-18       791588            754488 18-AUG-18
         2  984497954          1          2        791588 18-AUG-18       797102            754488 18-AUG-18
         3  984498663          1          3        797102 18-AUG-18       810193            754488 18-AUG-18
         4  984636466          1          4        810193 18-AUG-18       850428            754488 18-AUG-18
         5  984638345          1          5        850428 20-AUG-18       853286            754488 18-AUG-18


SQL> alter database open;

{% endhighlight %}


## ARCHIVE LOG MODE에서 COLD BACKUP과 HOT BACKUP 받기

{% highlight css %}

--0. 디렉토리 준비  
--os user의 홈 디렉토리로 가기 
[orcl2:~]$ mkdir -p backup/arch/hot

[orcl2:~]$ mkdir -p backup/arch/cold

--1. COLD BACKUP
[orcl:~]$ sqlplus / as sysdba

SQL> archive log list

Database log mode              Archive Mode
Automatic archival             Enabled
Archive destination            /home/oracle/arch2
Oldest online log sequence     6
Next log sequence to archive   8
Current log sequence           8
SQL> 

SQL> select * from v$log;
        @log

    GROUP#    THREAD#  SEQUENCE#      BYTES  BLOCKSIZE    MEMBERS ARC STATUS           FIRST_CHANGE# FIRST_TIM NEXT_CHANGE# NEXT_TIME
---------- ---------- ---------- ---------- ---------- ---------- --- ---------------- ------------- --------- ------------ ---------
         1          1          7   52428800        512          2 YES INACTIVE                851844 21-AUG-18       851848 21-AUG-18
         2          1          8   52428800        512          2 NO  CURRENT                 851848 21-AUG-18   2.8147E+14
         3          1          6   52428800        512          2 YES INACTIVE                851841 21-AUG-18       851844 21-AUG-18


SQL> exit

$ cd
$ cd test

[orcl:~]$ sqlplus / as sysdba
/* archive log mode 에서 cold backup 받기 */

SQL> select tablespace_name, logging from dba_tablespaces;

TABLESPACE_NAME                LOGGING
------------------------------ ---------
SYSTEM                         LOGGING
SYSAUX                         LOGGING
UNDOTBS1                       LOGGING
TEMP                           NOLOGGING
USERS                          LOGGING
EXAMPLE                        NOLOGGING	/nologging - redo기록을 안하겠다. redo기록이 필요없는 경우에만 안하겠다는의미

6 rows selected.

SQL> alter tablespace example logging;

Tablespace altered.

SQL> select tablespace_name, logging from dba_tablespaces;

TABLESPACE_NAME                LOGGING
------------------------------ ---------
SYSTEM                         LOGGING
SYSAUX                         LOGGING
UNDOTBS1                       LOGGING
TEMP                           NOLOGGING
USERS                          LOGGING
EXAMPLE                        LOGGING

SQL> select * from v$log;
    GROUP#    THREAD#  SEQUENCE#      BYTES  BLOCKSIZE    MEMBERS ARC STATUS           FIRST_CHANGE# FIRST_TIM NEXT_CHANGE# NEXT_TIME
---------- ---------- ---------- ---------- ---------- ---------- --- ---------------- ------------- --------- ------------ ---------
         1          1          7   52428800        512          2 YES INACTIVE                851844 21-AUG-18       851848 21-AUG-18
         2          1          8   52428800        512          2 NO  CURRENT                 851848 21-AUG-18   2.8147E+14
         3          1          6   52428800        512          2 YES INACTIVE                851841 21-AUG-18       851844 21-AUG-18

SQL> select 'cp -av '|| name ||' /home/oracle/backup/arch/cold/' from v$controlfile
  union all
  select 'cp -av '|| member ||' /home/oracle/backup/arch/cold/' from v$logfile
  union all
  select 'cp -av '||name ||' /home/oracle/backup/arch/cold/' from v$datafile
  union all
  select 'cp -av '||name ||' /home/oracle/backup/arch/cold/' from v$tempfile;

SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.

SQL> ! 

cp -av //u01/app/oracle/orcl2/control01.ctl /home/oracle/backup/arch/cold
cp -av //u01/app/oracle/orcl2/redo03.log /home/oracle/backup/arch/cold
.......

[orcl:~]$ ls /home/oracle/backup/arch/cold
control01.ctl redo01b.log  redo02b.log  redo03b.log  sysaux01.dbf  temp01.dbf     users01.dbf
  example01.dbf  redo01.log   redo02.log   redo03.log   system01.dbf  undotbs01.dbf
[orcl:~]$ 


[oracle@localhost ~]$ 

--2. /* archive log mode에서 hot backup 받기 1 */
orcl2 $  cd


SQL> startup
ORACLE instance started.

Total System Global Area  849530880 bytes
Fixed Size                  1339824 bytes
Variable Size             511708752 bytes
Database Buffers          331350016 bytes
Redo Buffers                5132288 bytes
Database mounted.
Database opened.

SQL> select a.file#,a.name,a.checkpoint_change#,           b.status,b.change#,b.time 
      from v$datafile a, v$backup  b 
   where a.file#=b.file#;

     FILE# NAME                                                                   CHECKPOINT_CHANGE# STATUS                CHANGE# TIME
---------- ---------------------------------------------------------------------- ------------------ ------------------ ---------- ---------
         1 /u01/app/oracle/orcl2/system01.dbf                                          854015 NOT ACTIVE                  0
         2 /u01/app/oracle/orcl2/sysaux01.dbf                                          854015 NOT ACTIVE                  0
         3 /u01/app/oracle/orcl2/undotbs01.dbf                                         854015 NOT ACTIVE                  0
         4 /u01/app/oracle/orcl2/users01.dbf                                           854015 NOT ACTIVE                  0
         5 /u01/app/oracle/orcl2/example01.dbf                                         854015 NOT ACTIVE                  0


SQL> select 'cp -av '||name ||' /home/oracle/backup/arch/hot/' from v$datafile
    union all
    select 'cp -av '||name ||' /home/oracle/backup/arch/hot/' from v$tempfile;

cp -av /u01/app/oracle/orcl2/system01.dbf /home/oracle/backup/arch/hot/
cp -av /u01/app/oracle/orcl2/sysaux01.dbf /home/oracle/backup/arch/hot/
cp -av /u01/app/oracle/orcl2/undotbs01.dbf /home/oracle/backup/arch/hot/
cp -av /u01/app/oracle/orcl2/users01.dbf /home/oracle/backup/arch/hot/
cp -av /u01/app/oracle/orcl2/example01.dbf /home/oracle/backup/arch/hot/
cp -av /u01/app/oracle/orcl2/temp01.dbf /home/oracle/backup/arch/hot/

6 rows selected.

SQL> alter database begin backup; 

Database altered.

SQL> select! a.file#,a.name,a.checkpoint_change#, b.status,b.change#,b.time from v$datafile a, v$backup b where a.file#=b.file#;

      FILE# NAME                                                                   CHECKPOINT_CHANGE# STATUS                CHANGE# TIME
---------- ---------------------------------------------------------------------- ------------------ ------------------ ---------- ---------
         1 /u01/app/oracle/orcl2/system01.dbf                                          854464 ACTIVE                 854464 22-AUG-18
         2 /u01/app/oracle/orcl2/sysaux01.dbf                                          854464 ACTIVE                 854464 22-AUG-18
         3 /u01/app/oracle/orcl2/undotbs01.dbf                                         854464 ACTIVE                 854464 22-AUG-18
         4 /u01/app/oracle/orcl2/users01.dbf                                           854464 ACTIVE                 854464 22-AUG-18
         5 /u01/app/oracle/orcl2/example01.dbf                                         854464 ACTIVE                 854464 22-AUG-18

SQL> ! . orcl2_backup_hot.sh

cp -av /u01/app/oracle/orcl2/system01.dbf /home/oracle/backup/arch/hot_20190415/
cp -av /u01/app/oracle/orcl2/sysaux01.dbf /home/oracle/backup/arch/hot_20190415/
cp -av /u01/app/oracle/orcl2/undotbs01.dbf /home/oracle/backup/arch/hot_20190415/
cp -av /u01/app/oracle/orcl2/users01.dbf /home/oracle/backup/arch/hot_20190415/
cp -av /u01/app/oracle/orcl2/example01.dbf /home/oracle/backup/arch/hot_20190415/
cp -av /u01/app/oracle/orcl2/temp01.dbf /home/oracle/backup/arch/hot_20190415/

[orcl:~]$ exit
exit

SQL> alter database end backup;

Database altered.

SQL>  select a.file#,a.name,a.checkpoint_change#, b.status,b.change#,b.time from v$datafile a, v$backup b where a.file#=b.file#;

     FILE# NAME                                                                   CHECKPOINT_CHANGE# STATUS                CHANGE# TIME
---------- ---------------------------------------------------------------------- ------------------ ------------------ ---------- ---------
         1 /u01/app/oracle/orcl2/system01.dbf                                          854464 NOT ACTIVE             854464 22-AUG-18
         2 /u01/app/oracle/orcl2/sysaux01.dbf                                          854464 NOT ACTIVE             854464 22-AUG-18
         3 /u01/app/oracle/orcl2/undotbs01.dbf                                         854464 NOT ACTIVE             854464 22-AUG-18
         4 /u01/app/oracle/orcl2/users01.dbf                                           854464 NOT ACTIVE             854464 22-AUG-18
         5 /u01/app/oracle/orcl2/example01.dbf                                         854464 NOT ACTIVE             854464 22-AUG-18

  -- control file의 온라인 백업 (물리적 백업)
SQL> alter database backup controlfile to '/home/oracle/backup/arch/hot/control01.ctl';

  -- control file의 온라인 백업 (논리적 백업)
SQL> alter database backup controlfile to trace;

SQL> select value from v$diag_info
where name ='Default Trace File';

SQL>  alter database backup controlfile to trace as '/home/oracle/test/control.sql';

--2. 방법 2
alter tablespace users begin backup;
! cp -av /u01/app/oracle/orcl2/users01.dbf     /home/oracle/backup/arch/hot
alter tablespace users end backup;   --이런식으로 모든 테이블스페이스 백업  

-- 콘트롤 파일도 온라인 백업을 수행한다.
alter database backup controlfile to '/home/oracle/backup/arch/hot/control.bak';
SQL>  select * from v$log;

    GROUP#    THREAD#  SEQUENCE#      BYTES  BLOCKSIZE    MEMBERS ARC STATUS           FIRST_CHANGE# FIRST_TIM NEXT_CHANGE# NEXT_TIME
---------- ---------- ---------- ---------- ---------- ---------- --- ---------------- ------------- --------- ------------ ---------
         1          1          7   52428800        512          2 YES INACTIVE                851844 21-AUG-18       851848 21-AUG-18
         2          1          8   52428800        512          2 NO  CURRENT                 851848 21-AUG-18   2.8147E+14
         3          1          6   52428800        512          2 YES INACTIVE                851841 21-AUG-18       851844 21-AUG-18



SQL> alter system archive log current;  

System altered.

SQL> select * from v$log;

    GROUP#    THREAD#  SEQUENCE#      BYTES  BLOCKSIZE    MEMBERS ARC STATUS           FIRST_CHANGE# FIRST_TIM NEXT_CHANGE# NEXT_TIME
---------- ---------- ---------- ---------- ---------- ---------- --- ---------------- ------------- --------- ------------ ---------
         1          1          7   52428800        512          2 YES INACTIVE                851844 21-AUG-18       851848 21-AUG-18
         2          1          8   52428800        512          2 YES ACTIVE                  851848 21-AUG-18       854607 22-AUG-18
         3          1          9   52428800        512          2 NO  CURRENT                 854607 22-AUG-18   2.8147E+14




SQL>  select sequence#, name from v$archived_log;

 SEQUENCE# NAME
---------- ----------------------------------------------------------------------
         8 /home/oracle/arch1/1_8_984672345.dbf
         8 /home/oracle/arch2/1_8_984672345.dbf


SQL> !ls /home/oracle/arch1/
1_8_984672345.dbf

SQL> !ls /home/oracle/arch2/
1_8_984672345.dbf

--참고 1 (archivelog 모드 상태의 COLD 백업) 전체 데이타 베이스 백업  
SQL > vi db_list
    db_list.sql
     ---------------------
     set pages 0 head off feed off
create pfile from spfile;
     spool backup.sh
      select 'cp -av ' || name||'  /home/oracle/backup/arch/cold'
 from v$controlfile
      union all
      select  'cp -av ' || name||'  /home/oracle/backup/arch/cold'
 from v$datafile
     union all
      select 'cp -av ' || name||' /home/oracle/backup/arch/cold'
 from v$tempfile
      union all
      select 'cp -av '|| member ||' /home/oracle/backup/arch/cold'
 from v$logfile;
select 'cp -av $ORACLE_HOME/dbs/initorcl2.ora /home/oracle/backup/arch/cold' from dual;
      spool off
      set feedback on head on pages 100
  !chmod 755 backup.sh
  
	  shutdown immediate
	  -----------------
      만약 제일 마지막에  shutdown immediate 없으면 추가해 준다.
SQL >  @db_list

SQL > exit

$ .  backup.sh  == cold backup 수행

백업 이전의 archive 는 필요없으므로 archve log file들을 삭제

$ rm /home/oracle/arch1/* home/oracle/arch2/*
  alter database begin backup;
    -- 전체 데이타파일 백업   -- 아래 참고2번에서 얻어진 open_db_backup.sh 수행
  alter database end backup;

   alter tablespace USERS begin backup;
     -- users 테이블스페이스의 데이타파일 백업
   alter tablespace USERS end backup;

--참고 2 archive log 모드 상태의 HOT 백업시 리스트 얻기
 db_list_datafile.sql
 ---------------------
 set pages 0 head off feed off
   
 spool open_db_backup.sh
     
 select 'cp ' || name||'  /home/oracle/backup/arch/hot'  
 from v$datafile;

 spool off
 set feedback on head on pages 100
 ! chmod 755 open_db_backup.sh
 -----------------
{% endhighlight %}

## RECOVERY_RESTORE NOARCHIVE LOG MODE : CONSISTENT BACKUP으로 복원만 가능 

{% highlight css %}
[orcl:noarch]$ sqlplus / as sysdba

SQL> select * from v$log;
       @log

    GROUP#    THREAD#  SEQUENCE#      BYTES  BLOCKSIZE    MEMBERS ARC STATUS           FIRST_CHANGE# FIRST_TIM NEXT_CHANGE# NEXT_TIME
---------- ---------- ---------- ---------- ---------- ---------- --- ---------------- ------------- --------- ------------ ---------
         1          1          7   52428800        512          2 NO  INACTIVE                851844 21-AUG-18       851848 21-AUG-18
         2          1          8   52428800        512          2 NO  CURRENT                 851848 21-AUG-18   2.8147E+14
         3          1          6   52428800        512          2 NO  INACTIVE                851841 21-AUG-18       851844 21-AUG-18


SQL> conn hr/hr
Connected.

SQL> create table test(id number) tablespace example;

Table created.

SQL> insert into test(id) values(1);

1 row created.

SQL> commit;

Commit complete.

SQL> select * from test;

        ID
----------
         1

SQL> conn / as sysdba
Connected.

SQL> alter system switch logfile;

System altered.

SQL> /

System altered.

SQL> /

System altered.

SQL> select * from v$log;

    GROUP#    THREAD#  SEQUENCE#      BYTES  BLOCKSIZE    MEMBERS ARC STATUS           FIRST_CHANGE# FIRST_TIM NEXT_CHANGE# NEXT_TIME
---------- ---------- ---------- ---------- ---------- ---------- --- ---------------- ------------- --------- ------------ ---------
         1          1         10   52428800        512          2 NO  INACTIVE                852593 21-AUG-18       852597 21-AUG-18
         2          1         11   52428800        512          2 NO  CURRENT                 852597 21-AUG-18   2.8147E+14
         3          1          9   52428800        512          2 NO  INACTIVE                852582 21-AUG-18       852593 21-AUG-18

SQL> SELECT f.file_name
     FROM dba_extents e, dba_data_files f
     WHERE e.file_id = f.file_id
     AND e.segment_name = 'TEST'
     AND e.owner = 'HR';   
 
FILE_NAME
---------------------------------------------
/u01/app/oracle/orcl2/example01.dbf

SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> 

---장애 유발 시키기
SQL>! rm -f /u01/app/oracle/orcl2/example01.dbf

SQL> startup
ORACLE instance started.

Total System Global Area  849530880 bytes
Fixed Size                  1339824 bytes
Variable Size             511708752 bytes
Database Buffers          331350016 bytes
Redo Buffers                5132288 bytes
Database mounted.
ORA-01157: cannot identify/lock data file 5 - see DBWR trace file
ORA-01110: data file 5: '/u01/app/oracle/oradata/orcl2/example01.dbf'


SQL> ! cp -av /home/oracle/noarch/example01.dbf /u01/app/oracle/orcl2/example01.dbf
`/home/oracle/noarch/example01.dbf' -> `/u01/app/oracle/orcl2/example01.dbf'

SQL> recover database;
ORA-00279: change 852305 generated at 08/21/2018 12:49:22 needed for thread 1
ORA-00289: suggestion : /u01/app/oracle/flash_recovery_area/ORCL/archivelog/2018_08_21/o1_mf_1_8_%u_.arc
ORA-00280: change 852305 for thread 1 is in sequence #8


Specify log: {<RET>=suggested | filename | AUTO | CANCEL}

auto
ORA-00308: cannot open archived log '/u01/app/oracle/flash_recovery_area/ORCL/archivelog/2018_08_21/o1_mf_1_8_%u_.arc'
ORA-27037: unable to obtain file status
Linux Error: 2: No such file or directory
Additional information: 3


ORA-00308: cannot open archived log '/u01/app/oracle/flash_recovery_area/ORCL/archivelog/2018_08_21/o1_mf_1_8_%u_.arc'
ORA-27037: unable to obtain file status
Linux Error: 2: No such file or directory
Additional information: 3


SQL> select status from v$instance;

STATUS
------------
MOUNTED

SQL> shutdown abort
ORACLE instance shut down.

/* Offline + Whole Backup을 Restore */
SQL> ! cp -av /home/oracle/noarch/*.* /u01/app/oracle/orcl2/

`/home/oracle/noarch/control01.ctl' -> `/u01/app/oracle/oradata/orcl2/control01.ctl'
`/home/oracle/noarch/control02.ctl' -> `/u01/app/oracle/oradata/orcl2/control02.ctl'
`/home/oracle/noarch/control03.ctl' -> `/u01/app/oracle/oradata/orcl2/control03.ctl'
`/home/oracle/noarch/example01.dbf' -> `/u01/app/oracle/oradata/orcl2/example01.dbf'
`/home/oracle/noarch/redo01.log' -> `/u01/app/oracle/oradata/orcl2/redo01.log'
`/home/oracle/noarch/redo01b.log' -> `/u01/app/oracle/oradata/orcl2/redo01b.log'
`/home/oracle/noarch/redo02.log' -> `/u01/app/oracle/oradata/orcl2/redo02.log'
`/home/oracle/noarch/redo02b.log' -> `/u01/app/oracle/oradata/orcl2/redo02b.log'
`/home/oracle/noarch/redo03.log' -> `/u01/app/oracle/oradata/orcl2/redo03.log'
`/home/oracle/noarch/redo03b.log' -> `/u01/app/oracle/oradata/orcl2/redo03b.log'
`/home/oracle/noarch/sysaux01.dbf' -> `/u01/app/oracle/oradata/orcl2/sysaux01.dbf'
`/home/oracle/noarch/system01.dbf' -> `/u01/app/oracle/oradata/orcl2/system01.dbf'
`/home/oracle/noarch/temp01.dbf' -> `/u01/app/oracle/oradata/orcl2/temp01.dbf'
`/home/oracle/noarch/undotbs01.dbf' -> `/u01/app/oracle/oradata/orcl2/undotbs01.dbf'
`/home/oracle/noarch/users01.dbf' -> `/u01/app/oracle/oradata/orcl2/users01.dbf'

SQL> startup
ORACLE instance started.

Total System Global Area  849530880 bytes
Fixed Size                  1339824 bytes
Variable Size             528485968 bytes
Database Buffers          314572800 bytes
Redo Buffers                5132288 bytes
Database mounted.
Database opened.
SQL> 

SQL> conn hr/hr
Connected.
SQL> select * from tab;

TNAME                          TABTYPE  CLUSTERID
------------------------------ ------- ----------
COUNTRIES                      TABLE
DEPARTMENTS                    TABLE
EMPLOYEES                      TABLE
EMP_DETAILS_VIEW               VIEW
JOBS                           TABLE
JOB_HISTORY                    TABLE
LOCATIONS                      TABLE
REGIONS                        TABLE

-- TEST 테이블은 없음
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
