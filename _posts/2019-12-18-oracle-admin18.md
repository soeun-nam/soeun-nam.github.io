---
layout: post
title: ORACLE ADMIN 11
excerpt: " MANAGING UNDO DATA "
categories: [ORACLE ADMINISTRATION]
comments: true
---


# UNDO TABLESPACE
  
  * CREATE UNDO TABLESPACE UNDOTBS1 DATAFILE '...' SIZE 100M;   
  * 로컬 관리 방식, AUTOALLOCATE   
  * 크기에 맞춰 여러개의 UNDO SEGMENT들이 자동으로 생성됨    
  * UNDO DATA만 저장   
  * INSTANCE 당 하나만 사용 (생성은 여러개 가능)   
  * UNDO_TABLESPACE   
  
{% highlight css %}

SYS@orcl> SELECT tablespace_name, contents, retention
FROM dba_tablespaces
WHERE contents = 'UNDO' ;

TABLESPACE CONTENTS  RETENTION
---------- --------- -----------
UNDOTBS1   UNDO      NOGUARANTEE

SYS@orcl> SELECT file_name, bytes/1024/1024 AS MB, autoextensible
FROM dba_data_files
WHERE tablespace_name = 'UNDOTBS1' ;

FILE_NAME                                                  MB AUT
-------------------------------------------------- ---------- ---
+DATA/orcl/datafile/undotbs1.258.1008856283               105 YES
{% endhighlight %} 


# UNDO SEGMENT  
  
  * EXTENT가 최소 2개로 시작  
  * 순환식으로 기록  -> 종료된 TX의 UNDO DATA는 OVERWRITE
  * 부족시 추가로 EXTENT할당(GROW), 공간이 남으면 EXTENT 반납(SHINK)  
  * 트랜잭션은 시작시 UNDO SEGMENT 하나를 지정 받음, 하나의 UNDO SEGMENT에는 여러 트랜잭션이 기록할 수 있음  
  * undo segment  ---------<- trasaction   

{% highlight css %}  

SYS@orcl> select segment_name, status 
          from dba_rollback_segs;

SEGMENT_NAME                   STATUS
------------------------------ ----------------
SYSTEM                         ONLINE
_SYSSMU10_4131489474$          ONLINE
_SYSSMU9_1735643689$           ONLINE
_SYSSMU8_3901294357$           ONLINE
_SYSSMU7_3517345427$           ONLINE
_SYSSMU6_2897970769$           ONLINE
_SYSSMU5_538557934$            ONLINE
_SYSSMU4_1003442803$           ONLINE
_SYSSMU3_1204390606$           ONLINE
_SYSSMU2_967517682$            ONLINE
_SYSSMU1_592353410$            ONLINE
  
SYS@orcl> SELECT *
FROM v$rollname ;

       USN NAME
---------- ------------------------------
         0 SYSTEM
         1 _SYSSMU1_592353410$
         2 _SYSSMU2_967517682$
         3 _SYSSMU3_1204390606$
         4 _SYSSMU4_1003442803$
         5 _SYSSMU5_538557934$
         6 _SYSSMU6_2897970769$
         7 _SYSSMU7_3517345427$
         8 _SYSSMU8_3901294357$
         9 _SYSSMU9_1735643689$
        10 _SYSSMU10_4131489474$

11 rows selected.
  
-- UNDO SEGMENT의 EXTENT들  
  1  select segment_name, sum(bytes)/1024 k, count(extent_id)
  2  from dba_undo_extents
  3* group by segment_name
SYS@orcl>/

SEGMENT_NAME                            K COUNT(EXTENT_ID)
------------------------------ ---------- ----------------
_SYSSMU7_3517345427$                 2176                4
_SYSSMU6_2897970769$                 2176                4
_SYSSMU10_4131489474$                2240                5
_SYSSMU9_1735643689$                 2176                4
_SYSSMU8_3901294357$                 1152                3
_SYSSMU5_538557934$                  2176                4
_SYSSMU3_1204390606$                 2176                4
_SYSSMU2_967517682$                  2304                6
_SYSSMU4_1003442803$                 2176                4
_SYSSMU1_592353410$                  2176                4

