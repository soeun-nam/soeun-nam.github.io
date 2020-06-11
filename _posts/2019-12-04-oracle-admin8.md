---
layout: post
title: ORACLE ADMIN 03
excerpt: "There are three major structures in Oracle Database server architecture: memory structures, process structures, and storage structures."
categories: [ORACLE ADMINISTRATION]
comments: true
---

### Explanation

> DB STARTUP 단계별 조회, Read only 모드로 startup 하기, 제한모드로 DB 시작하기


## Code Snippets

{% highlight css %}

--1. sys 유저로 접속
[oracle@localhost ~]$ sqlplus / as sysdba -- sqlplus sys/oracle as sysdba
SQL*Plus: Release 11.2.0.1.0 Production on Mon Jun 6 20:37:29 2016
Copyright (c) 1982, 2009, Oracle.  All rights reserved.
Connected to an idle instance.

SQL> select status from v$instance;
select status from v$instance
*
ERROR at line 1:
ORA-01034: ORACLE not available
Process ID: 0
Session ID: 0 Serial number: 0   -- 현재 shutdown 상태

--2. db startup 단계
SQL> startup nomount
ORACLE instance started.

Total System Global Area  849530880 bytes
Fixed Size                  1339824 bytes
Variable Size             503320144 bytes
Database Buffers          339738624 bytes
Redo Buffers                5132288 bytes

SQL> select status from v$instance;

STATUS
------------
STARTED

SQL> alter database mount;
Database altered.

SQL> select status from v$instance;
STATUS
------------
MOUNTED

SQL> alter database open;
Database altered.

SQL> select status from v$instance;
STATUS
------------
OPEN

SQL> select open_mode from v$database;
OPEN_MODE
--------------------
READ WRITE

SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.

SQL> startup nomount
ORACLE instance started.

Total System Global Area  849530880 bytes
Fixed Size                  1339824 bytes
Variable Size             503320144 bytes
Database Buffers          339738624 bytes
Redo Buffers                5132288 bytes

SQL> alter database open;
alter database open
*
ERROR at line 1:
ORA-01507: database not mounted

SQL> alter database mount;
Database altered.

SQL> alter database open;
Database altered.

--3. db 읽기모드로 시작

SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.

SQL> startup open read only;
ORACLE instance started.

Total System Global Area  849530880 bytes
Fixed Size                  1339824 bytes
Variable Size             503320144 bytes
Database Buffers          339738624 bytes
Redo Buffers                5132288 bytes
Database mounted.
Database opened.

SQL> select open_mode from v$database;

OPEN_MODE
--------------------
READ ONLY

SQL> select salary from hr.employees where employee_id = 100;

    SALARY
----------
     24000

SQL> update hr.employees set salary = 1000 where employee_id = 100;
update hr.employees set salary = 1000 where employee_id = 100
          *
ERROR at line 1:
ORA-16000: database open for read-only access


SQL> create table hr.emp as select * from hr.employees;
create table hr.emp as select * from hr.employees
                                        *
ERROR at line 1:
ORA-00604: error occurred at recursive SQL level 1
ORA-16000: database open for read-only access

SQL> select open_mode from v$database;

OPEN_MODE
--------------------
READ ONLY

SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> startup
ORACLE instance started.

Total System Global Area  849530880 bytes
Fixed Size                  1339824 bytes
Variable Size             503320144 bytes
Database Buffers          339738624 bytes
Redo Buffers                5132288 bytes
Database mounted.
Database opened.

SQL> select open_mode from v$database;

OPEN_MODE
--------------------
READ WRITE

SQL> update hr.employees set salary = 1000 where employee_id = 100;

1 row updated.

SQL> rollback;

Rollback complete.

SQL> create table hr.emp as select * from hr.employees;

Table created.

SQL> shutdown immediate 
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> startup mount
ORACLE instance started.

Total System Global Area  849530880 bytes
Fixed Size                  1339824 bytes
Variable Size             503320144 bytes
Database Buffers          339738624 bytes
Redo Buffers                5132288 bytes
Database mounted.

SQL> select status from v$instance;

STATUS
------------
MOUNTED

SQL> alter database open read only;
Database altered.

SQL> select status from v$instance;
STATUS
------------
OPEN

SQL> select open_mode from v$database;
OPEN_MODE
--------------------
READ ONLY

SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> startup
ORACLE instance started.

Total System Global Area  849530880 bytes
Fixed Size                  1339824 bytes
Variable Size             503320144 bytes
Database Buffers          339738624 bytes
Redo Buffers                5132288 bytes
Database mounted.
Database opened.

SQL> select status from v$instance;

STATUS
------------
OPEN

SQL> select open_mode from v$database;

OPEN_MODE
--------------------
READ WRITE

{% endhighlight %}

{% highlight css %}

--제한모드로 db시작하기
SQL> select * from session_privs 
     where privilege = 'RESTRICTED SESSION';

PRIVILEGE
----------------------------------------
RESTRICTED SESSION


SQL> select logins from v$instance;

LOGINS
----------
ALLOWED

SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.

SQL> startup restrict
ORACLE instance started.

Total System Global Area  849530880 bytes
Fixed Size                  1339824 bytes
Variable Size             503320144 bytes
Database Buffers          339738624 bytes
Redo Buffers                5132288 bytes
Database mounted.
Database opened.

SQL> select logins from v$instance;

LOGINS
----------
RESTRICTED

SQL> select account_status from dba_users 
     where username = 'HR';

ACCOUNT_STATUS
--------------------------------
OPEN

SQL> select privilege from dba_sys_privs 
     where grantee = 'HR';

PRIVILEGE
----------------------------------------
CREATE VIEW
UNLIMITED TABLESPACE
CREATE DATABASE LINK
CREATE SEQUENCE
CREATE SESSION
ALTER SESSION
CREATE SYNONYM

--다른 터미널창에서 hr로 접속
[oracle@localhost ~]$ sqlplus hr/oracle_4U
SQL*Plus: Release 11.2.0.1.0 Production on Mon Jun 6 21:02:56 2016
Copyright (c) 1982, 2009, Oracle.  All rights reserved.
ERROR:
ORA-01035: ORACLE only available to users with RESTRICTED SESSION privilege
Enter user-name: 


-- dba session 에서 수행

SQL> select logins from v$instance;

LOGINS
----------
RESTRICTED

SQL> alter system disable restricted session;

System altered.

SQL> select logins from v$instance;

LOGINS
----------
ALLOWED


-- hr session 에서 수행
[oracle@localhost ~]$ sqlplus hr/oracle_4U

SQL*Plus: Release 11.2.0.1.0 Production on Mon Jun 6 21:06:33 2016

Copyright (c) 1982, 2009, Oracle.  All rights reserved.
Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

-- dbs session 제한모드 설정

SQL> alter system enable restricted session;
System altered.
-- 시스템은 제한모드로 변경되었으나 전에 먼저 로그인한 세션은 영향을 줄 수 없음, 앞으로의 새로운 세션에 대해서만 restricted session 권한이 있는지 확인

SQL> select logins from v$instance;
LOGINS
----------
RESTRICTED

-- hr session 
SQL> select salary from employees where employee_id = 100;

    SALARY
----------
     24000

-- dba session에서 hr유저 kill
-- 필요하면 restricted session 권한은 없으나 이미 로그인 중이던 세션을 kill 
   
SQL> select sid, serial#,
     to_char(logon_time,'yyyymmdd hh24:mi:ss') 
	 from v$session 
	 where username = 'HR';

       SID    SERIAL# TO_CHAR(LOGON_TIM
---------- ---------- -----------------
        12         13 20190521 18:35:51


SQL> alter system kill session '12,13' immediate;
System altered.

SQL> select sid, serial#,
     to_char(logon_time,'yyyymmdd hh24:mi:ss') 
	 from v$session 
	 where username = 'HR';
no rows selected

-- hr session
SQL> select salary from employees where employee_id = 200;
select salary from employees where employee_id = 200
*
ERROR at line 1:
ORA-03135: connection lost contact
Process ID: 30649
Session ID: 29 Serial number: 8

SQL> conn hr/oracle_4U
ERROR:
ORA-01035: ORACLE only available to users with RESTRICTED SESSION privilege

-- dba session 제한모드 해지
SQL> alter system disable restricted session;

System altered.

SQL> select logins from v$instance;

LOGINS
----------
ALLOWED

-- hr session 접속
SQL> conn hr/oracle_4U
Connected.
SQL> select salary from employees where employee_id = 200;

    SALARY
----------
      4400
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
