---
layout: post
title: ORACLE ADMIN 10
excerpt: "데이터 동시성 관리"
categories: [ORACLE ADMINISTRATION]
comments: true
---


# ORACLE LOCK MECHANISM
  
  **여러 세션이 동시에 같은 데이터를 변경하지 못하도록 함, 오라클은 가장 최소 레벨의 LOCK을 자동으로 사용**  

 * DML - ROW LEVEL LOCK MODE (TX) : X(Exclusive)  
 * DML - TABLE LEVEL LOCK MODE (TM)  
       |	RS	Rx	S	SRx	X               
      ------------------------------------------------    
	  RS   |	O	O	O	O	X          O X 공유여부표시    
	  Rx   |	O	O	X	X	X    
	  S    |	O	X	O	X	X    
	  SRx  |	O	X	X	X	X   
	  X    |	X	X	X	X	X    
	  
   -RS(Row Share)     : SELECT FOR UPDATE    					
   -Rx(Row Exclusive) : DML    
   -S(Share)          : FK    
   -SRx(Share Row Exclusive) : FK on delete cascade    
   
   --테이블에 걸리는 락 
   RS -- SELECT FOR UPDATE 작업때 트랜잭션 진행중인 작업을 기다리게되므로 테이블에 RS lock 이 걸림   
   Rx -- DML작업이 있을때 행레벨에 x락을 걸게되는데 테이블에는 x락이 걸린 행이 있다라는걸 표시하기위해 테이블에 Rx lock 이 걸림   
   S  -- dept테이블의 40번부서에는 사원이 존재하지 않음. 따라서 그행을 지우게되어도 문제가되지않지만 emp테이블과의 참조관계에 의해 emp테이블에 S lock 이 걸림   
   SRX -- S lock의 경우와 다르게 10번부서를 삭제하는데 on delete dascade를 하게되면 emp에 SRx lock 이 걸림  
   
 * DDL - TABLE LEVEL LOCK MODE  : X(DROP, ALTER) , S (CREATE PROCEDURE, AUDIT)   

# LOCK CONFLICT   

  * 다른 세션에서 걸어놓은 LOCK 때문에 WAITING발생   
  * 발견  
    EM Performance Page의 blocking sessions    
    ADDM Report   
    v$session, v$lock   
    $ORACLE_HOME/rdbms/admin/utllockt.sql   
	
  * 해결  
    해당 트랜잭션을 종료  
    세션을 KILL - 진행중인 트랜잭션은 ROLLBACK됨  
  
  * DEAD LOCK   
    deadlock 은 서로가 서로를 기다리는 교착상태   
    ora-60 deadlock detected (alert_orcl.log)  
    일단 모두 허용, dead lock ring 발견시 발생 명령을 rollback(먼저 lock건)    
    deadlock 이 발생하지 않도록 하려면 일괄 작업(batch) 을 수행할 때 작업하는 행을 같은 순서로 처리   

{% highlight css %}

-- 데드락 예제 작업  
SCOTT> update emp
  2  ser sal=2000
  3  where empno=7788;

1 row updated.

ALLEN> update scott.emp
  2  set sal=1950
  3  where empno=7900;

1 row updated.

SCOTT> update emp
  2  set sal=100
  3  where empno=7900;

	wating...

ALLEN> update scott.emp
  2  set sal=3000
  3  where empno=7788;

	wating...

SCOTT> 
update emp
       *
ERROR at line 1:
ORA-00060: deadlock detected while waiting for resource		--서로 기다리는 교착상태가 걸리게됨

SCOTT> select * from v$diag_info;		--조회하여나오는 트레이스파일 경로
							/u01/app/oracle/diag/rdbms/orcl/orcl/trace/orcl_ora_14162.trc

orcl:trace]$ tail -5 orcl_ora_14162.trc
END OF PROCESS STATE

*** 2019-12-17 15:15:26.643
Attempting to break deadlock by signaling ORA-00060	--데드락에대한 트레이스가 기록된 것을 볼수 있음

--session sql 
  1  select sid,serial#, username, machine, osuser
  2  from v$session
  3* where username is not null
  
--lock_info.sql 
select s.sid,serial#, username,
    decode(l.type,'TM','TM(table)','TX(row)') type, id1,id2,
    decode(lmode,0,'NONE',1,'NULL',2,'RS',3,'RX',4,'S',5,'SRX',6,'X') lmode,
    decode(request,0,'NONE',1,'NULL',2,'RS',3,'RX',4,'S',5,'SRX',6,'X') request
from v$lock l, v$session s
where l.sid= s.sid and l.type in ('TM','TX')
order by s.sid, l.type
/

--blocking_session.sql 
SYS@orcl> get blocking_session
  1  select sid, serial#,username
  2  from v$session
  3* where sid in ( select blocking_session from v$session)

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
