---
layout: post
title: ORACLE ADMIN 12
excerpt: "Auditing"
categories: [ORACLE ADMINISTRATION]
comments: true
---


# Auditing

> Mandatory Auditing
  * AUDIT_FILE_DEST 위치에 sysdba connect만 기록
  * alert

> Standard Database Auditing    
  * AUDIT_TRAIL=db    
  * sys.aud$에 기록    
  * AUDIT, NOAUDIT 명령 사용   
  * 주의사항     
    1.  aud$ 테이블이 너무 커져서 system TS가 full 상태가 되면 데이터베이스는 비정상 종료됨     
	2. aud$ 테이블은 sys가 소유한 테이블 중 유일하게 DBA가 테이블을 관리가능 (DELETE, TRUNCATE TABLE, ALTER TABLE SYS.AUD$ MOVE TABLESPACE USERS)       
	3. AUDIT 작업은 명령처리과정중 execute 과정에서 수행되며 auditing이 실패할 경우 해당 명령어도 정상수행 될 수 없음     
	
> Fine-Grained Auditing
  * sys.fga_log$에 기록    
  * DBMS_FGA 패키지 이용   
 
> Value-based Auditing    
  * TRIGGER 이용  

> SYSDBA 감사 
  * SYS 작업 모두를 기록함  
  * AUDIT_SYS_OPERATIONS = TRUE  
  * AUDIT_FILE_DEST 경로의 *.aud 화일에 기록   
 
{% highlight css %}

SYS@orcl> show parameter audit_trail                --audit_trail초기화 파라미터 조회

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
audit_trail                          string      DB               --값이 DB이면 현재 STANDARD AUDIT이 실행 중

SYS@orcl> audit table;   -- statement auditing
-- CREATE TABLE, DROP USER등의 SQL문장에 대해서 Audit
SYS@orcl> audit select any table;  -- privilege auditing
-- 권한 감사는 시스템권한을 감사
SYS@orcl> audit all on hr.employees;  -- object auditing
-- 특정스키마 오브젝트에 수행되는 명령문을 감사

--시스템 전체 및 사용자가 감사하는 현재 시스템 권한을 설명
select privilege, success, failure
  2  from dba_priv_audit_opts
  3  order by 1;
PRIVILEGE                                SUCCESS    FAILURE
---------------------------------------- ---------- ----------
ALTER ANY PROCEDURE                      BY ACCESS  BY ACCESS
ALTER ANY TABLE                          BY ACCESS  BY ACCESS
ALTER DATABASE                           BY ACCESS  BY ACCESS
ALTER PROFILE                            BY ACCESS  BY ACCESS
ALTER SYSTEM                             BY ACCESS  BY ACCESS
...

24 rows selected.

--시스템 전체 및 사용자 별 현재 시스템 감사 옵션을 설명
select audit_option, success, failure
  2  from dba_stmt_audit_opts
  3  order by 1;

AUDIT_OPTION                             SUCCESS    FAILURE
---------------------------------------- ---------- ----------
ALTER ANY PROCEDURE                      BY ACCESS  BY ACCESS
ALTER ANY TABLE                          BY ACCESS  BY ACCESS
ALTER DATABASE                           BY ACCESS  BY ACCESS
ALTER PROFILE                            BY ACCESS  BY ACCESS
ALTER SYSTEM                             BY ACCESS  BY ACCESS
ALTER USER                               BY ACCESS  BY ACCESS
...

30 rows selected.

--현재 사용자와 관련된 모든 감사 내역 항목을 표시
SYS@orcl> SELECT username, timestamp, owner, obj_name, action_name, sql_text
	FROM dba_audit_trail ;
--테이블의 모든 로우를 제거하기 위한 명령어
SYS@orcl> truncate table aud$;
Table truncated.

SYS@orcl> select username, timestamp, owner, obj_name, action_name, sql_text
  2  from dba_audit_trail;

no rows selected    --제거됨 (DROP TABLE은 테이블 구조 자체를 제거하지만 TRUNCATE TABLE은 테이블의 내용만 제거)

SYS@orcl> CREATE TABLE t (id NUMBER) ;       --SYS에서테이블 생성
 
