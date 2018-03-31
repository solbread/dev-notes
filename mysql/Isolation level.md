## MySQL의 Isolation level (격리 수준)

#### 트랜잭션의 Isolation levle(격리 수준)

[mysql 5.7 innodb transaction isolation levels](https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html)

동시에 여러 트랜잭션이 처리될 때, 특정 트랜잭션이 다른 트랜잭션에서 변경하거나 조회하는 데이터를 볼 수 있도록 허용할지 말지를 결정하는 것

아래의 4가지 수준으로 나뉨

1. READ UNCOMMITTED
2. READ COMMITTED
3. REPEATABLE READ
4. SERIALIZABLE

##### READ UNCOMMITTED < READ COMMITTED < REPEATABLE READ < SERIALIZABLE

격리 수준이 뒤로 갈수록 점점 높아지며,

격리수준 높아지면

* 동시성이 떨어짐
* MySQL 서버의 처리 성능은 SERIALIZABLE 격리 수준이 아니라면 크게 성능의 개선이나 저하는 발생하지 않음



####3가지 부정합의 문제

데이터베이스의 격리 수준을 이야기 할 때 항상 언급되는 3가지 부정합 문제가 존재하며, 이 문제는 격리 수준의 레벨에 따라 발생할 수도 발생하지 않을수도 있음

|                  | DIRTY READ | NON-REPEATABLE READ | PHANTOM READ              |
| ---------------- | ---------- | ------------------- | ------------------------- |
| READ UNCOMMITTED | 발생         | 발생                  | 발생                        |
| READ COMMITTED   | 발생하지 않음    | 발생                  | 발생                        |
| REPEATABLE READ  | 발생하지 않음    | 발생하지 않음             | 발생<br />(InnoDB는 발생하지 않음) |
| SERIALIZABLE     | 발생하지 않음    | 발생하지 않음             | 발생하지 않음                   |

* 온라인 서비스 용도의 데이터베이스는 READ COMMITTED와 REPEATABLE READ 둘 중 하나를 주로 사용
* 오라클은 주로 READ COMMITTED 수준 사용
* MySQL은 주로 REPEATABLE READ 수준 사용



#### READ UNCOMMITTED (DIRTY READ)

![isolation - read uncommitted(dirty read)](C:\Users\jungsol\Documents\GitHub\dev-notes\mysql\picture\isolation - read uncommitted(dirty read).jpg)

1. 트랜잭션의 변경 내용이 COMMIT이나 ROLLBACK 여부에 상관 없이 다른 트랜잭션에서 보여짐
   * Transaction A에서 x=1인 레코드에 대해 쓰기 작업을 하고, Transaction B에서 x=1인 레코드에 대해 조회를 하면 Transaction A가 커밋되지 않은 상태에서도 새로 쓰여진 x=1에 대해 조회가 됨
2. 변경하던 트랜잭션이 처리 도중 알 수 없는 문제가 발생해 트랜잭션을 ROLLBACK 한다 하더라도 중간에 조회한 트랜잭션이 알 수 없음
   * Transaction A가 중간에 알 수 없는 문제가 발생해서 Rollback 하더라도 Transaction B는 쓰기 작업을 시도했던 레코드값으로 처리를 이어나감

* Dirty Read (더티 리드) 발생

  > Dirty Read
  >
  > 트랜잭션에서 처리한 작업이 완료되지 않았는데도 다른 트랜잭션에서 볼 수 있게 되는 현상
  >
  > 데이터가 나타났다가 사라졌다 하는 현상을 초래하므로 개발자와 사용자를 상당히 혼란스럽게 만듦

* RDBMS 표준에서 트랜잭션의 격리수준으로 인정하지 않을 정도로 정합성에 문제가 많은 격리수준이므로, 일반 데이터베이스에서는 거의 사용하지 않음. (READ COMMITTED 이상의 격리수준을 사용할것으로 권장됨)



#### READ COMMITTED (NON-REPEATABLE READ)

![isolation - read committed (non-repeatable read)](C:\Users\jungsol\Documents\GitHub\dev-notes\mysql\picture\isolation - read committed (non-repeatable read).jpg)

