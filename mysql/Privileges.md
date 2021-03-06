## Privileges

MySQL의 사용자 계정은 단순히 사용자의 아이디뿐 아니라 해당 사용자가 어느 IP에서 접속하고 있는지도 확인한다. 또한 MySQL 에서는 권한을 묶어서 관리하는 롤(Role)의 개념이 존재하지 않기 떄문에 일반적으로 모든 권한을 사용자에게 부여해 버릴 때가 많다.



#### 사용자의 식별

MySQL의 사용자는 다른 DBMS와는 조금 다르게 사용자의 계정 뿐 아니라 사용자의 접속 지점(클라이언트가 실행된 호스트 명이나 도메인 또는 IP 주소)도 계정의 일부가 된다. -> MySQL에서 계정을 언급할 때는 항상 아이디와 호스트가 같이 명시되어야 한다.

> 'solbread'@'192.168.0.10' (계정의 비밀번호는 123)
>
> 'solbread'@'%' (계정의 비밀번호는 abc)

* 만약 'solbread'@'192.168.0.10' 만 등록되어 있다면 '192.168.0.10'외에서는 solbread로 접속할 수 없음
* '%' : 모든 외부 컴퓨터에서 접속이 가능한 사용자 계정을 만들려면 호스트 부분을 '%'로 대체
* 권한이나 계정정보에 대해 MySQL은 범위가 가장 작은 것을 먼저 선택하므로, '192.168.0.10'에서 solbread로 MySQL에 서버에 접속할 때는 'solbread'@'192.168.0.10' 계정 정보를 이용해서 사용자 인증을 하게된다. (따라서 비밀번호를 abc로 입력하게 되면 "비밀번호가 일치하지 않는다"라는 이유로 접속이 거절됨)



#### 권한(Privileges)

MySQL에서는 권한을 묶어서 권한 그룹(Role)으로 관리하는 기능은 제공하지 않으며, 모든 사용ㅎ자는 하나하나의 권한을 가지고 있는다. -> 매번 사용자 계정을 새로이 생성하거나 사용자 계정의 권한을 변경할 때는 구넣나 아이템을 하나씩 제어해야 하는데, 이는 상당히 귀찮은 작업이 될 수 있기 때문에 권한을 미리 묶어서 템플릿처럼 만들어 두면 유용하다.

[MySQL 5.7 privileges-provided.html](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html)

| Privilege                                | Column                       | Context                               |
| ---------------------------------------- | ---------------------------- | ------------------------------------- |
| [`ALL [PRIVILEGES]`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_all) | Synonym for “all privileges” | Server administration                 |
| [`ALTER`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_alter) | `Alter_priv`                 | Tables                                |
| [`ALTER ROUTINE`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_alter-routine) | `Alter_routine_priv`         | Stored routines                       |
| [`CREATE`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_create) | `Create_priv`                | Databases, tables, or indexes         |
| [`CREATE ROUTINE`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_create-routine) | `Create_routine_priv`        | Stored routines                       |
| [`CREATE TABLESPACE`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_create-tablespace) | `Create_tablespace_priv`     | Server administration                 |
| [`CREATE TEMPORARY TABLES`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_create-temporary-tables) | `Create_tmp_table_priv`      | Tables                                |
| [`CREATE USER`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_create-user) | `Create_user_priv`           | Server administration                 |
| [`CREATE VIEW`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_create-view) | `Create_view_priv`           | Views                                 |
| [`DELETE`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_delete) | `Delete_priv`                | Tables                                |
| [`DROP`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_drop) | `Drop_priv`                  | Databases, tables, or views           |
| [`EVENT`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_event) | `Event_priv`                 | Databases                             |
| [`EXECUTE`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_execute) | `Execute_priv`               | Stored routines                       |
| [`FILE`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_file) | `File_priv`                  | File access on server host            |
| [`GRANT OPTION`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_grant-option) | `Grant_priv`                 | Databases, tables, or stored routines |
| [`INDEX`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_index) | `Index_priv`                 | Tables                                |
| [`INSERT`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_insert) | `Insert_priv`                | Tables or columns                     |
| [`LOCK TABLES`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_lock-tables) | `Lock_tables_priv`           | Databases                             |
| [`PROCESS`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_process) | `Process_priv`               | Server administration                 |
| [`PROXY`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_proxy) | See `proxies_priv` table     | Server administration                 |
| [`REFERENCES`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_references) | `References_priv`            | Databases or tables                   |
| [`RELOAD`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_reload) | `Reload_priv`                | Server administration                 |
| [`REPLICATION CLIENT`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_replication-client) | `Repl_client_priv`           | Server administration                 |
| [`REPLICATION SLAVE`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_replication-slave) | `Repl_slave_priv`            | Server administration                 |
| [`SELECT`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_select) | `Select_priv`                | Tables or columns                     |
| [`SHOW DATABASES`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_show-databases) | `Show_db_priv`               | Server administration                 |
| [`SHOW VIEW`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_show-view) | `Show_view_priv`             | Views                                 |
| [`SHUTDOWN`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_shutdown) | `Shutdown_priv`              | Server administration                 |
| [`SUPER`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_super) | `Super_priv`                 | Server administration                 |
| [`TRIGGER`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_trigger) | `Trigger_priv`               | Tables                                |
| [`UPDATE`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_update) | `Update_priv`                | Tables or columns                     |
| [`USAGE`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_usage) | Synonym for “no privileges”  | Server administration                 |

