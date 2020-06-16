---
layout: post
title: ORACLE ADMIN 04
excerpt: "SQL, SELECT 실행과정과 BUFFER CACHE의 구조에 대해 알아보기."
categories: [ORACLE ADMINISTRATION]
comments: true
---


# SQL문장이 실행되는 과정
> user process : sql을 작성하는 프로그램(SQL*PLUS, SQL Developer, Toad ···)  
> 권한 검사(shared pool): 어떤사용자가 해당 오브젝트에 접근 할 수 있는 권한여부를 확인하는 과정  
> Library Cache : 한번이라도 실행된 SQL 또는 PL/SQL 문장과 해당 문장의 실행계획이 공유되어 있는 공간    

 1. User Process가 서버 위치 정보가 들어있는 tnsnames.ora를 참고해 해당 서버로 찾아감    
 2. 서버 쪽 listener.ora 파일 안에 접속하려는 서버 IP가 있으면 리스너는 해당 Server Process를 불러옴    
 3. User Process는 Server Process에게 수행되어야 할 쿼리 전달    
 4. Server Prcoess는 Parse -> Bind -> Execute -> Fetch 과정을 통해 쿼리 수행 (select문 실행 과정)     
 5. 작업이 끝나면 Server Process는 결과값은 User Process에게 전달하고, User Process는 사용자에게 전달    

### SQL문장이 실행되는 과정에서 주의!

* 오라클과 리스너는 서로 다른 프로그램이기 때문에 server는 켜있는데 listener가 죽어있는 경우도 있음 --> 신규 접속 불가능    
* listener가 켜져있는지 확인하려면 # tnsping SID 명령어를 사용  
* 리스너는 새로 들어오는 client만 관리하는데 그 이유는 이미 들어와 있는 사용자는 리스너를 통하지 않고 바로 server process와 연결되기 때문  
* 즉 리스너가 장애가 났을 경우, 전에 미리 접속해 있던 사람들은 계속 서버 이용이 가능하지만 새로 접속해야 하는 사용자들은 접속할 수 없음  
* 리스너 로그 파일(listener.log)이 2G가 넘어가게 되면 리스너가 죽어버리기 때문에 항상 관리해줘야 함  
* 경로는 listener.ora파일에 있음, 용량이 거의 차버렸다고 리스너 로그 파일을 그냥 지워버리면 안됨!  

# SELECT문 실행 과정  
 **Parse --> Bind -->  Execute --> Fetch**      

> 1.PARSE      
  검사를 하기 위해서는 데이터 딕셔너리를 사용하게 되고 데이터 딕셔너리를 캐싱해 두고 성능을 높여주는 곳이 shared pool 안의 dictionary cache 또는 row cache    
  shared pool의 Library cache를 검사한 후 공유되어 있는 실행계획이 있는지 검사 있으면 바로 Execution 단계로 진행하면 soft parse(커서 공유), 없으면 Optimizer를 찾아가서 실행 계획을 만들어 달라고 요청하는 것이 Hard Parse  
  
  Soft Parse    
  Library Cache안에 있는 커서 즉, 공유커서는 이미 한번 수행되었던 SQL 문장의 실행계획과 관련 정보를 보관 하고 있다가 재활용 해줌  
  Optimizer 옵티마이저가 실행계획을 만들어주는 부담을 줄이게 됨으로써 SQL 수행 속도를 빠르게 함  
  
  Hard Parse  
  Optimizer 는 실행 계획을 생성시켜주는 역할  
  옵티마이저가 실행 계획을 세워주는 대로 실행을 하기 때문에 옵티머아저의 실행 계획이 SQL 수행 속도에 절대적인 영향  
  Staitc Data Dictionary는 자동으로 업데이트 되지 않기 때문에 직접 관리 해줘야 함    
  
> 2.BIND  
  실행 계획을 1개만 생성 한 후 바인드 변수 값을 바꾸어 여러 번 실행하는 것  
  
> 3.EXECUTE  
  데이터파일에서 데이터가 들어있는 블록을 찾아 Database Buffer Cache로 복사해오는 과정  
    
  서버 프로세스가 해당 데이터를 가져오기 위해 해당 데이터가 저장되어 있는 블록을 찾게 됨
  찾는 모든 데이터는 Database Buffer Cache에 있어야 하며,  서버 프로세스는 해당 블록을 찾기 위해서 우선 DB Buffer Cache를 확인하는데 DB Buffer Cache에 원하는 블록이 
  있으면 즉시 다음단계인 Fetch 단계로 진행    
  없으면 서버 프로세스가 하드 디스크로 가서 해당 블록을 찾아 DB Buffer Cache로 복사    
  디스크에서 메모리로 복사해 오는 시간이 굉장히 오래걸리는 편 --] 성능에 영향을 줌      
  인덱스 생성 해놓게 되면 디스크에 어떤 파일이 어디있는지 알기 때문에 성능 UP, 속도 UP    
  옵티마이저가 실행계획을 세울 때 인덱스가 있으면 인덱스를 보고 찾아가라고 하기 때문에 인덱스 생성은 성능과 연관됨 인덱스가 없다면 풀스캔을 해야하기 때문에 성능 DOWN  

