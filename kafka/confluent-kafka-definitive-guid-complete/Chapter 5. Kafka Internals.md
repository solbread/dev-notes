## Chapter 5. Kafka Internals

### Introduce

* Kafka를 production or write에 사용하는 application을 run하기 위해서 Kafka의 internals을 이해하는 것은 엄격하게 필요하지는 않음
* 그러나 Kafka의 works(작동방식)을 아는것은 troubleshooting을 하거나 Kafka의 왜 그런방식으로 행동하는지 이해하려고 할 때 상황을 알 수 있음
* 모든 single implementation detail(구현 세부사항)과 design decision(디자인 결정)을 다루는것은 책의 범위를 벗어나므로, 이 책에서는 실무자에게 관련된 아래 3가지 주제에 중점을 둠
  * How Kafka replication works (Kafka 복제의 작동 방식)
  * How Kafka handles requests from prducers and consumrs (Kafka가 생산자와 소비자의 요청을 처리하는 방법)
  * How Kafka handles storage such as file format and indexes (Kafka가 파일 형식 및 색인과 같은 저장소를 처리하는 방법)
* 위의 주제들을 이해하면 Kafka tuning할 때 무작위로 fidding(조작)하지 않고 정밀한 의도를 가지고 tuning할 수 있어 유용함




### Cluster Membership

* Kafka는 Apache Zookeeper를 사용하여 현재 cluster의 members인 list of brokers를 maintain(유지/관리)

* 모든 broker에는 broker configuration file에 설정되거나 자동으로 생성된 unique identifier가 있음

  * broker process가 시작할 때 마다, ephemeral node(임시 노드)를 만들어서 Zookeeper에 ID로 등록

* 다른 Kafka components(구성요소)는 broker가 등록되어 있는 Zokeeper의 /borkers/ids path에 subscribe되어 broker가 add되거나 remove될 때 nofify받음

  * broker의 stop, network partition의 결과, 긴 시작의 garbagecollection 일시중지 등의 이유로 broker가 Zookeeper와의 연결이 끊어지면,
    * broker가 생성될 때 만들어진 ephemeral node(임시 노드)는 자동으로 Zookeeper에서 삭제됨
    * brokers의 list를 watch하고 있는 Kafka components는 broker가 없어졌음을 notify받음

* broker가 중지될 때 broker를 나타내는 node가 사라지더라도, broker ID는 여전히 data structures에 존재

  * 예를들어, 각 topic의 list of replicas는 replica의 broker ID's가 포함됨


  * 이런 방식으로 broker를 완전히 lose하고, 기존의 broker의 ID로 new broker를 시작하면, 잃어버린 broker와 같은 partitions와 topics이 assign되어 cluster의 missing broker자리에 즉시 join됨



### The Controller

* definition
  * Kafka broker 중 하나
  * 일반적인 broker functionality(기능) 외에도 partition leader를 선출
* controller의 선출 과정
  * cluster에서 시작하는 first broker를 Zookeeper의 ephemeral node(임시 노드)가 ./controller를 call하여 controller로 만듦
  * other broker가 시작될때도, controller node로 만들려고 하지만 "node already exists" exception을 받으며, cluster에 controller node가 이미 있음을 인지하게 됨
* Zookeeper watch
  * brokers는 controller node에 대한 Zookeeper watch를 생성하여, controller node의 변경사항을 알림받음
* controller의 재선출
  * cluster는 한 번에 하나의 controller만 가질 수 있음
  * controller broker가 stop되거나 Zookeeper와의 connectivity를 lose하면, ephemeral node가 사라짐
  * cluster에 있는 다른 broker는 Zookeeper watch를 통하여 controller가 사라졌음을 notify받고, Zokeeper내에 controller node를 생성하려고 시도함
    * Zookeeper내의 first node가 new controller로 생성되며, 다른 nodes는 "node already exists" exception을 받고 new controller node에 대한 Zookeeper watch를 다시 생성
* epoch number (시대 번호)
  * controller가 elect(선출)될 때 마다, controller는 Zookeeper conditional increment operation(조건부 증가 연산)을 통해 새로운 higher controller epoch number(새로운 controller의 더 높은 epoch number)를 receive
  * brokers는 현재 controller epoch를 알고 있으며, older number의 controller부터 message를 받으면, 무시함
* elect leader
  * controller relevant Zookeeper path를 watch함으로써 broker가 cluster를 떠났음을 알리고, 떠난 broker가 leader로 있었던 모든 partitions에 new leader가 필요하다는 것을 알게 됨
  * new leader를 필요로하는 모든 partitions을 검사하고, partition의 replica 목록에 있는 broker 중 next replica broker로 new leader를 결정하여 partitions의 new leaders 이거나 existing followers인 모든 brokers에게 partition의 new leader와 followers 정보를 포함하는 request를 send
  * new leader는 clinets의 producer와 consumer request에 serve하는 것을 시작해야하며, followers는 new leader로부터 message를 replicate하기 시해야 함
* broker ID
  * cluster에 broker가 join했다는 것을 controller가 알게되면, broker ID를 사용하여 이 broker에 replica가 존재하는지를 확인
  * 만약 broker에 replica가 있으면, controller는 new brokers와 existing brokers에 변화를 notify하고, new broker의 replicas는 existing leaders로부터 message replicate를 시작
* Summary
  * Kafka는 Zookeeper의 ephemeral node 기능을 사용하여 controller를 선출
  * cluster에 node가 join 및 leave할 때 contoller에 알림
  * controller는 nodes가 cluster에 join하고 leave할 때 마다 partitions과 replicas 사이에서 leaders를 선출
  * controller는 epoch number를 사용하여 2개의 node가 각각 current controller라고 생각하는 "split brain"을 prevent(방지)



### Replication

