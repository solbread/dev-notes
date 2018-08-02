## Lock - InnoDB

InnoDB 스토리지 엔진은 MySQL에서 제공하는 잠금과는 별개로 스토리지 엔진 내부에서 레코드 기반의 잠금 방식을 탑재하고 있다.

InnoDB는 레코드 기반의 잠금 방식 때문에 MyISAM보다는 훨씬 뛰어난 동시성 처리를 제공할 수 있다.

잠금 정보가 상당히 작은 공간으로 관리되기 때문에 락 에스컬레이션은 없다.

* 락 에스컬레이션 : 레코드 락이 페이지 락 또는 테이블 락으로 레벨업 되는 경우





#### 잠금 정보를 진단 할 수 있는 도구

1. MySQL 5.0이하 : lock_monitor (innodb_lock_monitor라는 이름의 InnoDB 테이블을 생성하는 방법)를 이용

   ```mysql
   SHOW ENGINE INNODB STATUS
   ```

2. MySQL 5.1이상 : InnoDB 플러그인 스토리지 엔진이 도입되면서부터 InnoDB의 트랜잭션과 잠금 그리고 잠금 대기중인 트랜잭션의 목록을 조회할 수 있는 방법이 도입

   ```mysql
   -- // 아래의 테이블들을 조인해서 조회하면
   -- // 현재 어떤 트랜잭션이 어떤 잠금을 대기하고 있고
   -- // 해당 잠금을 어느 트랜잭션이 가지고 있는지를 확인할 수 있으며,
   -- // 장기간 잠금을 가지고 있는 클라이언트를 종료시키는 것도 가능

   INFORMATION_SCHEMA.INNODB_TRX
   INFORMATION_SCHEMA.INNODB_LOCKS
   INFORMATION_SCHEMA.INNODB_LOCK_WAITS
   ```

   ​

#### InnoDB의 잠금 방식

* 비관적 잠금 (Pessimistic locking)
  * 현재 트랜잭션에서 변경하고자 하는 레코드에 대해 잠금을 획득하고 변경 작업을 처리하는 방식
  * 현재 변경하고자 하는 레코드를 다른 트랜잭션에서도 변경할 수 있다 라는 비관적인 가정을 하기 때문에 먼저 잠금을 획득한 것이며, 일반적으로 높은 동시성 처리에는 비관적 잠금이 유리하다고 알려져 있으며 InnoDB는 비관적 잠금 방식을 채택
* 낙관적 잠금
  * 각 트랜잭션이 레코드를 변경할 가능성은 상당히 희박할 것이라고 (낙관적으로) 가정
  * 우선 변경 작업을 수행하고 마지막에 잠금 충돌이 있었는지를 확인해 문제가 있었다면 ROLLBACK 처리




#### InnoDB의 잠금 종류

[mysql 5.7 innodb locking](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html)

참고자료 : [토토 블로그 , InnoDB에서 사용되는 Lock의 종류](http://intomysql.blogspot.kr/2010/12/innodb-lock_2880.html)

![innoDB lock type](picture\innoDB lock type.jpg)

* Record lock (Record only lock, 레코드 락)
* Gap lock (갭 락)
* Next key lock (넥스트 키 락)
* Auto increment lock (자동 증가 락)





#### Record lock (Record only lock, 레코드 락)

* 레코드 자체만을 잠그는 것
* 다른 상용 DBMS 레코드 락과 동일한 역할을 하지만 한가지 중요한 차이는 InnoDB 스토리지 엔진은 레코드 자체가 아니라 인덱스의 레코드를 잠금 : 만약 인덱스가 하나도 없는 테이블이라 하더라도 내부적으로 자동 생성된 클러스터 인덱스를 이용해 잠금을 설정
* InnoDB에서는 대부분 보조 인덱스를 이용한 변경 작업은 Next key lock이나 Gap lock을 사용하지만, primary key 또는 unique key index에 의한 변경 작업은 gap에 대해서는 잠그지 않고 레코드 자체에서 락





#### Gap lock (갭 락)

* 다른 DBMS와의 차이
* 레코드 그 자체가 아니라 레코드와 바로 인접한 레코드 사이의 간격만을 잠그는 것
* 목적) Phantom Read 방지 : 레코드와 레코드 사이의 간격에 새로운 레코드가 생성(INSERT)되는 것을 제어
  * Phantom Read : 한 트랜잭션 내에서 일정 범위의 레코드를 두 번 이상 읽을 때, 이전 쿼리에 없던 레코드가 생기는 현상
  * 갭 락이 걸린 간격에 새로운 레코드가 INSERT 되려고 하면 걸린 락들이 모두 해제되어야 INSERT됨