> 4.FETCH  
   DB Buffer Cache에 복사된 블록 중에서 사용자가 요청한 원하는 데이터만 골라내는 과정  
   
# UPDATE문 실행 과정  
  
* 모든 dml문장의 수행원리는 동일, select문의 수행단계에서 fetch 과정이 없고 나머지는 동일하나 execute과정이 select문보자 더 복잡   

  excute 단계에서 필요한 데이터를 db buffer cache로 복사해오는 단계까지는 select문과 동일   
  execute단계에서 원하는 데이터가 들어있는 블록을 database buffer cache로 가져온 후 서버 프로세스가 변경되는 데이터의 변경 내역을 redo log buffer에 기록   
  redo log buffer: 데이터가 변경 될 때 만약의 장애를 복구하기 위해 변경내역을 기록해두는 공간   
  undo segment에 이전 이미지를 기록한 후 database buffer cache의 내용을 변경 => 데이터가 변경되는 것을 통틀어서 오라클에서는 트랜잭션이라고 부름   
  
## DATA BUFFER CACHE  
  *Datafile들로부터 읽은 Data Block의 복사본을 담고 있는 SGA의 한 부분
  *Oracle Instance에 동시 접속한 모든 User Process 은 Database Buffer Cache에 대한 Access를 공유 합니다.
  *Database buffer cache의 용도는 logical read, physical read

  logical read는 dbms의 메모리(buffer cache)에 적재된 데이터를 읽는 것
  physical read 는 하드디스크에서 데이터를 읽어 들이는 것   
  
## BUFFER CACHE STRUCTURE      
 *db버퍼캐시는 해시테이블 구조로 관리   
 *데이터블록주소(DBA): 데이터블록을 해싱하기 위해 사용되는 킷값    
 *해시 함수에 DBA를 입력해 리턴받은 해시 값이 같은 블록들을 해시 버킷에 연결리스트 구조로 연결    
 *연결리스트를 해시체인(cache buffer chain)이라고함    
 *버퍼 헤더만 체인에 연결되며 실제 데이터값이 필요해지면 버퍼 헤더에 있는 포인터를 이용해 다시 버퍼 블록을 찾아가는 구조    
 *오라클은 cache buffer를 관리하기 위해서는 세가지의 내부적인 structure를 사용하는데, 첫 번째 cache buffer chain, dirty list, LRU. 이 세가지 LIST를 관리하면서 사용자에게 필요한 BUFFER를 사용가능하도록 제공하여 주는 역할이 DBWR   
 *DBWR(백그라운드 프로세스)은 STARTUP시 online datafile에 대해서 media recovery lock을 획득하는 등 data file의 관리자로 간주    
 *DBWR은 Database Buffer Cache에 존재하는 Dirty blocks들을 check point가 발생하는 동시에 데이터 파일에 저장하는 작업을 수행    
 *DBWR(버퍼 캐시로부터 변경된 블록을 주기적으로 데이터파일에 기록하는 작업을 수행)에 의해서 관리    
 *LRU 알고리즘에 의하여 가장 오래 전에 사용된 것은 디스크에 저장하고 메모리에는 가장 최근에 사용된 데이터를 저장 함으로써, 디스크 I/O를 줄이고, 데이터베이스 시스템의 성능은 증가 하게 관리 함     

## LRU 알고리즘      
  오라클 LRU 접근 횟수의 조합을 사용하는 복합 알고리즘   
  MRU   -------------10-----------------  LRU   
                    ^(TOUCH COUNTER)       
 1. TOUCH COUNTER을 한번 터치할때마다 MRU로 감    
 2. 모든 버퍼 블록 헤더를 LRU체인에 연결해 사용빈도 순으로 위치를 옮겨가다가, FREE버퍼가 필요해 질때마다 액세스 빈도가 낮은 데이터 블록들을 우선하여 밀어냄으로써, 자주 액세스되는 블록들이 캐시에 더 오래 남아있도록 함    
 3. LRU리스트에는 DIRTY리스트, LRU리스트가 존재     
 4. DIRTY리스트(LRUW 리스트)는 캐시 내에는 변경되었지만, 아직 디스크에 기록되지 않은 버퍼블록을 관리     
 5. LRU리스트는 아직 DIRTY리스트로 옮겨지지 않은 나머지 버퍼 블록들을 관리    
 6. LRU 리스트의 버퍼 상태  
 
 *Free 버퍼 : 인스턴스 기동 후 아직 데이터가 읽히지 않아 비어있는 상태(Clean 버퍼)이거나 언제든지 덮어 써도 무방한 버퍼 블록      
 *Dirty 버퍼 : 버퍼에 캐시된 이후 변경이 발생했지만, 아직 디스크에 기록되지 않아 데이터 파일 블록과 동기화가 필요한 버퍼 블록       
 *Pinned 버퍼 : 읽기 또는 쓰기 작업을 위해 현재 액세스되고 있는 버퍼 블록       