* Introduce
  * Replication은 Kafka architecture의 핵심
  * Kafka documentation의 첫번째 문장에서 "a distributed, partitioned, replicated, commit log service"(분산되고, 나뉘어져있고, 복제되며는 commit log service)라고 설명함
* Replication의 효과
  * Kafka의 개별 nodes가 inevitably(필연적으로) fail할 때, availability(가용성)와 durability(내구성)를 보장
* Kafka는 topics별로 organize되어 있으며, 각 topic은 partition으로 나눠져있고, 각 partition은 multiple replicas를 가짐
  * replicas는 brokers에 저장되며, 각 broker는 일반적으로 서로 다른 topics 및 partitions에 속한 수백 또는 수천개의 replica를 저장
* replicas의 2가지 유형 : Leader replica, Follower replica
  1. Leader replica
     * 각 partition에는 leader로 지정된 single replica가 있음
     * consistency(일관성)을 보장하기 위해 모든 produce와 consume requests는 leader를 통하여 진행됨
     * follower replicas 중 어떤 replica가 leader와 up-to-date(최신)인지 파악
  2. Follower replica
     * 각 partition의 leader가 아닌 모든 replicas
     * followers는 client request를 serve(처리)하지 않음
     * 역할
       * 단지 leader로부터 messages를 replicate
       * stay up-to-date with the most recent message the leader has (leader가 가지는 가장 최근의 message를 up-to-date(최신으로) stay(유지)함)
     * partiton의 leader replica가 crash되는 경우, follower replicas 중 하나가 promote(승격)되어 partition의 new leader가 됨
* leader replica와 follower replicas의 sync
  * followers는 messages가 도착하면, leader로부터 모든 messages를 replicate하여 up-to-date(최신상태)로 유지하려고 하지만, network congestion으로 인해 replication 속도가 느려지거나, broker의 crash 등의 다양한 이유로 sync(동기화상태)를 유지하지 못할 수 있음
    * 문제가 생긴 broker의 모든 replicas는 broker가 다시 시작되어 replication을 다시 시작할 수 있을 때 까지 뒤떨어짐
  * leader replica와 follower replicas간의 sync를 유지하는 방법
    * follower replicas는 leader에게 consumers가 messages를 consume하기 위해 보내는 requests와 동일한 type의 requests인 Fetch requests를 send
    * 이러한 Fetch requests에 response하여, leader는 replicas에게 messages를 send
    * 이러한 Fetch requests에는 replica가 다음번에 receive하려고 하는 message의 offset이 포함되며, 항상 순차적임
    * Example
      * replica는 message1, message2, message3을 request하고, 모든 이전 messages를 get하기 전에 messge4를 request하지 않음.
        * replica가 message4를 requests하면, replica가 message3까지 모든 messages를 수신하였음을 leader가 알 수 있음
      * leader는 각 replica의 request되는 마지막 offset을 보고, 각 replica가 얼마나 멀리있는지를 알 수 있음
* in-sync replicas
  * replica가 message를 10초 이상 하지 않거나, messages를 요청했지만 recent message를 x초 이상 caught up(포착)하지 않은 경우, replica는 out of sync(동기화 되지 않은 상태)로 간주됨
    * 여기서 x초는 allowed lag(허용 지연)으로 replica.lag.time.max.ms configuration option에 명시되어있는 시간을 의미
  * replica가 leader를 따라잡지 못하면, 모든 messages를 포함하고 있지 않으므로 failure(장애)가 발생시 new leader가 될 수 없음
  * in-sync replicas
    * latest messages를 지속적으로 requests하는 replicas
    * existing leader(기존 리더)가 fail되었을 때, in-sync replicas만 partition의 leader로 선출 될 수 있음
* allowed lag
  * replica.lag.time.max.ms
    * broker configuration option
    * default value = 10000 (10s)
    * follower가 동기화되지 않은 것으로 간주되기 전에 inactive(비활성화)되거나 behind(뒤로밀리는)할 수 있는 time을 제어하는 configuraton parameter
  * 이러한 allowed lag(허용 지연)은 leader election 중에 clinet behavior와 data retention에 영향을 미침
  * *reliability guarantees에 관한 내용은 chapter 6에서 다룸*
* preferred leader
  * 각 partition에는 current leader외에도 preferred leader가 있음
  * 해당 topic이 처음 생성되었을 때의 leader replica
  * It is preferred because when partitions are first created, the leaders are balanced between brokers (partitions이 처음 생성될 때, leaders가 brokers 사이에서 균형을 이루기때문에 preferred)
    * As a result, we expect that when the preferred leader is indeed the leader for all partitions in the cluster, load will be evenly balanced between brokers. (preferred leader가 cluster내의 모든 partitions의 leader가 되면 brokers간의 load(부하)가 균형을 이룰 것이라고 expect)
  * auto.leader.rebalance.enable
    * broker configuration option
    * default value = true
    * true일 경우, preferred leader replica가 currnet leader가 아니지만 in-sync이면, preferred leader를 current leader로 만들기 위하여 leader election을 trigger
* Finding the Preferred Leaders
  * current preferred leader를 identify하는 가장 좋은 방법은, partition의 replicas 목록을 확인하는 것
  * kafka-topics.sh tool에서 partitions과 replicas의 세부사항을 확인 가능 *(chapter10에서 자세하게 다룸)*
  * replicas 목록의 first replica는 항상 preferred leaser
    * current leader가 누구인지에 관계 없으며, 심지어는 replicas가 replica reassignment tool을 사용하여 다른 brokers에 reassign되었더라도 성립함
  * replicas를 manually(수동으로) reassign하는 경우, 우선 지정한 replica가 preferred replica
  * 몇몇 brokers에 leaders가 overloading되는것을 막기 위해서 preferred replica를 다양한 brokers에 분산시켜야 함



### Request Processing

