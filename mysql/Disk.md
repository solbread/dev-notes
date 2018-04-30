## Disk

컴퓨터의 CPU나 메모리와 같은 전기적 특성을 띤 장치의 성능은 짧은 시간 동안 매우 빠른 속도로 발전했지만 디스크와 같은 기계식 장치의 성능은 상당히 제한적으로 발전했다.

데이터베이스의 성능 튜닝은 어떻게 디스크 I/O를 줄이느냐가 관건인 것들이 상당히 많다.



#### 저장 매체

데이터를 저장할 수 있는 매체

* Internal Disk (내장 디스크)
  * PC 본체 내에 장착된 디스크와 같은 매체
  * 컴퓨터의 본체 내부 공간은 제하적이어서 장착할 수 있는 디스크의 개수가 적고 용량도 부족할 수 있음
* DAS(Direct Attached Storage)
  * 내장 디스크 용량 문제를 해결하기 위해 사용
  * 디스크만 있으며, SATA나 SAS 또는 SCSI와 같은 케이블로 컴퓨터 본체에 연결해서 사용가능
  * 성능은 내장디스크와 비슷하며, 하나의 컴퓨터 본체에만 연결해서 사용할 수 있기 떄문에 디스크의 정보를 여러 컴퓨터가 동시에 공유하는 것이 불가능
* NAS(Network Attached Storage)
  * 내장 디스크와 DAS의 문제점을 동시에 해결
  * NAS는 TCP/IP를 통해 컴퓨터 본체와 연결되므로 여러 컴퓨터에서 공유해서 사용할 수 있는 저장소
  * SATA나 SAS 방식의 직접 연결보다는 속도가 매우 느리므로 빈번한 데이터의 읽기/쓰기가 필요한 데이터베이스 서버용으로는 거의 사용되지 않음
* SAN(Storage Area Network)
  * 내장 디스크와 DAS의 문제점을 동시에 해결
  * DAS로 구축할 수 없는 아주 대용량의  스토리지 공간을 제공
  * 여러 컴퓨터에서 동시에 사용할 수 있을 뿐더러 컴퓨터 본체와 광케이블로 연결되기 때문에 상당히 빠르고 안정적인 데이터 처리(읽고 쓰기)를 보장
  * 고가의 구축 비용이 들기 때문에 기업에서 중요한 데이터를 보관할 경우에만 쓰임 

> 고사양/높은 구축비용 : 내장 디스크 < DAS < SAN
>
> 각 장치가 얼마나 많은 디스크 드라이브를 장착할 수 있는지, 그리고 어떤 방식으로 컴퓨터 본체에 연결되는지에 따른 구분일 뿐, 여기에 언급된 모든 저장 매체는 내부적으로 1개 이상의 디스크 드라이브를 장착하고 있다는 점은 같다. 대부분의 저장 매체는 디스크 드라이브의 플래터(Platter, 디스크 드라이브 내부의 데이터 저장용 원판)를 회전시켜 데이터를 읽고 쓰는 기계적인 방식을 사용한다.



#### 디스크 드라이브와 솔리드 스테이트 드라이브(SSD)

컴퓨터에서 CPU나 메모리와 같은 주요 장치는 대부분 전자식 장치이지만 디스크 드라이브는 기계식 장치이다. 그래서 데이터베이스 서버에서는 항상 디스크 장치가 병목 지점이 된다. 

기계식 디스크 드라이브를 대체하기 위해 전자식 저장 매체인 SSD(Solid State Drive)가 많이 출시되고 있다. SSD로 기존 디스크 드라이브와 같은 인터페이스(SATA나 SAS)를 지원하므로 내장 디스크나 DAS 또는 SAN에 그대로 사용할 수 있다.



#### Solid State Drive (SSD)

* 기존의 Disk Drive에서 데이터 저장용 플래터(원반)을 제거하고 대신 플래시 메모리를 장착하여 디스크 원판을 기계적으로 회전시킬 필요가 없어 아주 빨리 데이터를 읽고 쓸 수 있게 함
  * 디스크의 헤더를 움직이지 않고 한번에 많은 데이터를 읽는 순차 I/O에서는 SSD가 디스크 드라이브보다 조금 빠르거나 거의 비슷한 성능이지만, 랜덤 I/O는 디스크 드라이브보다 훨씬 빠름 -> 데이터베이스 서버에서 순차 I/O 작업은 그다지 비중이 크지 않고 랜덤 I/O를 통해 작은 데이터를 읽고 쓰는 작업이 대부분이므로 SSD의 장점은 DBMS용 스토리지에 최적화되어 있음
