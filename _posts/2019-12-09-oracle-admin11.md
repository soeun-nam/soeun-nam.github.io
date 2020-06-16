---
layout: post
title: ORACLE ADMIN 06
excerpt: "ASM에 대해 알아보기."
categories: [ORACLE ADMINISTRATION]
comments: true
---


# ASM 

> 데이터베이스에서 사용하는 모든 파일에 대해 자동저장공간 관리를 제공  

 *장점   
 1) 효율적인 디스크 자원의 관리     
 2) 자원의 물리적 장애에 대한 관리 향상     
 3) 디스크 I/O의 효율적인 분산     
 4) 효율적인 업무에 대한 집중 

 *DB instance와 ASM instance startup, shutdown 순서   
  startup - ASM인스턴스 먼저   
  shutdown - DB인스턴스 먼저   
  
 * ASM DISK GROUP내의 Failure group이란?   
  하나의 컨트롤러로 묶인 디스크들을 Failure group으로 볼수 있는데 미러링을 수행할때 같은 Failure group에 속한 디스크에 미러링을 하게되면 해당 컨트롤러에 문제가 생겼을때 미러링한 데이터까지 사용하지 못하는 경우가 생김.
  따라서 다른 컨트롤러에 속한 디스크들 즉, 디스크그룹내의 다른 Failure group에 미러링을 수행해야함
 
# ASM INSTANCE    
 * shared pool -> meta data정보에 사용   
 * large pool -> 병렬 작업에 사용    
 * asm cache -> 리밸런스 작업 중 읽기 및 쓰기 블록에 사용   
 * 사용 가능 메모리 -> 사용 가능한 할당 해제된 메모리   
 * rbal -> 검색 중에 모든 장치를 열고 리밸런스 작업 조정  
 * arbn -> 리밸런스 작업을 수행하는 하나 이상의 slave process  
 
## ASM Instance 확인

{% highlight css %}

--ASM Instance 확인
[orcl:~]$ . oraenv
ORACLE_SID = [orcl] ? +ASM                 --ASM접속
The Oracle base for ORACLE_HOME=/u01/app/oracle/product/11.2.0/dbhome_1/grid is /u01/app/oracle
[+ASM:~]$ sqlplus / as sysasm

SQL*Plus: Release 11.2.0.1.0 Production on Tue Dec 10 10:01:03 2019

Copyright (c) 1982, 2009, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Automatic Storage Management option

SP2-0158: unknown SET option "sql"
SQL> select instance_name, status from v$instance;    --ASM인스턴스가 시작할수있는지 확인

INSTANCE_NAME    STATUS
---------------- ------------
+ASM             STARTED

SQL> !ps -ef|grep +ASM                 --OS에서 ASM인스턴스가 잘 시작 되어있는지 확인
oracle    5437     1  0 09:49 ?        00:00:00 asm_pmon_+ASM
oracle    5439     1  0 09:49 ?        00:00:02 asm_vktm_+ASM
oracle    5443     1  0 09:49 ?        00:00:00 asm_gen0_+ASM
oracle    5445     1  0 09:49 ?        00:00:00 asm_diag_+ASM
oracle    5447     1  0 09:49 ?        00:00:00 asm_psp0_+ASM
.
.
SQL> select name, value     
  2  from v$parameter
  3  order by name;                 --ASM관련 파라미터 검색

NAME
--------------------------------------------------------------------------------
VALUE
--------------------------------------------------------------------------------
asm_diskgroups
FRA, DATA

asm_diskstring


asm_power_limit
1

asm_preferred_read_failure_groups


audit_file_dest
/u01/app/oracle/product/11.2.0/dbhome_1/grid/rdbms/audit
.
.
74 rows selected.

SQL> select name, value from v$diag_info;   --ADR의 관련 정보 확인 (NAME은 인스턴스ID이다.)
                                            --ADR이란??  Fault Diagnosability Infrastructure에 속하는 하나의 component
										    --Fault Diagnosability Infrastructure이란?? 
											--critical errors를 발견(detect)한 후, 원인을 분석(diagnose)해서, 문제를 해결(resolve)할 수 있도록 도움을 주는 infrastructure

