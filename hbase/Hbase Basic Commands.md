## Hbase Basic Commands

#### hbase shell 실행

`bin/hbase shell`



#### 테이블 관련 명령어

* 전체 테이블 리스트 조회
  * `list`
* 테이블 생성
  * `create 'table_name', 'cf', 'cf', .... `
* Column family 추가
  * `alter 'table_name', 'cf'`



#### 데이터 조회 

* 테이블 전체 데이터 조회

  `scan 'table_name'`

* 테이블내의 특정 column들 조회

  `scan 'table_name', {"COLUMNS" => ['cf:cq', 'cf:cq', ...]}`

* 특정 row 데이터 조회

  `get 'table_name', 'row_key'`

* 특정 row의 특정 column family 조회

  `get 'table_name', 'row_key', 'cf'`

* 특정 row의 특정 column family의 특정 column qualifier 조회

  `get 'table_name', 'row_key', 'cf:cq'`

* 특정 row의 특정 column들 조회

  `get 'table_name', 'row_key', ['cf:cq', 'cf:cq', ...]`

  ​



#### 데이터 삽입

`put 'table_name', 'row_key', 'cf:cq', 'value'`

