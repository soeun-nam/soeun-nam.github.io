---
layout: post
title: ORACLE ADMIN 09
excerpt: "사용자 보안 관리"
categories: [ORACLE ADMINISTRATION]
comments: true
---


# TABLE A CONTENT

> DATABASE USER 생성  

> 권한 부여 및 취소  
   
> 롤 생성 및 관리  
  
> 프로파일 생성 및 관리   

## DATABASE USER 생성
  
  1.1 USER생성 시 정해지는 것들 
   - username : 30byte 이내, 특수문자 포함 안됨, 문자로 시작   
   - 인증방식 							--user생성후 접속시 인증방식에 관한내용   
      1) db 인증 (password)   
      2) OS 인증 (external)   
      3) Global 인증 (홍채인식이나 지문인식)   
   - default tablespace    
       : tablespace의 지정없이 segment 생성시 공간이 할당되는 곳   
         quota 또는 unlimited tablespace 권한이 필요   
   - temporary tablespace   
       : sort,hash join 작업을 위한 임시 저장공간이 할당되는 곳   
         quota 또는 unlimited tablespace 권한이 필요 없음   
   - user profile    
       : 자원, 패스워드 사용을 제한하기 위해 사용   
   - consumer group   
       : Resource Manager와 연동하여 사용하기 위한 지정   
   - 계정의 상태    
       : 로그인 가능여부(lock, unlock)      
	   
  create user demo   
  identified by demo ==> 인증방식 : 패스워드인증(db인증)   
  --default tablespace users   
  --temporary tablespace temp   
  --profile default    
  --account unlock		--아래 4줄은 입력하지않아도 디폴트로 설정되는 것들    
 
 * SELECT * FROM DBA_USERS WHERE USERNAME = 'DEMO';    
  조회하면 입력하지 않은 4줄의 정보들이 그대로 디폴트 값으로 지정해서 생성  
  
 * SELECT * FROM DATABASE_PROPERTIES;   
  데이터베이스의 영구적이 설정이 저장된 정보    
  
 * ALTER DATABASE DEFAULT TABLESPACE EXAMPLE;  
  설정을 변경하게 되면 DEFAULT로 USERS 테이블 스페이스를 받았던 것들은 EXAMPLE로 바뀜  
  
 * CREATE USER OPS$ORACLE IDENTIFIED EXTERNALLY;  
  인증방식: OS인증 -> OPS$ORACLE은 OS계정당 하나씩만 만들수 있음   
 
 * SHOW PARAMETER OS_AUTHENT_PREFIX    
  OS인증으로 접속할수 있느 USER는 OSos_authent_prefix파라메터에 명시된 ops$가 붙은 user만 가능함.   
  파라메터에 ops$대신 null이면 아무것도 안붙여도됨. 단, os계정당 하나의 os인증 user만가능함.    
 
 1.2 SCHEMA   
  * USER 생성시 같은 이름으로 정의됨  
  * 해당 사용자가 생성한 모든 OBJECT들의 모임  
  
  user DEMO: object의 소유자, 권한(privilige)
  schema  DEMO.emp
	          .dept		: 스키마란 오브젝트들의 집합,모임   
			  
 1.3 미리 정의된 DB관리계정   
  - sys : sysdba, sysoper로만(외부인증) 접속 가능, DD의 소유자, OWR의 소유자
          모든 권한을 with admin 옵션과 함께 가짐

  - system : dba , tool 등에 만드는 테이블의 소유자.
 
  - dbsnmp : EM을 위한 사용자
   
  - sysman : EM 저장소(OMR)의 소유자

  EM - database control, grid control  OMR  
  
 1.4 DATABASE USER 생성   
  * 패스워드로 인증하는 유저 생성   
    sqlplus / as sysdba    
	CREATE USER user1       
	IDENTIFIED BY newuser    -- 패스워드는 대소문자 구분(30byte이내),   
	PASSWORD EXPIRE   
	DEFAULT TABLESPACE users   
	QUOTA 1M ON users    
	QUOTA 5M ON example  
	TEMPORARY TABLESPACE temp   
	PROFILE DEFAULT   
	ACCOUNT UNLOCK ;   
  
  - quota: 테이블스페이스의 허가된 할당량, 기본적으로 0  
  - 일반사용자에게 RESOURCE 롤 부여시 UNLIMITIED TABLESPACE 권한이 추가부여되어 QUOTA를 지정하지 않아도 모든 테이블스페이스에서 공간할당이 가능  


