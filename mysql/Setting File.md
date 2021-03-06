## Setting File



MySQL은 단 하나의 설정 파일만 사용한다.

* 파일의 이름은 변경할 수 없다.
  * 유닉스 계열 : my.cnf
  * 윈도우 계열 : my.ini

* 파일의 경로가 딱 하나로 고정된 것은 아니다.

  * MySQL 서버는 지정된 여러 개의 디렉터리를 순차적으로 탐색하면서 처음 발견된 설정파일을 사용

  * 디렉토리 확인하는 방법

    ```
    mysql --help
    ...
    Default options are read from the following files in the given order:
    /etc/my.conf /etc/mysql/my.conf /usr/local/mysql/etc/my.conf ~/.my.conf
    ...
    ```



#### 설정 파일의 구성

```
[clinet] default-character-set = utf8

[mysql]
socket	= /usr/local/mysqL/tmp/mysql.sock
port	= 3304

[mysqldump]
socket	= /usr/local/mysqL/tmp/mysqL.sock
port	= 3305

[mysqlId_safe]

[mysqlId]
socket	=/usr/local/mysql/tmp/mysql.sock
port	=3306
```

* 여러개의 설정 그룹을 담으려면 \[ ${group_name} \]로 구분하여 사용
* MySQL 서버(mysqlId) 프로그램은[mysqlId\] 설정그룹을 참조하며, MySQL 클라이언트(mysql) 프로그램은 \[mysql]과 \[client\]를 동시에 참조한다.
* \[client]
  * MySQL 서버(mysqlId와 mysqlId_safe)를 제외한 대부분의 클라이언트 프로그램이 공유하는 영역
  * Mysql 클라이언트(mysql) 프로그램이나 MySQL 백업 프로그램인 mysqldump 프로그램 등은 모두 클라이언트 프로그램의 분류에 속하기 때문에 \[clinet\] 설정 그룹을 공유하며 동시에 각각 자기 자신의 그룹도 함께 읽어서 사용.
* \[mysqlId_safe]
  * MySQL 서버가 비정상적으로 종료됐을 때 재시작하는 일만 하는 프로세스
  * 일반적으로 잘 사용되지 않지만, MySQL 서버의 타임존을 설정할때는 이 설정그룹을 꼭 이용해야 함









#### 참고자료

개발자와 DBA를 위한 Real MySQL, 저 이성욱