10 rows selected.
{% endhighlight %}   

## UNDO DATA
  
  1. UNDO DATA의 용도     
  * 롤백      
  * 읽기 일관성      
  * RECOVERY - 중단된 트랜잭션의 복구      
  * FLASHBACK QUERY    

  2. 관리 
  * 자동 언두 관리 -> UNDO TABLESPACE 생성만 해주면 오라클 서버가 알아서   
  * 수동 언두 관리 -> UNDO SEGMENT의 사이즈와 갯수를 DBA가 정함      
 
{% highlight css %}  

--AUTOMATIC UNDO MANAGEMENT  
[orcl:~]$ sqlplus / as sysdba

SYS@orcl>show parameter undo

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
undo_management                      string      AUTO
undo_retention                       integer     900
undo_tablespace                      string      UNDOTBS1

undo_tablespace 
 : 인스턴스에서 사용하는 undo tablespace 지정

SQL> create undo tablespace undo2
  2  datafile '/home/oracle/labs/undo2.dbf' size 10m;

Tablespace created.

SQL> select segment_name, status, tablespace_name
  2*           from dba_rollback_segs
SQL> /

SEGMENT_NAME                   STATUS           TABLESPACE_NAME
------------------------------ ---------------- ------------------------------
SYSTEM                         ONLINE           SYSTEM
_SYSSMU1_592353410$            ONLINE           UNDOTBS1
_SYSSMU2_967517682$            ONLINE           UNDOTBS1
_SYSSMU3_1204390606$           ONLINE           UNDOTBS1
_SYSSMU4_1003442803$           ONLINE           UNDOTBS1
_SYSSMU5_538557934$            ONLINE           UNDOTBS1
_SYSSMU6_2897970769$           ONLINE           UNDOTBS1
_SYSSMU7_3517345427$           ONLINE           UNDOTBS1
_SYSSMU8_3901294357$           ONLINE           UNDOTBS1
_SYSSMU9_1735643689$           ONLINE           UNDOTBS1
_SYSSMU10_4131489474$          ONLINE           UNDOTBS1
_SYSSMU11_3700460593$          OFFLINE          UNDO2
_SYSSMU12_2139809194$          OFFLINE          UNDO2
_SYSSMU13_4096367303$          OFFLINE          UNDO2
_SYSSMU14_2246945915$          OFFLINE          UNDO2
_SYSSMU15_3814402214$          OFFLINE          UNDO2
_SYSSMU16_3042527912$          OFFLINE          UNDO2
_SYSSMU17_4012943329$          OFFLINE          UNDO2
_SYSSMU18_2588389998$          OFFLINE          UNDO2
_SYSSMU19_468035644$           OFFLINE          UNDO2
_SYSSMU20_1110266837$          OFFLINE          UNDO2

21 rows selected.
SQL> alter system set undo_tablespace=undo2;

System altered.

SQL> select segment_name, status, tablespace_name
  2*           from dba_rollback_segs
SQL> /

SEGMENT_NAME                   STATUS           TABLESPACE_NAME
------------------------------ ---------------- ------------------------------
SYSTEM                         ONLINE           SYSTEM
_SYSSMU1_592353410$            OFFLINE          UNDOTBS1
_SYSSMU2_967517682$            OFFLINE          UNDOTBS1
_SYSSMU3_1204390606$           OFFLINE          UNDOTBS1
_SYSSMU4_1003442803$           OFFLINE          UNDOTBS1
_SYSSMU5_538557934$            OFFLINE          UNDOTBS1
_SYSSMU6_2897970769$           OFFLINE          UNDOTBS1
_SYSSMU7_3517345427$           OFFLINE          UNDOTBS1
_SYSSMU8_3901294357$           OFFLINE          UNDOTBS1
_SYSSMU9_1735643689$           OFFLINE          UNDOTBS1
_SYSSMU10_4131489474$          OFFLINE          UNDOTBS1
_SYSSMU11_3700460593$          ONLINE           UNDO2
_SYSSMU12_2139809194$          ONLINE           UNDO2
_SYSSMU13_4096367303$          ONLINE           UNDO2
_SYSSMU14_2246945915$          ONLINE           UNDO2
_SYSSMU15_3814402214$          ONLINE           UNDO2
_SYSSMU16_3042527912$          ONLINE           UNDO2
_SYSSMU17_4012943329$          ONLINE           UNDO2
_SYSSMU18_2588389998$          ONLINE           UNDO2
_SYSSMU19_468035644$           ONLINE           UNDO2
_SYSSMU20_1110266837$          ONLINE           UNDO2