* Introduce

  * Kafka broker가 수행하는 대부분의 작업은 clients, partition replica, controller에서 partition leader에게 전송된 requests
  * Kafka는 requests의 format과 request가 성공하거나 오류가 발생했을 경우 brokers가 response하는 방식에 대해 specify(지정)하는  binary protocol(over TCP)를 가짐
    * 이 binary protocol의 내부에 대해서는 kafka documentation에 자세히 설명되어있기 때문에 책에서는 다루지 않음

* client와 broker 사이의 request

  * clients는 항상 connections을 initiate하고, requests를 send
  * brokers는 requests를 처리하고 repond
  * 특정 client에서 broker로 전송된 모든 requests는 수신된 순서대로 처리됨이 보장
    * Kafka는 message queue로 behave(작동)하고 message가 순서대로 message queue에 저장되기 때문

* request의 standard header에 포함되는 내용

  * Request type (=API Key)
  * Request version
    * broker가 다양한 version의 clients를 handle하며, 적절하게 response
  * Correlation ID 
    * Correlation : 연관성, 상관관계
    * request를 unique하게 identify(식별)하고, reponse와 error logs에 appear하는 number
    * 이 ID는 troubleshooting에 사용됨
  * Client ID
    * request를 보낸 application을 identify(식별)하는데 사용됨

* Broker가 requests를 처리하는 방법

  1. 각 broker가 listen하는 port에 대해, broker는 connection을 생성하고 이를 처리하기 위해 processor thread로 넘겨주는 aceeptor thread를 run
     * preocessor threads(or network threads)의 개수는 configurable(구성가능)
  2. network threads는 client connections으로부터 requests를 가져와서, reqeust queue에 place하고, response queue에서 response를 pick up하여 clinets에 다시 send![figure5-1](C:\Users\jungsol\Documents\kafka\pic\figure5-1.PNG)
     * requests가 request queue에 plcae되면, IO threads는 requests를 pick up하여 처리

* request의 common type  : Produce requests, Fetch requests

  1. Produce requests
     * producer가 보내고, clients가 Kafka broker에 쓰는 messages
  2. Fetch requests
     * Kafka brokers로부터 messages를 읽을 때, consumers와 follower replicas가 보냄

  * *여기에 추가적로 Meatadata requests도*

* Request를 받을 수 있는 Broker

  * prduce requests와 fetch reqeusts는 request와 관련된 partition의 leader replica가 포함된 broeker로 send되어야 함
    * "Not a Leader Partition"
      * broker가 특정 partition에 대한 produce request를 받고, 이 partition의 leader는 다른 broker인 경우에는 prduce request를 보낸 client에 해당 error 발생
      * 특정 partition에 대한 fetch request를 받은 broker가 해당 partition의 leader가 아닐 경우에도, fetch request를 보낸 client에 해당 error 발생
  * client는 requests를 어디로 보내야 할지를 어떻게 알 수 있는가?
    * Kafka clients는 client가 관심있는 topics list가 포함된 metadata request라고 불리는 another request type을 사용

* Metadata requests

  * metatdata request에 server는 topics에 존재하는 partitions, 각 partition의 replicas, replica의 leader를 reponse함

  * 모든 brokers는 metadata information이 포함된 metadata cache를 가지므로 metadata request는 모든 broker로 보낼 수 있음

  * clients는 metadata request로 받은 information을 cache하고, 이를 이용하여 각 partition에 대한 올바른 borker에게 produce requests 혹은 fetch requests

  * metadata request를 하는 경우

    1. 특정 interval을 가지고 주기적으로 metadata request

       ![figure5-2](C:\Users\jungsol\Documents\kafka\pic\figure5-2.PNG)

       * broker가 추가되거나 일부 replicas가 new broker로 이동되는 등의 이유로 topic의 metatdata가 변경될 수 있으므로, 변경되었는지를 확인하기 위하여 가끔씩 metadata request를 보내서 metadata information을 refresh
       *  metadata.max.age.ms
         *  producer configuration option
         * default value = 300000 (=300s =5min)
         * metadata informataion을 refresh하는 주기를 조절하는 configuration parameter

    2. clinet가 requests 중 하나에 "Not a Leader" error를 receive했을 경우

       * 이 경우에는 client가 오래된 metadata information을 사용하고 있는 것이고, requests를 잘못된 broker에게 send했음을 의미하므로 request를 다시 시도하기 전에 metatdata를 refreh


* Produce Requests

  1. partition의 lead replica를 포함한 broker가 이 partition에 대한 produce requests를 받으면 아래의 몇가지 vaildations(유효성검사)를 실행함으로써 시작
     * data를 send하는 user는 topic에 대한 wrtie privileges가 있는가?
     * request에 지정된 acks의 number가 vaild한가?
       * 오직 0, 1, all만 value값으로 가질 수 있음 
     * acks가 all로 set된 경우 안전하게 message를 write할 수 있는 충분한 in-sync replicas가 있는가?
       * in-sync replicas의 개수가 configurable number보다 작은 경우, broker는 new messages를 refuse하도록configure할 수 있음
  2. new messages를 local disk에 write
     * Linux상에서는 messages는 filesystem cache에 write되며, 언제 disk에 write되는 지에 대한 guarantee(보증)이 없음
     * kafka는 data가 disk에 저장될때 까지 기다리지 않고, message durability(내구성)을 위해 replication에 의존
  3. message가 partiton의 leader에 쓰여지면 broker는 ack configuration을 examin(검사)하여 이 값에 따라 action을 취하며 respond
     * acks가 0이나 1이면 broker는 즉시 respond
     * acks이 all이면 leader가 follower replicas가 message를 replicate한 것을 확인 하여 client에게 response를 send할 때 까지 purgatory라고 불리는 buffer에 request를 저장

