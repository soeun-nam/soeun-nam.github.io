---
layout: post
title: ORACLE ADMIN 05
excerpt: "DB 생성 실습하기."
categories: [ORACLE ADMINISTRATION]
comments: true
---


### Explanation

> DB CREATION PRACTICE  
  
  1. DB NAME   : ERPDB    
  2. 파일 위치 : datafile       : /u01/app/oracle/oradata/ERPDB/   
               controlfile    : /u01/app/oracle/oradata/ERPDB/   
			   redo log file  : /u01/app/oracle/oradata/ERPDB/   
  3. 백업 위치 : FRA : /home/oracle/FRA   

## Code Snippets

{% highlight css %}

export ORACLE_SID=ERPDB
mkdir -p /u01/app/oracle/oradata/ERPDB		--ERPDB설치를 위한 디렉토리 경로설정(하위디렉토리전부생성)

cd $ORACLE_HOME/dbs
orapwd file=orapwERPDB password=oracle_4U	--dbs경로에 패스워드파일생성

vi initERPDB.ora
db_name=ERPDB
service_names=ERPDB
control_files='/u01/app/oracle/oradata/ERPDB/control01.ctl'
sga_target=400M
pga_aggregate_target=150M
db_block_size=8192
remote_login_passwordfile='EXCLUSIVE'
undo_tablespace='UNDOTBS1'		--pfile을 생성하여 컨트롤파일생성 등 기초작업을 함


sqlplus / as sysdba
CREATE SPFILE FROM PFILE ;		--pfile을 이용하여 spfile생성
STARTUP NOMOUNT				--nomount상태로 시작

CREATE DATABASE ERPDB
USER SYS IDENTIFIED BY oracle_4U
USER SYSTEM IDENTIFIED BY oracle_4U					--sys와 system의 패스워드설정
LOGFILE GROUP 1 ('/u01/app/oracle/oradata/ERPDB/redo01a.log'
                ,'/u01/app/oracle/oradata/ERPDB/redo01b.log') SIZE 100M,
        GROUP 2 ('/u01/app/oracle/oradata/ERPDB/redo02a.log'
		        ,'/u01/app/oracle/oradata/ERPDB/redo02b.log') SIZE 100M		--로그파일생성
CHARACTER SET AL32UTF8
NATIONAL CHARACTER SET AL16UTF16
EXTENT MANAGEMENT LOCAL
DATAFILE '/u01/app/oracle/oradata/ERPDB/system01.dbf' SIZE 400M AUTOEXTEND ON
SYSAUX 
DATAFILE '/u01/app/oracle/oradata/ERPDB/sysaux01.dbf' SIZE 200M AUTOEXTEND ON
DEFAULT TABLESPACE USERS
DATAFILE '/u01/app/oracle/oradata/ERPDB/users01.dbf' SIZE 50M AUTOEXTEND ON
DEFAULT TEMPORARY TABLESPACE TEMP 
TEMPFILE '/u01/app/oracle/oradata/ERPDB/temp01.dbf' SIZE 100M AUTOEXTEND ON
UNDO TABLESPACE UNDOTBS1
DATAFILE '/u01/app/oracle/oradata/ERPDB/undotbs01.dbf' SIZE 200M AUTOEXTEND ON ;	--여러 데이터파일들을 생성

@?/rdbms/admin/catalog.sql		--데이터딕셔러리 사용을 위한 설치
@?/rdbms/admin/catproc.sql		--여러 함수사용을 위한설치
connect  system/oracle_4U		--system에 연결
@?/sqlplus/admin/pupbld.sql		--sqlplus환경 사용을 위한 설치

exit

cat >> /etc/oratab <<EOF				--호출하여 oratab에 바로 입력하겠다는 의미
ERPDB:/u01/app/oracle/product/11.2.0/dbhome_1:N		--이것을 입력하면 . oraenv를 통해 ERPDB에 접속가능
EOF
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