21 rows selected.

{% endhighlight %}  

## UNDO RETENTION 
    
  * TX이 종료된 후에도 UNDO를 유지해주는 시간   
  * FLASHBACK QUERY를 위해 지켜져야함   
  update 500 -> 1000       undo data: 500    status : 'ACTIVE'    
  commit:                                             'UNEXPIRED'    
  undo_retention=900      이 경과 된 다음             'EXPIRED'    
  
  1. ACTIVE: 현재 트랜잭션이 진행중 상태, 언두 정보는 겹쳐쓰이지 않음   
  2. UNEXPIRTED: 트랜잭션은 끝났지만 UNDO RETENTION을 보장하기 위해서 여전히 필요, 커밋된 언두 정보는 언두 공간의 여유공간이 있는 한도 내에서 보존    
  3. EXPIRED: 트랜잭션이 끝난 상태, 만료된 언두는 새로운 트랜잭션에서 공간이 필요하면 겹쳐씀   
  
  * UNDO_RETENTION  
    공간이 부족하면 지켜지지 않음    
	반드시 지켜지는 경우    
	 1. undo datafile이 autoextend on    
	 2. alter tablespace undotbs1 retention guarantee;   
	self-tuning 기능을 이용해 undo_retention을 오라클이 지정   
	 1. auextend on : undo_retention 보장 & long running query보다 크게    
	 2. fixed size: undo_retention 지정은 무시하고 self-tuning의 결과에 따름   
	 
{% highlight css %} 
 
-- 트랜잭션에 대한 Undo Segment 할당 확인
SYS@orcl> CONN system/oracle_4U

SYSTEM@orcl> CREATE TABLE t1
AS SELECT rownum AS ID, object_name, created
FROM dba_objects ;

SYSTEM@orcl> UPDATE t1
SET created = created + 1
WHERE id = 1 ;
1 row updated.

SYSTEM@orcl> SELECT xid, xidusn,xidslot, ubafil, ubablk
FROM v$transaction ;
XID                  XIDUSN    XIDSLOT     UBAFIL     UBABLK
---------------- ---------- ---------- ---------- ----------
09001C004A030000          9         28          3       2762

--system 유저가 수행한 트랜잭션 정보가 저장된 언두 세그먼트를 확인
select segment_id, segment_name, status
from dba_rollback_segs
where segment_id=&usn			--위에 조회한 xidusn번호(9)를 입력!  

SYSTEM@orcl>select extent_id, block_id, status 
  2  from dba_undo_extents 
  3   where segment_name = '_SYSSMU9_1735643689$';  -- 위에서 확인한  v$transaction의 xidusn(9)의 segment_name
	 
 EXTENT_ID   BLOCK_ID STATUS
---------- ---------- ---------
         0        256 EXPIRED
         1        264 EXPIRED
         2       2688 ACTIVE
         3        512 EXPIRED

--SYSTEM user가 commit을 수행한후 언두 세그먼트를 확인 
SYSTEM@orcl>commit;

Commit complete.

SYSTEM@orcl>select extent_id, block_id, status 
from dba_undo_extents 
where segment_name = '_SYSSMU9_1735643689$';  2    3  

 EXTENT_ID   BLOCK_ID STATUS
---------- ---------- ---------
         0        256 EXPIRED
         1        264 EXPIRED
         2       2688 UNEXPIRED
         3        512 EXPIRED

-- Automatic undo management는 transaction 당 하나의 undo segment 할당을 최대한 보장해 줌으로써 경합을 최소화 할 수 있으며 
--undo_retention의 값을 최대한 보장할 수 있는 환경을 구성할 수 있음 !!