* Fetch Requests

  * Brkers는 produce requests와 비슷한 방식으로 fetch requests를 처리

  1. client가 broker에게 topics, partitions, offsets list에서 messages를 보내달라고 요청
     * e.g) test topic의 partition 0의 53 offset에서 시작하는 messages와 test topic의 partiton3의 64 offset에서 시작하는 messages를 보내달라고 요청
  2. requests를 leader replica가 수신하며 requests의 vaildation을 확인
     * reqeusts는 requests에 지정된 partitions의 leader에 도착해야하며, client는 fetch reqeusts를 정확하게 routing하기 위해 metadata requests가 필요
     * leader는 reqeust를 받으면 request가 vaild한지 먼저 확인하고, 특정 partition에 대한 offset이 존재하는지를 확인하여 reqeust가 vaild한지를 확인
       * client가 message가 너무 오래되어 partition에서 삭제되었거나, 아직 존재하지 않는 offset을 request하면 broker는 error로 respond
  3. reqeust가 vaild하다면 broker는 requests에서 client에 의해 upper boundary limit까지 partition에서 messages를 read하고 client에게 messages를 send
     * Kafka는 clintes에게 messages를 send하기 위해 zero-copy method를 사용
       * zero-copy method
         * Kafka는 Linux의 file(or more likely, the Linux filesystem cache)에서 intermediate buffers(중간 버퍼)없이 network channel에 직접 message를 보냄
         * data가 clients에 전송되기 전에 local cache에 저장되는 database와 다름
         * 이 techinque는 memory에 bytes를 copy하고 buffer를 manage하는 overhead를 제거하여 결과적으로 performance가 향상됨

  * clients는 broker가 각 partition에 대해 return할 수 있는 data의 양에 대한 boundary(경계)가 필요

    - upper boundary (상한선)

      - broker가 clients에게 각 partition에 대해 return할 수 있는 data 양의 최대값
      - The limit is important because clients need to allocate memory that will hold the response sent back from the broker (clents는 broker로부터 전송되는 response를 hold(가지고있음)할 memory를 allocate해야 하므로 limit은 중요함)
      - limit이 없다면 brokers는 clients가 memory를 다 소진할 만큼의 큰 응답을 보낼 수 있음

    - lower boundary (하한선)

      - broker가 client에게 각 partition에 대해 return할 때 data 양의 최소값

        - e.g) lower boundary를 10K로 설정하면, client가 "최소한 10K bytes를 가질 때 clients에게 result를 return해라"라고 broker에게 tell

      - CPU와 network utilization(네트워크 사용)을 줄임

        ![figure5-3](C:\Users\jungsol\Documents\kafka\pic\figure5-3.PNG)

        * clients는 매 few millisecnds마다 data를 요청하고 few(거의 없음) or no messages를 얻는 broker에게 requests를 보내는 대신, clients는 reqeust를 send하고 broker는 decent(충분한) 양의 data가 있을 때까지 대기한 후에 data를 return
        * 전체적으로는 동일한 양의 data가 read되지만, back and forth(앞 뒤로) read하는 횟수가 훨씬 줄어들어 overhead도 감소

      - wait time

        - broker가 충분한 data를 가질 때 까지 영원히 기다리기 보다는, 잠시 기다린 후 존재하는 data를 take하여 처리하는 것이 합리적임
        - 따라서 clietns는 broker에게 timeout을 지정할 수 있음
          - "broker는 x milliseconds 안에 보낼 데이터의 최소량을 충족하지 못하면, 지금까지 모은 것을 보내어라"

  * consumer가 읽을 수 있는 data (replication)

    * clients가 partitoon의 leader에 있는 모든 data를 읽을 수 있는 것은 아님
    * 대부분의 cleints는 in-sync replicas에 쓰여진 messages만 읽을 수 있음

  * message의 replication과 consume data의 관계

    ​	![figure5-4](C:\Users\jungsol\Documents\kafka\pic\figure5-4.PNG)

    * follower replicas는 consumer지만 in-sync replicas에서 exempt(면제)됨. 면제 되지 않는다면 replication이 작동하지 않음
    * partition의 leader는 replica에 replication된 messages를 알고 있으며, message가 모든 in-sync replica에 write될 때 까지, consumer는 message를 read할 수 없음
      * in-sync replicas에 write되지 않은 messages를 fetch하려고 attempt하면, error가 나는 것이 아니고 result가 empty


    * 이러한 동작 방식의 이유는?
      * 충분한 replicas로 replicate되지 않은 messages는 아직 "unsafe"하다고 간주됨
        * leader가 crash하고 another replica가 leader가 되면,  해당(덜 replication된) message는 더이상 Kafka에 존재하지 않게 됨
      * inconsistent behavior 가능성 존재
        * 오직 leader에만 존재하는 data를 clients가 read할 수 있도록 하면 inconsistent behavior(일관성 없는 동작)이 나타날 수 있음
        * e.g) consumer가 message를 읽고 ledaer가 crash(손상)되고 이 message를 포함하는 다른 broker가 없다면, message가 사라짐. 이렇게 되면 다른 consumer는 이 message를 read할 수 없게 되므로 이 message를 읽은 consumer와의 모순이 발생
    * latency time의 발생
      * broker 간의 replication이 어떠한 이유로 느려지면, messages가 replicate될 때까지 대기하기 때문에 new messages가 consumers에게 도착하는데 오래걸리게 됨
      * replica.lag.time.max.ms
        * broker configuration
        * default value = 10000
        * in-sync로 간주되는 replica가 new messages를 replicate하는데 delay될 수 있는 시간
        * *(javadoc에 의하면)* If a follower hasn't sent any fetch requests or hasn't consumed up to the leaders log end offset for at least this time, the leader will remove the follower from isr (이 시간까지 follower가 fetch requests를 보내지 않았거나, leaders log end offset까지 consume하지 않은 경우 isr로부터 follower를 remove)