NAME
----------------------------------------------------------------
VALUE
--------------------------------------------------------------------------------
Diag Enabled
TRUE

ADR Base
/u01/app/oracle

ADR Home
/u01/app/oracle/diag/asm/+asm/+ASM
.
.
11 rows selected.
{% endhighlight %}

##Dynamic Performance View 종류 확인   

>오라클 인스턴스가 동작할때마다 자동 갱신되는 뷰들이며 오라클의 상태, 성능, 모니터링, 감사 등을 위한 뷰   

{% highlight css %}
 
--Dynamic Performance View 종류 확인
SQL> select * from dictionary;
select * from dictionary
              *
ERROR at line 1:
ORA-01219: database not open: queries allowed on fixed tables/views only
--ERROR가 나는 이유는 ASM Instance 는 Dictionary Table 이 존재하지 않으므로 
--Dictionary View 의 접근은 불가능하다.
--Dynamic Performance View 는 이용 가능하며 ASM Instance 와 관련된 뷰는 다음과 같다.
--V$ASM_ALIAS, V$ASM_ATTRIBUTE, V$ASM_CLIENT, V$ASM_DISK, V$ASM_DISK_IOSTAT, V$ASM_DISK_STAT....

--ASM 관리에 사용되는 뷰 
--ASM DISK확인
SQL> select instance_name from v$instance;   

INSTANCE_NAME
----------------
+ASM
--ASM을 사용하고 있는 DB인스턴스 상황 (ASM, ORCL 두개 연결되어있음을 확인)
SQL> select group_number, instance_name, db_name, status
  2  from v$asm_client;

GROUP_NUMBER INSTANCE_NAME
------------ ----------------------------------------------------------------
DB_NAME  STATUS
-------- ------------
           1 +ASM
+ASM     CONNECTED

           1 orcl
orcl     CONNECTED

           2 orcl
orcl     CONNECTED

--v_asm_diskgroup는 ASM인스턴스가 감지한 모든 ASM디스크 그룹에 대해 검색
--asm instance에 연결되어 있는 disk group확인 (DATA, FRA)
SQL> select name, state, type, total_mb      
  2  from v$asm_diskgroup;

NAME                           STATE       TYPE     TOTAL_MB
------------------------------ ----------- ------ ----------
DATA                           MOUNTED     NORMAL       9216
FRA                            MOUNTED     NORMAL       9216

SQL> select group_number, disk_number, name, failgroup, total_mb, free_mb, mode_status 
2 from v$asm_disk;          --disk_group별 세부 상세 정보 확인

GROUP_NUMBER DISK_NUMBER NAME
------------ ----------- ------------------------------
FAILGROUP                        TOTAL_MB    FREE_MB MODE_ST
------------------------------ ---------- ---------- -------
           0           8
                                       0          0 ONLINE
           0           9
                                        0          0 ONLINE
           0          10
                                        0          0 ONLINE
           0          11
                                        0          0 ONLINE
           0          12
                                        0          0 ONLINE
           1           0 ASMDISK01
ASMDISK01                            2304       1391 ONLINE

           1           3 ASMDISK02
ASMDISK02                            2304       1393 ONLINE

           1           2 ASMDISK03
ASMDISK03                            2304       1403 ONLINE
.
.
13 rows selected.
{% endhighlight %}

## ASM Instance 시작 및 종료 수행 

{% highlight css %}

--• ASM Instance 시작 및 종료 수행 
SQL> !ps -ef|grep orcl         --현재 프로세스 확인
oracle    5565     1  0 09:50 ?        00:00:00 ora_pmon_orcl
oracle    5567     1  0 09:50 ?        00:00:15 ora_vktm_orcl
oracle    5571     1  0 09:50 ?        00:00:00 ora_gen0_orcl
oracle    5573     1  0 09:50 ?        00:00:00 ora_diag_orcl
oracle    5575     1  0 09:50 ?        00:00:00 ora_dbrm_orcl
oracle    8270  5896  0 13:10 pts/1    00:00:00 /bin/bash -c ps -ef|grep orcl
oracle    8272  8270  0 13:10 pts/1    00:00:00 grep orcl

