---
layout: post
title: ORACLE ADMIN 08
excerpt: "데이터베이스 저장영역 구조 관리."
categories: [ORACLE ADMINISTRATION]
comments: true
---


# TABLE A CONTENT

> 논리적 및 물리적 데이터베이스 구조

> 테이블에 데이터가 저장되는 방식
   
> TABLESPACE 생성
  
> TABLESPACE 변경 

> TABLESPACE 삭제  

## 논리적 및 물리적 데이터베이스 구조

<img src="./image/datas.png" width="400" height="300" >

** 논리적 구조 **

 * 테이블스페이스   
   하나의 데이터베이스를 구성하는 여러 개의 논리적 구조들, 유니버셜 인스톨러에 의해 나눠짐    
   - system테이블스페이스: 기본적으로 data dictionary테이블을 제공하는데, 이와 같은 테이블들이 저장되는 논리적 구조    
                          SQL>select * from dba_data_files;, select * from v$datafile;     
   - undotbs테이블스페이스: 사용자들이 데이터베이스에 접속한 다음 DML을 실행한 후 만약, 트랜잭션을 취소해야 한다면 ROLLBACK문 수행     
	                      undotbs 테이블 스페이스의 공간에는 사용자의 ROLLBACK 데이터가 ROLLBACK문장이 수행될 때까지 잠시 저장되어 있는 임시 공간   
		                  SCOTT@orcl> select empno, ename, sal          
					               2  from emp    
					               3  where empno = 7934;    

					              EMPNO ENAME             SAL        
					              ---------- ---------- ----------       
					              7934 MILLER           1300     

					              SCOTT@orcl> update emp set sal = sal*1.1 where empno = 7934;      
					              1 row updated.     
					              SCOTT@orcl> rollback;    
					              Rollback complete.    						  
   - temporary테이블 스페이스: 사용자들이 데이터베이스에 접속하여 실행하는 문장들은 내부적으로 분류작업 발생(order by, group by, distinct 등)     
							 그 분류 작업을 하기 위해서는 별도의 임시 공간이 필요하게 되는데 그 공간이 temporary테이블스페이스     
                             SCOTT@orcl> select empno, ename, sal from emp order by deptno;  
							  
 
 * 데이터블록   
 오라클 데이터베이스 구조에서 가장 작은 저장 구조, 익스텐트 구조가 블록구조가 여러 개 모여 만들어지는 하나의 공간   
 
 * EXTENT      
   하나의 세그먼트도 여러 개의 익스텐트가 모여서 구성, 하나의 테이블은 하나의 연속적인 공간으로 생성되는 것이 아니라 익스텐트라는 작은 구성요소로 생성되어 있는 것         
 
 * SEGMENT  
 데이터베이스 내의 생성되는 모든 객체(테이블,인덱스,뷰,시퀀스 등)    
 DATA: TABLE (dba_tables), index (dba_indexes), undo( dba_rollback_segs), temporary( v$temp_stat)    
 
** 물리적 구조 **

 * 데이터 파일: 테이블 스페이스의 데이터 파일은 지원되는 모든 저장 기술을 사용하여 물리적으로 저장     
	          저장 영역 시스템(san, nfs, nas, asm, exadata, raw)     
  - SYSTEM, SYSAUX : 필수 테이블스페이스     
    SYSTEM : Data Dictionary Table 저장       
	         항상 Online + Read Write    
    SYSAUX : 기타 오라클이 사용하는 구성요소들 저장    
  - segment type : table, index, undo, temporary, ...     

## 테이블에 데이터가 저장되는 방식

 * 행 구조 
	row header    
	1 col header    
	1 col value   
	2 col header    
	2 col value    
	null    --> null은 col header정보가 0으로 저장    
		    --> trailing nulls 는 행의 뒤쪽에 null로 연속된 공간이며, 따로 저장을 하지 않고 다음 헤더가 옴    
			--> 선택적 입력 컬럼들을 테이블 설계시 뒤쪽에 배치하는 것을 고려    
 * pctfree     
	해당 블록에 저장된 행에 늘어나는 updatae를 위해서 예비한 공간의 pct   
	10에서 30으로 늘렸을 경우 row migration가능성이 줄어드는 장점이 있지만, 더 많은 저장공간이 필요하며 성능이 저하된다는 단점     
	
 * 테이블(segment)가 공간을 할당 받는 tablespace는 생성 user의 default tablespace 또는 지정된 tablespace !  
 