* Other Requests

  * Kafka에는 Apache Kafka project contributors가 implement(구현) 및 maintain(관리)하는 Java client외에도 C, Python, Go 등의 다양한 언어의 clients가 있음

    * Apache Kafka webstie에서 전체 목록을 확인할 수 있음

  * binary protocol

    * requests를 보내는 Kafka clients는 어떠한 언어로 구현되든 모든 언어의 clients는 binary protocol을 이용하여 Kafka broker와 통신
    * Kafka brokers 간의 통신도 binary prtocol을 이용
      * e.g) controller가 partitions의 new leader를 알리면, Leader Andlsr request를 client requests를 받을 수 있도록 new leader에게 send하며, new leader를 follow하기 위해 follwers에게 send
    * Kafka protocol은 현재는 20가지의 request types를 처리하며, 더 추가될 것임


  * client의 capabilities(기능)이 추가됨에 따라 capabilities와 match되는 진화된 protocol이 필요하므로 protocol은 끊임없이 evolve(진화)됨
    * new reqeust type을 추가하거나 존재하는 reqeust에 some capabilities를 추가하여 수정함으로써 evolve


  * protocol의 진화 예시

    * OffsetCommitRequest
      * 과거에는 Kafka Consumers는 Apache Zokeeper를 사용하여 Kafka로부터 받은 offsets을 ekep track(추적)하였음 -> consumer가 시작되면 consumer는 Zookeeper를 확인하여 partitions에서 read한 last offset을 얻어서 처리를 시작할 위치를 알수 있음 -> 그 이후 다양한 이유로 Zookeeper 사용을 중단하고 Kafka topic 에 offsets을 저장하기로 결정. 이로 인해 아래와 같은 Protocol에 여러 requests가 추가됨
        * OffsetCommitRequest
          * application이 commitOffset() client API를 call하면 client는 더이상 Zookeeper에 offset을 쓰지 않고 Kafka에 OffsetCommitRequest 를 send
        * FetchRequest
        * ListOffsetsRequest
    * Create Topic Request
      * Topic Creation은 topics의 list를 Zookeeper가 직접 update하는 command-line tools로 수행됨 -> brokers는 Zookeeper의 topic list를 보고 새로운 topics가 add되는 것을 알 수 있음 -> 이를 clients가 broker에게 직접 요청하는 new request가 추가됨
      * Kafka를 개선하고 모든 clietns(Zookeeper libarary가 가지고 있지 않는 언어로 쓰인 clients도 포함)가 Kafka brokeer에게 직접적으로 Create TpicRequest를 요청
    * a new version to the Metadata request  / response
      * Kafka 0.9.0에서 Kafka0.10.0으로 넘어갈 때 현재 controller informatin을 추가하여 clients에게 Metadata response를 하는 것으로 추가됨
      * 0.9.0 clients는 versin 0의 Metadata requesets를 send하며 brokers는 controller information을 포함하지 않는 version 0으로 respnse해야 하는 0.9.0인지 0.10.0인지를 확인
    * ApiVersionRequest
      * 0.10.0에서 추가됨
      * clients는 ApiVersionRequest로 broker에게 각 request의 지원되는 version을 물어보고 그에 따라 올바른 version을 사용할 수 있음
      * 이 new capability를 올바르게 사용하는cleitns는 연결된 broker에서 지원하는 protocol version을 사용하여 older broekrs와 통신할 수 있음

  * request / response의 version 호환성

    * 상위 version의 request / response가 있을 경우, 버전의 호환성에 주의해야 함

    1. 하위 version의 client가 하위 version의 reqeust를 broker에게 보낸 경우 -> broker의 version에 관계 없이 broker는 하위 version의 response를 clinet에게 보냄
       * e.g) 0.9.0 clients가 0 version의 Metadata request를 broker에게 보내면, broker는 0.9.0이든지 0.10.0이든지 controller information을 포함하지 않는 version 0의 response를 clients에게 보냄
    2. 상위 version의 client가 상위 version의 request를 broker에게 보낸 경우 -> broker가 상위 version일 경우 1 version의 response를 보내지만, broker가 하위 version일 경우 broker는 new version의 request를 처리하는 방법을 모르므로 error로 response

    * new version의 broker는 old version의 requests를 처리하는 방법을 알고 있지만, 그 반대는 아니므로, clients를 update하기 전에 brokers를 update하기를 권장

      ​

### Physical Storage

* Kafka의 기본 storage는 partition replica

* Partisions는 오직 single broker, single disk에 저장될 수 있음

  * multiple brokers사이에 분할될 수 없으며, 동일한 broker의 multiple disks에도 분할될 수 없음
  * partition의 크기는 single mount point에서 available한 공간으로 제한 
    * JBOD configuration을 사용하는 경우, a mount point는 single disk
    * RAID configuration을 사용하는 경우, a mount point는 multiple disks

* Kafka를 configure할 때, administrator는 log.dirs parameter는 이용하여 partitions이 저장될 directories의 목록을 정의

