---
layout: post
title: ORACLE ADMIN 04
excerpt: "There are three major structures in Oracle Database server architecture: memory structures, process structures, and storage structures."
categories: [ORACLE ADMINISTRATION]
comments: true
---


# SQL문장이 실행되는 과정
* user process : sql을 작성하는 프로그램(SQL*PLUS, SQL Developer, Toad ···)  
* 권한 검사(shared pool): 어떤사용자가 해당 오브젝트에 접근 할 수 있는 권한여부를 확인하는 과정  
* Library Cache : 한번이라도 실행된 SQL 또는 PL/SQL 문장과 해당 문장의 실행계획이 공유되어 있는 공간    

> 1. User Process가 서버 위치 정보가 들어있는 tnsnames.ora를 참고해 해당 서버로 찾아감  

> 2. 서버 쪽 listener.ora 파일 안에 접속하려는 서버 IP가 있으면 리스너는 해당 Server Process를 불러옴  
 
> 3. User Process는 Server Process에게 수행되어야 할 쿼리 전달  

> 4. Server Prcoess는 Parse -> Bind -> Execute -> Fetch 과정을 통해 쿼리 수행 (select문 실행 과정)  

> 5. 작업이 끝나면 Server Process는 결과값은 User Process에게 전달하고, User Process는 사용자에게 전달  

### SQL문장이 실행되는 과정에서 주의!

* 오라클과 리스너는 서로 다른 프로그램이기 때문에 server는 켜있는데 listener가 죽어있는 경우도 있음 --> 신규 접속 불가능    
* listener가 켜져있는지 확인하려면 # tnsping SID 명령어를 사용  
* 리스너는 새로 들어오는 client만 관리하는데 그 이유는 이미 들어와 있는 사용자는 리스너를 통하지 않고 바로 server process와 연결되기 때문  
* 즉 리스너가 장애가 났을 경우, 전에 미리 접속해 있던 사람들은 계속 서버 이용이 가능하지만 새로 접속해야 하는 사용자들은 접속할 수 없음  
* 리스너 로그 파일(listener.log)이 2G가 넘어가게 되면 리스너가 죽어버리기 때문에 항상 관리해줘야 함  
* 경로는 listener.ora파일에 있음, 용량이 거의 차버렸다고 리스너 로그 파일을 그냥 지워버리면 안됨!  

# SELECT문 실행 과정  
 **Parse --> Bind -->  Execute --> Fetch      

1.PARSE      
  검사를 하기 위해서는 데이터 딕셔너리를 사용하게 되고 데이터 딕셔너리를 캐싱해 두고 성능을 높여주는 곳이 shared pool 안의 dictionary cache 또는 row cache    
  shared pool의 Library cache를 검사한 후 공유되어 있는 실행계획이 있는지 검사 있으면 바로 Execution 단계로 진행하면 soft parse(커서 공유), 없으면 Optimizer를 찾아가서 실행 계획을 만들어 달라고 요청하는 것이 Hard Parse  
  
  *Soft Parse    
  Library Cache안에 있는 커서 즉, 공유커서는 이미 한번 수행되었던 SQL 문장의 실행계획과 관련 정보를 보관 하고 있다가 재활용 해줌  
  Optimizer 옵티마이저가 실행계획을 만들어주는 부담을 줄이게 됨으로써 SQL 수행 속도를 빠르게 함  
  
  *Hard Parse  
  Optimizer 는 실행 계획을 생성시켜주는 역할  
  옵티마이저가 실행 계획을 세워주는 대로 실행을 하기 때문에 옵티머아저의 실행 계획이 SQL 수행 속도에 절대적인 영향  
  Staitc Data Dictionary는 자동으로 업데이트 되지 않기 때문에 직접 관리 해줘야 함    
  
2.BIND  
  실행 계획을 1개만 생성 한 후 바인드 변수 값을 바꾸어 여러 번 실행하는 것  
  
3.EXECUTE  
  데이터파일에서 데이터가 들어있는 블록을 찾아 Database Buffer Cache로 복사해오는 과정  
    
  서버 프로세스가 해당 데이터를 가져오기 위해 해당 데이터가 저장되어 있는 블록을 찾게 됨
  찾는 모든 데이터는 Database Buffer Cache에 있어야 하며,  서버 프로세스는 해당 블록을 찾기 위해서 우선 DB Buffer Cache를 확인하는데 DB Buffer Cache에 원하는 블록이 
  있으면 즉시 다음단계인 Fetch 단계로 진행    
  없으면 서버 프로세스가 하드 디스크로 가서 해당 블록을 찾아 DB Buffer Cache로 복사    
  디스크에서 메모리로 복사해 오는 시간이 굉장히 오래걸리는 편 --] 성능에 영향을 줌      
  인덱스 생성 해놓게 되면 디스크에 어떤 파일이 어디있는지 알기 때문에 성능 UP, 속도 UP    
  옵티마이저가 실행계획을 세울 때 인덱스가 있으면 인덱스를 보고 찾아가라고 하기 때문에 인덱스 생성은 성능과 연관됨 인덱스가 없다면 풀스캔을 해야하기 때문에 성능 DOWN  

4.FETCH  
   DB Buffer Cache에 복사된 블록 중에서 사용자가 요청한 원하는 데이터만 골라내는 과정  
   
   
  
  


 
 

## Buttons

Make any link standout more when applying the `.btn` class.

{% highlight html %}
<a href="#" class="btn btn-success">Success Button</a>
{% endhighlight %}

<div markdown="0"><a href="#" class="btn btn-info">Info Button</a></div>

## Notices

**Watch out!** You can also add notices by appending `{: .notice}` to a paragraph.
{: .notice}