SYS@orcl> DROP TABLE t PURGE ;               --테이블 삭제

SYS@orcl> SELECT empno, sal
	FROM scott.emp
	WHERE empno = 7788 ;                       --조회가능

SYS@orcl> CONN system/oracle_4U

SYSTEM@orcl> CREATE TABLE t (id NUMBER) ;       --SYSTEM에서 테이블 생성

SYSTEM@orcl> DROP TABLE t PURGE ;               --테이블 삭제

SYSTEM@orcl> SELECT empno, sal
	FROM scott.emp 
	WHERE empno = 7788 ;                      --조회가능

SYSTEM@orcl> CONN / as sysdba

SYS@orcl> SELECT username, timestamp, owner, obj_name, action_name, sql_text, sql_bind
	FROM dba_audit_trail ;          --AUDIT 조회하면 테이블 생성,삭제 언제 했는지까지 기록 남음

USERNAME   TIMESTAMP OWNER      OBJ_NAME   ACTION_NAME                  SQL_TEXT           SQL_B
---------- --------- ---------- ---------- ---------------------------- -------------------- -----
SYSTEM     28-MAY-19 SYSTEM     T          CREATE TABLE
SYSTEM     28-MAY-19 SYSTEM     T          DROP TABLE
SYSTEM     28-MAY-19 SCOTT      EMP        SESSION REC

--db,extended값으로 audit_trail초기화 파라미터 값 변경
SYS@orcl> ALTER SYSTEM SET audit_trail = db,extended SCOPE=SPFILE ;
/*
audit_trail = DB   -- aud$   dba_audit_trail로 확인   
                  OS   -- audit_file_dest 파라메터가 지정하는 경로에 

  aud$ table 을 system이 아니 다른 테이블스페이스로 변경하는 것 고려.
  aud$ table 에 대한 관리( 주기적으로 백업하고 truncate)
*/
  
SYS@orcl> STARTUP FORCE           --다시 시작

SYS@orcl>NOAUDIT select on scott.emp ;     --활성화 증인 audit를 비활성화  (선택한 감사를 중지시킴)

Noaudit succeeded.

SYS@orcl>NOAUDIT CREATE TABLE ;

Noaudit succeeded.

SYS@orcl>NOAUDIT TABLE ;

Noaudit succeeded.

SYS@orcl>AUDIT CREATE SESSION ;       

Audit succeeded.

SYS@orcl> TRUNCATE TABLE aud$ ;
Table truncated.
{% endhighlight %}

## DML 트리거  
  * 사용하는 이유  
   1. dba: auditing, 보안
    특정 data를 누가 조작하고 삭제했는지를 쉽게 확인   
	누가 db에 접속했는지 그 정보를 확인하고자 할 때   
	특정시간에 db작업 못하게 막고 싶을 때  
   2. 개발자  
    두개의 테이블을 동기화하고자 할 때  
	데이터 무결성을 지키기 위해  
	
  * 구성 요소 
   1. event : insert, update, delete
   2. timing : before -> validation    
               after -> 값에 의한 auditing (감사), 복제   
   3. scope : row, statement
   
{% highlight css %}

--FGA(Fined Grained Auditing) 
/*
데이터베이스 감사(Audit)는 작업이 발생했다는 사실은 기록하지만 이 작업을 발생시킨 명령문에 대한 정보는 캡처하지 않는데 
FGA(Fine-Grained Auditing)는 이러한 기능을 확장한 것으로 데이터를 query 또는 조작하는 실제 SQL 문을 캡처가능
"audit_trail"값이 "none"이라고 해도, FGA기능을 사용가능
*/

SYS@orcl>
    BEGIN
        DBMS_FGA.ADD_POLICY(
                 object_schema => 'SCOTT',
                 object_name => 'EMP',
                policy_name => 'CHK_SCOTT_EMP',
                 enable => TRUE,
                statement_types => 'INSERT, UPDATE, SELECT, DELETE' ,
                audit_condition => 'DEPTNO = 10',
                audit_column => 'SAL,COMM',
                audit_column_opts => DBMS_FGA.ALL_COLUMNS,
                audit_trail => DBMS_FGA.DB + DBMS_FGA.EXTENDED);     --sal,comm, deptno=10일때 audit
   END;
