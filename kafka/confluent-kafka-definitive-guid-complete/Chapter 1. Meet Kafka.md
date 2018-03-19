## Chapter 1. Meet Kafka



* Publish / Subscribe Messaging

  * Publisher(sender)
    * 데이터를 분류하여 전송
  * Subscriber(receiver)
    * 특정 class의 메세지를 구독
  * Publish system 과 Subscribe system 사이에는 message가 published되는 broker라는 central point 존재
* Kafka

  * Publish / Subscribe Messaging Model + Queuing Messaging Model
    * Publish / Subscribe Messaging Model
      * Publisher가 Subscriber로 데이터를 Broadcast하기 때문에 확장이 어려움
    * Queuing Messaging Model
      * Multi Subscriber를 지원하지 않기 때문에 한번에 하나의 프로세스만 접근 가능
  * distributing streaming platform
  * durable (내구성있는)
  * in order (순차적인)
  * distribution (분산) -> provide additional protection against failures
  * scalable (확장성있는)
* Messages

  * Kafka의 데이터 단위 (The unit of data within Kafka)
  * 데이터베이스의 Row 혹은 Record
  * Kafka는 message를 byte array로 처리
    * message를 byte array로 처리하는 이유 https://kafka.apache.org/documentation/#maximizingefficiency
  * Message는 Metadata 포함
    * Key
      * byte array
      * message의 partition을 결정
  * Kafka는 Message와 Key의 포맷에 관심없음
* Batches

  * just collection of messages, all of which are being produced to the same topic and same partition (같은 토픽과 파티션에 produced 되는 message의 collection)
  * For effciency (효율성을 위함)
  * message는 batches로 Kafka에 쓰임
  * tradeoff between latency(지연시간) and throughtput(처리량)
    * larger the batches(한번에 처리되는 message양을 늘리면)
      * throughtput은 향상(the more messages that can be handled per unit of time)
      * lantency이 늘어남(the longer it takes an individual message to propagate)
  * 압축 가능
    * more efficient data transfer and storage (데이터 전송과 저장 비용에 효율적)
* Schemas

  * message는 이해하기 힘든(opaque) byte array므로 추가적인 이해하기 쉬운 schema를 갖는것을 추천
  * 가장 간단한 schema는 JSON(JavaScript Object Notation), XML(eXtenible Markup Language)
    * low robust type handling
    * low compatibility between schema version (schema 버전이 바뀌면 호환성 유지가 되지 않음)
  * Apache Avro
    * 하둡에서 개발된 serialization framework 
    * provies a compact serialization format
    * schemas that are separate from the message payloads (shema와 message 내용이 분리)
    * schemas that do not require code to be generated when they chage (schema의 변경에도 code 수정이 불필요)
    * strong datatyping (강한 타입 체크)
    * schema evolution, with both backward and forward compatibility (schema 버전이 올라가도 에도 호환성 유지 가능)
  * decoupled(낮은 결합도) between writing and reading
    * consistent(일관성 있는) data format은 wrriting과 reading을 분리
    * kafka의 message format은 writing과 reading간의 format의 coupling이 낮아야함
    * tightly coupled의 경우, publisher 혹은 subsriber의 format이 변경될 경우 양쪽 모두 수정이 필요
* Topics

  * 데이터베이스의 테이블 혹은 파일시스템의 폴더와 비슷한 역할
  * message는 topic으로 분류됨
  * message의 writing은 append-only
  * message의 reading은 from beginning to end
* Partitions

  * topic은 partition으로 분리됨
    * 개별 partition은 message의 time-ordering이 보장되지만 전체 topic에서는 보장되지 않음
  * Redundancy(중복성/여유도) and Scalability
    - partition은 여러대의 서버에 분산될 수 있음
    - 여러대의 서버에 partition을 분산하여 하나의 topic이 여러대의 서버로 확장될 수 있으므로 하나의 서버를 사용하는 경우보다 성능이 좋아짐  
  * Stream
    * single topic of data, regardless of the number of partitions (파티션의 변호에 상관 없는 하나의 토픽)
    * 데이터의 single stream은 producer에서 consumer로 이동된다.
    * stream processing framework VS offline framework
      * stream processing framework인 Kafka Streams, Apache Samza, Sotrm등은 real time(실시간)으로 message를 operate
      * offlien framework인 Hadoop등은 bulk data를 모은 후 operate
* Producers(생산자)

  * Publish / Subscribe system에서의 publisher 혹은 writer
  * create new messages
  * producer는 message를 특정 topic에 produce
  * producer는 message가 저장될 partition과 partition에 저장되는 message의 balance(균형)에 대해 고려하지 않아도 Key를 이용한 partitioning과 balance관리가 됨
    * 기본 partitioner는 key의 hash값과 partitions의 개수의 modulo 연산의 결과값으로 partition을 결정
    * partitioner를 custom하여 사용 가능