{% endhighlight %}  


{% highlight css %} 

--실습 ! 
--0.happy user를 생성  
--# Terminal 1

[orcl:~]$ sqlplus / as sysdba

create user happy identified by tiger; 
grant update on hr.employees to happy;

grant connect to happy;

--1. 현재 db의 언두 설정 파라미터 확인
SQL> show parameter undo

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
undo_management                      string      AUTO
undo_retention                       integer     900
undo_tablespace                      string      UNDOTBS1

--2. 언두 데이터 파일의 정보를 확인 
SQL> select file_name, file_id, bytes/1024/1024 m, status, autoextensible
  2  from dba_data_files
  3* where tablespace_name='UNDOTBS1'

FILE_NAME                                             FILE_ID          M STATUS    AUT
-------------------------------------------------- ---------- ---------- --------- ---
+DATA/orcl/datafile/undotbs1.258.1008675815                 3        105 AVAILABLE YES 

--3. undo segment에 대한 정보확인  
SQL>  SELECT segment_id,segment_name,owner,tablespace_name,status FROM dba_rollback_segs;

SEGMENT_ID SEGMENT_NAME                   OWNER  TABLESPACE_NAME                STATUS
---------- ------------------------------ ------ ------------------------------ ----------------
         0 SYSTEM                         SYS    SYSTEM                         ONLINE
         1 _SYSSMU1_592353410$            PUBLIC UNDOTBS1                       ONLINE
         2 _SYSSMU2_967517682$            PUBLIC UNDOTBS1                       ONLINE
         3 _SYSSMU3_1204390606$           PUBLIC UNDOTBS1                       ONLINE
         4 _SYSSMU4_1003442803$           PUBLIC UNDOTBS1                       ONLINE
         5 _SYSSMU5_538557934$            PUBLIC UNDOTBS1                       ONLINE
         6 _SYSSMU6_2897970769$           PUBLIC UNDOTBS1                       ONLINE
         7 _SYSSMU7_3517345427$           PUBLIC UNDOTBS1                       ONLINE
         8 _SYSSMU8_3901294357$           PUBLIC UNDOTBS1                       ONLINE
         9 _SYSSMU9_1735643689$           PUBLIC UNDOTBS1                       ONLINE
        10 _SYSSMU10_4131489474$          PUBLIC UNDOTBS1                       ONLINE

11 rows selected.

--4. # Terminal 2로 hr유저로 접속해서 2002년도에 입사한 사원들의 급여를 10%인상하세요 단, 트랜잭션은 종료하지 마세요  
SQL> update hr.employees
set salary = salary * 1.1
where hire_date>= to_date('2002-01-01','yyyy-mm-dd') 
       and hire_date < to_date('2003-01-01','yyyy-mm-dd');  

7 rows updated.

--5. # Terminal 3로 allen유저로 접속하셔서 60번 부서 사원들의 급여를 10%인상하세요 단, 트랜잭션은 종료하지 마세요 
SQL> update hr.employees
set salary = salary * 1.1
where department_id = 60;
  
5 rows updated.

--6. # Terminal 1에서 현재 트랜잭션을 수행하고 있는 유저들의 정보를 확인  
SQL>  SELECT s.username,t.xidusn,t.ubafil,t.ubablk,t.used_ublk
FROM v$session s,v$transaction t WHERE s.saddr = t.ses_addr;    2  

USERNAME                           XIDUSN     UBAFIL     UBABLK  USED_UBLKL\
------------------------------ ---------- ---------- ---------- ----------
HAPPY                                   4          3        186          1
HR                                      2          3       1364          1

--7. allen유저는 rollback을 수행한 후 트랜잭션의 정보를 확인
--# Terminal 3  
SQL> rollback;

Rollback complete.

--# Terminal 1

SQL> SELECT s.username,t.xidusn,t.ubafil,t.ubablk,t.used_ublk
FROM v$session s,v$transaction t WHERE s.saddr = t.ses_addr;  2  