/
anonymous block completed

SYS@orcl>SELECT object_schema,
  object_name,
  policy_owner,
  policy_name,
  policy_text,
  policy_column_options
FROM dba_audit_policies ;   --데이터베이스의 모든 세분화 된 감사 정책을 설명

OBJECT_SCH OBJECT_NAM POLICY_OWN POLICY_NAME          POLICY_TEXT                    POLICY_COLU
---------- ---------- ---------- -------------------- ------------------------------ -----------
SCOTT      EMP        SYS        CHK_SCOTT_EMP        DEPTNO = 10                    ALL_COLUMNS

SYS@orcl> SELECT policy_name, policy_column
FROM dba_audit_policy_columns ;               --
 
POLICY_NAME          POLICY_COLUMN
-------------------- ------------------------------
CHK_SCOTT_EMP        SAL
CHK_SCOTT_EMP        COMM

SYS@orcl> conn system/oracle_4U

SYSTEM@orcl> select empno, sal from scott.emp
  2  where deptno=10;

     EMPNO        SAL
---------- ----------
      7782       2450
      7839       5000
      7934       1300
	  

SYSTEM@orcl>select empno, sal from scott.emp
  2* where deptno =20;

     EMPNO        SAL
---------- ----------
      7369        800
      7566       2975
      7788       3000
      7876       1100
      7902       30
	  
SYSTEM@orcl> select empno, sal , comm from scott.emp
  2  where deptno =10;                                  --sal, comm, deptno =10 audit설정 해놓았는데 검색

     EMPNO        SAL       COMM
---------- ---------- ----------
      7782       2450
      7839       5000
      7934       1300

SYSTEM@orcl> select empno,sal,comm from scott.emp
  2  where deptno =20;

     EMPNO        SAL       COMM
---------- ---------- ----------
      7369        800
      7566       2975
      7788       3000
      7876       1100
      7902       3000

SYSTEM@orcl> CONN / as sysdba

SYS@orcl> SELECT timestamp, username, owner, obj_name, action_name, sql_text
	FROM dba_audit_trail ;
no rows selected                            --그래서 검색이 안됨      

SYS@orcl> SELECT timestamp, db_user, object_name, policy_name, statement_type, sql_text
	FROM dba_fga_audit_trail ;              --날짜와 검색 안되는 이유에 대한 레코드=audit 세분화 된 감사에 대한 모든 감사 레코드 표시 

TIMESTAMP DB_USER    OBJECT_NAM POLICY_NAME          STATEME SQL_TEXT
--------- ---------- ---------- -------------------- ------- --------------------------------------------------
28-MAY-19 SYSTEM     EMP        CHK_SCOTT_EMP        SELECT  SELECT empno, sal,comm FROM scott.emp
                                                                WHERE deptno = 10         --이유

SYS@orcl> SELECT audit_type, db_user, policy_name, statement_type, action, audit_option, sql_text
FROM dba_common_audit_trail ;                         --FGA TYPE(감사 추적 항목), 검색 안되는 이유에 대한 레코드

AUDIT_TYPE             DB_USER    POLICY_NAME          STATEMENT_     ACTION AUDIT_OPTI SQL_TEXT
---------------------- ---------- -------------------- ---------- ---------- ---------- --------------------------------------------------
Fine Grained Audit     SYSTEM     CHK_SCOTT_EMP        SELECT                           SELECT empno, sal,comm FROM scott.emp
                                                                                                WHERE deptno = 10	  
--fga_log$: standard auditing의 "aud$" table에 해당하며, 
--모든 FGA records가 저장되어 있으며, dba권한을 가진 user가 data를 delete할 수 있음
SYS@orcl> truncate table fga_log$;  --fga_log$ truncate 함     

Table truncated.

--값 기준 감사 (TRIGGER활용)
SYS@orcl> CREATE TABLE system.audit_emp
               ( os_user VARCHAR2(10),
				dml_date DATE,
				addr VARCHAR2(15),
				description VARCHAR2(1000) ) ;


