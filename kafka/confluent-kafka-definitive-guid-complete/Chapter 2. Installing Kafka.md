## Chapter 2. Installing Kafka



* Installing Kafka (책 기준)

  * Java : 1.8.0_51
  * Zookeeper : 3.4.6
  * Kafka : 0.9.0.1
  * Scala : 2.11.0

* Choosing an Operating System

  * Kafka는 Java applicatin
  * 많은 operation system에서 run 가능
    * Windows, MacOS, Linux and the tohers
    * Linux environment가 가장 많이 쓰이는 OS이며 deploying Kafka를 위해 추천되는 OS

* Installing Java

  * Zookeeper와 Kafka를 installing 하기 전에 Java environment를 set up 및 functioning(작동)해야 함
  * Zookeeper와 Kafka는 runtime edition of Java에서 작동하므로 full Java Development Kit(JDK)가 편리함

* Installing Zookeeper

  * Zookeeper : Kafka cluster의 metadata(broker와 topic의 metadata)와 consumer client(old)의 세부사항(consumer metadata와 partiton offsets)을 저장

* Zookeeper ensemble

  * Zookeeper cluster는 ensemble이라고 불림
    * Zookeeper의 알고리즘 때문에 ensembles는 홀수개의 servers를 포함하도록 권장됨
    * requests에 대한 Zookeeper의 response는, three-node ensemeble의 경우 하나의 node가 missing이더라도(request를 잃더라도) run 될 수 있으며, five-node ensemble의 경우 두개의 node가 missing이더라도 run될 수 있음
  * Sizing your Zokeeper Ensemble
    * 5개의 node를 사용하기를 권장
      * ensemble이 하나 이상의 node가 내려갔을 때 이를 계속 유지한다면 추가적인 risk가 심하므로 남아있는 nodes간의 합의하여 configuration changes가 필요 
      * ensemble의 configuration의 changes를 위해서는 모든 nodes를 한번에 reload 해야함.
      * 7개 이상의 node를 사용하는 것은 nature of the consensus protocol (합의 프로토콜의 본질) 때문에 성능이 저하되므로 추천되지 않음

* Zookeeper configuration Options

  ```
  tickTime=2000
  dataDir=/var/lib/zookeeper
  clientPort=2181
  initLimit=20
  syncLimit=5
  server.1=zoo1.example.com:2888:3888
  server.2=zoo2.example.com:2888:3888
  server.3=zoo3.example.com:2888:3888
  ```

  * tickTime
    * 기준 시간 (ms 단위)
    * initLimit과 syncLimit의 time을 계산하는데 기준이됨
  * dataDir
    * tarnsaction log등을 저장
  * clientPort
  * initLimit
    * the amount of time to allow followers to connection with a leader (leader와 followers 간의 connection이 허용되는 시간)
    * tickTime units을 기준으로 계산
      * tickTime=2000, initLimit=20이면 실제 intiLimit은 20*2000ms = 40seconds
  * syncLimit
    * limits how out-of-sync followers can be with the leader. (leader와 동기화 되지 않은 followers의 limit)
    * tickTime units을 기준으로 계산
  * server.x = hostname:peerPort:leaderPort
    * x
      * The ID number of the server (server의 id)
      * 반드시 Integer
      * 반드시 zero-based이거나 sequentail(순차적)일 필요는 없음
    * hostname
      * server의 hostname or IP address
    * peerPort
      * TCP Port over which severs in the ensemble communicate with each other (ensemble의 서버간 혹은 다른것들과의 통신을 위한 TCP Port)
    * leaderPort
      * The TCP port over which leader election is performed (leader election(선출)에 사용되는 TCP port)
  * cleints는 ensemble에 clientPort로만 connect할 수 있지만 ensemble mebers간에는 3개의 ports 모두로 communication할 수 있음 
    * 3개의 port는 책에는 명시되어있지 않지만 clientPort, peerPort, leaderPort인듯


  * myid
    * echo x > myid (x는 id number)
    * dir drectory에 myid라는 이름의 file에 server의 unique한 id number를 저장
    * 이 in number는 configuration file가 match되어야 함

* Installing Broker

  * Zookeeper 설치 및 configuration 설정 이후 Kafka를 설치 하고 Broker Configuration을 설정하면 됨