SQL> shutdown normal    --현재 처리 중인 트랜잭션이 있다면 트랜잭션 끝난 후 종료 , 새로운 session 접근 차단
ORA-15097: cannot SHUTDOWN ASM instance with connected client
SQL> shutdown transactional      --클라이언트의 진행중인 트랜잭션 모두 마치면 서버 종료  (normal가 비슷)
ORA-15097: cannot SHUTDOWN ASM instance with connected client
SQL> shutdown immediate    --현재 처리중인 SQL statement all stop, commit안된 트랜잭션 모두 rollback
ORA-15097: cannot SHUTDOWN ASM instance with connected client
SQL> shutdown abort  --정상적인 방법으로 내려가지 않는 긴급한 경우 강제로 shutdown을 진행 (system에 무리를 줄 수있음)
ASM instance shutdown

SQL> !ps -ef|grep orcl     --프로세스 닫힌 것 확인
oracle    8270  5896  0 13:10 pts/1    00:00:00 /bin/bash -c ps -ef|grep orcl
oracle    8272  8270  0 13:10 pts/1    00:00:00 grep orcl

SQL> STARTUP   --오라클 인스턴스 시작
ASM instance started
Total System Global Area 284565504 bytes
Fixed Size 1336036 bytes
Variable Size 258063644 bytes
ASM Cache 25165824 bytes
ASM diskgroups mounted
ASM diskgroups volume enabled

select name, state, type, total_mb  -- Database Instance에서 현재 연결되어있는 ASM Disk Group 정보 확인
  2  from v$asm_diskgroup;

NAME                           STATE       TYPE     TOTAL_MB
------------------------------ ----------- ------ ----------
DATA                           MOUNTED     NORMAL       9216
FRA                            MOUNTED     NORMAL       9216
SQL> exit

[+ASM:~]$ . oraenv
ORACLE_SID = [+ASM] ? orcl
The Oracle base for ORACLE_HOME=/u01/app/oracle/product/11.2.0/dbhome_1 is /u01/app/oracle

[orcl:~]$ sqlplus / as sysdba 
SQL*Plus: Release 11.2.0.1.0 Production on Tue Dec 10 13:17:54 2019
Copyright (c) 1982, 2009, Oracle.  All rights reserved.

Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options

select instance_name, status from v$instance;   --DB인스턴스 상태 확인 OPEN!
                                                --OPEN이 아닌 STARTED이면 NOMOUNT상태

INSTANCE_NAME    STATUS
---------------- ------------
orcl             OPEN

SYS@orcl> select dbid, name, open_mode, log_mode from v$database;  --데이터베이스가 open상태여서 읽고쓰기가 가능한지 확인

      DBID NAME      OPEN_MODE            LOG_MODE
---------- --------- -------------------- ------------
1553401296 ORCL      READ WRITE           NOARCHIVELOG
{% endhighlight %}

## ASM Fast Mirror Resync 확인    
 
 *11g에 도입 된 새로운 기능으로 ASM Fast Mirror Resync는 디스크의 일시적 장애를 재 동기화하는데 필요한 시간을 줄임    
 *일시적 장애 후 디스크가 OFFLINE 상태가 되면 ASM은 중단 중에 수정 된 범위를 추적    
 *일시적인 오류가 복구되면 ASM은 중단 중에 영향을 받은 ASM 디스크 범위만 빠르게 재 동기화함    
 *ASM 디스크 경로가 실패하면 해당 디스크 그룹에 대해 DISK_REPAIR_TIME 속성을 설정 한 경우 ASM디스크가 오프라인 상태가 되지만 삭제가 안됨         
 *이 속성의 설정은 복구를 완료한 후에도 재 동기화 할 수 있는 동안 ASM이 허용하는 디스크 중단 기간을 결정    
 
{% highlight css %}