## 테이블스페이스 생성  

 * 생성 명령  
	CREATE TABLESPACE new_tw   
	DATAFILE '+DATA' SIZE 100M    
	EXTENT MANAGEMENT LOCAL   
	AUTOALLOCATE    
	SEGMENT SPACE MANAGEMENT AUTO   
	NOLOGGING    
	BLOCKSIZE 16K;   
	
 * smallfile tablespace(default)  
  데이터 화일을 여러개 가질 수 있음 (400만개 블럭까지 가능)  
 
 * bigfile tablespace  
  테이블 스페이스의 데이터파일을 1개만 설정    
  tera byte 단위이상으로 지정할 경우 40억개 블럭까지 가능   
  
 * extent management local(default)  
   익스텐트 할당과 간련된 정보를 테이블스페이스의 데이타파일의 헤더에 비트맵으로 기록하고 사용 -> 데이터 딕셔너리에 대한 경합이 줄고, 헤더에 기록되어 있어서 local에서 처리하므로 처리속도 빠름   
   extent management dictionary : users01.dbf라는 테이블스페이스에 프리한 extent 정보를 공간에 적어두고 프리한 extent공간이 할당되었을 때 system01.dbf 데이터 딕셔너리에 적힌 정보를 갱신  

 * contents   
   permanent(create tablespace~)     
   temporary         
			   (create temporary tablespace xxx tempfile ~)     
   undo (create undo tablespace xxx datefile ~)  

## 테이블스페이스의 종류  

 * permanent : permanent object를 저장하는 일반적인 테이블스페이스      
 * temporary : 임시로 data 를 저장하는데 사용하는 임시테이블스페이스, sort segment나 임시테이블을 저장    
 * undo  : undo data 를 저장하는 테이블스페이스 (commit, rollback 을 가능하게 해주는 undo data를 저장하는 테이블스페이스)    
 * status   :  online - read write    
                      - read only (select만 할수 있고, dml 불가)    
               offline 
 * Automatic Segement Space Management (ASSM)    
    1. Segement Space Management AUTO (default)      
     : insert 시 필요한 free block에 대한 정보를 BitMap Block에 저장하고 사용      
       대량의 동시 Insert가 일어날 때 좋은 성능을 보장 (여러 extent가 있을때 여러곳에 비트맵블록이 존재하여 동시 인서트가일어날때 프리스페이스를 빨리 찾을수있음)   
	   
    2. Segement Space Management MANUAL    
     : insert 시 필요한 free block에 대한 정보를 Segment header block의 freelist에 저장하고 사용    
	   여러 extent가 있을때 가장 첫번째 extent의 첫번째 블록에 freelist를 저장      
	   처음 찾은 프리스페이스에 인서트한내용이 들어갈수 없는경우 다시 프리스페이스를 찾아야함      

 * logging : 저장된 segment에서의 모든 작업에 log를 기록(default)   
              nologging : 일부 명령에 대해서는 log를 기록 X       
                          장애시 복구할 수 없음    
              CTAS(create table as select), Direct Load 의 경우 다시 데이터를 가져올수있으므로 꼭 리두를 기록할 필요x    
	          OLTP작업에 대해서는 리두를 기록해야함    

 * blocksize : standard block size (=db_block_size) 아닌 block size를 이용하여는 경우       
               2k, 4k, (8k:standard block size), 16k, 32k중 선택가능   
               먼저 같은 크기의 buffer cache가 준비되어져야 함(db_nk_cache_size)   				
			   블럭사이즈를 16k로 만들어서 사용하고싶은경우 같은크기의 버퍼캐시   
			   즉, 16k의 버퍼캐시가 필요함 (db_16k_cache_size)   

 * compress : 데이터 저장시 압축해서 저장
              스토리지를 덜 차지하나 압축작업, dml을 수행하기 위한 cpu 소비가 발생    
   compress basic : 일부 Load 작업시만 압축   
                    (CTAS, Direct Load)   
   compress for oltp : basic + dml   (11g)   
   nocompress : 압축 안함 (default)    			   


{% highlight css %}

select * from dba_tablespaces;    --테이블스페이스 정보 확인
create table hr.t1(a number, b varchar2(2000));     --테이블 생성
select * from dba_tables where table_name = 't1';
select * from dba_users where username = 'HR';
select * from dba_segments where segment_name='T1';

