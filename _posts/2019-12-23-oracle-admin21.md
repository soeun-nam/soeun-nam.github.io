---
layout: post
title: ORACLE ADMIN 14
excerpt: "데이타 이동 2"
categories: [ORACLE ADMINISTRATION]
comments: true
---


# TABLE A CONTENT

> SQL*Loader 사용

> External table 사용
   
> DB Link 사용

## SQL*Loader 사용

{% highlight css %}
[orcl:data]$ ls *.dat
library_items.dat  new_emp.dat
[orcl:data]$ ls *.ctl
sql_loader.ctl
[orcl:data]$ cat sql_loader.ctl
LOAD DATA													
   INFILE 'new_emp.dat'											--infile : 임포트 하려는 데이터 파일의 경로와 이름
   BADFILE 'new_emp.bad'										--badfile : 적재되지 못한 데이터들 ( 데이터 입력 과정 중에서 데이터 형식이나 조건 등의 문제 때문에 입력되지 않은 파일이 저장)
   DISCARDFILE 'new_emp.dis'									--discardfile : 적재되지 못한 데이터들에 대한 정보를 저장 ( 콘트롤 파일의 when 절과 맞지 않는 데이터 저장)
																	--badfile은 insert실행 시 오류가 난 데이터를, discardfile은 insert구문도 실행할 수 없는 레코드를 남김)
   APPEND INTO TABLE new_emp
     WHEN deptno = '30'
     FIELDS TERMINATED BY ',' 
   TRAILING NULLCOLS
   (empno INTEGER EXTERNAL,
    ename CHAR,
    job   CHAR,
    mgr   INTEGER EXTERNAL NULLIF mgr=BLANKS,
    hiredate DATE "YYYY-MM-DD",
    sal   INTEGER EXTERNAL,
    comm  INTEGER EXTERNAL NULLIF comm=BLANKS,
    deptno INTEGER EXTERNAL) 
[orcl:data]$ vi sql_loader.ctl
	--여기서
	LOAD DATA
   INFILE 'new_emp.dat'
   BADFILE 'new_emp.bad'
   DISCARDFILE 'new_emp.dis'
   APPEND INTO TABLE new_emp
     FIELDS TERMINATED BY ',' 
   TRAILING NULLCOLS
   (empno INTEGER EXTERNAL,
    ename CHAR,
    job   CHAR,
    mgr   INTEGER EXTERNAL ,
    hiredate DATE "YYYY-MM-DD",
    sal   INTEGER EXTERNAL,
    comm  INTEGER EXTERNAL,
    deptno INTEGER EXTERNAL) 
이렇게 만들어줌(     WHEN deptno = '30'없애줌)
[orcl:data]$ conn hr/hr     
-bash: conn: command not found
[orcl:data]$ sql

SQL*Plus: Release 11.2.0.1.0 Production on Mon Dec 23 15:51:53 2019

Copyright (c) 1982, 2009, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options


SYS@orcl> conn hr/hr 
Connected.

HR@orcl> CREATE TABLE new_emp                    
(empno number(4),  
ename  varchar2(10),  
job  varchar2(10),  
mgr  number(4),  
hiredate date,  
sal  number(7,2),  
comm  number(7,2),  
deptno  number(2)) ;    

Table created.

HR@orcl>  host sqlldr hr/hr control=sql_loader.ctl
		--host = !
SQL*Loader: Release 11.2.0.1.0 - Production on Mon Dec 23 15:54:17 2019

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.

Commit point reached - logical record count 14

HR@orcl> select * from new_emp;

     EMPNO ENAME      JOB               MGR HIREDATE         SAL       COMM
---------- ---------- ---------- ---------- --------- ---------- ----------
    DEPTNO
----------
      7839 KING       PRESIDENT             17-NOV-81       5000
        10

      7782 CLARK      MANAGER          7839 09-JUN-81       2450
        10

      7566 JONES      MANAGER          7839 02-APR-81       2975
        20

      7654 MARTIN     SALESMAN         7698 28-SEP-81       1250       1400
        30

      7499 ALLEN      SALESMAN         7698 20-FEB-81       1600        300
        30

      7844 TURNER     SALESMAN         7698 08-SEP-81       1500          0
        30

      7900 JAMES      CLERK            7698 03-DEC-81        950
        30

      7521 WARD       SALESMAN         7698 22-FEB-81       1250        500
        30

      7902 FORD       ANALYST          7566 03-DEC-81       3000
        20

      7369 SMITH      CLERK            7902 17-DEC-80        800
        20

      7788 SCOTT      ANALYST          7566 09-DEC-82       3000
        20

      7876 ADAMS      CLERK            7788 12-JAN-83       1100
        20

      7934 MILLER     CLERK            7782 23-JAN-82       1300
        10