* Broker Configuration

  * Kafka distribution(배포판)의 configuration 예제는 standalone server를 실행하기에는 충분하지만 대부분의 installations에서 부적절함
  * Kafka의 setup 및 tuning의 모든 측면을 제어하는 많은 configuration option이 존재
  * 많은 configuration options는 default value로 남겨둠 (use case에 따라 특정 option을 조정하는 것이 필요한 경우에 변경)

* Broker Configuration Options

  * broker.id
    * Integer형의 Kafka broker identifier(식별자)
    * default값은 0
    * Kafka cluster 내에서 unique 값이어야 함
    * broker id는 임의로 선택할 수 있으며 유지보수 작업에 필요할 경우 brokers 간에 id를 이동할 수 있음
    * 유지보수를 수행할 때 broker의 맵핑이 번거롭지 않도록 hosts에 대한 id number를 broker id로 사용하는 것이 좋은 guidline
  * port
    * 지정된 TCP port의 listener에서 Kafka가 시작됨
    * default 값은 9092
    * 사용가능한 어떠한 port로 설정 가능
    * lower than 1024일 경우 Kafka는 반드시 root로 시작되어야 하는데 이는 추천되지 않는 configuration
  * zookeeper.connect
    * broker metadata를 저장하는 Zookeeper의 위치
    * \${hostname}:\${port}/\${path}
    * hostname : Zookeeper server의 hostname 혹은 ip주소
    * port : server의 client port
    * path (optional) : 생략할 경우 root path가 사용되며 명시할 경우 Kafka cluster의 chroot environment로써 사용할 Zookeeper path
      * Chroot : 실행중인 프로세스와 차일드 프로세스 그룹에서 루트 디렉토리를 변경하는 작업
      * Chroot path가 존재하지 않는다면 broker가 시작될 때 만들어짐
      * 왜 Chroot Path를 사용하는가?
        * Kafka cluster에서 Chroot path를 사용하는 것은 권장됨
        * Zookeeper ensemble은 다른 Kafka clusters를 포함한 applications와 충돌없이 공유할 수 있도록 해줌 -> kafka 외 다른 application도 zookeeper를 사용중이면 chroot path를 추가해서 사용
        * 같은 ensemble에 속하는 mutliple Zookeeper servers를 열거하는 것이 가장 좋은 방법임
        * server failure일 때 kafka broker가 다른 Zookeeper ensemble의 member에 connect할 수 있도록 해줌
  * log.dirs
    * kafka는 모든 messages를 disk에 저장하고 log segments는 log.dirs에 명시된 directory에 저장됨
    * 형식은 local system의 comma-seperated의 path list
      * 둘 이상의 path가 명시되어 있다면 broker는 partition별로 저장하게 되는데 "least-used(최소 사용)" 방식을 이용하여, 새로운 partition이 추가되었을 때 저장할 path의 선정은 disk의 사용이 가장 적은 path가 아닌, 저장된 partition의 수가 가장 적은 path로 선정 됨 
  * num.recovery.threads.per.data.dir
    * log segments를 다루는 pool of threads
    * thread pool이 사용되는 경우
      * 정상적으로 시작되어 각 partition의 log segments를 열 때
      * 장애 발생 후 각 partition의 log segment를 추출 및 확인 할 때
      * 종료할 때 log segments를 닫을 때
    * default value는 1
    * threads는 시작 및 종료시에만 사용되므로 parallelize operations를 위해서는 더 많은 threads의 개수가 필요
    * log directory별로 설정되기 때문에, 전체 threads 개수는 num.recover.threads.per.data.dir설정값 * log.dirs개수 
  * auto.create.topics.enable
    * broker가 아래와 같은 상황일 때 자동으로 topic을 생성하도록 함
      * producer가 messages를 topic에 쓸 때
      * consumer가 topic으로부터 messages를 읽을 때
      * client가 topic에 대한 metadata를 요청할 때 
    * default value는 true
    * Kafka protocol로는 topic이 생성되어있는지를 확인할 수 없기 때문에 존재하지 않는 topic에 대해서 작업을 수행하면 자동으로 topic이 생성되어 버림 -> 수동 혹은 provisioning system에 의해 topic 생성을 명시적으로 하길 원하는 경우 해당 옵션은 false로 변경해야 함