* Consumers

  * Publish / Subscribe sytsem 에서의 subscriber 혹은 reader

  * read messages

  * consumer는 하나이상의 topic을 subscribe하고 produced된 순서로 messages를 읽음

  * consumer는 이미 consumed된 messages의 offsets을 기억

  * offset

    * continually increasing하는 integer value의 bit metadata
    * 각 message가 prduced 될 때 결정됨
    * 각 message는 partition 내에서 unique한 offset을 가짐

  * Zookeeper나 Kafka 내부에 마지막으로 각 partition별로 consumed된 offset을 저장해두어 consumer가 중단 혹은 재시작 되더라도 위치를 잃어버리지 않음

  * Consumer group

    * 하나 이상의 consumer가 하나의 topic을 consume
    * Consumer group은 각 partition이 하나의 member에 의해 consumed 되도록 보장
    * consumer와 parition의 mapping은 consumer에 의해 ownership of the partition을 call하여 이뤄짐
      * consumer는 large number of mesage의 topic을 consumer하도록 horizontally 확장할 수 있음
      * 하나의 consumer가 fail하더라도, consumer group의 남아있는 member들이 rebalancing 하여 missing member의 partition을 분배함

* Brokers
  * single Kafka server (하나의 카프카 서버를 broker라고 함)
  * Broker의 역할
    * recieves messages from producers (producer로부터 messages를 수신)
    * assigns offsets to messages (message들의 offset을 할당)
    * commits the message to storage on disk (disk의 저장소에 message를 commit)
    * services consumers, responding to fetch requests for partitions (partition에 대한 요청에 대한 응답)
    * responding with the messages that have been committed to disk (disk에 message를 commit하고 응답)
  * 하드웨어의 성능에 따라 하나의 broker에서 1초당 수백만개의 message를 처리할 수 있음
* Clusters
  * Kafka broker는 cluster의 부분으로 동작(operate)하도록 설계되어 있음
  * cluster의 brockers 중 하나의 brocker는 cluster controller 역할을 함
    * cluster controller
      * 살아있는 cluster members 중 투표를 통해 자동으로 선정
      * administrative operations (관리자 역할을 함)
        * assigning partitions to brokers (brokers에 partition을 할당)
        * monitoring for broker failures (broker의 장애상황을 모니터링)
  * Replication (복제)
    * 하나의 파티션이 여러대의 broker에 replicated 되어 assigned 가능
    * 하나의 파티션을 소유하는 하나의 leader broker가 존재
      * 해당 파티션에서 opertaing하는 모든 consumers와 producers는 leader broker와 연결되어어야 함 
    * provides redundancy of messages in the partition (partition의 중복성/여유도를 제공)
      * leader broker에 장애가 발생하면 다른 broker가 leader가 됨
  * Retention (리텐션)
    * messages를 consumed 한 후에도 일정기간동안 durable storage에 저장
    * topic별로 retention 설정 가능 -> consumer의 필요에 따라 messages의 retention을 다르게 설정 가능
    * period of time 혹은 certain size in bytes 기준으로 retention 설정 가능
      * limit에 도달하면 messages는 expired and deleted
    * log compacted
      * 특정 key에 대해 마지막으로 produced된 message만 retain(보존함)
      * last update data가 interesting한 changelog-type data에 유용함
* Multiple Clusters
  * Multiple Clusters의 유용성
    * Segregation of types of data (데이터 타입의 분리)
    * Isolation for security requirements (보안 조건의 분리)
    * Multiple datacenters (disaster recovery) (다수의 데이터 센터 (장애 복구)) 
  * MirrorMaker
    * multiple clusters에서 messages를 copy하여 single cluster처럼 설계되도록 해주는 tool
    * simply a Kafka consumer and producer, linked together with a queue (queue로 서로서로 연결된 Kafka consumer와 producer)
      * MirrorMaker는 여러대의 cluster들의 데이터를 consume하여 Kafka Cluster Aggregate(queue)에 produce하고 이를 또다른 MirrorMaker가 consume하여 다른 cluster에 produce함 (multiple cluster 환경에서 data sync가 맞게됨)