13 rows selected.

HR@orcl> host ls *.log
expdp_dump.log  expdp_pump.log  expdp_pump2.log  export.log  impdp_pump.log  impdp_pump2.log  import.log  sql_loader.log

HR@orcl> host cat sql_loader.log

SQL*Loader: Release 11.2.0.1.0 - Production on Mon Dec 23 15:54:17 2019

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.

Control File:   sql_loader.ctl
Data File:      new_emp.dat
  Bad File:     new_emp.bad
  Discard File: new_emp.dis 
 (Allow all discards)

Number to load: ALL
Number to skip: 0
Errors allowed: 50
Bind array:     64 rows, maximum of 256000 bytes
Continuation:    none specified
Path used:      Conventional

Table NEW_EMP, loaded from every logical record.
Insert option in effect for this table: APPEND
TRAILING NULLCOLS option in effect

   Column Name                  Position   Len  Term Encl Datatype
------------------------------ ---------- ----- ---- ---- ---------------------
EMPNO                               FIRST     *   ,       CHARACTER            
ENAME                                NEXT     *   ,       CHARACTER            
JOB                                  NEXT     *   ,       CHARACTER            
MGR                                  NEXT     *   ,       CHARACTER            
    NULL if MGR = BLANKS
HIREDATE                             NEXT     *   ,       DATE YYYY-MM-DD      
SAL                                  NEXT     *   ,       CHARACTER            
COMM                                 NEXT     *   ,       CHARACTER            
    NULL if COMM = BLANKS
DEPTNO                               NEXT     *   ,       CHARACTER            

Record 2: Rejected - Error on table NEW_EMP, column JOB.
ORA-12899: value too large for column "HR"."NEW_EMP"."JOB" (actual: 14, maximum: 10)


Table NEW_EMP:
  13 Rows successfully loaded.
  1 Row not loaded due to data errors.
  0 Rows not loaded because all WHEN clauses were failed.
  0 Rows not loaded because all fields were null.


Space allocated for bind array:                 132096 bytes(64 rows)
Read   buffer bytes: 1048576

Total logical records skipped:          0
Total logical records read:            14
Total logical records rejected:         1
Total logical records discarded:        0

Run began on Mon Dec 23 15:54:17 2019
Run ended on Mon Dec 23 15:54:17 2019

Elapsed time was:     00:00:00.17
CPU time was:         00:00:00.00


HR@orcl> !ls *.bad
new_emp.bad

HR@orcl> host cat new_emp.bad
7698,BLAKE,ABCDEFGHIJKLMN,7839,1981-05-01,2850,,30

-------------------------------------------------------------------------
HR@orcl> !ls *.bad
new_emp.bad

HR@orcl> host cat new_emp.bad
7698,BLAKE,ABCDEFGHIJKLMN,7839,1981-05-01,2850,,30

HR@orcl> !ls *.ctl
sql_loader.ctl

HR@orcl> !cp sql_loader.ctl sql_loader2.ctl

HR@orcl> ed sql_loader2.ctl

	--여기서
	LOAD DATA
   INFILE 'new_emp.dat'
   BADFILE 'new_emp.bad'
   DISCARDFILE 'new_emp.dis'
   APPEND INTO TABLE new_emp
     WHEN deptno = '30'
     FIELDS TERMINATED BY ',' 
   TRAILING NULLCOLS
   (empno INTEGER EXTERNAL,
    ename CHAR,
    job   CHAR,
    mgr   INTEGER EXTERNAL ,
    hiredate DATE "YYYY-MM-DD",
    sal   INTEGER EXTERNAL,
    comm  INTEGER EXTERNAL,
    deptno INTEGER EXTERNAL) 
이렇게 만들어줌

HR@orcl> truncate table new_emp;