1. MVCC 변경방식을 이용하여 트랜잭션에서 변경한 내용은 COMMIT이 완료되기 전까지는 언두(Undo) 영역에 있는 정보로 조회되며, COMMIT이 완료되어야만 다른 트랜잭션에서 조회할 수 있음

   * Transaction B에서 x=1인 레코드에 대해 쓰기 작업을 하고면 해당 내용이 테이블에 즉시반영 되고, 이전 내용은 언두(Undo) 영역으로 백업됨 -> Transaction B가 커밋을 수행하기 전에 Transaction A에서 x=1인 레코드를 읽으면 테이블이 아니라 언두 영역에 백업된 레코드에서 가져와서 기존의 레코드를 보여줌 -> Transaction B가 커밋한 후 Transaction A에서 x=1인 레코드를 읽으면 쓰기작업이 반영된 레코드를 보여줌

   > MVCC 변경방식
   >
   > 트랜잭션이 ROLLBACK될 가능성에 대비해 변경되기 전 레코드를 언두(Undo) 영역에 백업해두고 실제 레코드 값을 반영하는 MVCC(Multi Version Concurrency Control) 변경방식

- Dirty Read (더티 리드) 발생하지 않음

- NON-REPEATABLE READ 발생

  > NON-REPEATABLE READ
  >
  > 하나의 트랜잭션 내에서 같은 SELECT 쿼리를 실행했을 때 결과 값이 달라짐 (위의 예제에서 Transaction A의 두개의 read 쿼리가 동일 트랜잭션이었다면, 하나의 트랜잭션 내에서 같은 SELECT 쿼리를 실행했을 때 결과 값이 달라짐)
  >
  > 일반적인 웹 프로그램에서는 크게 문제되지는 않지만 하나의 트랜잭션에서 동일한 데이터를 여러 번 읽고 변경하는 작업이 금전적인 처리와 연결되면 문제가 될 수 있음

- 오라클 DBMS에서 기본적으로 사용

- 온라인 서비스에서 가장 많이 선택되는 격리 수준

- STATEMENT 포맷을 사용하는 바이너리 로그가 활성화된 MySQL 서버에서 사용 불가 (MySQL 5.0이하 버전에서는 경고 메시지를 출력하지는 않지만 그렇다고 안정적인 것은 아니며, MySQL 5.1부터는 경고 메시지를 출력하는 형태로 바뀜)



#### REPEATABLE READ (PHANTOM READ)

![isolation - repeatable read (phantom read)](C:\Users\jungsol\Documents\GitHub\dev-notes\mysql\picture\isolation - repeatable read (phantom read).jpg)

1. READ COMMITTED와 동일하게 MVCC 변경방식을 이용하는데,  언두 영역에 백업되는 레코드 버전에 차이가 있음. REPEATABLE READ에서는 실행중인 트랜잭션보다 오래된 트랜잭션 번호들의 언두 영역의 데이터는 삭제 할 수 없으며 이 언두 영역의 데이터를 이용해서 하나의 트랜잭션 내에서 똑같은 SELECT 쿼리를 실행했을 때는 항상 같은 결과를 가져옴
   * 모든 InnoDB의 트랜잭션은 고유한 트랜잭션 번호(순차적으로 증가하는 값)를 가지며, 언두 영역에 백업된  모든 레코드에는 변경을 발생시킨 트랜잭션의 번호가 포함되어 있음. 언두 영역의 백업된 데이터는 InnoDB 스토리지 엔진이 불필요하다고 판단하는 시점에 주기적으로 삭제하는데, REPEATABLE REAED에서는 현재 트랜잭션보다 이전 트랜잭션들의 언두데이터가 보존되므로, 동일 트랜잭션 내에서는 동일 SELECT 쿼리를 실행하면 현재 트랜잭션보다 이전 트랜잭션의 레코드를 보여주므로 결과가 동일함
   * Transaction A의 트랜잭션이 시작하면 트랜잭션 번호(11이라고 가정)가 주어지며, 이 트랜잭션에서 데이터를 SELECT 하면 11 이전의 트랜잭션 번호의 데이터를 읽어옴. 트랜잭션 번호 12를 가지는 Transaction B가 시작하여 쓰기작업을 하고 커밋을 하였고 그 후에 Transaction A에서 읽기작업을 한번 더 하더라도, Transaction A의 트랜잭션 번호인 10보다 더 작은 트랜잭션 번호의 데이터를 언두영역에서 불러오므로 2번의 읽기작업 모두 데이터가 일치하게 됨

* Dirty Read (더티 리드)와 Non-Repeatable Read 부정합이 발생하지 않음