* Partition Allocation

  * topic을 create할 때 brokers간에 partitions을 allocate하는 방법

  * 각 partition의 각 replica는 다른 broker에 있어야 함

  * Kafka 0.10.0 이상의 version에서는 broker가 rack information을 가지고 있으므로, 각 partition에 대한 replicas를 다른 rack에 assign

    - 전체 rack에 대한 downtime이 발생하더라도 partitions에 대한 완전한 unavailability(비가용성)을 초래하지는 않음

  * Round-Robin을 이용한 parittion leaders 결정

    - 임의의 broker에서 시작하여 각 broker에 round-robin 방식으로 paerttiitons의 leader로 assign

  * Rack awarencess

    ![figure5-5](C:\Users\jungsol\Documents\kafka\pic\figure5-5.PNG)

    * When rack awareness is taken into account, instead of picking brokers in numerical order, we prepare a rack-alternating broker list (rack awarencess(렉 인식)을 고려할 때, 숫자 순서로 broker를 고르는 대신 rack-alternating broker list를 준비하여 해당 list 순으로 broker를 고름)
      - rack-alternating broker list
        - 다음 broker가 다른 lack에 있는 broker가 되도록 (즉, lack이 교차되도록) 정렬해놓은 broker list
    * Round-Robin을 이용하여 Partition allocation을 할 때  rack-alternating broker list를 이용하여 rack awarencess를 하여 partition allocation을 하는 것이 안전함
      - 특정 rack이 offline이 되더라도, 해당 rack에 assign된 partitions들은 다른 rack에 replica가 있으므로 partitions을 계속 사용할 수 있게 됨
      - rack failure 발생시 availability(가용성)을 제공함 

  * e.g) 6 brokers, a topic with 10 partitions, replication factor of 3의 경우

    * 총 30개의 partition replicas를 6개의 brokers에게 allocate해야 함
    * brokers 간에 replica를 고르게 분배하려면 각 broker당 5개의 replicas를 allocate
    * 각 partition의 각 replica가 다른 broker에 있어야 하므로 partition0은 broker2를 leader로 가지면, followers로 broker3, 4를 설정할 수 있지만 2, 3에는 설정할 수 없음
    * Round-Robin을 이용한 partition allocation
      * broker4(random으로 결정된  시작 broker)를 partition0의 leader로 결정하면, 첫번째 follower는 broker5, 두번째 follower는 broker0(broker6은 없음)
      * partition0의 leader가 broker4이므로, partition1의 leader는 broker5이며, 첫번째 follower는 broker1, 두번째 follower는 broker1
    * Round-Robin에 rack-alternating broker list를 이용하여 rack awarencess을 적용
      * brokers 0, 1, 2가 동일한 rack에 있고, brockers 3, 4, 5가 별도의 rack에 있다고 가정
      * rack-alternating broker list를 0,3,1,4,2,5로 작성
      * partition0의 leader가 broker4로 결정되면, 첫번째 follower는 다른 rack에 있는 broker2

* Decision diectory

  * 각 partitions과 replica에 대해 올바른 broker를 선택하였으면, 새로운 partitions에 사용할 directory를 결정해야 함
  * 각 partition마다 independent(독립적)으로 directory selection 작업을 수행하며 알와 같은 간단한 rule을 가짐
    * 각 directory의 partitions의 number를 count하고, the fewest partitions(가장 적은 수의 partitions)을 가지는 directory에 new partition을 추가
    * new disk를 추가하면 다른 disks와 균형을 이루기 전까지는 new disk가 가장 적은 수의 partitions를 가지기 때문에 new disk에 new partitiions이 생성됨
  * Mid the Disk Space
    * brokers에 partitions을 allcoate하는 것은 available spcae(가용 공간) 이나 existing load into account(기존 부하)가 고려되지 않고, 오직 partitions의 개수만 고려됨
    * cluster가 older servers와 new servers가 혼합되어 있는 등의 이유로 일부 brokers가 다른 brokers보다 더 많은 disk의 공간을 가지고 있는 경우, some partitions이 abnormally(비정상적으로) 커지거나, 같은 broker에 다른 size의 disks를 가지게 되므로 partition allocation에 주의해야 함

* File Management

  * Rentention은 Kafka에서 중요한 concept로 Kafka는 data를 forever 유지 하지도 않으며, message가 삭제되기 전에 모든 consumer가 message를 read하도록 기다리지도 않음
  * Kafka administrator는 각 topic별로 retention period(보관 주기) configure
    * retention period를 time과 size를 기준으로 configuration 가능
  * larget file에서 삭제가 필요한 messages를 찾은 다음 file의 일부를 삭제하면 time-consuming(시간이 오래걸림) and error-prone(에러가 발생하기 쉬움)하기 때문에, 각 partition을 segments로 split하여 관리
  * segment file
    * 각 segment는 single data file에 저장됨
    * 기본적으로 1gb 혹은 1주일 중 먼저 충족하는 조건에 따라 segment를 나눔
    * Kafka broker는 partition에 write할 때, segment limit에 도달하면 file을 닫고 new file에 write함
  * active segment
    * 현재 wrtie중인 segment
    * active segment는 절대 삭제 되지 않음
    * active segment는 절대 삭제되지 않으므로, log retention의 주기보다 segment retention의 주기보다 길 경우 segment가 닫히기 전에는 log data를 삭제 할 수 없으므로, log retention 조건이 만족되더라도 segment retention 조건이 만족되어 현재 segment가 롤링될떄까지 data를 보관하게 됨
  * segment retention 관련 configuration parameter
    * log.segment.bytes (broker | default = )
    * segment.bytes (topic| default = $\{log.segment.bytes \})
    * log.segment.hours/minutes/ms (broker | null / null / null)
    * log.roll.hours/ms (broker | defulat = 168 / null)
    * segment.ms (topic | default = 604800000)
  * log rolling time (kafka 1.0.0 docs 보고 정리)
    * log rolling time은 더이상 log segment create time이 아닌 messages의 timestamp를 이용
    * segment의 첫번쨰 message의 timestamp에서 log.roll.ms parameter value의 시간만큼 지나면 log가 roll out
  * Kafka broker는 모든 partitins의 각 segement file들을 open한 상태를 유지
    * inactive segments들도 open file로 유지
    * open file의 개수가 많아지므로 적절하게 OS를 튜닝해야 함