* 플래시 메모리이므로 전원이 공급되지 않아도 데이터가 삭제되지 않음
* 속도(빠른순) : CPU > D-RAM(메모리) > SSD > DISK(HDD)


#### 랜덤 I/O와 순차 I/O

![random io and sequentail io definition](./picture/random io and sequentail io definition.jpg)

​					위의 그림에서 1, 2, 3, 4, 6은 Random I/O, 5는 Sequentail I/O



랜덤 I/O와 순차 I/O는 모두 디스크 드라이브의 플래터(원판)를 돌려서 읽어야 할 데이터가 저장된 위치로 디스크 헤더를 이동시킨 다음 데이터를 읽는 것을 의미한다.

차이점은,

![reandom io and sequential io](./picture/reandom io and sequential io.jpg)

* 랜덤 I/O는 3개의 페이지를 디스크에 기록하기 위해 3번 시스템 콜 (=디스크에 기록해야 할 위치를 찾기 위해 디스크 헤더를 3번 움직임)
* 순차 I/O는 3개의 페이지를 디스크에 기록하기 위해 1번 시스템 콜 (=디스크에 기록해야 할 위치를 찾기 위해 디스크의 헤드를 1번 움직임)

디스크에 데이터를 쓰고 읽는데 걸리는 시간은 디스크 헤더를 움직여서 읽고 쓸 위치로 옮기는 단계에서 결정되므로, 위의 경우에 순차I/O는 랜덤I/O보다 거의 3배정도 빠름

디스크의  성능은 디스크 헤더의 위치 이동 없이 얼마나 많은 데이털르 한 번에 기록하느냐에 의해 결정됨 -> 여러 번 쓰기 또는 읽기를 요청하는 랜덤 I/O 작업이 훨씬 부하가 큼



#### 랜덤 I/O와 MySQL

* 데이터베이스 대부분의 작업은 작은 데이터를 빈번히 읽고 쓰기 때문에 랜덤 I/O가 발생하게 되므로, MySQL 서버에는 그룹커밋이나 바이너리 로그 버퍼 또는 InnoDB 로그 버퍼 등의 기능이 내장되어 있음
* 쿼리를 튜닝해서 랜덤I/O를 순차I/O로 바꿔서 실행할 방법은 그다지 많지 않음. 일반적으로 쿼리를 튜닝하는 것은 랜덤 I/O 자체를 줄여주는 것이 목적이라고 할 수 있음. 여기서 랜덤 I/O를 줄인다는 것은 쿼리를 처리하는데 꼭 필요한 데이터만 읽도록 쿼리를 개선하는 것을 의미



#### 순차 I/O와 MySQL

* 랜덤I/O나 순차I/O나 모두 파일에 쓰기를 실행하면, 반드시 동기화(fsync 또는 flush 작업)가 필요함. 그런데 순차 I/O인 경우에도 이런 파일 동기화 작업이 빈번히 발생하면 랜덤I/O와 같이 비효율적인 형태로 처리될 때가 많음 -> 기업용으로 사용하는 데이터베이스 서버에는 캐시 메모리가 장착된 RAID 컨트롤러가 일반적으로 사용되는데, RAID 컨트롤러의 캐시 메모리는 아주 빈번한 파일 동기화 작업이 호출되는 순차 I/O를 효율적으로 처리될 수 있게 변환하는 역할을 함
* 인텍스 레인지 스캔은 데이터를 읽기 위해 주로 랜덤 I/O를 사용하며, 풀 테이블 스캔은 순차 I/O를 사용한다. 그래서 큰 테이블의 레코드 대부분을 읽는 작업에서는 인덱스를 사용하지 않고 풀 테이블 스캔을 사용하도록 유도할 때도 있다. 이는 순차 I/O가 랜덤 I/O보다 훨씬 많은 레코드를 읽어올 수 있기 떄문에, 이런 형태는 OLTP 성격의 웹 서비스보다는 데이터 웨어하우스나 통계작업에서 자주 사용된다.







#### 참고자료

개발자와 DBA를 위한 Real MySQL, 저 이성욱