show parameter deferred                          --변경된 값이 다음 session부터 바뀌게 된 것을 보여줌
NAME                                               TYPE        VALUE                                                                                                
-------------------------------------------------- ----------- --------------
deferred_segment_creation                          boolean     TRUE 

select * from dba_extents where segment_name='T1'; 
insert into hr.t1 values (1,'abc');
select owner, segment_name, extents, blocks, bytes/1024 kb from dba_segments where segment_name = 'T1'; 
select owner, segment_name, extent_id, bytes/1024 kb from dba_extents where segment_name='T1'; 
insert into hr.t1 select object_id, object_name from all_objects where rownum <=100; 
insert into hr.t1 select object_id, object_name from all_objects;
select owner, segment_name, extents, blocks, bytes/1024 kb from dba_segments where segment_name = 'T1';  
select owner, segment_name, extent_id, bytes/1024 kb from dba_extents where segment_name='T1'; 

CREATE TEMPORARY TABLESPACE temp1
TEMPFILE '+DATA' SIZE 10M;

CREATE UNDO TABLESPACE undo1
datafile '+DATA' size 20m;

select tablespace_name, extent_management, allocation_type, 
segment_space_management, bigfile, contents, block_size from dba_tablespaces; --테이블스페이스 정보확인

--테이블 스페이스 공간관리
--테이블 스페이스 용량이 10m인데 용량이 가득 차면 용량을 늘려 20을 되게 하는게 아니라 그냥 두고 새로운 스페이스 이름을 가진 데이터 파일을 생성하기 위한 코드
create tablespace test_tbs2 datafile '/home/oracle/test_tbs2_1.dbf' size 10m;   
create temporary tablespace temp1 tempfile '+DATA' size 10m;
create undo tablespace undo1 datafile '+DATA' size 20m;

select * from hr.t1 where rownum<=3;
delete hr.t1 where rownum <= 100;
commit;

alter tablespace users read only; 
select * from hr.t1 where rownum <=3;   --됨 왜냐면 read는 가능
delete hr.t1 where rownum<=100;         --안됨 왜냐면 write 막아놓음

alter tablespace users read write;  -- read only 도 online이라 online이라고 바꿔주면 변경안됨 
									-- 읽고 쓰기 가능하게 하려면 read write로 써야함
alter tablespace users offline;  
/*
offline상태에서 검색은안됨, online으로 바꿔야하고,
offline상태에서 테이블 drop이 되는 이유는 ddl이기 때문에 데이터 블락에 가지 않음
데이터 딕셔너리 작업이기 때문에 데이터는 남아있지만 oracle과의 접속만 끊어진 상태(되살리기 가능)
offline 시점에 체크포인트 발생(offline normal), 검색해서 체크포인트 보면 제일 첫번째에 있음
백업 하기 위해서 offline(offline immediate)하는 것은 체크포인트 발생 x ->recovery하기 위해서
(offline temporary) option은 일부만 체크포인트
*/
select * from hr.t1 where rownum <=3;
delete hr.t1 where rownum<=100;  
drop table hr.t1;   

alter tablespace users online;

select * from hr.t1;   --삭제해서 없지만 데이터는 데이터 딕셔너리에 남아있는 상태 
					   --그래서 휴지통에서 되살릴 수 있는 것 처럼 되살리기 가능 (free가 overwrite 되기 전까지는)
select * from scott_emp;   --그래서 scott.emp 조회가능

create tablespace test_tbs3 blocksize 16k
datafile '/home/oracle/test_tbs3_01.dbf' size 10m;

alter system set db_16k_cache_size = 1m;

drop tablespace test_tbs2 including contents;  --테이블스페이스의 모든 세그먼트삭제 
drop tablespace test_tbs3 including contents and datafiles;   --물리적파일까지 전부 삭제
		   
select * from dba_data_files; 
/home/oracle/a02.dbf	10	A_TBS	17825792	2176	AVAILABLE	10ONLINE 
create tablespace a_tbs datafile '/home/oracle/test_tbs3_01.dbf' size 5m;  --새로운 테이블스페이스를 만들고 데이터파일의 경로와 사이즈 설정
insert into hr.t1 select * from hr.t1;   --error becouse 5라는 용량을 설정했지만 데이터를 더 추가하면 용량이 넘어서 에러