{% highlight css %}
SYS@orcl>CREATE USER user1    
IDENTIFIED BY newuser	 -- 암호로 인증
PASSWORD EXPIRE 	 -- 생성하고 첫번째 로그인시 패스워드 만료됨
DEFAULT TABLESPACE users -- 테이블스페이스를 명시하지 않으면 segment가 생성될 위치
QUOTA 1M ON users 	 -- 테이블스페이스에 대한 할당량
QUOTA 5M ON example
TEMPORARY TABLESPACE temp -- disk sort시에 사용할 테이블스페이스 
PROFILE DEFAULT 	  -- default 라는 프로파일이 할당됨
ACCOUNT UNLOCK ;  	  -- 계정의 상태가 open

User created.


SYS@orcl>select username from dba_users
  2  order by username;

USERNAME
------------------------------
ANONYMOUS
APEX_030200
...

SCOTT
SH
SI_INFORMTN_SCHEMA
SPATIAL_CSW_ADMIN_USR
SPATIAL_WFS_ADMIN_USR
SYS
SYSMAN
SYSTEM
USER1
WMSYS
XDB
XS$NULL

37 rows selected.

SYS@orcl>conn user1/newuser
ERROR:
ORA-28001: the password has expired

Changing password for user1
New password: 
Retype new password: 
ERROR:
ORA-01045: user USER1 lacks CREATE SESSION privilege; logon denied

Password changed
Warning: You are no longer connected to ORACLE.
@>conn / as sysdba
Connected.
SYS@orcl>grant create session to user1;

Grant succeeded.

SYS@orcl>conn user1/oracle_4U
Connected.
USER1@orcl>show user
USER is "USER1"
{% endhighlight %}


 * OS 인증 가능한 USER 생성  
   os_authent_prefix = ops$  
   CREATE USER ops$oracle   
   IDENTIFIED EXTERNALLY ;
   
   
{% highlight css %}
drop user ops$oracle cascade;
drop user oracle cascade;

SYS@orcl>show parameter os_authent_prefix

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
os_authent_prefix                    string      ops$


SYS@orcl>CREATE USER ops$oracle 
IDENTIFIED EXTERNALLY ;  2  

User created.
 

SYS@orcl>grant create session to ops$oracle;

Grant succeeded.


SYS@orcl>conn /
Connected.
OPS$ORACLE@orcl>show user
USER is "OPS$ORACLE"
OPS$ORACLE@orcl>conn / as sysdba

SYS@orcl> drop user ops$oracle;

SYS@orcl> exit
 
{% endhighlight %}  
   
 1.5 관리자 인증 방법 2가지  
  * OS 측 권한 : DBA로 작업시 OS화일의 생성, 삭제를 위한 OS 쪽 권한이 필요     
  * DB 측 권한 : SYSDBA, SYSOPER, SYSASM DML 의 자격으로 접속하려면 OS인증과 PASSWORD FILE 인증을 해야함    
   - OS 인증 : DB 생성시에 지정한 OSDBA, OSOPER -> USER GROUP에 속하는 OS USER로 OS에 로그인 한 후 DB에 접속   
                                            -> $SQLPLUS / AS SYSDBA  (관리자 접속시 OS인증 방법 시 비밀번호가 틀리더라도 OS인증 접속이기 때문에 접속 됨)   
											
   - PASSWORD FILE 인증 : PASSWORD FILE 생성(화일 위치와 이름은 정해짐, SYS의 패스퉈드 변경시 재생성 필요)    
                         인증 가능한 유저 확인 -> SELECT * FROM V$PWFILE_USERS;    
						 인증 불가하게 하려면 -> REMOTE_LOGIN_PASSOWORDFILE = NONE;    
						 
 1.6 USER변경  
  alter user scott identified by lion   
  default tablespace ts1   
  temporary tablespace temp2    
  quota 100m on users   
  quota 0m on users    
  account lock  