* 갭 락은 개념일 뿐 자체적으로 사용되지는 않고, 넥스트 키 락의 일부로 사용됨



#### Next key lock (넥스트 키 락)

* 레코드 락 + 갭 락
* STATEMENT 포맷의 바이너리 로그를 사용하는 MySQL 서버에서는 REPEATABLE READ 격리수준을 사용해야 함
* innodb_locks_unsafe_for_binlog 파라미터가 비활성화되면(파라미터 값이 0으로 설정되면) 변경을 위해 검색하는 레코드에는 넥스트 키 락 방식으로 잠금이 걸림
* 목적) 바이너리 로그의 정확한 기록을 통한 Master와 Slvae의 데이터 동기화
  * 바이너리 로그에 기록되는 쿼리가 슬레이브에서 실행될 때 마스터에서 만들어 낸 결과와 동일한 결과를 만들어내도록 보장하는 것
* Concurrency 저하 발생
  * 넥스트 키 락과 갭 락으로 인해 데드락이 발생하거나 다른 트랜잭션을 기다리게 만드는 일이 자주 발생



#### Auto increment lock (자동 증가 락)

[mysql 5.7 Innodb auto increment handling](https://dev.mysql.com/doc/refman/5.7/en/innodb-auto-increment-handling.html)

* 테이블에 AUTO INCERMENT 칼럼이 있을 경우 INSERT와 REPLCAE 쿼리 문장과 같이 새로운 레코드를 저장하는 쿼리에서 중복없이 순서대로 증가하는 일련번호 값을 가지게 하기 위해서 발생하는 잠금

* 레코드 락이나 넥스트 키 락과 달리 트랜잭션과 관계 없이 INSERT나 REPLCAE 쿼리에서 AUTO_INCREMENT 값을 가져오는 순간만 AUTO_INCREMENT 라깅 걸렸다가 즉시 해제

* AUTO_INCREMENT 락은 테이블에서 단 하나만 존재하기 때문에 두 개의 INSERT 쿼리가 동시에 실행되는 경우 하나의 쿼리가 AUTO_INCREMENT 락을 걸게 되면 나머지 쿼리는 AUTO_INCREMENT 락을 기다려야 하는데, 아주 짧은 시간동안만 걸렸다가 해제되는 잠금이기 때문에 대부분의 경우 문제되지 않음

* "자동 증가 값은 한 번 증가함녀 절대 줄어들지 않음" 

  * AUTO INCREMENT LOCK을 최소화하기 위하여 INSERT 쿼리가 실패했떠라도 한번 증가된 AUTO INCREMENT 값은 다시 줄어들지 않고 그대로 남음

* MySQL 5.1 이상부터는 "innodb_autoinc)lock_mode"를 이용하여 INCREMENT LOCK의 작동 방식 변경 가능

  * innodb_autoinc_lock_mode = 0

    * MySQL 5.0이하 버전과 동일하게 작동
    * AUTO_INCREMENT 락을 명시적으로 획득하고 해제하는 방법이 없으며, 모든 INSERT와 REPLACE 쿼리에 대해 AUTO INCREMENT LOCK을 사용
    * AUTO_INCREMENT 칼럼에 명시적으로 값을 주더라도 락이 발생 

  * innodb_autoinc_lock_mode = 1

    * MySQL 서버가 INSERT 되는 레코드의 건수를 정확히 예측할 수 있을 떄는 AUTO INCREMENT LOCK을 사용하지 않고, 빠른 latch(mutex)를 이용해 처리 -> 아주 짧은 시간만 잠금을 걸고 필요한 AUTO INCREMENT 값을 가져오면 즉시 잠금이 해제
    * ISNERT .. SELECT와 같이 MySQL 서버가 쿼리를 실행하기 전에 INSERT 되는 레코드 건수를 예측할 수 없을 때는 AUTO INCREMENT LOCK을 사용 -> INSERT 문장이 완료되기 전까지는 AUTO INCREMENT LOCK은 해제되지 않기 때문에 다른 커넥션에서는 INSERT 실행하지 못하고 대기
    * Consecutive mode (연속 모드)
      * 하나의 INSERT 문장으로 INSERT되는 레코드는 연속된 자동 증가 값을 보장
      * ISNERT .. SELECT와 같은 대량 INSERT가 수행될 때는 InnoDB 스토리지 엔진은 한번에 여러 개의 AUTO INCREMENT 값을 할당받아서 INSERT 되는 레코드에 사용하므로, 대량 ISNERT 되는 레코드는 AUTO INCREMENT 값이 누락되지 않고 연속되게 INSERT 되지만, 한번에 할당 받은 AUTO INCREMENT 값이 남아서 사용되지 못하면 폐기하므로 대량 INSERT 문장의 실행 이후에 INSERT 되는 레코드의 자동 증가 값은 연속되지 않고 누락된 값이 발생할 수 있음

  * innodb_autoinc_lock_mode = 2

    * AUTO_INCREMENT LOCK을 사용하지 않고 경량화된 latch(mutex)를 사용
    * 인터리빙 모드(Interleaved mode)
      * 하나의 INSERT 문장으로 INSERT 되는 레코드라 하더라도 연속된 자동 증가 값을 보장하지 않음
      * INSERT ... SELECT와 같은 대량 INSERT 문장이 실행되는 중에도 다른 커넥션에서 INSERT를 수행할 수 있게 되므로 동시 처리 성능이 높아지지만, 이 설정에서 작동하는 AUTO INCREMENT 기능은 유니크한 값이 생성된다는 것만 보장하며, 복제를 사용하는 경우에는 마스터와 슬레이브의 AUTO INCREMENT 값이 달라질 가능성도 있기 때문에 주의해야 함

    > ​

> mutext, latch, lock 개념
>
> 출처 https://kldp.org/node/68437
>
> mutex나 semaphore는 시스템에서 동기화를 위해서 제공하는 자원(리소스)이고, latch와 lock은 db에서 사용하는 동기화 기법이다. latch와 lock을 mutex 또는 semaphore로 구현 할 수 있다.
>
> * mutext
>   * 상호배제를 구현하기 위한 동시성 제어 단위
>   * mutex는 단지 어떤 리소스에 대한 권한을 획득/비획득 이라는 2개의 상태로 나뉘어 짐
>   * ex) pthread_mutex_lock / unlock 함수
> * latch 
>   * 물리적인 대상(특정 버퍼 블럭 혹은 낮은 수준의 객체)의 리소스에 대해 lock에 비해 짧은 시간동안 권한(읽기/쓰기)를 획득하여 작업하고자 할 경우 사용되는 동시성 제어 단위
>   * 가능한 빠른 연산을 목표
>   * mutex와 달리 모드가 있음 : 읽기 모드 / 쓰기 모드
>   * dead lock이 발생하지 않는다고 가정하므로 dead lock 상황에 대한 해결책을 가지지 않음 (잘못된 순서로 객체에 대한 latch를 잡는 경우에는 dead lock이 발생할 수 있지만, 이는 버그일 뿐 latch의 본래 목적과 상반됨)
>   * latch의 대상이 되는 객체는 엄격한 순서로 연산이 이루어 짐 (특히 인덱스나 레코드 페이지에 대한 순서가 어긋나면 DBMS가 정지하는 상황이 발생하고, 개발자는 혹독한 시련을 겪음)
> * lock
>   * 논리적인 대상(단순 메모리 대상 객체 뿐 아니라 테이블 혹은 레코드)에 대한 작업시 사용되는 동시성 제어 단위로서 비교적 긴 시간동안 발생하는 연산
>   * 상당히 긴 시간동안 발생할 수 있음
>   * 다양한 모드가 있음 : S / X / IS / IX / SIX / U 등
>   * dead lock이 발생하는 것을 허용하고, dead lock에 대한 내부적인 방지책( prevention과 resolution)을 가지고 있음