[+ASM:~]$ sqlplus / as sysasm
/*
V $ ASM_DISK 의 REPAIR_TIMER 열에 는 오프라인 디스크가 삭제되기까지 남은 시간 (초)이 표시됩니다. 
지정된 시간이 경과 한 후 ASM은 디스크를 삭제합니다.
ALTER DISKGROUP DISK OFFLINE 문 및 DROP AFTER 절을 사용 하여이 속성을 대체 할 수 있습니다 .
*/

--현재 오프라인 상태인 디스크가 있는 디스크 그룹에서 발행되면 새 속성 값은 현재 오프라인 모드가 아닌 디스크에만 적용
--ONLINE 모드인 디스크는 ALTER DISKGROUP DROP DISK문으로 삭제 불가능, 만약 시도하면 오류 반환
SQL> ALTER DISKGROUP DATA OFFLINE DISK ASMDISK02 DROP AFTER 0.0 h ; 
--디스크를 오프라인으로 만드려면 
SQL> ALTER DISKGROUP ASMDISK02 OFFLINE DISKS
--compatible.asm 및 compatible.rdbms 매개 변수 는 11.1.0.0.0 이상이거나 COMPATIBILITY와 DATABASE_COMPATIBILITY이 같아야함
SQL> ALTER DISKGROUP DATA SET ATTRIBUTE 'compatible.rdbms' = '11.2.0.0.0' ; 
--디스크를 복구하는 데 걸리는 시간을 제공하는 DISK_REPAIR_TIME 매개 변수 를 설정, 설정 안하면 기본 시간은 3.6 시간이지만 0.0으로 설정
SQL> ALTER DISKGROUP ASMDISK02 SET ATTRIBUTE 'DISK_REPAIR_TIME'= '0.0H';
--어떤 이유로 복구 시간이 만료되기 전에 디스크를 제거해야하는 경우(디스크를 복구할 수 없는 경우)
--DROP AFTER 절이 0H, 0M인 두번째 OFFLINE문을 실행하여 디스크 즉시 삭제 
SQL> ALTER DISKGROUP DATA OFFLINE DISK ASMDISK02 DROP AFTER 0.0 h ;

SQL> SELECT group_number, name, mount_status, mode_status, free_mb
FROM v$asm_disk
WHERE group_number = 2 ;
GROUP_NUMBER NAME                           MOUNT_S MODE_ST    FREE_MB
------------ ------------------------------ ------- ------- ----------
           2 ASMDISK01                      CACHED  ONLINE        1113
           2 ASMDISK03                      CACHED  ONLINE        1118
           2 ASMDISK04                      CACHED  ONLINE        1112
		   
SQL> exit

--삭제된 디스크 생성
[+ASM:~]$ . oraenv
ORACLE_SID = [+ASM] ? orcl
The Oracle base for ORACLE_HOME=/u01/app/oracle/product/11.2.0/dbhome_1 is /u01/app/oracle
[orcl:~]$ su -
Password: oracle

[root@edydr1p0 ~]# oracleasm listdisks

ASMDISK01
ASMDISK02
ASMDISK03
ASMDISK04
.
.
[root@edydr1p0 ~]# oracleasm deletedisk ASMDISK02

Clearing disk header: done
Dropping disk: done

[root@edydr1p0 ~]# oracleasm createdisk ASMDISK02 /dev/xvdc

Writing disk header: done
Instantiating disk: done

[root@edydr1p0 ~]# exit

[+ASM:labs]$ sqlplus / as sysasm

SQL> alter diskgroup data add disk 'ORCL:ASMDISK02' size 2304m
  2  rebalance power 11;
  
Diskgroup altered.

SQL> select group_number, name, mount_status, mode_status, free_mb
  2  from v$asm_disk where group_number =2;

GROUP_NUMBER NAME                           MOUNT_S MODE_ST    FREE_MB
------------ ------------------------------ ------- ------- ----------
           2 ASMDISK01                      CACHED  ONLINE        1113
           2 ASMDISK02                      CACHED  ONLINE        2210
           2 ASMDISK03                      CACHED  ONLINE        1118
           2 ASMDISK04                      CACHED  ONLINE        1112

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