* Topic Configuration Options

  * 이전에는 topic별로 재정의 할 수 있었지만 더이상 지원하지 않으며 administrative tools를 이용하여 지정해야하는 options 리스트
    * log.retention.hours.per.topic
    * log.retention.bytes.per.topic
    * log.segment.bytes.per.topic
  * num.partitions
    * 새로운 topic이 생성될 때 만들어지는 partition의 수
    * 주로 automatic topic creation(자동 topic 생성)이 가능한 경우 사용되는 옵션
    * default value는 1
    * topic에 대한 partitions의 개수는 늘릴 수는 있지만 줄일 수는 없음
    * 전체 cluster에 brokers가 추가될 때 message load의 균형을 위한 적절한 partition number를 설정하는 것은 매우 중요
    * topic에 대한 partitions number를 broker 수와 같게 하거나 여러배로 설정 -> partitions이 brokers에 균등하게 분배되어 message load가 균등하게 분배됨 -> 그러나 이는 mutiple topics를 갖는것으로도 message load의 균형을 맞출 수 있으므로 필수가 아님
    * How to Choose the Number of Partitions(Paritions의 개수를 정하는 방법)
      * topic에서 기대되는 처리량은 몇인지?
      * single partition으로부터 consuming할 때 기대하는 최대 throughput(처리량)이 몇인지?
        * 대부분의 one consumer는 하나의 partition으로부터 reading하고 database에 쓸 때 초당 50MB 이상을 처리하지 않는다면 단일 partition에서 consuming할 때의 최대 throughtput은 60MB
      * single partition에 대해 위와 같은 방법으로 producer의 최대 throughtput per second를 측정할 수 있지만, producers는 일반적으로 consumers에 비해 훨씬 빠르기 때문에 이 단계를 skip하여도 무관
      * keys를 기반으로 partition에 messages를 sending하는 경우, 나중에 partitions을 추가하는 것은 데이터 일관성이 깨질 수 있기 떄문에 매우 어려울 수 있으므로 현재가 아닌 미래의 사용량에 따라 throughput을 계산해야 함
      * broker에 배치할 partitions의 개수와 사용가능한 diskspace(디스크공간) 및 broker당 network bandwidth(대역폭)을 고려해야 함
      * 각 partition은 broker에서 memory와 other resources를 사용하고 leader election(리더 선출)에 시간이 증가하게 되므로 과하게 측정하는 것을 피해야 함
      * 요약 : paritions 개수의 증가는 balancing, throughtput이 증가하여 좋아보일 수 있으나 memory와 other resources 사용의 문제가 발생할 수 있으므로 disk space, network bandwidth등을 고려하여 적정선을 찾을 필요가 있음
        * topic당 target throughput이 있고, consumers당 기대되는 throughput이 있따면, target throughput per topics / expected throughput per consumers로 partitions의 개수를 구할 수 있음
        * 책에서는 partition당 6GB정도 늘어날 때 가장 만족스러웠다고 함
  * log.retention.hours/minutes/ms
    * kafka에 messages를 보관하는 기간
    * individual messages가 아닌 log segments 단위 에서 동작
    * default는 log.rentention.hours를 사용하며 value는 168, 즉 1주일
    * 해당 option은 2개 이상이 설정 되었을 때는 작은 단위가 우선적용 되므로 log.rentention.ms를 사용하는 것이 좋음
    * Retention By Time and Last Modified Times
      * time에 의해 retention을 수행될 떄 disk 상의 각 log segment file이 마지막으로 수정된 시간(mtime)을 기준으로 수행됨.
        * 정상적인 cluster opertions에서는 이 시간은 log segment file이 닫힐 때이며 파일의 마지막 message의 timestamp를 나타냄
        * 반면 administrative tools에 의한 brokers 간의 partitions이 이동될 때는, 이 시간이 정확하지 않게 되어 partitions에 대한 retention이 지나치게 길게 되어 버림
  * log.retention.bytes
    * 각 partition에 메세지를 보관하는 최대 bytes size
      * 특정 topic에 대해 8개의 partitions이 있는 경우, log.retention.bytes=1gb이면 하나의 topic에 보유되는 데이터양은 최대8gb
    * individual messages가 아닌 log segments 단위 에서 동작
    * Configuring Retention by Size and Time
      * log.rentention.hours/minutes/ms와 log.rentention.bytes options 모두가 지정되어 있는 경우 두 기준 중 하나의 기준이 충족되면 message가 삭제됨
  * log.segment.bytes
    * log segment(단일 로그)의 최대 크기
      * Kafka broker에 messages가 produced되면 해당 partition의 current log segment에 appended되는데, log segment의 size가 해당 option의 size에 도달하면 log segment를 닫고 새로운 log segment를 open
      * log segment가 닫히면 message 삭제가 고려될 수 있음
        * log.retention.hours/minutes/ms와 log.retention.bytes는 individual messages가 아닌 log segments 단위 에서 동작하므로 log segement가 닫힌 시점 이후부터 해당 기준을 충족해야 mesage의 삭제가 고려됨
        * 즉, log segment의 마지막 message까지 모두 log.retention.hours/minutes/ms와 log.retention.bytes기준에 만족될 때까지 해당 log segment는 삭제되지 않음
    * default value는 1gb
    * 해당 value가 작으면 file을 닫고 allocated(할당)하는 빈도가 높아져 disk writes의 효율이 낮아짐
    * topic이 낮은 produce rate(생산률)를 가진다면 해당 option값을 조정해야 함
      * topic이 하루에 100mb의 message만 수신하고 해당 option이 default value인 1gb로 설정되어 있다면 하나의 log segment를 채우는데 10일이 소요되며 log segment가 닫혀야만 message를 expired 할 수 있음. log.rentention.ms가 1주로 설정되어 있을 경우 log segment가 expired일 때 까지 실제로 17일 동안 messages가 유지되게 됨 (log segment가 10일동안 채워지고 닫히면 이 시점 이후부터 7일동안 유지되어야 하기 때문에 총 17일이 소요됨)
    * Retrieving Offsets by Timestamp (Timestamp를 이용한 Offsets 검색)
      * log segment의 size는 timestamp별로 offsets을 가져오는 동작에도 영향을 줌
      * 특정 timestamp에서 특정 partition에 대한 offsets을 요청하면 Kafka는 그 시점에 기록된 log segment file을 찾을 수 있음
      * 파일의 생성 및 최종 수정 시간을 이용하며 지정된 timestamp 이전에 만들어지고 지정된 timestamp 이후에 마지막으로 수정된 file들 중 찾음
      * log segment(or file name)의 시작 offset이 반환됨
  * log.segment.hours/minutes/ms
    * log segment를 닫아야하는 시간을 명시
    * default value는 존재하지 않으므로, 기본적으로는 log segment는 size별로만 닫음
    * log.segment.bytes와 log.segment.hours/minutes/ms option은 서로 exclusive 하지 않으므로 두개의 option 모두가 지정되어 있는 경우 두 기준 중 하나의 기준이 충족되면 message가 삭제됨
    * Disk Performane When Using Time-Based Segments (시간기반 log segment를 사용할 때의 disk 성능)
      * 여러 log segment가 동시에 닫힐 때dml disk performance에 대한 고려가 필요함
      * broker가 시작될 때 제한시간을 세는 clock이 시작되며, log segment가 size limit에 도달하지 않은 partitions이 많으면 이러한 low-volumn partitions들에 대해 동시에 log segment를 닫는 작업을 수행함
  * message.max.bytes
    * broker에 produced될 수 있는 최대 message의 size
    * default value는 1000000 혹은 1mb
    * 이 option의 value보다 큰 message를 보내려는 produer는 broker로 부터 error를 받으며 message는 accepted되지 않음
    * broker에 지정되는 모든 byte단위의 configuration options와 마찬가지로 이 옵션 역시 compressed message size를 의미하므로, producers는 compressed한 message를 produced한다면, 해당 value보다 훨씬 큰 size의 uncompressed value를 produced할 수 있음
    * 해당 option value가 증가하면 network connection 및 requests를 처리하는 broker threads가 각 요청에 대해 오랫동안 작동하며, disk writes의 size가 증가하여 I/O throughput에도 영향을 줌
    * Cordinating Message Size Configuration
      * Kafka broker의 message size configuration는 broker의 replica.fetch.max.bytes와 consumer의 fetch.message.max.bytes configuration을 고려하여 설정해야 함
        * replica.fetch.max.bytes : 각 partition에 대해 messages fetch(messages 일괄처리)를 시도할 bytes
      * 만약 fetch.message.max.bytes가 message.max.bytes보다 작으면, consumers는 큰 size의 messages를 consumed할 수 없게 되며, consumer가 stuck되고 proceed할 수 없게됨