## 권한 부여 및 취소   

  * 시스템 권한  : 데이터베이스에서 수행할 수 있는 권한 (dba가 권한 부여), with admin option   
   - 부여 명령    
   GRANT  ...  TO ...   -- 시스템권한        
   WITH ADMIN OPTION;   --> 권한 부여자, 받은자의 자격이 동등해짐     
                            직접 권한부여하지 않았어도 회수 가능
							
   - 회수 명령  
   REVOKE ...  FROM ....;    -- 시스템 권한은 연쇄적으로 회수하지 않음    
   
  * 객체 권한 : 특정 object를 액세스하거나 조작할 수 있는 권한  (object 소유자가 권한 부여), with grant option   
   - 부여 명령    
   GRANT ...            -- 객체권한   
   ON   
   TO   
   WITH GRANT OPTION;   
   
   - 회수 명령   
   REVOKE ...                -- 직접 부여한 권한만 회수 가능                                
   ON ...                    -- 연쇄적으로 회수함     
   FROM ....;        
 
{% highlight css %}

--demo가 가진 권한
--system: create table
select privilege from user_sys _privs;

--role: connect, resource
select* from user_role_privs;
select roll||'  :  '||privilege from role_sys_privs;
select privilege||'  ON '||owner||'....'||table_name from user_role_privs;

--object: select on scott.emp
--col level: update(sal, job) on scott.emp
select * from user_tab_privs;
select * from user_col_privs;

[sql developer]
create user demo2
identified by demo2
password expire;

select * from dba_users
where username like 'DEMO%';

create user demo3
identified by demo3
password expire
account lock;           --비밀번호 lock 걸리게함

grant create session to demo2, demo3;

[orcl:~]$ sqlplus demo2/demo2         --demo2 접속

SQL*Plus: Release 11.2.0.1.0 Production on Fri Dec 13 14:35:47 2019

Copyright (c) 1982, 2009, Oracle.  All rights reserved.

ERROR:
ORA-28001: the password has expired      --새로운 패스워드 입력


Changing password for demo2
New password: 
Retype new password: 
Password changed

Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, Automatic Storage Management, OLAP, Data Mining
and Real Application Testing options

DEMO2@orcl> conn demo3/demo3          --demo3 접속
ERROR:
ORA-28000: the account is locked      --lock이 걸림

Warning: You are no longer connected to ORACLE.
@> 

[sql developer]
alter user demo3 account unlock;   --lock 걸린거 풀리게함	

{% endhighlight %}

## 롤 생성 및 관리   
  
  - 권한 부여, 회수 등의 관리작업의 편의를 위해 나온 개념   
  - 같이 부여하려는 권한들을 그룹화   
  - 부여받은 후 다음 세션부터 사용 가능  
  - 롤을 롤에게 부여 가능 (자신에게 X)   

 * 롤의 enable/disable   
  --set role 명령으로 지정된 것만 enable 상태가 됨   
   ALLEN> set role ROLE_B;            
   ALLEN> update ...  -- ROLE_B 가 활성화 됐으므로 수행	--이때 rola_a가 비활성화됨    
   
   --롤의 생성시 패스워드를 설정한 경우는 활성화시 암호를 지정해야 함    

  	CREATE ROLE r1 IDENTIFIED BY oracle_4U;   
  	SET ROLE r1 IDENTIFIED BY oracle_4U;    

   --다시 role_b를 활성화하면 role_a가 비활성화됨. 하지만 디폴트롤이 role_a이므로 세션을 다시접속하면 role_a만 활성화되어있음     
 
 * 프로그램 내에서 롤의 enable    
  create or replace procedure compute_salary ....    
    begin    
    dbms_session.set_role('role_b');   --프로시저에서 활성화 해서 사용.    
	
 * Application Role    
   프로그램내에서 롤 활성화 명령을 수행 
   dbms_session.set_role('role_b');  

  
{% highlight css %}

SQL> conn / as sysdba
Connected.
SQL>  create user u1 identified by oracle;

User created.

SQL> create role r1;

Role created.

SQL> create role r2;

Role created.

SQL> grant select on scott.emp to r1;

Grant succeeded.

SQL> grant update on scott.emp to r2;

Grant succeeded.

SQL> grant connect, r1, r2 to u1;  

Grant succeeded.

SQL> alter user u1 default role connect, r1;

User altered.
-- 자동적으로 default role이 아닌 r2 는 비활성화

SQL> conn u1/oracle
Connected.
SQL> select count(*) from scott.emp;    -- default role

  COUNT(*)
----------
        14