순수하게 DB 관리와 연관된 권한이므로 일반 사용자나 서비스 계정에 부여할 때 주의가 필요한 옵션

*  GRANT OPTION, ALL, CREATE USER, SHUTDOWN, SUPER



#### 전역 권한과 로컬 권한

* 전역 권한
  * DB나 테이블 단위의 권한이 아니라 MySQL 서버 전역적으로 작동하는 권한
  * MySQL 서버의 프로세스나 복제에 관련된 정보와 같이 테이블이나 칼럼이 아니라 MySQL 서버 전체에 영향을 미치는 권한
* 로컬 권한
  * 기본적으로는 DB 단위로 부여하는 권한이지만 테이블이나 칼럼 단위이까지 부여 가능
  * MySQL에서는 권한을 칼럼 단위로 부여하지 않는 것이 성능상 좋음



#### MySQL의 사용자 계정 초기화

MySQL에서 root 사용자 계정은 기본적으로 모든 권한을 가지고 있는 것으로 초기화돼 있다. 처음 설치된 MySQL 서버에는 비밀번호가 없는 root 계정과 아이디가 없는 계정도 있다. 따라서 항상 새로운 MySQL을 설치하면 모든 사용자 계정을 삭제하고 다시 재설정 할 것을 권장한다. MySQL 관리자 계정은 항상 root인 것으로 생각하는 사용자가 많은데 이는 초기에 설치된 MySQL의 root 계정이 모든 권한을 가지고 있게 되어 있기 때문이다. MySQL에서 'root'라는 이름의 계정이 항상 필요한 것은 아니기 때문에 필요하다면 MySQL 서버의 관리자 계정을 'admin'으로 변경해도 무방하다(단 관리자로서 필요한 권한은 모두 별도로 부여해야 한다)



#### 권한 부여