SYS@orcl> CREATE OR REPLACE TRIGGER system.sal_audit
	AFTER UPDATE OF sal ON scott.emp
	FOR EACH ROW
	BEGIN
	   IF :old.sal <> :new.sal THEN
	      INSERT INTO system.audit_emp
	      VALUES (sys_context('userenv','os_user'), 
                                 sysdate, sys_context('userenv','ip_address'),
	      :new.empno ||' salary changed from '||:old.sal||' to '||:new.sal);
	   END IF;
	END;
	/

SYS@orcl> CONN system/oracle_4U@ORCL

SYSTEM@ORCL> UPDATE scott.emp
	SET sal = sal * 1.1
	WHERE empno = 7788 ;

SYSTEM@ORCL> SELECT * FROM audit_emp ;
	OS_USER DML_DATE ADDR DESCRIPTION
	---------- --------- --------------- ----------------------------------------
	oracle 22-MAY-13 127.0.0.1 7788 salary changed from 3000 to 3300

SYSTEM@ORCL> ROLLBACK ;

SYSTEM@ORCL>SELECT * FROM audit_emp ;

no rows selected

SYSTEM@ORCL> DROP TABLE audit_emp PURGE ;

SYSTEM@ORCL> DROP TRIGGER sal_audit ;

SYSTEM@ORCL> exit

--SYSDBA 감사 
[orcl:~]$ sql

SQL*Plus: Release 11.2.0.1.0 Production on Fri Dec 20 10:09:29 2019

Copyright (c) 1982, 2009, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options
 
SYS@orcl> show parameter audit                  --audit 파라미터 조회

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
audit_file_dest                      string      /u01/app/oracle/admin/orcl/adu
                                                 mp
audit_sys_operations                 boolean     FALSE
audit_syslog_level                   string
audit_trail                          string      NONE
SYS@orcl> exit  
Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options

[orcl:~]$ cd /u01/app/oracle
[orcl:oracle]$ ls
admin  cfgtoollogs  checkpoints  diag  oradata  oradiag_oracle  product
[orcl:oracle]$ cd admin
[orcl:admin]$ ls
+ASM  orcl
[orcl:admin]$ cd orcl
[orcl:orcl]$ cd adump
[orcl:adump]$ ls
orcl_ora_10057_1.aud  orcl_ora_17134_1.aud  orcl_ora_5773_1.aud
orcl_ora_1011_1.aud   orcl_ora_17140_1.aud  orcl_ora_5783_1.aud
orcl_ora_10120_1.aud  orcl_ora_17141_1.aud  orcl_ora_5784_1.aud
orcl_ora_1032_1.aud   orcl_ora_17142_1.aud  orcl_ora_5791_1.aud
orcl_ora_1034_1.aud   orcl_ora_17148_1.aud  orcl_ora_5797_1.aud
.
.
[orcl:adump]$ rm *
[orcl:adump]$ ls
[orcl:adump]$ sql

SQL*Plus: Release 11.2.0.1.0 Production on Fri Dec 20 10:11:51 2019

Copyright (c) 1982, 2009, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options

SYS@orcl> create table t(a number);             --테이블 생성

Table created.

SYS@orcl> select * from scott.emp;            --테이블 검색

     EMPNO ENAME      JOB              MGR HIREDATE         SAL       COMM
---------- ---------- --------- ---------- --------- ---------- ----------
    DEPTNO
----------
      7369 SMITH      CLERK           7902 17-DEC-80        800
        20

      7499 ALLEN      SALESMAN        7698 20-FEB-81       1600        300
        30

      7521 WARD       SALESMAN        7698 22-FEB-81       1250        500
        30
...

14 rows selected.

SYS@orcl> drop table t;             --테이블 삭제

Table dropped.

SYS@orcl> exit
Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options

[orcl:adump]$ ls 
orcl_ora_6056_1.aud
[orcl:adump]$ cat orcl_ora_6056_1.aud   --audit 출력해보지만 아무것도 없음
							--왜냐하면 show parameter audit에서 audit_sys_operations false로 되어있기 때문