alter tablespace a_tbs add datafile '/home/oracle/a02.dbf' size 5m;   --사이즈 5 더 추가
insert into insert into hr.t1 select * from hr.t1; -- error 안되고 데이터 들어감
alter database datafile 10 resize 10m;     --dba_data_files에서 검색했을때 경로가 아닌 번호로 찍어서 resize로 사이즈 변경 가능

select count(*) from hr.t1;    --140000데이터가 있음
insert into hr.t1 select * from hr.t1;    
select * from hr.t1 where rownum <=70000;   --70000개만 데이터 추가
select count(*) from hr.t1;       --210000개 데이터 있음
commit;

alter database datafile 10 autoextend on;   --데이터 파일 용량 초과시 자동 증가를 설정해줌(MAXSIZE 1000)
--OMF란??? 파일이름이 아닌 데이터베이스 객체 관점에서 파일 작업 지정(DB_CREATE_FILE_DEST, +DATA)
show parameter db_create_file_dest      --OMF 확인
alter system set db_create_file_dest = '/home/oracle/test/';   -- 경로 변경
create tablespace b_tbs;                           --OMF O
create tablespace c_tbs datafile size 10m;         --OMF O
create tablespace d_tbs datafile '/home/oracle/d01.dbf' size 5m;   --OMF X
/*
OMF란>> ORACLE 관리 파일이며 오라클에서 경로, 데이터파일이름, 사이즈를 지정해주면 OMF
                            사용자가 경로, 데이터파일이름, 사이즈를 지정해주면 OMF가 아님
*/
drop tablespace b_tbs;             --b_tbs, c_tbs는 OMF이며, including contents and datafiles를 안해줘두 drop success
drop tablespace c_tbs;
drop tablespace d_tbs including contents and datafiles;   --OMF이기 때문에 including contents and datafiles해주어야 drop success
{% endhighlight %}

## 테이블스페이스 변경  
 
> ALTER TABLESPACE ....

 * 이름 변경  
 ALTER TABLESPACE NEW_TS RENAME TO TS_NEW;   
 
 * STATUS 변경   
 ALTER TABLESPACE TS_NEW READ ONLY;    
 ALTER TABLESPACE TS_NEW OFFLINE;   
(OFFLINE 수행시 CHECKPOINT가 수행됨, 수행되지 않게 하려면 IMMEDIATE를 지정)   

* SIZE 변경  
 DATAFILE을 추가 -> ALTER TABLESPACE TS_NEW ADD DATABASE DATAFIE '/home/oracle/a02dbf' size 5m;   
 DATAFILE의 RESIZE  -> ALTER DATABASE DATAFILE '/home/oracle/a01.dbf' resize 10m;   
 AUTOEXTEND ON  
 (주의해야할 점 :  resize 10m => 100m   
	            100m => 10m   	
				둘다 가능하지만 100m에서 10m로줄이는경우는 데이터가 채워졌다가 삭제되었더라도 10m보다 많이 쓰였으면 high watermark가 데이터에도 있어서 줄이기 불가능)   
  
* LOGGING 변경  
 ALTER TABLESPACE TS_NEW NOLOGGING;   
 
* TABLESPACE USAGE THRESHOLD  
 테이블스페이스 사용 % 가 85%, 97% 이면 경고 메시지를 내보냄   
 
## 테이블스페이스 삭제  

> DROP TABLESPACE ....  
 
  * DROP TABLESPACE NEW_TS;  
  
  * DROP TABLESPACE NEW_TS INCLUDING CONTENTS; : 저장된 SEGMENT 먼저 삭제 후 DROP   
   
  * DROP TABLESPACE NEW_TS INCLUDING CONTENTS AND DATAFILES; : 관련된 DATAFILE도 같이 삭제   
  
  * DROP TABLESPACE NEW_TS INCLUDING CONTENTS AND DATAFILES CASCADE CONSTRAINTS; : SEGMENT 삭제 시 FK 제약조건을 삭제하고 삭제   
  
## Buttons

Make any link standout more when applying the `.btn` class.

{% highlight html %}
<a href="#" class="btn btn-success">Success Button</a>
{% endhighlight %}

<div markdown="0"><a href="#" class="btn btn-info">Info Button</a></div>

## Notices

**Watch out!** You can also add notices by appending `{: .notice}` to a paragraph.
{: .notice}