* File Format

  * segment file 내부에는 Kafka messages와 messages의 offsets이 저장됨

  * disk 상의 data의 format은 producer에서 broker로 보내는 messages와, broker에서 consumers로 보내는 messages의 format과 동일함

  * disk와 wire에서 동일한 format을 사용하는 것의 장점

    * Kafka가 consumer에게 message를 send할 때 zero-copy optimization을 사용할 수 있게 함
    * producer가 이미 compress한 messages를 decompressing 하고 recompressing하는 것을 피함

  * message에 포함되는 항목

    * key

    * value

    * offset

    * message size

    * checksum code

      * corruption(손상)을 감지

    * magic byte

      * message format의 version을 나타냄

    * compression codec

      * Snappy, GZip, LZ4가 될 수 있음

        ![figure5-6](C:\Users\jungsol\Documents\kafka\pic\figure5-6.PNG)

      * producer가 compressed messages를 보낼 때는 single producer batch안의 모든 messges가 함께 compress된 wrapper message를 value하여 send함

      * consumer가 message의 value를 decompress하면, batch에 포함된 자신의 timestamp와 offsets을 가지는 개별 message들을 얻을 수 있음

      * producer에서 compression을 사용하는 것은 권장됨

      * large batches를 보내는 것은 network와 broker disks 모두에 better compression을 의미함

      * consumers가 사용하는 message에 timestamp를 추가하는 등과 같이 message format을 바꾸기로 결심한다면?

        * wire protocol과 on-disk의 format 모두 바꿔야 함
        * broker는 upgrades로 인해 두 가지 format의 messges를 포함하는 file을 처리하는 방법을 알아야 함 *(messages를 compress한 message를 value로한 wrapper message의 format과 wrapper message의 value를 decompress하여 얻은 messages의 format 2가지를 모두 처리할 수 있어야 한다는 의미인 것 같음*

    * timestamp

      * Kafka 0.10.0이후 추가됨
      * messages가 producer에 의해 messages가 send될 때 혹은 broker에 의해 messages가 arrive되었을 때로 configuration 할 수 있음

  * DumpLogSegment tool

    > bin/kafka-run-class.sh kafka.tools.DumpLogSegments

    * Kafka brokers는 DumpLogSegment tool을 이용하여 filesystem안의 partition segment를 보고, contents(내용)을 examine(검사)할 수 있음 
    * 각 message의 offset, checksum, magic byte size, compression codec을 보여줌
    * parameter
      * --deep-iteration
        * wraper message안의 compressed messages에 대한 정보를 표시

* Indexex

  * index의 필요 이유
    * consumer는 available offset에서 messages를 fetch하기 시작함
    * consumer가 특정 offset에서 시작하여 특정 크기의 messages를 요청하면, broker는 빠르게 해당 offset의 message를 찾아야 하고 그 offset에서 부터 messges를 read해야 함
      * 특정 offset의 message는 partition의 모든 segment에 있을 수 있음
      * 각 partition의 index를 이용하여 빠르게 주어진 offset의 message를 찾을 수 있음 
  * The index maps offsets to segment files and positions within the file. (index는 segment files의 offsets과 file내의 positions을 map)
  * index도 segment로 분리되므로, messages를 삭제 할 때 오래된 index를 삭제할 수 있음
  * Kafka는 index의 checksums을 유지하지 않음
  * index의 regenerate
    * index가 corrupt(손상)되면, messages를 다시 읽고, offsets과 locations을 기록해서 간단하게 log segment와 matching되는 index를 regenerate
    * 필요한 경우 administrator가 안전하게 idnex segmetns를 삭제하고 자동으로 재생성할 수 있음

* Compaction (압축)

  * Kafka의 messages 보관 유형 (retention policy)
    1. 일반적인 messages 보관 유형으로 retention period동안 messages를 저장하고, retention period가 지나면 message를 purge(삭제)
    2. retnetion time이 지나면 topic에 저장된 각 key에 대해 가장 최신 value만 저장하여 compact(압축,소형)하게 만드는 delete evnets *(이 section에서는 이러한 compaction을 다룸)*
  * policy to compact
    * Kafka를 long-term data store로 전환하는 고급 기능
    * state가 변경될 때 마다, application은 Kafka에 new state를 write
    * latest(current) state를 저장하는 Kafka에 주로 사용 됨
      * e.g) 고객의 배송주소를 저장해야 하는 경우는 지난주 1년 동안의 데이터가 아닌 각 고객의 마지막 주소를 저장하는것이 적합
    * crash되었을 때는 -> 실행 중 발생한 모든 변화가 아닌 crash 이전의 latest state에만 관심이 있으므로, crash에서 recover되었을 때, application은 Kafka로 부터 messages를 읽어 latest state를 recoever
    * compact 정책은 applications이 key와 value를 모두 포함하는 events를 produce하는 topic에만 설정할 수 있음
      * 만약 topic에 null keys가 존재하면 compaction은 실패