USERNAME                           XIDUSN     UBAFIL     UBABLK  USED_UBLK
------------------------------ ---------- ---------- ---------- ----------
HR                                      2          3       1364          1

--8. hr유저가 수행한 트랜잭션 정보가 저장된 언두 세그먼트를 확인  
SQL> @undo_extents

SQL> select * from dba_undo_extents where segment_name = '_SYSSMU2_967517682$'   <- 위 결과의 XIDUSN(=2)의 segment_name을 앞서 수행한 결과에서 찾으세요

OWN SEGMENT_NAME                   TABLESPACE_NAME                 EXTENT_ID    FILE_ID   BLOCK_ID      BYTES     BLOCKS RELATIVE_FNO COMMIT_JTIME COMMIT_WTIME         STATUS
--- ------------------------------ ------------------------------ ---------- ---------- ---------- ---------- ---------- ------------ ------------ -------------------- ---------
SYS _SYSSMU2_967517682$            UNDOTBS1                                0          3        144      65536          8           3                                    EXPIRED
SYS _SYSSMU2_967517682$            UNDOTBS1                                1          3        152      65536          8           3                                    EXPIRED
SYS _SYSSMU2_967517682$            UNDOTBS1                                2          3       1280    1048576        128           3                                    ACTIVE

--9. # Terminal 2에서 hr commit을 수행한후 언두 세그먼트를 확인
SQL> commit;

Commit complete.

# Terminal 1

SQL> select * from dba_undo_extents where segment_name = '_SYSSMU2_967517682$'

OWN SEGMENT_NAME                   TABLESPACE_NAME                 EXTENT_ID    FILE_ID   BLOCK_ID      BYTES     BLOCKS RELATIVE_FNO COMMIT_JTIME COMMIT_WTIME         STATUS
--- ------------------------------ ------------------------------ ---------- ---------- ---------- ---------- ---------- ------------ ------------ -------------------- ---------
SYS _SYSSMU2_967517682$            UNDOTBS1                                0          3        144      65536          8           3                                    EXPIRED
SYS _SYSSMU2_967517682$            UNDOTBS1                                1          3        152      65536          8           3                                    EXPIRED
SYS _SYSSMU2_967517682$            UNDOTBS1                                2          3       1280    1048576        128           3                                    UNEXPIRED

--10. db의 undo 발생량을 확인  
SQL> select to_char(begin_time,'yyyy/mm/dd hh24:mi:ss') begin_time
	, to_char(end_time,'yyyy/mm/dd hh24:mi:ss') end_time, undoblks
from v$undostat;  

TO_CHAR(BEGIN_T TO_CHAR(END_TIM   UNDOBLKS
--------------- --------------- ----------
20190528 054622 20190528 055331         10
20190528 053622 20190528 054622         11

--11. 현재 undo 발생량을 이용하여 undo tablespace크기를 계산 
SQL> SELECT d.undo_size/(1024*1024) "ACTUAL UNDO SIZE [MByte]",
  2      SUBSTR(e.value,1,25) "UNDO RETENTION [Sec]",
  3     (TO_NUMBER(e.value) * TO_NUMBER(f.value) * g.undo_block_per_sec) / (1024*1024) "NEEDED UNDO SIZE [MByte]"
  4  FROM (
  5     SELECT SUM(a.bytes) undo_size
  6     FROM v$datafile a, v$tablespace b, dba_tablespaces c
  7     WHERE c.contents = 'UNDO'
  8     AND c.status = 'ONLINE'
  9     AND b.name = c.tablespace_name
 10     AND a.ts# = b.ts#
 11       ) d,
 12       v$parameter e,
 13       v$parameter f,
 14       (
 15     SELECT MAX(undoblks/((end_time-begin_time)*3600*24)) undo_block_per_sec
 16          FROM v$undostat
 17       ) g
 18  WHERE e.name = 'undo_retention'
 19* AND f.name = 'db_block_size'


ACTUAL UNDO SIZE [MByte] UNDO RETENTION [Sec] NEEDED UNDO SIZE [MByte]
------------------------ -------------------- ------------------------
                     105 900                                 1.1484375

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