#### 인덱스와 잠금

InnoDB의 잠금과 인덱스는 상당히 중요한 연관 관계가 있기 때문에 자세하게 살펴 볼 필요가 있다.

InnoDB의 잠금은 레코들르 잠그는 것이 아니라 인덱스를 잠그는 방식으로 처리하므로, 변경해야 할 레코드를 찾기 위해 검색한 인덱스의 레코드를 모두 잠가야 한다. 



#### 트랜잭션 격리 수준과 잠금

갭 락과 넥스트 키 락에 의해 불필요한 레코드의 잠금이 발생하는데, InnoDB에서 넥스트 키 락을 필요하게 만드는 주 원인은 복제를 위한 바이너리 로그 때문이다. 

- 넥스트 키 락과 갭 락을 줄이는 방법

  | 버전           | 설정의 조합                                   |
  | ------------ | ---------------------------------------- |
  | MySQL 5.0    | innodb_locks_unsafe_for_binlog = 1<br /> 트랜잭션 격리 수준을 READ-COMMITED로 설정 |
  | MySQL 5.1 이상 | 바이너리 로그를 비활성화 <br /> 트랜잭션 격리 수준을 READ-COMMITTED로 설정 |
  | MySQL 5.1 이상 | 레코드 기반의 바이너리 로그 사용<br />innodb_locks_unsafe_for_binlog = 1<br />트랜잭션 격리 수준을 READ-COMMITTED로 설정 |

  - 갭 락이나 넥스트 키 락을 줄일 수 있다는 것은, 사용자의 쿼리 요청을 동시에 더 많이 처리하는 것을 의미
  - 대부분의 갭 락과 넥스트락이 사라지지만 유니크키나 외래키에 대한 갭 락은 사라지지 않음
  - ROW 포맷의 바이너리 로그(레코드 기반의 바이너리 로그, Row based binary log)를 사용하거나 바이너리 로그를 사용하지 않는 경우에는 갭 락과 넥스트 키 락의 사용을 대폭 줄일 수 있음
    - MySQL 5.0 버전에서는 바이너리 로그가 비활성화 되지 않아도 트랜잭션의 격리 수준을 READ-COMMITED로 설정하는 것이 가능했기 때문에 바이너리 로그의 사용 여부와 관계 없이 innodb_locks_unsafe_for_binlog 시스템 설정값을 1로 설정하고 트랜잭션 격리 수준을 READ-COMMITTED로 설정해 대부분의 갭 락이나 넥스트 키 락을 제거할 수 있음
    - MySQL 5.1 이상의 버전에서는 문장 기반의 바이너리 로그의 경우에, 바이너리 로그가 활성화 되면 최소 REPEATABLE-READ 이상의 격리수준을 사용하도록 강제되고 있음
    - 하지만 ROW 포맷의 바이너리 로그(레코드 기반의 바이너리 로그, Row based binary log)는 그다지 널리 사용되지 않기 때문에 안정성을 확인하는 것이 어려운 상태이며, 또한 STATEMENT 포맷의 바이너리 로그에 비해 로그 파일의 크기가 상당히 커질 가능성이 많음