SQL> update scott.emp set sal=1000;     -- r2 는 비활성화 상태라서
update scott.emp set sal=1000
             *
ERROR at line 1:
ORA-01031: insufficient privileges

SQL> set role r2;                      -- r2만 활성화됨

Role set.

SQL> update scott.emp set sal=1000;

14 rows updated.

SQL> rollback;  

Rollback complete.

SQL> select count(*) from scott.emp;     -- r1은 자동으로 비활성화
select count(*) from scott.emp
                           *
ERROR at line 1:
ORA-01031: insufficient privileges

SQL> conn u1/oracle 
Connected.
SQL> select count(*) from scott.emp;     -- 접속할 때마다 default role만 활성화	 

  COUNT(*)
----------
        14

SQL> conn / as sysdba
Connected.
SQL> drop role r1;  

Role dropped.

SQL> drop role r2;

Role dropped.

SQL> drop user u1;

User dropped.

--부여된 롤, 권한을 확인하는 data dictionary view들   
user2 >   SELECT * FROM USER_SYS_PRIVS;  -- SYSTEM 권한
          SELECT * FROM USER_TAB_PRIVS; -- 객체 권한
          SELECT * FROM USER_COL_PRIVS; -- 객체 권한 (컬럼 레벨로 준 권한)
                                           UPDATE (SAL,DEPTNO) on EMP
          SELECT * FROM USER_ROLE_PRIVS; -- ROLE로 부여된 권한
          SELECT * FROM ROLE_SYS_PRIVS;  -- ROLE내의 시스템 권한
          SELECT * FROM ROLE_TAB_PRIVS; -- ROLE내의 객체 권한


	select * from session_privs;	session 에서 사용할 수 있는 system privs
	
{% endhighlight %}  


## PROFILE 생성 및 관리   

 * 오라늘의 자원 사용에 제한을 둘 수 있으며, 특정 사용자가 오라클의 자원을 무한히 사용하지 못하도록 제한을 두는 기능    
 * 패스워드 관리 기능  
 
{% highlight css %}
 
--사용자들에게 할당된 PROFILE? 
select username, profile    --'DEFAULT'
from dba_users;

SQL> select username, profile from dba_users;

USERNAME                       PROFILE
------------------------------ ------------------------------
HR                             DEFAULT
SCOTT                          DEFAULT
SPATIAL_WFS_ADMIN_USR          DEFAULT
SPATIAL_CSW_ADMIN_USR          DEFAULT
APEX_PUBLIC_USER               DEFAULT
OE                             DEFAULT
DIP                            DEFAULT
SH                             DEFAULT
...

--DEFAULT 프로파일의 내용은? 
select *
from dba_profiles
where profile='DEFAULT';

PROFILE                        RESOURCE_NAME                    RESOURCE LIMIT
------------------------------ -------------------------------- -------- ----------------------------------------
DEFAULT                        COMPOSITE_LIMIT                  KERNEL   UNLIMITED
DEFAULT                        SESSIONS_PER_USER                KERNEL   UNLIMITED
...

--자원 관리를 활성화하기 위해서는 RESOURCE_LIMIT 파라미터를 TRUE로 설정해야하며, 암호관리는 항상 활성화됨  ]
--예를 들어 sessions_per_user 2 라고 설정되어 있다면 resource_limit이 true로 되어있으면 설정대로 세션이 2개밖에 열리지 않음. 3개째부터 에러  
alter system set resource_limit = true;
 
--SQL 명령어 수행 시 LOGICAL READ가 10회를 넘지 않도록 DEFAULT PROFILE을 변경
SQL> alter profile default limit
  2  failed_login_attempts 3
  3  password_lock_time 3;         -- 기간 (일)

Profile altered.

SQL> alter profile default limit
  2  logical_reads_per_call 10; 

Profile altered.


SQL> show parameter resource_limit

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
resource_limit                       boolean     TRUE

  
SQL> conn / as sysdba
Connected.
SQL> alter user sh identified by oracle_4U account unlock;

User altered.

SQL> conn sh/oracle_4U
Connected.
SQL> select count(*)   
  2  from customers c, sales s;
from customers c, sales s
     *
ERROR at line 2:
ORA-02395: exceeded call limit on IO usage

SQL> conn / as sysdba
Connected.

SQL> alter profile default limit
  2  logical_reads_per_call unlimited;

