## Variables - System Variables

#### 시스템 변수

MySQL 서버는 기동하면서 설정 파일의 내용을 읽어 메모리나 작동 방식을 초기화하고, 접쏙된 사용자를 제어하기 위해 시스템 변수들을 별도로 저장해 둔다.

[mysql 5.7 server-system-variables.html](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html)



#### 시스템 변수 확인

```mysql
SHOW (GLOBAL) VARIABLES
```



#### 시스템 변수 5가지 선택사항

| Name                   | Cmd-Line | Option-file | System-Var | Var Scope | Dynamic |
| ---------------------- | -------- | ----------- | ---------- | --------- | ------- |
| autocommit             | Yes      | Yes         | Yes        | Session   | Yes     |
| basedir                | Yes      | Yes         | Yes        | Global    | No      |
| delete_format          |          |             | Yes        | Both      | No      |
| general-log            | Yes      | Yes         |            |           | Yes     |
| -Varaiable:general_log |          |             | Yes        | Global    | Yes     |

* Cmd-Line
  * MySQL 서버의 명령형 인자로 설정될 수 있는지의 여부
* Option-file
  * MySQL의 설정 파일인 my.cnf(또는 my.ini)로 제어할 수 있는지의 여부
* System Var
  * 시스템 변수인지 아닌지의 여부를 나타냄
  * 설정파일이나 MySQL 서버의 명령행 인자로는 하이픈으로 구분된 변수명을 사용하지만 MySQL 서버가 시작된 이후 MySQL 서버 내에서는 언더스코어로 구분된 변수명으로 통용되는 경우가 있는데 이런 경우에는 System Var 항목의 값이 비어있으며 두 가지 모두 표시됨. MySQL 서버 내에서 통용되는 이름은 "-Variable:"이라는 추가 내용이 붙어있음
    * MySQL 서버가 예전부터 수많은 사람들의 손을 거쳐오면서 생긴 일관성 없는 변수의 명명 규칙 때문으로, 어떤 변수는 하이픈으로 구분되고 어떤 시스템 변수는 언더스코어로 구분되는 등 상당히 애매모호한 부분들이 있는데, 뒤늦게 이런 부분을 언더스코어로 통일하려는 단계이다 보니 지금과 같은 현상이 나타남
    * 예를 들어, 표의 가장 아래에 있는 general-log 라는 설정 변수는 MySQL 서버의 명령행 인자로 설정될 떄는 하이픈으로 구분된 general-log 라는 이름을 쓰지만 MySQL 서버 내에서 통용되는 이름은 언더스코어로 구분된 general_log라는 이름임
* Var Scope
  * 시스템 변수의 적용 범위, 즉 영향을 미치는 곳 : Global, Session, Both
* Dynamic
  * 시스템 변수가 동적인지 정적인지 구분



#### 글로벌 변수와 세션 변수

시스템 변수 값이 어떻게 MySQL 서버와 클라이언트에 영향을 미치는지 판단하려면 각 변수가 글로벌 변수인지 세션 변수인지 구분할 수 있어야 한다.

* 글로벌 변수
  * 하나의 MySQL 서버 인스턴스에서 전체적으로 영향을 미치는 시스템 변수
  * 주로 MySQL 서버 자체에 관련된 설정
* 셰션 변수 
  * MySQL 클라이언트가 MySQL 서버에 접속할 때 기본적으로 부여하는 옵션의 기본값을 제어
  * 각 커넥션별로 설정값을 다르게 지정할 수 있으며, 한번 연결된 커넥션의 세션변수는 서버에서 강제로 변경할 수 없음
  * 클라이언트가 처음에 접속하면 기본적으로 부여하는 디폴트 값을 가지고 있는데, 클라이언트가 별도로 그 값을 변경하지 않은 경우에는 그대로 값이 유지되지만 클라이언트의 필요에 따라 개별 커넥션 단위로 다른 값으로 변경할 수 있는 것이 세션 변수



#### 동적 변수와 정적 변수

* MySQL 서버가 기동 중인 상태에서 변경가능한지의 여부



#### MySQL의 시스템 변수를 변경하는 방법

* 디스크에 저장돼있는 설정 파일(my.conf 또는 my.ini)를 변경하는 경우

  * MySQL 서버가 재시작 하기 전에는 변경되지 않음

* 이미 기동중인 MySQL 서버의 메모리에 있는 MySQL 서버의 시스템 변수를 변경하는 경우

  * 현재 기동 중인 MySQL 인스턴스에서만 유효하며, MySQL 서버가 재시작하면 다시 설정 파일의 내용으로 초기화
  * GLOBAL 키워드를 사용하면 글로벌 시스템 변수을 조회/변경 할 수 있으며 GLOBAL 키워드를 빼면 세션 변수를 조회/변경 할 수 있음
  * 시스템 변수의 범위가 Both인 경우 글로벌 시스템 변수의 값을 변경해도 이미 존재하는 커넥션의 세션 변경되지 않고 그대로 유지됨

  ```mysql
  SHOW (GLOBAL) variables LIKE ${variable_name};
  SET (GLOBAL) ${variable_name} = ${variable_value};
  ```

  ​





#### 참고자료

개발자와 DBA를 위한 Real MySQL, 저 이성욱