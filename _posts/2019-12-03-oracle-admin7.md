---
layout: post
title: ORACLE ADMIN 02
excerpt: "There are three major structures in Oracle Database server architecture: memory structures, process structures, and storage structures."
categories: [ORACLE ADMINISTRATION]
comments: true
---

### shared pool
> shared pool이란?    
오라클 SGA를 구성하는 가장 기본적인 구성 요소에 해당
 
> shared pool의 용도란?    
 sql문 실행할때 parse단계에서 shared pool에 저장된 내용을 확인하고 soft parse할건지 hard parse할건지 결정
 동일한 sql문을 실행할때 shared pool에 저장된 내용을 확인후 parse단계를 건너뛰는 soft parse를 함

> shared pool에 저장되는 정보?      
 shared pool속 라이브러리 캐시에 sql문,실행계획,p-code가 저장됨
 LC : SQL문, 실행계획, p-code
 DC : data dictionary 내용을 저장
 
### database buffer cache

> database buffer cache이란?        
 Datafile 들로부터 읽은 Data Block의 복사본을 담고 있는 SGA의 한 부분

> database buffer cache의 용도는?        
 sql문 처리단계에서 Execute단계를 수행할때 데이터버퍼캐시에 저장된 내용을 조회하여 조회가 가능하면 logical read, 데이터 버퍼캐시에 저장된 내용중에 일치하는것이 없어서 디스크에서 데이터버퍼캐시로 해당 데이터를 받아오는 것이 physical read.
 DB작업의 효율적인 작업을 위해 디스크의 데이터를 데이터버퍼캐시에 복사하여 저장해두는 곳

### parameter, parameter file 

> 파라미터 파일이 있는 경로    
 $ORACLE_HOME/dbs
 
> 파라미터 파일의 종류    
 서버 파라미터 파일 (spfile, spfile<SID>.ora) : spfile, binary parameter file
 텍스트 초기화 파라미터 파일 (init<SID>.ora) : init 파일, pfile, text parameter file
 
 동적 파라미터 : DB 운영 중에 수정 가능한 파라미터
 정적 파라미터 : DB 재기동해야 반영되는 파라미터
 

## Code Snippets
{% highlight css %}

--파라미터 정보확인
SQL> select count(*) from v$parameter;

  COUNT(*)
----------
       342    -- Oracle 전체 Parameter 갯수

SQL> select name, value
  2  from v$spparameter
  3  where value is not null  
  4  order by name;

NAME                           VALUE
------------------------------ --------------------------------------------------
audit_file_dest                /u01/app/oracle/admin/orcl/adump
audit_trail                    db
compatible                     11.2.0.0.0
control_files                  +DATA/orcl/controlfile/current.260.1008856401
control_files                  +FRA/orcl/controlfile/current.256.1008856403
db_block_size                  8192
db_create_file_dest            +DATA
db_domain                      example.com
db_name                        orcl
db_recovery_file_dest          +FRA
db_recovery_file_dest_size     4039114752
diagnostic_dest                /u01/app/oracle
dispatchers                    (PROTOCOL=TCP) (SERVICE=orclXDB)
memory_target                  576716800
open_cursors                   300
processes                      150
remote_login_passwordfile      EXCLUSIVE
undo_tablespace                UNDOTBS1

18 rows selected.

--파라미터 생성 및 확인
SQL> show parameter spfile   -- startup시 spfile로 시작되었는지를 확인

NAME TYPE VALUE
------------------------------------ ----------- ------------------------------
spfile string +DATA/orcl/spfileorcl.ora

SQL> ! cat $ORACLE_HOME/dbs/initorcl.ora
SPFILE='+DATA/orcl/spfileorcl.ora'

SQL> ! cp $ORACLE_HOME/dbs/initorcl.ora $ORACLE_HOME/dbs/initorcl.bak		--initorcl.ora를 복사하여 다른이름으로 저장

SQL> CREATE PFILE FROM SPFILE ;  -- spfile의 내용을 읽어 pfile 형식으로 만들어줌
                                    (경로 지정이 없는 경우 $ORACLE_HOME/dbs/initorcl.ora로 생성)  --spfile을 이용해서 pfile을 생성
File created.

SQL> CREATE SPFILE FROM PFILE ;   
     <- initorcl.ora의 내용으로 $ORACLE_HOME/dbs에 spfile을 새로 만듦

--위의 작업으로 다시 spfile이 생성되었으므로 db재시작이후 우선순위에의해서 spfile을 읽어들여 인스턴스 실행하기

SQL> SHUTDOWN IMMEDIATE
SQL> STARTUP
SQL> show parameter spfile