* Hardware selection

  * Kafka 자체는 hardware configuration(구성)에 대한 엄격한 요구사항이 없으며 어떠한 system에서도 문제없이 실행됨
  * Kafka 전반적인 성능에 영향을 줄 수 있는 몇가지 요소 (performance type)
    * disk throughput (디스크 처리량), disk capacity (디스크 용량), memory (메모리), networking (네트워킹), CPU 
    * system environment에서 가장 중요한 performance type을 결정한 후 해당 performance type을 향상시키기 위한 최적의 hardward configuration을 선택하면 됨
  * Disk Throughput
    * producer clients의 성능은 log segments가 저장되는 broker disk의 throughput에 가장 직접적인 영향을 받음
    * Kafka messages는 produced될 때 local storage에 committed되어야 하며, 대부분의 clients는 전송이 성공으로 간주되기전에 하나이상의 broker가 messages가 committed되었음을 확인하기 전까지는 기다림 -> disk writes 속도가 빨라지면 produce latency(지연)이 줄어듦
    * traditional spinning hard drives(HDD) VS solid-state disks (SSD)
      * SSD
        * lower seek and access times(탐색 및 액세스 시간의 감소)
      * HDD
        * more economical(경제적)
        * more capacity per unit(단위당 더 많은 용량)
        * HDD의 성능을 향상시키는 방법
          * broker에서 multiple data directories를 사용하거나 RAID(Redundant Array of Independent Disk)으로 drives를 설정
    * drive controller의 품질 뿐 아니라 특정 drive technology(e.g. serial attached storage or serial ATA)도 throughput에 영향을 줌
  * Disk Capacity
    * disk capacity를 결정하는 요인들
      * 언제든지 보유되어야 하는 messages의 수에 따라 결정됨
        * broker가 하루에 받는 traffic에 retention기간을 곱하여 반드시 확보되어야 하는 disk 공간을 정할 수 있음
        * 실제 disk capacity는 위의 disk size에 10% overhead가 필요 (다른 files과 시간이 지남에 따라 발생하는 traffic이나 growth의 변화에 대비하여)
      * cluster에 선택된 replication strategy(복제 정책)에 의해서도 결정됨 (6장에서 다룸)
    * 하나의 clsuter에 대한 전체 traffic은 topic당 mutliple partitions을 가짐으로써 균형을 맞출 수 있음
    * single broker의 density(밀집도)가 충분하지 않으면 brokers를 추가함으로써 사용가능한 capacity를 늘릴 수 있음
  * Memory
    * 일반적으로 consumer는 producer보다 뒤쳐짐
    * page cache
      * page cache는 consumer의 performance 향상을 도움
      * consumer는 system의 page cache에 optimally stored 되어있는 messages를 읽으는데, 이는 disk에서 message를 읽는것보다 훨씬 빠름
    * systme에서 page cache를 위한 memory를 더 많이 사용할 수 있게 된다면 consumer clients의 performance가 향상됨
    * System에 Kafka를 다른 중요한 applicatios과 함께 배치하는 것은 권장되지 않음
      * Kafka와 다른 applications이 page cache를 공유해야 하므로, Kafka의 consumer performance가 낮아짐
  * Networking
    * available network throughput : Kafka가 처리할 수 있는 최대 traffic양
    * Networking에 영향을 주는 요소
      * cluster의 size를 조정하기 위한 disk storage
      * the inherent imbalance between inbound and outbound  network usage (inbound와 outbound network usage의 고유한 불균형)
        * Kafka가 multiple consumers를 지원하면서 만들어지는 outbund network이inbound network보다 훨씬 큼
        * 이로인해 network throughput을 결정하기 복잡함
    * network interface가 saturated(포화상태)가 되면, cluster repliacation이 실패를 흔하게 하여 cluster가 vulnerable state(결함적인 상태)가 될 수 있음
  * CPU
    * Processing power는 disk와 memory에 비해 덜 중요하지만 borker의 전체 performance에 어느정도 영향을 미침
    * CPU는 message 압축과 관련하여 사용됨 (이것이 hardware를 선택하는 주요 요소가 되어서는 안 됨)
    * Compress Messages in Kafka
      * clients는 messages를 compress하여 network와 disk usage를 최적화해야함
      * messages를 compress하는 경우 개별 message의 checksum을 확인하고 offses를 할당하기 위해 Kafka broker는 모든 message batches를 decompress해야 함. 또한 이후 message bathe를 disk에 저장하기 위해 recompress해야 함

