## Transaction

#### 트랜잭션

꼭 여러개의 변경 작업을 수행하는 쿼리가 조합됐을 때만 의미 있는 개념은 아니다.

트랜잭션은 하나의 논리적인 작업 셋에 하나의 쿼리가 있든 두 개 이상의 쿼리가 있든 관계없이

* 논리적인 작업 셋 자체가 100% 적용되거나(COMMIT을 실행했을 때)
* 아무것도 적용되지 않아야(ROLLBACK 또는 트랜잭션 ROLLBACK 시키는 오류가 발생했을 때)

함을 보장해 주는 것이다.



#### Partial Update(부분 업데이트)

트랜잭션이 없는 MyISAM, MEMORY, MERGE 스토리지 엔진에서 일부분만 업데이트가 되는 현상으로 데이터의 정합성을 맞추는데 상당히 어려운 문제를 만들어낸다.



#### 트랜잭션 예제

```mysql
CREATEA TABLE test_myisam (id INT NOT NULL, PRIMARY KEY (id)) ENGINE = MyISAM;
INSERT INTO test_myisam (id) VALUES (3);
	ERROR 1062 (23000): Duplicate entry '3' for key 'PRIMARY'
SELECT * FROM test_myisam;
	+---+
	|	|
	|  1|
	|  2|
	|  3|
	+---+
	
CREATE TABLE test_innodb (id INT NOT NULL, PRIMARY KEY (id)) ENGINE = INNODB;
INSERT INTO tesT_innodb (id) VALUES (3);
	ERROR 1062 (23000): Duplicate entry '3' for key 'PRIMARY'
SELECT * FROM test_innodb
	+---+
	|	|
	|  3|
	+---+
```

MyiSAM과 InnoDB 모두 에러가 발생하지만 트랜잭션이 없는 MyISAM 엔진에서는 Partial Update가 발생하여 1과 2는 INSERT 된 상태로 남아있지만, 트랜잭션이 있는 InnoDB는 INSERT 쿼리 문장이 실행하기 전 상태로 그대로 복구되어 있다.



#### 트랜잭션 주의사항

프로그램 코드에서 데이터베이스 커넥션을 가지고 있는 범위와 트랜잭션의 범위는 꼭 필요한 최소의 코드에만 적용하여 최소화 해야한다.

네트워크 작업이 있는 경우에는 이로 인해 DBMS 서버가 높은 부하 상태나 위험한 상태로 빠지는 경우가 발생할 수 있으므로 반드시 트랜잭션에서 배제해야 한다. 







#### 참고자료

개발자와 DBA를 위한 Real MySQL, 저 이성욱