NAME     TYPE        VALUE
-------- ----------- ------------------------------
spfile   string      /u01/app/oracle/product/11.2.0/dbhome_1/dbs/spfileorcl.ora		--spfile로 실행했다는 것을 조회할수있음

SQL> create spfile from pfile;		--현재 spfile로 실행되었기 때문에 spfile을 또 생성할수 없어서 오류! (startup 때 사용한 parameter file은 재생성할 수 없음)
create spfile from pfile
*
ERROR at line 1:
ORA-32002: cannot create SPFILE already being used by the instance

SQL> !ls -l $ORACLE_HOME/dbs/*.ora
-rw-r--r-- 1 oracle oinstall 2851 May 15  2009 /u01/app/oracle/product/11.2.0/dbhome_1/dbs/init.ora
-rw-r----- 1 oracle oinstall  913 May 21 17:49 /u01/app/oracle/product/11.2.0/dbhome_1/dbs/initorcl.ora
-rw-r----- 1 oracle dba      2560 May 21 17:54 /u01/app/oracle/product/11.2.0/dbhome_1/dbs/spfileorcl.ora
...
SQL> ! cat $ORACLE_HOME/dbs/initorcl.ora
...
*.db_recovery_file_dest_size=4039114752
*.db_recovery_file_dest='+FRA'
...
SQL> ! cat $ORACLE_HOME/dbs/spfileorcl.ora
...
*.db_recovery_file_dest_size=4039114752
*.db_recovery_file_dest='+FRA'
...
SQL> ALTER SYSTEM SET db_recovery_file_dest_size = 3G ;

System altered.  -- spfile로 startup 시행했으므로 scope 옵션 생략시 scope=both가 적용되어 시스템과 spfile의 내용이 변경됨
   
SQL> ! cat $ORACLE_HOME/dbs/initorcl.ora
...
*.db_recovery_file_dest_size=4039114752
*.db_recovery_file_dest='+FRA'
...
SQL> ! cat $ORACLE_HOME/dbs/spfileorcl.ora
...
*.db_recovery_file_dest_size=3221225472
*.db_recovery_file_dest='+FRA'
...

SQL> show parameter recovery   -- Parameter 확인

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
db_recovery_file_dest                string      +FRA
db_recovery_file_dest_size           big integer 3G
recovery_parallelism                 integer     0

{% endhighlight %}

{% highlight css %}

-- session레벨로 파라메터 변경시 

SQL>alter syetem set sql_trace=true;

SQL> show parameter sql_trace		--변경내용 확인가능

SQL> ALTER SESSION SET statistics_level = ALL ;	
	
/*
디폴트값인 scope=both(메모리와 spfile둘다 변경)현재 조회하면 TYPICAL→ALL 껏다켜도 ALL
scope=spfile(spfile에서만 변경)이면 현재 조회하더라도 안바뀜. 껏다키면 ALL
scope=memory(메모리에서만 변경)이면 현재조회하면 ALL로바뀜. 껏다키면 원래상태인 TYPICAL이됨
*/
Session altered.

SQL> SELECT name, display_value
FROM v$parameter        -- 현재 SESSION 설정값
where name = 'statistics_level' ;
--or SQL>show parameter statistics_level

NAME DISPLAY_VALUE
------------------------------ ------------------------------
statistics_level ALL

SQL> SELECT name, display_value
FROM v$system_parameter  --현재 SYSTEM 설정값
where name = 'statistics_level' ;

NAME DISPLAY_VALUE
------------------------------ ------------------------------
statistics_level TYPICAL

SQL> ALTER SESSION SET large_pool_size = 20M ;

ERROR at line 1:
ORA-02096: specified initialization parameter is not modifiable with this option -- session레벨의 변경이 안됨

SQL> ALTER SYSTEM SET large_pool_size = 20M ;

System altered.

SQL> show parameter large_pool_size

NAME TYPE VALUE
------------------------------------ ----------- ------------------------------
large_pool_size big integer 20M

SQL> ALTER SYSTEM SET processes = 200 ;

ERROR at line 1:
ORA-02095: specified initialization parameter cannot be modified

SQL> ALTER SYSTEM SET processes = 200 SCOPE = SPFILE ;

System altered.
-- 원상복구

SQL> !mv $ORACLE_HOME/dbs/spfileorcl.ora  $ORACLE_HOME/dbs/spfileorcl.bak
SQL> !cp $ORACLE_HOME/dbs/initorcl.bak    $ORACLE_HOME/dbs/initorcl.ora
SQL> startup force
SQL> show parameter spfile

NAME     TYPE        VALUE
-------- ----------- ------------------------------
spfile   string      +DATA/orcl/spfileorcl.ora

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