* Kafka in the Cloud

  * Select the correct instance configuration to use in Amazon (아마존에서의 instance configuration의 올바른 선정 절차)
    1. producers에서 요구되는 damount of rentention을 바탕으로 disk를 선정
       * low latency(대기시간)이 필요한 경우 SSD를 이용
       * 이외에는 ephemeral storage(임시 저장소 e.g. AWS Elastic Block Store)로도 충분
    2. 적절한 performance의 CPU 및 memory options을 결정
    3. AWS instance types의 결정
       * 대부분 m4혹은 r3을 이용
       * m4와 r3의 장점을 혼합하려면 상당히 비싼 i2 또는 d2를 이용해야함
       * m4
         * greater rentention periods (더 많은 보존기간)
         * using elastic block storage -> lower the disk throughput (낮은 디스크 처리량)
       * r3
         * using SSD -> better disk throughput  (좋은 디스크 처리량)
         * limited retention periods (제한된 데이터 유지량)

* Kafka Clusters

  * single Kafka server로도 잘 작동하지만 아래 사진처럼 하나의 cluster에 multiple brokers를 configured(구성)하는 경우 많은 이점이 있음

    ![figure2-2_A](C:\Users\jungsol\Documents\kafka\pic\figure2-2_A.png)

    * multiple brokers 구성의 이점
      * multiple server에 부하를 확장할 수 있음
      * replication을 이용하여 data 손실을 방지가능
      * replication을 이용하여 clients에 대한 maintaining availability를 유지하면서, kafka나 underlying systems의 maintenance(유지보수) 작업을 할 수 있게 함

  * How Many Brokers?

    * Kafka cluster의 크기를 결정하는 몇가지 요인
      1. How much disk capacity is required for retaining messages and how much storage is available on a single broker (messages를 유지하는데 필요한 disk capacity, singel broker에서 사용 가능한 storage 용량)
         * broker의 개수 = required disk capacity * replication number / disk capacity of single broker
           * replication을 할 경우 storage 요구사항이 최소 100% 증가
      2. capacity of the cluster to handle requests (cluster가 요청을 처리할 수 있는 용량)
         * multiple consumers이거나 rentention period of data(데이터 보관주기)동안 traffic이 일정하지 않을 경우 (peak times 동안의 traffic 급증이 있을 경우) client가 traffic을 처리할 수 있는지, 즉 capacity of the network interfaces를 고려해야함
           * single borker의 network interface가 peak 시간대에 80%정도 사용된다면 2개의 consumers는 2개의 brokers가 필요 (2개의 brokers가 없다면 peak traffic을 따라잡을 수 없음)
           * If replication is being used in cluster, this is an additiional consumer of the data that must be taken into account (cluster에서 replication이 사용되는 경우 data consumer의 추가를 고려해야 함)
           * 적은 disk throughput 혹은 적은 사용가능 system memory로 인한 성능 문제를 해결하기 위하여 cluster 내의 brokers를 추가하는 것이 좋음

  * Broker Configuration

    * multiple kafka brokers가 single cluster에 합류할 수 있는 2가지 요구사항
      1. 모든 brokers는 같은 zookeeper.connect paramter에 대해 같은 configuration을 가져야 함
      2. 모든 brokers는 unique한 broker.id parameter 값을 가져야 함
         * 2개의 brokers가 동일한 broker.id로 같은 cluster에 join하려고 하면 2번째 broker는 error log와 함께 시작하지 못함

  * OS Tunning

    * 대부분의 Linux 배포판은 대부분의 applications에서 잘 작동하게 하는 kerneltuning parameters(커널 튜닝 매개변수)에 대한 좋은 configuration(구성)을 가지고 있지만, kafka broker의 performance를 향상시키는 몇가지 변경사항이 존재

      1. virtual momory (가상 메모리)
      2. networking sybsystem (네트워킹 하위시스템)
      3. log segment를 저장하는데 사용되는 disk mount point

      * 위의 parameters는 일반적으로 /etc/sysctl.conf 파일에서 configured (자세한 kernel configuration 조정에 대한 내용은 Linux 배포판의 documenntation을 참고)

    * Virtual memory

    * Disk

    * Networking