Table truncated.

HR@orcl> host sqlldr hr/hr control=sql_loader2.ctl direct=y

SQL*Loader: Release 11.2.0.1.0 - Production on Mon Dec 23 16:01:13 2019

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.


Load completed - logical record count 14.

HR@orcl> select * from new_emp;

     EMPNO ENAME      JOB               MGR HIREDATE         SAL       COMM
---------- ---------- ---------- ---------- --------- ---------- ----------
    DEPTNO
----------
      7654 MARTIN     SALESMAN         7698 28-SEP-81       1250       1400
        30

      7499 ALLEN      SALESMAN         7698 20-FEB-81       1600        300
        30

      7844 TURNER     SALESMAN         7698 08-SEP-81       1500          0
        30

      7900 JAMES      CLERK            7698 03-DEC-81        950
        30

      7521 WARD       SALESMAN         7698 22-FEB-81       1250        500
        30

	--조건 줘서 (when deptno='30') 5개만 들어감
HR@orcl> host ls new_emp*.*
new_emp.bad  new_emp.dat  new_emp.dis

HR@orcl> host ls *.log
expdp_dump.log  expdp_pump.log  expdp_pump2.log  export.log  impdp_pump.log  impdp_pump2.log  import.log  sql_loader.log  sql_loader2.log

HR@orcl> ed sql_loader2.log

HR@orcl> drop table new_emp;

Table dropped.
{% endhighlight %}

## External table 사용

{% highlight css %}

HR@orcl> connect scott/tiger
Connected.


SCOTT@orcl> CREATE TABLE emp_ext ( empno number(4)
                               , ename  varchar2(10)
                               , job     varchar2(9)
                               , mgr   number(4)
                               )
ORGANIZATION EXTERNAL
 (TYPE ORACLE_LOADER 									--type은 oracle_loader 방식과 data_pump 방식이 있음
  DEFAULT DIRECTORY data_dir							--파일이 있는 곳의 오라클 디렉토리 명을 정의
  ACCESS PARAMETERS (RECORDS DELIMITED BY NEWLINE			--레코드구분자는 개행문자. 만약 레코드 구분이 특정 길이로 되어 있고, 개행문자가 아니라면 records fixed 80이렇게 정의 하면 됨
                     FIELDS TERMINATED BY ',')				--필드 구분자는 ',' . 
 LOCATION ('new_emp.dat')								--파일명을 정의. 여러개인 경우 ,로 구분해서 넣어주면 됨
 )
REJECT LIMIT UNLIMITED;									--조건에 맞지 않는 오류데이터가 있는 경우 몇개이상의 에러가 발생하면 중단할지 정의
																--unlimited는 무한을 의미, 에러가 있어도 무시하고 계속 데이터를 읽어들인다
																--대량의 데이터를 처리해야하는경우, 병렬처리를 위해 reject limit unlimited 아래에 parallel을 정의하면 됨
Table created.

SCOTT@orcl> select * from emp_ext;

     EMPNO ENAME      JOB              MGR
---------- ---------- --------- ----------
      7839 KING       PRESIDENT
      7782 CLARK      MANAGER         7839
      7566 JONES      MANAGER         7839
      7654 MARTIN     SALESMAN        7698
      7499 ALLEN      SALESMAN        7698
      7844 TURNER     SALESMAN        7698
      7900 JAMES      CLERK           7698
      7521 WARD       SALESMAN        7698
      7902 FORD       ANALYST         7566
      7369 SMITH      CLERK           7902
      7788 SCOTT      ANALYST         7566
      7876 ADAMS      CLERK           7788
      7934 MILLER     CLERK           7782

13 rows selected.


SCOTT@orcl> CREATE TABLE  dept_ext (dept_id,
                          dept_name,
                          location
                        )
ORGANIZATION EXTERNAL
(
TYPE ORACLE_DATAPUMP
DEFAULT DIRECTORY data_dir
LOCATION ('ora1.exp','ora2.exp')
)
PARALLEL
AS 
SELECT deptno, dname, loc
FROM dept;  

Table created.

SCOTT@orcl> ! ls *.exp
ora1.exp  ora2.exp

SCOTT@orcl> select * from dept_ext;

   DEPT_ID DEPT_NAME      LOCATION