* How Compaction Works

  * 각 log는 두부분으로 분할됨

    ![figure5-7](C:\Users\jungsol\Documents\kafka\pic\figure5-7.PNG)

    * Clean
      * 이전에 compact된 messages
      * 이 section에서는 각 key에 대해 one value만 포함되며, compaction 이전의 가장 최신 value
    * Dirty
      * 마지막 compact 이후 wirte 된 messages

  * Compaction 과정

    1. Kafka가 시작될 때 compation이 사용되면, 각 broker는 compaction manager thread와 많은 compaction threads를 시작시킴
    2. compaction threads가 compaction tasks를 수행
    3. 각 threads는 dirty messages의 비율이 가장 높은 partition을 선택하여 partition을 clean

  * Offset Map

    * partition을 compact하기 위해, cleaner thread는 partition의 dirty section을 reads하고 memory 내에 map을 create
    * map의 entry는 동일한 key에 대한 이전 message의 16byte의 message hash와 8byte의 offset로 구성됨
    * 각 map의 entry는 24bytes를 사용하며, 큰 size의 segment를 작은 size의 map에 저장할 수 있으므로 매우 효율적임
      * e.g) 1gb segment를 보고, segment의 message가 각 1kb라고 하면, 해당  segment에는 1백만개의 messages가 있는 것이고, 이 messages를 compact하기 위해서는 최대 24mb의 map이 필요 (keys가 반복되면 same hash entries를 자주 재사용하므로 memory가 24mb보다 더 적게 쓰일수도 있음)
    * administrator는 compaction threads가 offset map에 사용할 수 있는 memory의 양을 configure
      * 각 thread마다 자신만의 map을 가지더라도, configuration은 전체 threads의 전체 memory에 대한 값
        * e.g) compaction offset map에 1gb를 configure하고, 5개의 cleaner threads를 사용하면 각 thread는 200mb의 offset map을 얻음
      * Kafka는 partition의 모든 dirty section을 map에 할당된 size에 맞출 필요는 없지만, 최소한 하나의 전체 segment는 수용할 수 있어야 함
        * 하나의 segment의 dirty section을 할당할 수 없다면 Kafka는 error를 log하며, adminstrator는 offset map에 memory를 더 allocate하거나 더 적은 수의 cleaner threads를 사용하여 개별 thread에 할당되는 offset map의 memory를 늘려야 함
        * 만약 몇개의 segments만 offset map에 fit하면, Kafka는 offset map에 fit(들어맞는)하는 가장 오래된 segments를 compact하고, 나머지는 dirty 상태 그대로 다음 compactiondmf rlekfla

  * Clenaer thread가 segment를 clean하는 방법

    ![figure5-8](C:\Users\jungsol\Documents\kafka\pic\figure5-8.PNG)

    1. Cleaner thread가 offset map을 만들면, 가장 오래된 segment에서 시작하여 clean할 segment를 read하며, 그 내용을 offset에서 점검
    2. 각 message의 key가 offset map에 존재하는지를 확인
       * key가 offset map에 존재하지 않으면, message의 value는 여전히 latest 값이며 replcaement segment에 해당 message를 copy
       * key가 offset map에 존재하면, 해당 partition에 동일한 key를 갖는 더 최신 value가 존재하기 때문에 생략
    3. key에 대해 가장 최신의 value를 가지는 모든 messages를 replacement segment에 copy한 후 original segment를 replcaement segment로 교체하고 다음 clean할 segment로 이동
       * process가 끝나면 key 당 가장 최신 value를 가지는 하나의 message가 남음

* Deleted Events in Compaction

  * 각 key에 대한 최신 message를 보관한다면 특정 key의 messages를 delete하는 방법은?

  1. last message를 system에 저장하지 않고 key를 완전히 삭제하려면, 해당 key와 null value가 포함된mesage를 produce하면 됨

  2. cleaner thread가 value가 null인 message를 발견하면, normal compaction을 하고, null value를 갖는 message만 retain

  3. configuarable time 동안 tombstone이라고 부르는 이 message를 keep함

     * 이 시간동안에는 consumers는 이 message를 보고 value가 delete되었다는 것을 알 수있음


     * consumer는 Kafka로부터 data를 relational dababase에 copy한다면, tombstone message를 보면 database로부터 delete
     * 이 시간이 지나면, cleaner thread는 tombstone message를 remove하고, Kafka의 partition으로부터 key가 사라짐
     * 이 시간을 consumer가 tombstone message를 볼 수 있도록 충분히 configure 하는 것이 중요함
       * consumer가 몇시간동안 down되어 tombstone message를 miss(놓침)하게 되면, consume할 때 key를 볼 수 없으므로 kafka에서 삭제된건지 database에서 삭제된건지를 알 수 없음

* When Are Topics Compacted?

  * delete policy이 현재 active segments를 삭제하지 않는 것 처럼, compact policy도 current segment를 compact하지 않음
  * inactive segments의 messages만 compact될 수 있음
  * Compact되는 시기의 결정 
    * 0.10.0 이전 version의 Kafka에서는 When 50% of the topic contains dirty records(topic의 50%가 dirty records를 포함할 때) compact가 시작됨
      * compact는 topic의 read/write performance에 영향을 줄 수 있기 때문에 너무 자주 compact를 하지 않기 위함이었으나, dirty records는 disk space를 차지하기 때문에 너무 많은 dirty records를 남겨두면 안 됨
    * Wasting 50% of the disk space used a topic on dirty records (dirty records가 있는 topic이 disk space의 50%를 넘은 다음에) , 한번에 compact하면 reasonable trade-off
      * administrator에 의해 튜닝 될 수도 있음
  * 관련 configuration parameter
    * log.cleaner.enabled (broker | default value = true)
      * server에서 log cleander process를 사용하는지를 설정
      * clean.up.policy=compact를 사용하는 하나 이상의 topics이 있다면 반드시 true로 해야 함
      * false로 설정하면 topic은 compact되지 않으며 지속적으로 size가 증가
    * log.cleanup.policy (broker | default.value = delete)
      * 가능한 value : delete, compact
      * retention 이후의 segemtns에 대한 cleanup policy
    * log.cleaner.min.compaction.lag.ms
      * broker configuration parameter
      * default value = 0
      * topic level configuration parameter : min.compaction.lag.ms
      * Kafka 0.10.1에서 추가됨
      * message가 uncompact된 상태로 남아있는 것을 guarantee(보증)하는 grace period를 추가할 계획
      * 이는 topic에 write된 모든 messages를 보는 application이 조금 lag(뒤쳐짐)되더라도 모든 message를 실제로 볼 수 있는 충분한 시간을 가질 수 있게 함




### Sumarry

* There is obviously more to Kafka than we could cover in this chapter, but we hope this gave you a taste of the kind of design decisions and optimizations we’ve made when working on the project and perhaps explained some of the more obscure behaviors and configurations you’ve run into while using Kafka. (이 Chapter에서 다룬 것 보다 더 많은 내용이 실제 Kafka에 있지만, 이 Chapter를 공부함으로써, project에서 작업할 때 소개된 design decision과 optimization을 이해하고, Kafka를 use할 때 obscure(모호한) behavior과 confuguration에 대해 설명할 수 있어야 함)