* Production Concerns (reliable messaging service를 설정하는데 도움이 되는 요소들)

  * Garbage Collector Options

    * application에 대한 Java garbage collection ptions을 tuning하는 것은 application의 memory 자세한 사용 방법과 상당한 관찰과 시행착오가 필요로 했음 -> Java 7 이후 Garbage First(G1) garbage collector 도입 이후로 달라짐

    * Garbage First(G1) garbage collector

      * 자동으로 다양한 workloads(작업 부하)에 맞게 조정되고 application의 lifetime(수명주기)동안 일관된 garbage collection을 위한 pause times(중단 시간)을 제공하도록 설계 됨
      * heap을 smaller zones으로 분리하여 garbage collection을 하므로 large heap을 쉽게 처리함
      * G1 garbage collector의 performance를 조정하는 2가지 configuration options 
        1. MaxGCPauseMillis
           * garbage collection cycle(주기)의 preferred pause time을 지정
           * 고정된 최대값이 아니며 필요할 경우 해당 time을 넘길 수 있음
           * default value는 200 milliseconds
             * G1은 GC cycles의 빈도와 각 cycle에서 collected되는 zones의 수를 계획하여, 각 cycle이 200ms 정도 걸리도록 함
        2. InitiatingHeapOccupancyPercent
           * G1이 garbage collection cycle을 시작하기 전에 사용중인 전체 heap의 백분율을 지정
           * default value는 45
             * G1은 heap의 45%가 사용되기 전까지는 garbage collection cycle을 시자하지 않음
           * This includes both the new (Eden) and old zone usage in total. (new (Eden)과 old zone usage가 모두 포함됨)

    * Kafka broker는 heap memory를 사용하고 garbage objects를 생성하는데 굉장히 효율적이므로 gc option을 낮게 사용할 수 있음

      * (책에서 권고하는 gc tunning option)
        * 64gb memory sever에서 5gb heap에 Kafka를 running
        * MaxGCPauseMillis는 20ms
        * InitiatingHeapOccupancyPercent는 35

    * Kafka 시작 script는 G1 collector를 사용하지 않고 parallel new and concurrent mark and sweep garbage collection을 default로 사용

    * Garbage Collector의 변경은 environment variables로 쉽게 가능

      ```
      # export JAVA_HOME=/usr/java/jdk1.8.0_51
      # export KAFKA_JVM_PERFORMANCE_OPTS="-server -XX:+UseG1GC
      -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35
      -XX:+DisableExplicitGC -Djava.awt.headless=true"
      # /usr/local/kafka/bin/kafka-server-start.sh -daemon
      /usr/local/kafka/config/server.properties
      #
      ```

  * Datacenter Layout

    * physical locaion of the Kafka brokers(Kafka brokers의 물리적 위치)

      * cluster가 부분적 혹은 전체적으로 단시간동안 사용할 수 없는 것은 development systems에 심각한 영향을 주지 않기 때문에, development systems의 경우, datacenter내의 Kafka brokers의 물리적 위치에 관심이 없음. 
      * production traffic을 serving할 때 downtime은 users에게 services가 손실되든 users가 수행하는 작업에 대한 telemetry(원격측정)이 손실되든 dollars lost를 의미
      * Kafka cluster내의 replication configuration이 중요해지면, Kafka brokers의 datacenter의 racks에 있는 물리적 위치도 중요해짐
      * Kafka를 deploying(배치)하기 전에 해결되지 않으면 servers의 이동이 필요하여 비싼 유지보수가 들 수 있음

    * No Rack-awareness

      * Kafka broker는 새로운 partitions에 할당될 때 rack-awareness(랙 인식 기능)이 없음
        * This means that it cannot take into account that two brokers may be located in the same physical rack, or in the same availability zone (if running in a cloud service like AWS), and therefore can easily assign all replicas for a partition to brokers that share the same power and network connections in the same rack.  (2개의 brokers가 같은 physical rack 또는 (AWS와 같은 cloud service에서 running되는 경우) 같은 availability zone(가용성 영역)에 위치할 수 없으므로, 하나의 partition에 대한 모든 replica를 같은 rack안에서 같은 power(전원)과 network를 공유하는 brokers에 쉽게 할당할 수 있음)
      * rack에 장애가 발생하면
        * partitions는 offline이 되어 clients가 접근할 수 없게 됨
        * unclean leader election으로 인해 data 손실이 발생할 수 있음 

    * Recommended (장애대비를 위한 권고사항)

      1. 각각의 Kafka broker는 다른 rack에 installed
      2. 최소한 power와 network같은 infrastructure services(인프라 서비스)에 대한 single points of failure(단일 장애 지점)을 공유하지 않아야 함
         * brokers들이 실행되는 servers를 dual power와 dual network를 이용하여 배포한다는 것을 의미
           * dual power : 2개의 다른 circuits(회로)에 연결
           * dual network : servers의 연결된 interface를 통한 원활한 failover(장애극복)
           * dual connections은 brokers를 완진히 분리된 racks에 배치하는데 얻는 이점을 얻을 수 있음

      * offline 상태의 rack 또는 cabinet에서 수행해야 하는 servers 이동 혹은 power connection 재배선과 같은 physical maintenance를 수행해야 할 수도 있음 -> offline상태에서 작업을 하더라도 장애가 발생하지 않도록 1번 혹은 2번의 대처 필요

  * Colocating Applications on Zookeeper (Zookeeper에 applications을 배치)

    * Zookeeper for Kafka (Kafka를 위한 Zookeeper)
      * Kafka는 Zookeeper를 사용하여 brokers, topics, partitions에 대한 metadata 정보를 저장
      * Zookeeper는 cnsumer groups의 member 혹은 Kafka cluster 자체의 변경사항에 대해서만 Writing 작업을 수행
      * traffic의 양이 최소화되며 single Kafka cluster에 대해 전용 Zookeeper ensemble을 사용하는 것은 옳지 않음(not justify)
      * 많은 deployments(배포)에서는 multiple Kafka clusters에 대해 각 Kafka cluster에 대해 chroot Zookeeper path를 사용 하여 sigle Zookeeper ensemble을 사용
    * Kafka Consumers and Zookeeper
      * Old Consumer (Kafka 0.9.0.0 이전)
        * 동일 group의 consumers간에 failover를 가능하게 하기 위해, consumer group, consumed topics, periodically commit offsets for each consumed partition과 같은 정보를 Zookeeper를 사용하여 저장
      * New Consumer(Kafka 0.9.0.0 이후)
        * Kafka brokers 내에서 직접적으로 metadatas를 관리
        * zookeeper.connect -> bootstrap.server
    * Old Consumer VS New Consumer
      * Consumer는 committing offsets할 때 Zookeeper과 Kafka중 무엇을 사용할지와 commits interval을 configuration으로 선택할 수 있음
        * Zookeeper를 선택하면 Old Consumer
        * Kafka를 선택하면 New Consumer
      * Old Consumer의 한계
        * 각각의 consumer는 매 interval마다 consuming하는 모든 partition에 대해  Zookeeper에 write 작업을 해야 함
          * interval은 1분이 합리적임 (consumer가 failure한 경우, 이 interval은 consumer group이 duplicate messages를 읽는 기간이기 때문)
        * 이러한 commit(write) 작업은 많은 cnsumers가 있는 cluster의 Zookeeper에 상당한 양의 traffic일 수 있음 -> Zookeeper ensemble이 traffic을 처리하지 못할 경우 더 긴 commit interval을 사용해야 함 -> Zokeeper에 대한 dependency(종속성) 발생
      * Old Consumer에는 위와같은 한계가 있으므로, 최신 Kafka libraries를 사용하여 Zookeeper에 대한 dependency를 제거하고, Kafka를 사용하여 offsets을 committing하는 것이 권장사항
    * Is recommended to colocate applications on Zookeeper?
      * multiple Kafka clusters에 대해 single Zookeeper ensemble을 사용하는 것 외에는 다른 applications과 Zookeeper ensemble을 공유하는 것은 권장되지 않음
      * Colcating Applications on Zookeeper가 권장되지 않는 이유
        * Kafka는 Zookeeper의 latency(지연시간)과 timeout(시간초과)에 민감
        * Zookeeper ensemble과 brokers의 통신이 중단되면 brokers는 unpredictably(예상치 못하게) 행동을 함
          * 여러대의 brokers가 동시에 offline이 될 수 있으며 Zookeeper와의 연결이 끊어지면 여러개의 partitions가 offline이 됨
        * 제어된 broker의 종료를 수행할 때와 같이 interrupion가 경과한 후에 subtle(미묘한) errors로 인해 cluster controller에 stress(압박)이 될 수 있음
      * Zookeeper ensemble의 heavy usage(과도한 사용)과 improper opertaions(부적절한 동작)으로 인해 stress(압박)을 줄 수 있는 other applications는 자체 ensemble로 분리해야 함



