---------- -------------- -------------
        10 ACCOUNTING     NEW YORK
        20 RESEARCH       DALLAS
        30 SALES          CHICAGO
        40 OPERATIONS     BOSTON

{% endhighlight %}

## DB Link 사용  

{% highlight css %}
export ORACLE_SID=ERPDB
mkdir -p /u01/app/oracle/oradata/ERPDB  --파라미터 파일 생성

cd $ORACLE_HOME/dbs
orapwd file=orapwERPDB password=oracle_4U --SYS유저의 패스워드 저장

vi initERPDB.ora  --파라미터파일 편집
db_name=ERPDB
service_names=ERPDB
control_files='/u01/app
/oracle/oradata/ERPDB/control01.ctl' --컨트롤파일 생성
sga_target=400M
pga_aggregate_target=150M 
db_block_size=4096  --다른사이즈로 생성 가능
remote_login_passwordfile='EXCLUSIVE' 
undo_tablespace='UNDOTBS1'  

--SAVE

sqlplus / as sysdba    --SYS접속
CREATE SPFILE FROM PFILE ;  --INSTANCE가 없어도 쓸 수 있는 명령 (SPFILE, PFILE)
STARTUP NOMOUNT


--DB생성 (DB생성하기 위해 파라미터 파일을 편집함) 
CREATE DATABASE ERPDB     --CREATE DATABASE를 함으로써 컨트롤파일과 리두 로그파일이 만들어짐
USER SYS IDENTIFIED BY oracle_4U
USER SYSTEM IDENTIFIED BY oracle_4U
LOGFILE GROUP 1 ('/u01/app/oracle/oradata/ERPDB/redo01a.log'
                ,'/u01/app/oracle/oradata/ERPDB/redo01b.log') SIZE 100M,
        GROUP 2 ('/u01/app/oracle/oradata/ERPDB/redo02a.log'
		        ,'/u01/app/oracle/oradata/ERPDB/redo02b.log') SIZE 100M
CHARACTER SET AL32UTF8
NATIONAL CHARACTER SET AL16UTF16
EXTENT MANAGEMENT LOCAL
DATAFILE '/u01/app/oracle/oradata/ERPDB/system01.dbf' SIZE 400M AUTOEXTEND ON  --데이터 딕셔너리에 들어감
SYSAUX 
DATAFILE '/u01/app/oracle/oradata/ERPDB/sysaux01.dbf' SIZE 200M AUTOEXTEND ON
DEFAULT TABLESPACE USERS
DATAFILE '/u01/app/oracle/oradata/ERPDB/users01.dbf' SIZE 50M AUTOEXTEND ON
DEFAULT TEMPORARY TABLESPACE TEMP 
TEMPFILE '/u01/app/oracle/oradata/ERPDB/temp01.dbf' SIZE 100M AUTOEXTEND ON
UNDO TABLESPACE UNDOTBS1
DATAFILE '/u01/app/oracle/oradata/ERPDB/undotbs01.dbf' SIZE 200M AUTOEXTEND ON ;


--select * from tab$-- 데이터베이스의 실질적 형태이며,카탈로그를 연결하지 않으면 select * from dbs_home이 보이지 않음
@?/rdbms/admin/catalog.sql   --데이터 딕셔너리를 만드는 카탈로그 연결   ?: oracle_home이라는 뜻
@?/rdbms/admin/catproc.sql   --패키지와 함수를 사용하기 위해 연결
connect  system/oracle_4U --sqlplus를 접속하기 위해 connect
@?/sqlplus/admin/pupbld.sql  --sqlplus를 사용하기 위해 연결

exit

cat >> /etc/oratab <<EOF   cat : 출력,  >>이 경로로 EOF사이에 있는 경로를 복사해서 보여줌<<
ERPDB:/u01/app/oracle/product/11.2.0/dbhome_1:N
EOF
{% endhighlight %}

> ORACLE ADMIN COMPLETION !!!

## Buttons

Make any link standout more when applying the `.btn` class.

{% highlight html %}
<a href="#" class="btn btn-success">Success Button</a>
{% endhighlight %}

<div markdown="0"><a href="#" class="btn btn-info">Info Button</a></div>

## Notices

**Watch out!** You can also add notices by appending `{: .notice}` to a paragraph.
{: .notice}