#### 갭 락과 넥스트 키 락 관련 예시

```mysql
-- // animals 테이블에는 300만개의 레코드가 있다.

-- // animals 테이블에서 type='cat'인 동물은 전체 300마리가 있으며,
-- // name='bori'인 고양이는 딱 1마리만 있다.
SELECT COUNT(*) FROM animals WHERE `type` = 'cat';
+-----------+
+		300	+
+-----------+

SELECT COUNT(*) FROM animals WHERE `name` = 'bori'
+-----------+
+		1	+
+-----------+

-- // animals 테이브렝서 type='cat'이고, name='bori'인 고양이의
-- // 색을 회색으로 변경하는 쿼리를 실행
UPDATE animals SET `color` = 'gray' WHERE `type` = 'cat' AND `name` = 'bori';

```

- 인덱스가 하나도 없다면?
  - 테이블을 풀 스캔하면서 UPDATE를 진행하는데, 300만개의 모든 레코드가 lock
- type 칼럼만 멤버로 담긴 idx_type이라는 인덱스가 있다면?
  - 넥스트 키 락과 갭 락을 사용하는 경우
    - 'name' 칼럼에 대한 인덱스는 없기 때문에 type = 'cat'인 300건의 레코드만 lock
  - 넥스트 키 락과 갭 락을 사용하지 않는 경우
    - 인덱스만으로 일치 여부를 판단하는 1차 비교 단계에서 type = 'cat'인 300건의 레코드를 모두 잠그는데, 인덱스를 사용하지 않는 나머지 조건의 일치 여부를 판단하는 2차 비교에서 실제 업데이트 대상이 아니라는 것을 알게됨과 동시에 1차 비교에서 걸었던 잠금을 해제하므로, type = 'cat' AND name = 'bori'인 레코드에 대해서만 배타적 잠금을 가지게 되므로 불필요한 잠금이 생기는 현상이 줄어듦