Profile altered.

--프로파일을 새로 만들어서 IDLE_TIME을 3분으로 설정해서 SCOTT에게 그 프로파일을 할당하고 테스트 
SQL> create profile idle_profile limit
  2  idle_time 3;

Profile created.

SQL> select * from dba_profiles
  2* where profile='IDLE_PROFILE'
SQL> /

PROFILE                        RESOURCE_NAME                    RESOURCE LIMIT
------------------------------ -------------------------------- -------- ----------------------------------------
IDLE_PROFILE                   COMPOSITE_LIMIT                  KERNEL   DEFAULT
IDLE_PROFILE                   SESSIONS_PER_USER                KERNEL   DEFAULT
IDLE_PROFILE                   CPU_PER_SESSION                  KERNEL   DEFAULT
IDLE_PROFILE                   CPU_PER_CALL                     KERNEL   DEFAULT
IDLE_PROFILE                   LOGICAL_READS_PER_SESSION        KERNEL   DEFAULT
IDLE_PROFILE                   LOGICAL_READS_PER_CALL           KERNEL   DEFAULT
...

16 rows selected.

SQL> alter system set resource_limit=true;

System altered.


SQL>  alter user scott                      
  2  profile idle_profile;

User altered.

SQL> select username, profile
  2  from dba_users
  3  where username='SCOTT';

USERNAME                       PROFILE
------------------------------ ------------------------------
SCOTT                          IDLE_PROFILE

SQL> conn scott/tiger
Connected.
SQL> select * from emp;

     EMPNO ENAME      JOB              MGR HIREDATE         SAL       COMM     DEPTNO
---------- ---------- --------- ---------- --------- ---------- ---------- ----------
      7369 SMITH      CLERK           7902 17-DEC-80        800                    20
      7499 ALLEN      SALESMAN        7698 20-FEB-81       1600        300         30
      7521 WARD       SALESMAN        7698 22-FEB-81       1250        500         30
      7566 JONES      MANAGER         7839 02-APR-81       2975                    20
...
14 rows selected.

--10분 정도 지난 후 다음의 명령을 수행하여 세션이 종료된 것을 확인 !!!

SQL> l
  1* select * from emp
SQL> /
select * from emp
*
ERROR at line 1:
ORA-02396: exceeded maximum idle time, please connect again

SQL> conn / as sysdba
Connected.
SQL> drop profile idle_profile;
drop profile idle_profile
*
ERROR at line 1:
ORA-02382: profile IDLE_PROFILE has users assigned, cannot drop without CASCADE

SQL> drop profile idle_profile cascade;

Profile dropped.

--$ORACLE_HOME/rdbms/admin/utlpwdmg.sql 스크립트 실행하여 엄격한 패스워드 관리

[orcl:~]$ sqlplus / as sysdba

SQL> alter user scott identified by lion;

User altered.

SQL> conn scott/lion
Connected.

SQL> conn / as sysdba
Connected.
SQL> @$ORACLE_HOME/rdbms/admin/utlpwdmg.sql

Function created.

Profile altered.

Function created.

SQL> alter user scott identified by tiger;     
alter user scott identified by tiger
*
ERROR at line 1:
ORA-28003: password verification for the specified password failed
ORA-20001: Password length less than 8

SQL> alter user scott identified by oracle_4U;

User altered.

SQL> conn scott/oracle_4U
Connected.

SQL> conn / as sysdba
Connected.

SQL> select * from dba_profiles
  2  where profile='DEFAULT';

PROFILE                        RESOURCE_NAME                    RESOURCE LIMIT
------------------------------ -------------------------------- -------- ----------------------------------------
DEFAULT                        COMPOSITE_LIMIT                  KERNEL   UNLIMITED
DEFAULT                        SESSIONS_PER_USER                KERNEL   UNLIMITED
DEFAULT                        CPU_PER_SESSION                  KERNEL   UNLIMITED
DEFAULT                        CPU_PER_CALL                     KERNEL   UNLIMITED
DEFAULT                        LOGICAL_READS_PER_SESSION        KERNEL   UNLIMITED
DEFAULT                        LOGICAL_READS_PER_CALL           KERNEL   UNLIMITED
DEFAULT                        IDLE_TIME                        KERNEL   UNLIMITED
DEFAULT                        CONNECT_TIME                     KERNEL   UNLIMITED
DEFAULT                        PRIVATE_SGA                      KERNEL   UNLIMITED
DEFAULT                        FAILED_LOGIN_ATTEMPTS            PASSWORD 10
DEFAULT                        PASSWORD_LIFE_TIME               PASSWORD 180
DEFAULT                        PASSWORD_REUSE_TIME              PASSWORD UNLIMITED
DEFAULT                        PASSWORD_REUSE_MAX               PASSWORD UNLIMITED
DEFAULT                        PASSWORD_VERIFY_FUNCTION         PASSWORD VERIFY_FUNCTION_11G
DEFAULT                        PASSWORD_LOCK_TIME               PASSWORD 1
DEFAULT                        PASSWORD_GRACE_TIME              PASSWORD 7