[MySQL 5.7 grant.html](https://dev.mysql.com/doc/refman/5.7/en/grant.html)

GRANT SQL 문장을 사용한다.

```mysql
GRANT
    priv_type [(column_list)]
      [, priv_type [(column_list)]] ...
    ON [object_type] priv_level
    TO user [auth_option] [, user [auth_option]] ...
    [REQUIRE {NONE | tls_option [[AND] tls_option] ...}]
    [WITH {GRANT OPTION | resource_option} ...]

GRANT PROXY ON user
    TO user [, user] ...
    [WITH GRANT OPTION]
    
    
GRANT
    priv_type [(column_list)]
      [, priv_type [(column_list)]] ...
    ON [object_type] priv_level
    TO user [auth_option] [, user [auth_option]] ...
    [REQUIRE {NONE | tls_option [[AND] tls_option] ...}]
    [WITH {GRANT OPTION | resource_option} ...]

GRANT PROXY ON user
    TO user [, user] ...
    [WITH GRANT OPTION]

object_type: {
    TABLE
  | FUNCTION
  | PROCEDURE
}

priv_level: {
    *
  | *.*
  | db_name.*
  | db_name.tbl_name
  | tbl_name
  | db_name.routine_name
}

user:
    (see Section 6.2.3, “Specifying Account Names”)

auth_option: {
    IDENTIFIED BY 'auth_string'
  | IDENTIFIED WITH auth_plugin
  | IDENTIFIED WITH auth_plugin BY 'auth_string'
  | IDENTIFIED WITH auth_plugin AS 'hash_string'
  | IDENTIFIED BY PASSWORD 'hash_string'
}

tls_option: {
    SSL
  | X509
  | CIPHER 'cipher'
  | ISSUER 'issuer'
  | SUBJECT 'subject'
}

resource_option: {
  | MAX_QUERIES_PER_HOUR count
  | MAX_UPDATES_PER_HOUR count
  | MAX_CONNECTIONS_PER_HOUR count
  | MAX_USER_CONNECTIONS count
}
```

* GRANT 문장이 실행될 때, 지정된 사용자가 존재하지 않으면 해당 사용자를 생성하고 권한을 부여
* WITH GRANT OPTION
  * GRANT OPTION은 다른 권한과 달리 GRANT 무장의 마지막에 "WITH GRANT OPTION"을 이용해 부여



#### 글로벌 권한

```mysql
GRANT SUPER ON *.* TO 'user'@'localhst';
```

* 글로벌 권한은 특정 DB나 테이블에 부여될 수 없기 때문에 ON 절에는 항상 "\*.\*"를 사용

  ​

#### DB 권한

```mysql
GRANT EVENT ON *.* TO 'user'@'localhost';
GRANT EVENT ON dbname.* TO 'user'@'localhost';
```

* DB 권한은 특정 DB에 대해서만 권한을 부여하거나, 서버에 존재하는 모든 DB에 대해 권한을 부여할 수 있기 때문에 ON  절에 "\*.\*" 이나 "dbname.\*"을 사용 가능



#### 오브젝트 권한

```mysql
GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO 'user'@'localhost';
GRANT SELECT, INSERT, UPDATE, DELETE ON dbname.* TO 'user'@'localhost';
GRANT SELECT, INSERT, UPDATE, DELETE ON dbname.tablename TO 'user'@'localhost';
```

* 오브젝트 권한은 서버의 모든 DB / 특정 DB의 오브젝트 / 특정 ㅇB의 특정 테이블에 대해서 권한 부여 가능



#### 칼럼 권한

```mysql
GRANT SELECT(columnname), INSERT(columnname), UPDATE(columnname) ON dbname.tablename TO 'user'@'localhost';
```

* 각 권한의 뒤에 칼럼을 명시
* 칼럼 이름을 생략하면 모든 칼럼에 대해 수행할 수 있는 권한을 부여



#### 뷰(View)

여러 가지 레벨이나 범위로 권한을 설정하는 것이 가능하지만 테이블이나 칼럼 단위의 권한은 잘 사용하지 않는다.

칼럼 단위의 권한이 하나라도 설정되면 나머지 모든 테이블의 모든 칼럼에 대해서도 권한 체크를 하기 때문에 칼럼 하나에 대해서만 권한을 설정하더라도 결과적으로 성능에 좋지 않은 영향을 끼친다.

칼럼 단위의 접근 권한이 꼭 필요하다면 GRANT로 해결하기보다는 테이블에서 권한을 허용하고자 하는 칼럼만으로 별도의 뷰(VIEW)를 만들어 사용하는 방법을 권장한다.

하나의 테이블로 인식되기 때문에 뷰를 만들어 두면 뷰의 칼럼에 대해 권한을 체크하지 않고 뷰 자체에 대한 권한만 체크하면 된다.







#### 참고자료

개발자와 DBA를 위한 Real MySQL, 저 이성욱