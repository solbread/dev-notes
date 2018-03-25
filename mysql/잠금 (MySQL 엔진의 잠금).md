## MySQL 엔진의 잠금

#### 잠금의 레벨

* 스토리지 엔진 레벨
  * 스토리지 엔진 간 상호 영향을 미치지는 않음
* MySQL 엔진 레벨
  * MySQL 엔진 : MySQL 서버에서 스토리지 엔진을 제외한 나머지 부분
  * 모든 스토리지 엔진에 영향을 미치게 됨



#### GLOBAL LOCK(글로벌 락)

*  FLUSH TABLES WITH READ LOCK 명령으로만 획득 가능
* 잠금 범위 : 서버 전체
  * 한 세션에서 글로벌 락을 획득하면 다른 세션에서 SELECT를 제외한 대부분의 DDL/DML 문장을 실행하는 경우 글로벌 락이 해제될 때 까지 대기상태로 남음
* 사용하는 경우
  * MyISAM이나 MEMORY 테이블에 대해 mysqldump로 일관된 백업을 받아야 할 때

> 글로벌 락을 거는 "FLUSH TABLES WITH READ LOCK" 명령은 실행과 동시에 MySQL 서버에 존재하는 모든 테이블에 잠금을 건다. "FLUSH TABLES WITH READ LOCK" 명령이 실행되기 전에 테이블이나 레코드에 쓰기 잠금을 걸고 있는 SQL이 실행되고 있었다면, 이 명령은 해당 테이블의 읽기 잠금을 걸기 위해 먼저 실행된 SQL이 완료되고 그 트랜잭션이 완료될 때 까지 기다려야 한다. 그런데 "FLUSH TABLES WITH READ LOCK" 명령은 테이블에 읽기 잠금만 걸기 전에 먼저 테이블을 플러시해야 하기 때문에 테이블에 실행되고 있는 모든 종류의 쿼리가 완료돼야만 테이블을 플러시하고 잠금을 걸 수 있다. 그래서 장시간 SEELCT 쿼리가 실행되고 있을 떄는 "FLUSH TABLES WITH READ LOCK" 명령은 SELECT 쿼리가 종료될 떄까지 기다려야만 한다.
>
> 장시간 실행되는 쿼리와 "FLUSH TABLES WITH READ LOCK" 명령이 최악의 케이스로 실행되면 MySQL 서버의 모든 테이블에 대한 INSERT, UPDATE, DELETE 쿼리가 아주 오랜 시간 동안 실행되지 못하고 기다려야 할 수도 있다. 글로벌 락은 MySQL 서버의 모든 테이블에 큰 영향을 미치기 때문에 웹 서비스용으로 사용되는 MySQL 서버에서는 가급적 사용하지 않는 것이 좋다. 또한 mysqldump 같은 백업 프로그램은 우리가 알지 못하는 사이에 이 명령을 내부적으로 실행하고 백업할 때도 있다. 만약 mysqldump를 이용해 백업을 수행한다면 mysqldump에서 사용하는 옵션에 따라 MySQL 서버에 어떤 잠금을 걸게 되는지 자세히 확인해보는 것이 좋다.



#### TABLE LOCK (테이블 락)

* 개별 테이블 단위로 설정되는 잠금
* 명시적으로는
  * "LOCK TABLES table_name [READ | WRITE] 명령으로 획득
  * "UNLOCK TALBES" 명령으로 잠금을 반납(해제)
* 묵시적으로는
  * MyISAM이나 MEMORY 테이블에 데이터를 변경하는 쿼리를 실행하면 발생
  * InnoDB 테이블의 경우 스토리지 엔진 차원에서 레코드 기반의 잠금을 제공하기 때문에 단순 데이터 변경 쿼리로 인해 묵시적인 테이블 락이 설정되지는 않음. 정확히는 InnoDB 테이블에서도 테이블 락이 설정되지만 대부분의 데이터 변경(DML) 쿼리에서는 무시되고 스키마를 변경하는 쿼리(DDL)의 경우에만 영향을 미침



#### USER LOCK (유저 락)

* 단순히 사용자가 지정한 문자열(String)에 대해 획득하고 반납(해제)하는 잠금
* 명령어
  * 잠금 설정 : GET_LOCK(string, wait_second)
  * 잠금 확인 : IS_FREE_LOCK(string)
  * 잠금 반납(해제) : RELEASE_LOCK(string)
* 사용하는 경우
  * 데이터베이스 서버 1대에 5대의 웹서버가 접속해서 서비스를 하고 있는 상태에서 5대의 웹 서버가 어떤 정보를 동기화해야 하는 요건처럼 여러 클라이언트가 상호 동기화를 처리해야 할 때

```mysql
-- // "mysql"이라는 문자열에 대해 잠금을 획득한다.
-- / 이미 잠금이 사용중이면 2초 동안만 대기한다.
SELECT GET_LOCK('mylock', 2);

-- // "mylock"이라는 문자열에 대해 잠금이 설정돼 있는지 확인한다.
SELECT IS_FREE_LOCK('mylock');

-- // "mylock"이라는 문자열에 대해 획득했던 잠금을 반납(해제)한다.
SELECT RELEASE_LOCK('mylock');

-- // 3개 함수 모두 정상적으로 락을 획득하거나 해제한 경우에는 1을, 아니면 NULL이나 0을 반환한다.
```



#### NAME LOCK (네임 락)

* 데이터베이스 객체(대표적으로 테이블이나 뷰 등)의 이름을 변경하는 경우 획득하는 잠금

* 명시적으로 획득 불가

* 테이블의 이름을 병경하는 경우 획득

* Example

  ```mysql
  -- // 배치 프로그램에서 별도의 임시 테이블(rank_new)에 서비스용 랭킹 데이터를 생성

  -- // 랭킹 배치가 완료되면 현재 서비스용 랭킹 테이블(rank)를 rank_backup으로 백업하고
  -- // 새로 만들어진 랭킹 테이블(rank_new)을 서비스용으로 대체하고자 하는 경우

  -- // 하나의 RENAME TABLE 명령문에 두 개의 RENAME 작업을 한꺼번에 실행하면
  -- // 실제 애플리케이션에서는 "Table not found 'rank'"와 같은 상황이 발생되지 않고 적용됨
  RENAME TABLE rank TO rank_backup, rank_new TO rank;

  -- // 2개의 RENAME TABLE 명령문으로 나눠서 실행하면
  -- // 아주 짧은 시간이지만 'rank' 테이블이 존재하지 않는 순간이 생기며,
  -- // 그 순간에 쿼리는 "Table not found 'rank'" 오류를 발생시킴
  RENAME TABLE rank TO rank_bakcup;
  RENAME TABLE rank_new TO rank;
  ```



