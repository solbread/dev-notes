## Lock - MyISAM, MEMORY

MyISAM이나 MEMORY 스토리지 엔진은 자체적인 잠금을 가지지 않고 MySQL 엔진에서 제공하는 테이블 락을 그대로 사용한다.

MyISAM이나 MEMORY 스토리지 엔진에서는 쿼리 단위로 필요한 잠금을 한꺼번에 모두 요청해서 획득하기 때문에 데드락이 발생할 수 없다.

별도로 표기하진 않지만 여기서 언급되는 잠금은 MySAM 스토리지 엔진 뿐 아니라 MySQL에서 기본적으로 제공되는 MEMORY나 ARCHIVE, 그리고 MERGE 등과 같은 스토리지 엔진에도 동일하게 적용된다.



#### 읽기 잠금 획득

테이블에 쓰기 잠금이 걸려 있지 않으면 바로 읽기 잠금을 획득하고 읽기 작업을 시작할 수 있다.



#### 쓰기 잠금 획득

테이블에 아무런 잠금이 걸려 있지 않아야만 쓰기 잠금을 획득할 수 있고, 그렇지 않다면 다른 잠금이 해제될 때까지 기다려야 한다.



#### 잠금 튜닝

테이블 락에 대한 작업 상황은 MySQL의 상태 변수를 통해 확인할 수 있다.

````mysql
SHOW STATUS LIKE 'Table%';
+---------------------------+-----------------+
| Variable_name				+ Value			  |
+---------------------------+-----------------+
| Table_locks_immediate		| 1151552		  |
| Table_locks_waited		| 15324			  |
+---------------------------+-----------------+
````

* Table_locks_immediate
  * 다른 잠금이 풀리기를 기다리지 않고 바로 잠금을 한 횟수
* Table_locks_waited
  * 다른 잠금이 이미 테이블을 사용하고 있어서 기다야 했던 횟수를 누적
* 잠금 대기 쿼리 비율
  * Table_locks_wait / (Table_locks_immediate + Table_locks_waited) * 100
  * 이 수치가 높고 테이블 잠금 떄문에 경합(Lock connection)이 많이 발생하고 있으면 처리 성능이 영향을 받고 있음을 의미하므로 테이블을 분리하거나 InnnoDB 스토리지 엔진으로 변환하는 방법을 고려해야 함
  * InnodDB 스토리지 엔진의 경우에는 레코드 단위의 잠금을 사용하기 때문에 집계에 포함되지 않음. 집계된 수치는 MyISAM 또는 MEMORY 또는 MERGY 스토리지 엔진을 사용하는 테이블이 대상이 됨



#### 테이블 수준의 잠금

MyISAM이나 MEMORY 등과 같은 스토리지 엔진을 사용하는 테이블은 모두 테이블 단위의  잠금이므로 테이블을 해체하지 않으면 다른 클라이언트에서 그 테이블을 사용하는 것은 불가능하다.

하나의 테이블에서 전혀 다른 레코드라 하더라도 동시에 변경하는 것은 불가능하기 때문에 쿼리 처리의 동시성이 떨어지게 된다.



#### 테이블 수준의 잠금 확인

```mysql
SHOW OPEN TABLES;
SHOW OPEN TABLES FROM db_name;
SHOW OPEN TABLES FROM db_name LIKE 'table_pattern';

+---------------+---------------+-----------+---------------+
| Database		| Table			| In_use	| Name_locked	|
+---------------+---------------+-----------+---------------+
+ employees		| employees		| 		3	|			0	|
+---------------+---------------+-----------+---------------+
```

* MySQL 서버의 모든 테이블 / db_name의 테이블 / db_name의 talbe_pattern의 테이블 에 대해 잠금 여부를 명시
* In_use : 해당 테이블을 잠그고 있는 클라이언트의 수 + 테이블의 잠금을 기다리는 클라이언트의 수 
* Name_locked : 테이블 이름에 대한 NAME LOCK(네임 락)이 걸려있는지를 표시

```mysql
SHOW PROCESSLIST;

-- // 1번 커넥션으로 인해 3번 4번 커넥션이 Lock걸리 상태
-- // 1번 커넥션을 종료시키면 3번과 4번이 차례대로 처리됨
+---+-------+-----------+-------+---------+-------+--------+----------------------------+
| Id| User	| Host		| db	| Command | Time  | State  | Info						|
+---+-------+-----------+-------+---------+-------+--------+----------------------------+
|  1| root  | localhost | test  | Sleep   |    46 |		   | NULL						|
|  3| root  | localhost | test  | Query   |    22 | Locked | UPDATE a SET b = NOW()		|
|  4| root  | localhost | test  | Query   |    7  | Locked | UPDATE a SET C = NOW()		|
|  5| root  | localhost | test  | Query   |    0  | NULL   | SHOW PROCESSLIST			|
+---+-------+-----------+-------+---------+-------+--------+----------------------------+

```

* 어떤 클라이언트의 커넥션이 잠금을 기다리고 있는지를 보여줌



#### 테이블 수준의 잠금 해제

```mysql
-- // 클라이언트가 실행하고 있는 쿼리 종료
KILL QUERY client_id;

-- // 클라이언트 커넥션을 종료
KILL client_id;
```







#### 참고자료

개발자와 DBA를 위한 Real MySQL, 저 이성욱