## REDO LOG BUFFER   
 *오라클은 오브젝트가 변경되거나 DML 작업에 의해 데이터가 변경되는 경우 변경에 대한 로그를 버퍼에 생성   
 *해당 로그들은 서버 프로세스에 의해 리두 로그 버퍼에 기록된 후 백그라운드 프로세스인 LGWR프로세스에 의해 리두 로그 파일에 저장   
 *commit이 발생하면 변경된 데이터가 디스크로 저장된다고 생각하기 쉽지만, 오라클에서 commit했을 때, 수정된 데이터가 디스크에 기록되는건 맞지만 데이터가 저장되는 위치는 리두 로그 파일로 저장   
 *LGWR : Redo Log buffer의 내용을 Redo Log file에 기록하기 위해서는 어떤 프로세스에 의해 작업을 수행해야함. 이와 같이 Redo Log Buffer의 내용을 Redo Log File에 기록하는 프로세스가 LGWR에 해당   
 *기록 -> Redo Log Buffer는 다음과 같은 경우에 LGWR에 의해 Redo Log Bufferd의 Active한 영역을 Redo Log File에 기록      
  1)commit발생, 2)Log buffer의 공간 부족시, 3)Log Buffer의 1/3이 채워졌을시, 4)Redo Record의 크기가 1mb 이상일 경우, 5)Thread Close, 6)3초에 한번씩(Time out)    

## LARGE POOL    
  *공유 푸르 데이터 버퍼캐쉬 및 리두 로그 버퍼가 필수 SGA 영역인 반면, Large pool은 반드시 지정해야할 SGA영역이 아님. Large pool을 지정하게 되면 공유풀의 부하를 감소시키게 됨    
  
## CKPT(체크포인트 프로세스)   
  *SGA의 변경된 데이터베이스 버퍼 캐시와 리두 로그 버퍼의 내용이 데이터 파일에 저장 되도록 DBWR, LGWR를 호출하는 기능    
  *체크포인트가 발생 시 각 데이터 파일의 헤더 부분에 체크 포인트 이벤트의 현재 시점을 그리고 컨트롤 파일에는 발생된 체크포인트 이벤트의 정보를 기록하는 프로세스        
  --> 이 기록을 이용해 추후 장해 발생 시 현재 기록된 시점까지 복구가 가능     
  --> 주된 임무는 동기화!! 동기화 정보가 일치하지 않으면 데이터베이스는 STARTUP되지 않고 복구     
  --> CHECK POINT는 LGWR에 의해서 작동하며 커밋문이 실행될 때마다 오라클 서버가 관리하는 시스템 변경번호 및 데이터베이스의 상태정보를 컨트롤 파일과 데이터파일에 저장하는 작업    
  
  *발생시기   
   1)새로운 데이터 블록을 데이터 버퍼캐시로 불러들이고자 할 때 여유공간이 없는 경우      
   2)Time out(3초) 발생하는 경우 (log_checkpoint_timeout파라미터로 설정할수 있으며, 지정한 시간이 경과시 체크포인트 발생)    
   3)온라인 리두로그 파일에 기록된 데이터 블록의 수가 임의의 개수에 도달하는 경우(log_checkpoint_interval 파라미터로 설정할 수 있으며, 지정한 수만큼의 redo log block이 기록되었을 때 체크포인트 발생 )     
   4)데이터베이스를 종료하는 경우    
   5)dirty buffer 수가 어느 정도의 threshold값에 도달하는 경우    
   6)테이블 스페이스가 drop되거나 truncate되는 경우    

   *복구와 성능    
   체크포인트의 잦은 사용으로 기록을 남기면 저장은 잘되나 DISK I/O가 늘어 성능이 떨어질 수 있음 단, 상대적으로 많은 데이터 파일을 가지고 있는 경우에는 CKPT프로세스를 사용함으로써 성능 향상이 가능   
 
#오라클의 시작과 종료   
 **시작 단계 : 종료-> 노마운트->마운트->오픈**
 **종료 단계 : 오픈-> 마운트 -> 노마운트 -> 종료**  
 
 *종료(shutdown)   
 데이터베이스에 대한 엑세스를 수행할 수 없는 상태, DB를 중지한 상태   
 OPEN하기 위해서는 SYS유저로 SYSDBA, SYSOPER 권한이 있어야함   
 *노마운트(nomount)   
 인스턴스만 시작된 상태   
 오라클 DB생성, 파라미터 파일 읽기, SGA할당, 백그라운드 프로세스 기동   
 *마운트(mount)   
 인스턴스에 대한 control file을 open, open후 데이터 파일과 리두 로그파일을 인지    
 일반 user은 접속 불가능하며 sysdba권한만 접근   
 *시작(open)   
 데이터 파일과 리두로그 파일의 존재 및 정합성을 확인한 후 해당 파일들을 열어 실제 데이터베이스를 사용할 수 있는 상태   
 select status from v$instance;로 확인 가능 ----> OPEN   

## Buttons

Make any link standout more when applying the `.btn` class.

{% highlight html %}
<a href="#" class="btn btn-success">Success Button</a>
{% endhighlight %}

<div markdown="0"><a href="#" class="btn btn-info">Info Button</a></div>

## Notices

**Watch out!** You can also add notices by appending `{: .notice}` to a paragraph.
{: .notice}