- Phantom Read( (팬텀 리드) 발생

  - 동일 트랜잭션 상에서 다른 트랜잭션에서 UDPATE 하는 것은 반영되지 않지만 INSERT 하는것은 반여되므로 하나의 트랜잭션이 진행되는 동안 다른 트랜잭션에서 데이터를 삽입하게 되면 SEELCT 결과가 달라질 수 있음
  - SELECT ... FOR UPDATE나 SELECT ... LOCK IN SHARE MODE로 조회되는 레코드는 쓰기 잠금을 걸어야 하는데, 언두 영역에는 잠금을 걸 수 없으므로 이 경우에는 언두 영역의 변경 전 데이터를 가져오는 것이 아니라 현재 레코드의 값을 가져오게 됨

  > Phantom Read (Phantom Row)
  >
  > 다른 트랜잭션에서 수행한 변경 작ㄱ업에 의해 레코드가 보였다가 안 보였다가 하는 현상

- MySQL의 InnoDB 스토리지 엔진에서 기본적으로 사용

- 바이너리 로그를 가진 MySQL 장비에서는 최소 REPEATABLE READ 격리 수준 이상을 사용해야 함

- 오랜 시간동안 트랜잭션이 지속되면 언두 영역이 백업 데이터로 무한정 커져서 MySQL 서버의 처리 성능이 하락될 수 있음



>  **트랜잭션 내에서 실행되는 SELECT 문장과 트랜잭션 없이 실행되는 SELECT 문장의 차이** 
>
> * READ COMMITTED 격리 수준에서는 트랜잭션 내에서 실행되는 SELECT 문장과 트랜잭션 외부에서 싫애되는 SELECT 문장의 차이가 거의 없음
> * REPEATABEL READ 격리 수준에서는 기본적으로 SELECT 쿼리도 트랜잭션 범위 내에서만 작동하기 때문에, 트랜잭션을 시작한 상태에서는 동일한 쿼리를 계속 실행하면 아무리 다른 트랜잭션에서 그 데이터를 변경하고 COMMIT을 실행한다고 하더라도 동일한 결과를 보게됨



#### SERIALIZABLE

* 가장 단순한 격리수준 이지만 가장 엄격한 격리수준 -> 동시 처리 성능도 다른 트랜잭션 격리 수준보다 떨어짐 -> 동시성이 중요한 데이터베이스에서는 거의 사용되지 않음

* Non-locking consistent read 작업도 공유 잠금(읽기 잠금)을 획득해야 하며, 동시에 다른 트랜잭션은 잠긴 레코드를 변경할 수 없게 됨 -> 한 트랜잭션에서 읽고 쓰는 레코드를 다른 트랜잭션에서는 절대 접근 할 수 없음

  > Non-locking consistent read(잠금이 필요 없는 일관된 읽기)
  >
  > - INSERT ... SELECT ... 또는 CREATE TABLE ... AS SELECT ...가 아닌 순수한 SELECT 작업
  > - 보통 아무런 레코드 잠금도 설정하지 않고 실행됨

* PHANTOM READ 문제가 발생하지 않지만, InnoDB 스토리지 엔진에서는 REPEATABLE READ 격리 수준에서도 이미 PHANTOM READ가 발생하지 않기 때문에 굳이 SERIALIZABLE을 사용할 필요성이 없음



#### REPEATABLE READ 격리수준과 READ COMMITTED 격리 수준의 성능 비교

실제 온라인 서비스  상황에서는 발생할 가능성이 거의 없지만 굳이 만들려고 한다면 REPEATABLE READ가 상당히 성능이 떨어지게 만들 수 있다. 

예를 들어, 하나의 트랜잭션을 열어 그 트랜잭션에서 모든 테이블의 데이터를 SELECT 한 후, 그대로 계속 놔두면 InnoDB의 언두(Undo) 영역이 계속 커져서 시스템 테이블 스페이스의 I/O가 유발되는 경우가 대표적인 예이다.

하지만 위의 경우가 아니라면 READ COMMITTED나 REPEATABLE READ 격리 수준의 성능 차이는 크지 않다.

> 벤치마크 결과로는 1gb와 30gb 크기의 테이블에서는 REPEATABLE READ가 2% 정도 높은 성능을 보였고,
>
> 100gb 크기의 테이블에서는 READ COMMITTED가 7% 정도 높은 성능을 보였다.



#### 격리수준 조회

| Name                                     | Cmd-Line | Option File | System Var | Var Scope | Dynamic |
| ---------------------------------------- | -------- | ----------- | ---------- | --------- | ------- |
| [tx_isolation](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_tx_isolation) |          |             | Yes        | Both      | Yes     |

```mysql
SHOW VARIABLES LIKE 'tx_isolation';
```



#### 격리 수준 변경

````mysql
SELECT * FROM table WITH (isolation_level);
또는,
SET TRANSACTION ISOLATION LEVEL isolation_level
````





#### 참고자료

개발자와 DBA를 위한 Real MySQL, 저 이성욱

[Easy Code - Transaction isolation level](http://easycode.onewebapp.com/transaction-isolation-level/)

[달토끼 블로그, 잠금에 관한 고찰(2) - 격리 수준(Transaction Isolation Level)에 대하여](http://kuaaan.tistory.com/98)