#### 레코드 수준의 잠금 확인 및 해제

InnoDB 스토리지 엔진을 사용하는 테이블의 레코드 수준 잠금은 테이블 수준의 잠금보다는 조금 더 복잡하다. 테이블 잠금에서는 잠금의 대상이 테이블 자체이므로 쉽게 문제의 원인이 발견되고 해결될 수 있지만, 레코드 수준의 잠금은 테이블의 레코드 각각에 잠금이 걸리므로 그 레코드가 자주 사용되지 않는다면 오랜 시간 잠겨진 상태로 남아있어도 잘 발견되지 않는다.

##### MySQL 5.0 이하의 잠금 확인 및 해제

레코드 잠금에 대한 메타 정보(Dictionary table)을 제공하지 않음

* 방법1

  ```mysql
  SHOW PROCESSLIST;
  ```

  해당 명령 결과 중 State를 보면 Lock에 의해 대기하고 있는 프로세스이지만 "Updating"으로 표시되어 잠금을 기다리는 것인지, 실제 쿼리를 실행하고 있는 것인지 알 수 없음 -> State가 'Updating" 이면서 Time 칼럼의 값인 쿼리 실행시간을 보고 잠금상태를 대략적으로 파악해야 함

* 방법2

  ```mysql
  SHOW ENGINE INNODB STATUS

  ....
  --TRANSACTION 0 1770, not started, OS thread id 5472
  ...
  --TRANSACTION 0 1780, ACTIVE 1 sec, OS thread id 5148 starting inddex read
  ...
  --TRANSACTION 0 1790, ACTIVE 32 sec, OS thread id 2116
  ```

  트랜잭션의 상태가 ACTIVE 이면서 트랜잭션이 오랜시간동안 실행되고 있는 줄을 찾으면 레코들르 오랫동안 잠그고 있는 프로세스를 찾을 수 있음. 하지만 어떤 잠금을 가지고 있는지의 정보는 얻을 수 없기 떄문에 쿼리로 필요한 잠금을 예측해야 함

  만약 근본적인 원인에 해당하는 트랜잭션을 찾기가 어렵다면 오래 기다리고 있는 트랜잭션의 커넥션을 모두 종료해버리는 것이 가장 빠른 해결책

  * TRANSACTION 뒤의 숫자 2개는 트랜잭션 번호를 의미 (64비트 숫자를 상위 4바이트와 하위 4바이트로 구분하여 표기)
  * not started / ACTIVE : 트랜잭션 상태
  * MySQL thread id <숫자> : 숫자 값은 이 트랜잭션을 실행하고 있는 MySQL 프로세스 아이디(커넥션 번호)



##### MySQL 5.1 이상(더 정확하게는 InnoDB 플러그인 버전부터)의 잠금 확인 및 해제

레코드 잠금과 잠금 대기에 대한 조회가 가능하므로 쿼리 하나만 실행해보면 잠금과 잠금 대기를 확인할 수 있음