Audit file /u01/app/oracle/admin/orcl/adump/orcl_ora_6056_1.aud
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options
ORACLE_HOME = /u01/app/oracle/product/11.2.0/dbhome_1
System name:    Linux
Node name:      edydr1p0.us.oracle.com
Release:        2.6.18-164.el5
Version:        #1 SMP Thu Sep 3 02:16:47 EDT 2009
Machine:        i686
Instance name: orcl
Redo thread mounted by this instance: 1
Oracle process number: 30
Unix process pid: 6056, image: oracle@edydr1p0.us.oracle.com (TNS V1-V3)

Fri Dec 20 10:11:51 2019 +09:00
LENGTH : '160'
ACTION :[7] 'CONNECT'
DATABASE USER:[1] '/'
PRIVILEGE :[6] 'SYSDBA'
CLIENT USER:[6] 'oracle'
CLIENT TERMINAL:[5] 'pts/1'
STATUS:[1] '0'
DBID:[10] '1553401296'

[orcl:adump]$ sql

SQL*Plus: Release 11.2.0.1.0 Production on Fri Dec 20 10:14:38 2019

Copyright (c) 1982, 2009, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options

SYS@orcl> show parameter audit

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
audit_file_dest                      string      /u01/app/oracle/admin/orcl/adu
                                                 mp
audit_sys_operations                 boolean     FALSE
audit_syslog_level                   string
audit_trail                          string      NONE
SYS@orcl> alter system set audit_sys_operations=true;
alter system set audit_sys_operations=true
                 *
ERROR at line 1:
ORA-02095: specified initialization parameter cannot be modified


SYS@orcl> i scope=spfile
SYS@orcl> l
  1  alter system set audit_sys_operations=true       --audit_sys_operations 을 true로 바꿈
  2* scope=spfile
SYS@orcl> /

System altered.

SYS@orcl> startup force                            --DB재시작
ORACLE instance started.

Total System Global Area  577511424 bytes
Fixed Size                  1338000 bytes
Variable Size             448791920 bytes
Database Buffers          121634816 bytes
Redo Buffers                5746688 bytes
Database mounted.
Database opened.

SYS@orcl> show parameter audit                           --true로 바뀐거 확인

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
audit_file_dest                      string      /u01/app/oracle/admin/orcl/adu
                                                 mp
audit_sys_operations                 boolean     TRUE
audit_syslog_level                   string
audit_trail                          string      NONE

SYS@orcl> exit
Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options
[orcl:adump]$ sql

SQL*Plus: Release 11.2.0.1.0 Production on Fri Dec 20 10:16:59 2019

Copyright (c) 1982, 2009, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options

SYS@orcl> create table hr.t(a number);                        --테이블 생성
se
Table created.

SYS@orcl       
SYS@orcl> select *from scott.emp;                               --테이블 검색

     EMPNO ENAME      JOB              MGR HIREDATE         SAL       COMM
---------- ---------- --------- ---------- --------- ---------- ----------
    DEPTNO
----------
      7369 SMITH      CLERK           7902 17-DEC-80        800
        20

      7499 ALLEN      SALESMAN        7698 20-FEB-81       1600        300
        30
.
.
14 rows selected.

SYS@orcl> drop table hr.t;                           --테이블 삭제

Table dropped.

SYS@orcl> exit
Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options
[orcl:adump]$ ls -lrt                               --true로 바꾼 후 audit 확인 했더니 생김
total 28
-rw-r----- 1 oracle dba  799 Dec 20 10:11 orcl_ora_6056_1.aud
-rw-r----- 1 oracle dba  752 Dec 20 10:15 orcl_ora_6101_1.aud
-rw-r----- 1 oracle dba 1007 Dec 20 10:15 orcl_ora_6086_1.aud
-rw-r----- 1 oracle dba  759 Dec 20 10:15 orcl_ora_6101_2.aud
-rw-r----- 1 oracle dba 1460 Dec 20 10:15 orcl_ora_6232_1.aud
-rw-r----- 1 oracle dba 1510 Dec 20 10:16 orcl_ora_6260_1.aud
-rw-r----- 1 oracle dba 1878 Dec 20 10:18 orcl_ora_6299_1.aud