* Why Kafka? (많은 publish / subscribe messaging system 중에서 kafka를 선택하는것이 좋은 이유는?)
  * Multiple Producers
    * 다수의 Producers가 한 개 이상의 topic에 produced 가능
    * 다수의 frontend sytsems에서 data를 aggregating하고 이를 consistnent(일관되게) 만드는데 이상적인 시스템임
      * 다수의 microservices룰 통해 user에게 content를 제공하는 사이트는 다수의 page views를 위해 하나의 토픽을 가질 수 있음. 그러면 consumer application은 다수의 topics의 consuming을 위하여 coordinate할 필요 없이 page views의 single stream을 받을 수 있음  
  * Multiple Consumers
    * 다수의 consumers간에 interfering(간섭) 없이 message의 single stream을 자유롭게 read할 수 있음 (하나의 message가 하나의 client에 의해 consumed되는 다른 queuing sytem들과 대조적임)
    * Multiple Kafka Consumers는 group으로 묶을 수 있으며 group간에는 stream을 share하여 message가 한번만 processes되도록 보장할 수 있음
  * Disk-Based Retention
    * durable message retention의 의미
      * consumer는 항상 실시간으로 동작하지 않아도 됨
      * 데이터 유실 위험 방지
        * 저장되어 있는 message를 다시 읽거나 real time이 아닌 later time에 읽을 수 도 있으므로 데이터 유실 위험에 대해 방지가 됨
        * processing이 느려지거나 traffic이 급증하여 consumer가 뒤쳐지더라도 데이터 유실 위험이 없음
        * maintenance(유지보수)가 consumer에서 수행됨 (application이 단기간동안 offline이더라도 messages 백업이나 유실에 대해 걱정하지 않아도 됨)
        * consumers가 중단되더라도 messages는 kafka에 보존되며 consumer를 재시작하여 중단된 부분부터 다시 messages를 읽을 수 있음
  * Scalable
    * flexible scalability (유연한 확장성)
    * 하나의 broker에서 시작하여 점차 늘려가며 수십 수백개의 broker를 갖는 large cluster로 expansions(확장)할 수 있음
    * kafka의 expansions
      * cluster가 online 상태에서 수행 가능
      * 시스템의 availability(가용성)에 영향을 주지 않음
    * 다수의 brokers를 가지는 하나의 cluster에서 하나의 broker가 장애가 발생하더라도 service에 영향을 주지 않음
    * replication(복제) factors를 이용하여 더 많은 동시적인 장애를 견딜 수 있는 clusters를 구성할 수 있음
  * High Performance
    * 높은 부하에서 높은 성능을 낼 수 있음
    * Producer, Consumers, Brokers는 큰 message streams를 쉽게 처리할 수 있도록 빠르게 확장될 수 있음
      * This can be done while still providing subsecond message latency from producing a message to availability to consumers. 
* The Data Ecosystem
  * Kafka는 messages schmas를 제공하는 system에서 producers와 consumers간의 tight coupling 또는 direct connection이 필요하지 않도록 함
    * producers는 누가 data를 사용하는지 혹은 consumer의 개수를 고려할 필요가 없음
* Use Cases(사용예시)
  * Activity tracking (사용자 활동 추적)
    * website의 frontend application에서 user의 활동 정보를 하나이상의 topic에 produce한 후backend application에서 이 datas를 consume하여 활용
  * Messaging
    * application에서 notifications를 보내는 messaging에 사용
    * messages의 format이나 sending방법에 대해 고려하지 않고 messagefmf produce한 후 single application에서 meesages를 consum하여 일관되게 handle하여 send 가능
      * single application을 사용하면 multiple applications간의 기능적 중복이 필요 없으며 aggregation(집계)이 가능
  * Metrics and logging
    * system metrics랑 logs를 수집하는데 유용
    * multiple applications에서 같은type의 messages를 producing하면 monitoring과 alerting systmes에서 이들을 consumed하여 사용
    * 이러한 방식은 longer-term analysis를 수행하는 Hadop이나 dedicated log search systems인 Elastisearch나 security analysis applications에서 사용됨
  * Commit log
    * Kafka는 commit log를 기반으로하여, database의 변화를 published하여 applications이 쉽게 이러한 stream의 변화를 monitor할 수 있게 함
    * changelog stream은 remote system에 databse updates를 remote system에 복제하거나 mutiple applications의 변경사항들을 single database view로 통합하는데 사용할 수 있음
    * durable retention(견고한 유지)는 changelog를 위한 buffer를 제공함 (즉, application에서 consuming이 실패하더라도 retry)
    * log-compacted topics : key 당 하나의 변경사항만 보관하여 더 오래 retention(보관)할 수 있음
  * Stream processing
* 참고자료
  * Reference Guide for Deploying and Configuring Apache Kafka
    * https://www.cloudera.com/content/dam/www/marketing/resources/datasheets/deploying-and-configuring-apache-kafka-reference-guide.pdf.landing.htmlApache
  * Kafka Documentation
    * https://kafka.apache.org/documentation
  * 발행-구독 모델
    * https://ko.wikipedia.org/wiki/%EB%B0%9C%ED%96%89-%EA%B5%AC%EB%8F%85_%EB%AA%A8%EB%8D%B8