16 rows selected.

SQL> alter profile default limit 
  2  password_verify_function null;

Profile altered.

SQL> alter user scott identified by tiger;

User altered.

{% endhighlight %}

 * 패스워드 관리를 엄격하게 제한하는 방법 (함수 사용)   
  verify_function_11g  (p 8-30)   $ORACLE_HOME/rdbms/admin/utlpwdmg.sql      
  위의 함수를 DEFAULT 프로파일에 적용을 하게 되면 다음과 같은 제한을 받음      
  1.  패스워드를 8자 이상으로     
  2.  유저이름, 유저이름을 거꾸로 해서 생성 못함, 유저이름에 숫자를 붙인 형태도 설정 못함    
  3.  데이터베이스 이름도 사용못함 ( connect scott/orcl )    
  4.  최소 하나의 문자, 하나의 숫자, 하나의 특수문자를 넣어줘야 패스워드가 생성이 됨    
  5.  직전 패스워드와 3자이상 달라야 함     

 * 패스워드를 엄격하게 관리하기 위한 함수 생성과 default 프로파일에 적용하는 스크립트    
  SQL> @?/rdbms/admin/utlpwdmg.sql   
       ↑ 
    오라클 홈디렉토리     

  SQL> ed ?/rdbms/admin/utlpwdmg.sql  

  ALTER PROFILE DEFAULT LIMIT
  PASSWORD_LIFE_TIME 180    --- 180일동안 현재 패스워드 사용가능   
  PASSWORD_GRACE_TIME 7 -- 패스워드 만료시 유예기간 7일    
  PASSWORD_REUSE_TIME UNLIMITED   
  PASSWORD_REUSE_MAX UNLIMITED   
  FAILED_LOGIN_ATTEMPTS 10    
  PASSWORD_LOCK_TIME 1    
  PASSWORD_VERIFY_FUNCTION verify_function_11G;   

 - 사용자가 지정된 profile의 삭제시는 cascade 옵션을 사용 : 관련 사용자는 default profile로 재지정됨   
 
### user 관리 실습 복습하기  

{% highlight css %}

--8장 user 관리 실습 복습
SYS@orcl> create role query_role;     --롤 생성
Role created.

SYS@orcl> grant select any table to query_role;       --롤에게 권한 부여
Grant succeeded.

SYS@orcl> create user james identified by oracle        --유저 생성
		  default tablespace users 
		  temporary tablespace temp  
		  quota 1m on users;                    
User created.

SYS@orcl> grant create session to james;            --유저에게 세션 권한 부여
Grant succeeded.

SYS@orcl> select * from dba_users where username='JAMES';        --james유저가 가지고 있는 정보

USERNAME                          USER_ID PASSWORD
------------------------------ ---------- ------------------------------
ACCOUNT_STATUS                   LOCK_DATE EXPIRY_DA
-------------------------------- --------- ---------
DEFAULT_TABLESPACE             TEMPORARY_TABLESPACE           CREATED
------------------------------ ------------------------------ ---------
PROFILE                        INITIAL_RSRC_CONSUMER_GROUP
------------------------------ ------------------------------
EXTERNAL_NAME
--------------------------------------------------------------------------------
PASSWORD E AUTHENTI
-------- - --------
JAMES                                 113
OPEN                                       14-JUN-20
USERS                          TEMP                           17-DEC-19
DEFAULT                        DEFAULT_CONSUMER_GROUP

10G 11G  N PASSWORD

SYS@orcl> grant create session, query_role to james;  -- jame 유저에게 create session 권한, query_role를 부여
Grant succeeded.