[orcl:adump]$ cat orcl_ora_6299_1.aud
Audit file /u01/app/oracle/admin/orcl/adump/orcl_ora_6299_1.aud
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options
ORACLE_HOME = /u01/app/oracle/product/11.2.0/dbhome_1
System name:    Linux
Node name:      edydr1p0.us.oracle.com
Release:        2.6.18-164.el5
Version:        #1 SMP Thu Sep 3 02:16:47 EDT 2009
Machine:        i686
Instance name: orcl
Redo thread mounted by this instance: 1
Oracle process number: 22
Unix process pid: 6299, image: oracle@edydr1p0.us.oracle.com (TNS V1-V3)

Fri Dec 20 10:16:59 2019 +09:00
LENGTH : '160'
ACTION :[7] 'CONNECT'
DATABASE USER:[1] '/'
PRIVILEGE :[6] 'SYSDBA'
CLIENT USER:[6] 'oracle'
CLIENT TERMINAL:[5] 'pts/1'
STATUS:[1] '0'
DBID:[10] '1553401296'

Fri Dec 20 10:16:59 2019 +09:00
LENGTH : '159'
ACTION :[6] 'COMMIT'
DATABASE USER:[1] '/'
PRIVILEGE :[6] 'SYSDBA'
CLIENT USER:[6] 'oracle'
CLIENT TERMINAL:[5] 'pts/1'
STATUS:[1] '0'
DBID:[10] '1553401296'

Fri Dec 20 10:16:59 2019 +09:00
LENGTH : '159'
ACTION :[6] 'COMMIT'
DATABASE USER:[1] '/'
PRIVILEGE :[6] 'SYSDBA'
CLIENT USER:[6] 'oracle'
CLIENT TERMINAL:[5] 'pts/1'
STATUS:[1] '0'
DBID:[10] '1553401296'

Fri Dec 20 10:17:51 2019 +09:00
LENGTH : '181'
ACTION :[27] 'create table hr.t(a number)'            --테이블 생성했다는 audit
DATABASE USER:[1] '/'
PRIVILEGE :[6] 'SYSDBA'
CLIENT USER:[6] 'oracle'
CLIENT TERMINAL:[5] 'pts/1'
STATUS:[1] '0'
DBID:[10] '1553401296'

Fri Dec 20 10:18:07 2019 +09:00
LENGTH : '176'
ACTION :[22] 'select *from scott.emp'               --테이블 검색했다는 audit
DATABASE USER:[1] '/'
PRIVILEGE :[6] 'SYSDBA'
CLIENT USER:[6] 'oracle'
CLIENT TERMINAL:[5] 'pts/1'
STATUS:[1] '0'
DBID:[10] '1553401296'

Fri Dec 20 10:18:18 2019 +09:00
LENGTH : '169'
ACTION :[15] 'drop table hr.t'                      --테이블 삭제했다는 audit
DATABASE USER:[1] '/'
PRIVILEGE :[6] 'SYSDBA'
CLIENT USER:[6] 'oracle'
CLIENT TERMINAL:[5] 'pts/1'
STATUS:[1] '0'
DBID:[10] '1553401296'

[orcl:adump]$ sql

SQL*Plus: Release 11.2.0.1.0 Production on Fri Dec 20 10:21:11 2019

Copyright (c) 1982, 2009, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options

SYS@orcl> show parameter audit;

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
audit_file_dest                      string      /u01/app/oracle/admin/orcl/adu
                                                 mp
audit_sys_operations                 boolean     TRUE
audit_syslog_level                   string
audit_trail                          string      NONE
SYS@orcl> alter system set audit_sys_operations=false       --다시 false로 바꾼후 
  2  scope=spfile;

System altered.

SYS@orcl> startup force                                      --db재시작
ORACLE instance started.

Total System Global Area  577511424 bytes
Fixed Size                  1338000 bytes
Variable Size             448791920 bytes
Database Buffers          121634816 bytes
Redo Buffers                5746688 bytes
Database mounted.
Database opened.

SYS@orcl> show parameter audit

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
audit_file_dest                      string      /u01/app/oracle/admin/orcl/adu
                                                 mp
audit_sys_operations                 boolean     FALSE
audit_syslog_level                   string
audit_trail                          string      NONE

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