* 방법

  ```mysql
  SELECT * FROM information_schema.innodb_locks;
  SELECT * FROM information_schema.innodb_trx;
  SELECT * FROM information_schema.innodb_lock_waits;
  ```

  INFORMATION_SCHEMA라는 DB의 innodb_trx, innodb_locks, innodb_lock_waits 테이블들 존재

  잠금이나 대기가 발생할 경우 InnoDB 스토리지 엔진에서 관련 정보를 해당 테이브들로 업데이트 하기 때문에 해당 테이블들을 조회하여, 각 트랜잭션이 어떤 잠금을 기다리고 있는지, 기다리고 있는 잠금은 어떤 트랜잭션이 가지고 있는지를 쉽게 메타 정보를 얻을 수 있음

  * innodb_locks : 어떤 잠금이 존재하는지를 관리
  * innodb_trx : 어떤 트랜잭션이 어떤 클라이언트(프로세스)에 의해 기동되고 있으며, 어떤 잠금을 기다리고 있는지를 관리
  * innodb_lock_waits : 잠금에 의한 프로세스 간의 의존 관계를 관리

  ```mysql
  SELECT 
  r.trx_id waiting_trx_id,
  r.trx_mysql_thread_id waiting_thread,
  r.trx_query waiting_query,
  b.trx_id blocking_trx_id,
  b.trx_mysql_thread_id blocking_thread,
  b.trx_query blocking_query 
  FROM information_schema.innodb_lock_waits w 
  	INNER JOIN information_schema.innodb_trx b ON b.trx_id = w.blocking_trx_id 
  	INNER JOIN informtaion_schema.innodb_trx r ON r.trx_id = w.requesting_trx_id;
  	
  **************** 1. row ****************
  waiting_trx_id : 34A7
  waiting_thread : 100
  waiting_query : UPDATE animals SET name = 'rion', birth_date = NOW() WHERE id = 100
  blocking_trx_id : 34A6
  blocking_thread : 99
  blocking_query : UPDATE animals Set NAME = 'rion' WHERE id = 100

  **************** 2. row ****************
  waiting_trx_id : 34A7
  waiting_thread : 100
  waiting_query : UPDATE animals SET name = 'rion', birth_date = NOW() WHERE id = 100
  blocking_trx_id : 34A5
  blocking_thread : 18
  blocking_query : NULL

  **************** 3. row ****************
  waiting_trx_id : 34A6
  waiting_thread : 99
  waiting_query : UPDATE animals Set NAME = 'rion' WHERE id = 100
  blocking_trx_id : 34A5
  blocking_thread : 18
  blocking_query : NULL	
  ```


  위의 예시로 상황을 판단하건데, 34A5 -> 34A6 -> 34A7 순서로 실행된 트랜잭션이 잠가진 34A5 트랜잭션에 의해 나머지 두개의 34A6, 34A7 트랜잭션이 기다리고 있음. 34A6은 34A5만 기다리지만, 34A7은 34A5와 34A6 모두 기다리고 있음

  * "waiting..." 칼럼은 잠금을 기다리는 트랜잭션이나 프로세스를 의미

  * "blocking..." 칼럼은 잠금을 해제하지 않아서 다른 트랜잭션을 막고(기다리게 하고) 있는 트랜잭션이나 프로세스를 의미

  * 예시 설명

    1. 첫 번째 레코드는 트랜잭션 34A7번(커넥션 100번)이 트랜잭션 34A6번(커넥션 99번)을 기다림
    2. 두 번째 레코드는 트랜잭션 34A7번(커넥션 100번)이 트랜잭션 34A5번(커넥션 18번)을 기다림
    3. 세 번째 레코드는 트랜잭션 34A6번(커넥션 99번)ㅣ 트랜잭션 34A5번(커넥션 18번)을 기다림

    * 위의 정보로 알 수 있는 것은 34a5번 트랜잭션이 다른 트랜잭션의 진행을 막고있으므로, Kill 명령어를 이용해 18번 커넥션을 종료하면 됨
    * 가장 중요한 34A5번(커넥션 18번)에 대한 정보가 표시되지 않는데, 이 정보는 innodb_trx 테이블에서 trx_id='34A5'로 조회하면 알 수 있음  







#### 참고자료

개발자와 DBA를 위한 Real MySQL, 저 이성욱