SYS@orcl> conn james/oracle
Connected.
JAMES@orcl> select * from session_privs;   -- james가 가지고 있는 세션 정보 확인

PRIVILEGE
----------------------------------------
CREATE SESSION
SELECT ANY TABLE

JAMES@orcl> select * from user_sys_privs;

USERNAME                       PRIVILEGE                                ADM
------------------------------ ---------------------------------------- ---
JAMES                          CREATE SESSION                           NO

JAMES@orcl> set lines 80
JAMES@orcl> select * from user_role_privs;

USERNAME                       GRANTED_ROLE                   ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
JAMES                          QUERY_ROLE                     NO  YES NO

JAMES@orcl> conn / as sysdba
Connected.
SYS@orcl> alter user james default role none;          --DEF YES에서 NO로 변경

User altered.

SYS@orcl> conn james/oracle
Connected.
JAMES@orcl> select * from user_role_privs;

USERNAME                       GRANTED_ROLE                   ADM DEF OS_
------------------------------ ------------------------------ --- --- ---     --DEF가 NO이면 평상시에 사용하지 못하게 설정함
JAMES                          QUERY_ROLE                     NO  NO  NO

JAMES@orcl> select * from dual;                    --dual은 검색이 되지만

D
-
X

JAMES@orcl> select * from scott.emp;               --scott는 검색이 안됨
select * from scott.emp
                    *
ERROR at line 1:
ORA-00942: table or view does not exist


JAMES@orcl> set role query_role;           --롤을 처음 로그인 할때만 활성화

Role set.

JAMES@orcl> select * from scott.emp;       --다시 검색해봤을때 활성화 시켰으니까 검색 됨

     EMPNO ENAME      JOB              MGR HIREDATE         SAL       COMM
---------- ---------- --------- ---------- --------- ---------- ----------
    DEPTNO
----------
      7369 SMITH      CLERK           7902 17-DEC-80        800
        20
      7499 ALLEN      SALESMAN        7698 20-FEB-81       1600        300
        30
14 rows selected.

JAMES@orcl> conn james/oracle               --다시 로그인
Connected.
JAMES@orcl> select * from scott.emp;          --다시 검색이 안됨
select * from scott.emp
                    *
ERROR at line 1:
ORA-00942: table or view does not exist


JAMES@orcl> desc dbms_session                 --alter session, set role 명령문 및 기타 세션 정보에 대한 정보 제공

JAMES@orcl> exec dbms_session.set_role('QUERY_ROLE');   --set_role이라는 프로시저를 사용해서 롤 활성화

PL/SQL procedure successfully completed.

JAMES@orcl> select * from scott.emp;           --활성화 해서 검색 가능

     EMPNO ENAME      JOB              MGR HIREDATE         SAL       COMM
---------- ---------- --------- ---------- --------- ---------- ----------
    DEPTNO
----------
      7369 SMITH      CLERK           7902 17-DEC-80        800
        20
      7499 ALLEN      SALESMAN        7698 20-FEB-81       1600        300
        30
14 rows selected.

/*
set role과 차이점
set role은 ddl만 실행 가능하지만 dbms_session은 dml만 가능(create, alter등등 불가능)
create ..
begin
set role query_role  -- 실행불가
=>dbms_session을 이용해서 실행 할 수 있게 해주는 프로시저
end;
/
*/
--PL/SQL -> PROCEDURE, FUNCTION, PACKAGE, TRIGGER
--TRIGGER: 특정 EVENT가 발생하면 자동으로 실행되는 프로그램

--매일 오후 16시에 접속을 하면 james user가 받은 query_role 활성화 되도록 trigger를 생성하세요.
CREATE OR REPLACE TRIGGER logon_trig
AFTER LOGON ON SCHEMA.JAMES           --로그온 할때마다 자동 실행이 되도록 해주는 프로그램(EVENT: LOGON)
BEGIN
IF (TO_CHAR (SYSDATE, 'HH24MISS') BETWEEN '160000' AND '165959') --TIME(AFTER, BEFORE) 160000(4시),165959(5시 되기 전까지)
     THEN   
        DBMS_SESSION.SET_ROLE('query_role');
     ELSE
	DBMS_SESSION.SET_ROLE('NONE');
     END IF;
END;
/
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
