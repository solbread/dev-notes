## Chapter 9. Administering Kafka

### Introduce (About Administrative tools)

* CLI utilites
  * Kafka는 clusters의 변경에 대한 관리를 위해 여러 command-line interface(CLI) utilities를 제공
  * tools은 Java classes로 implement되어 있으며, 해당classes를 호출할 수 있도록 일련의 scripts가 제공됨
* Open source
  * 기본 기능은 제공하지만 더 복잡한 작업을 수행하는데는 부족하므로, Apache Kafka open source project의 일부로 available한 tools이 존재
  * 외부의 community에서 개발된 advanced tools에 대해서는 Apache Kafka website에서 확인 가능
* Authorizing Admin Operations (관리자 작업 권한 부여)
  * Apache Kafka는 topic operation을 제어하기 위하여 authentication(인증)과 authorization(권한)을 implement하지만, 대부분의 cluster operation은 아직 지원하지 않음
  * CLI tools는 authentication이 필요하지 않고 사용할 수 있으므로, security check나 audit(감사) 없이 topic 변경과 같은 작업을 실행할 수 있음
  * 이 기능은 개발중이므로 곧 추가될 예정 *Kafka 최신버전(1.0.0)에서 지원하는지 확인하려고 했지만 해당 부분 못찾아서 확인 못함*



### Topic Operations

* Check the version
  * Kafka의 많은 command-line tools는 broker와 직접 연결하는 대신 Zookeeper에 저장된 metadata를 직접 operate(조작)함
  * 이러한 이유로 사용중인 tools의 version이 cluster의 broker version과 일치하는지 확인해야 함
  * 가장 안전한 방법은 deployed version(배포된 버전)을 사용하여, Kafka brokers의 tools을 사용하는 것
* 사용하는 script tool
  * kafka-topic.sh
  * 해당 tool은 대부분의 topic operation(조작)에 쉽게 access할 수 있게 함
  * configuration변경은 더이상 사용되지 않으며 kafka-config.sh 도구로 이동됨

#### Creating a New Topic

> kafka-topics.sh --zookeeper <zookeeper connect> --create --topic <string> --replication-factor <integer> --partitions <integer>

* argument (3가지)

  1. Topic Name (mandatory) : 생성하고자 하는 topic의 이름
  2. Replication Factor (mandatory) : cluster내에서 maintain(유지관리)할 topic의 replicas개수
  3. Partition (mandatory) : 생성할 topic의 parttions 개수
  4. --disable-rack-aware (optional) : rack aware이 필요하지 않을 경우 지정
  5. --if-not-exists (optional) : topic이 이미 존재할 경우 error를 return하지 않도록 지정

  * argument 중 일부는 broker-level의 기본값이 configure되어 있더라도 반드시 제공해야 함

* Command info

  * cluster가 지정된 이름과 partition수를 가지는 topic을 생성하도록 함
  * 각 partition에 대해 지정된 replicas의 수를 적절하게 선택
  * cluster가 rack-aware replica assignment를 위해 설정되어 있는 경우, 각 partition에 대한 replicas는 racks에 분리됨

* Naming Topics

  * 영어, 숫자, underscores(_), dashes(-), periods(.) 가능
  * 두개의 밑줄로 시작하는 Topic  Name은 내부 topic으로 간주되므로 권장되지 않음
    - e.g) e.g) __consumer_offsets : consumer group offset storage
  * 단일 cluster에서 periods(.)와 undersocres(_)를 함께 사용하는 것은 권장되지 않음
    - topic names이 Kafka내부의 metric names으로 사용될 때, 마침표는 underscores(_)로 변경되기 때문 
    - e.g) topic.1이라는 이름의 topic은 metrics에서 topic\_1로 변경됨

* Specifying Topic Configurations

  * kafkatopics.sh에서 topic을 생성하는 중에 topic의 replicas를 명시적으로 설정하거나 topic의 confiiguration override를 설정할 수 있음

* Example

  ```
  # kafka-topics.sh --zookeeper zoo1.example.com:2181/kafka-cluster --create
  --topic my-topic --replication-factor 2 --partitions 8
  Created topic "my-topic".
  #
  ```

#### Adding Partitions

> kafka-topics.sh --zookeeper <zookeeper connect> --alter--topic <string> --partitions <integer>

* consumer의 관점에서 partition 수를 변경하면 mapping of key가 변경되므로 권장되지 않지만 partitions을 추가하여 단일 partition의 throughput을 줄여야 할 경우가 있음

* Skipping Errors for Nonexistent Topics

  * --if-exists argument는 --alter command에 대해 제공되지만 권장되지 않음
  * 해당 argument를 사용하면 변경중인 topic이 존재하지 않을 경우 error를 return하지 않는데, 존재하지 않는 topic이 생성되는 문제가 발생

* Reducing Partition Counts

  * topic에 대한 partitions의 개수를 줄이는것은 불가능하므로 topic을 삭제 후 다시 생성해야 함
  * 불가능한 첫번째 이유는, topic에서 partition을 삭제하면 topic의 일부 data가 삭제되어 client 관점에서 inconsistent(일관적이지않은)하기 때문
  * 불가능한 두번째 이유는, 남아있는 partitions에 data를 redistribute(재배포)하는 것은 어려우며, out-of-order(순서가 잘못된) message가 발생하기 때문

* Example

  ```
  # kafka-topics.sh --zookeeper zoo1.example.com:2181/kafka-cluster
  --alter --topic my-topic --partitions 16
  WARNING: If partitions are increased for a topic that has a key,
  the partition logic or ordering of the messages will be affected
  Adding partitions succeeded!
  #
  ```

#### Deleting a Topic

> kafka-topics.sh --zookeeper <zookeeper connect> --delete --topic <string>

* messages가 없는 topic도 disk space, open filehandles, memory를 포함한 cluster resources를 사용

* topic이 더이상 필요하지 않으면 resources를 free up하기 위해서 삭제할 수 있음 

* topic을 삭제하려면 cluster의 broker의 delete.topic.enable configuration option이 true여야 함

* topic의 모든 messages가 삭제되므로 data loss(손실)에 주의해야 함

* Example

  ```
  # kafka-topics.sh --zookeeper zoo1.example.com:2181/kafka-cluster
  --delete --topic my-topic
  Topic my-topic is marked for deletion.
  Note: This will have no impact if delete.topic.enable is not set
  to true.
  #
  ```

#### Listing All Topics in a Cluster

> kafka-topics.sh --zookeeper <zookeeper connect> --list

* topics tool은 cluster내의 모든 topics을 나열할 수 있음

* list는 특별한 순서 없이, one line에 one topic의 형식으로 출력됨

* Example

  ```
  # kafka-topics.sh --zookeeper zoo1.example.com:2181/kafka-cluster
  --list
  my-topic - marked for deletion
  other-topic
  #
  ```

#### Describing Topic Details

> kafka-topics.sh --zookeeper <zookeeper connect> --describe

* cluster에 하나 이상의 topics에 대한 detailed information을 얻을 수 있음

* argument

  1. --topic (optional) : 사용할 경우 singel topic에 대해 세부정보를 출력하도록 제한
  2. --topics-withoverrides (optional) : configuration override가 있는 모든 topics을 찾음 (cluster의 기본값과 다른 구성이 있는 tpics만 describe)
  3. --underreplicated-partitions (optional) : partition에 대해 하나 이상의 replicas가 leader와 in-sync하지 않은 모든 partitions을 찾음
  4. --unavailable-partitions (optional) : leader가 없는 모든 partitions을 찾음 (partition이 현재 offline이며 producer나 consumer에 unavailable한 것을 의미하므로 심각한 상황)

  * 2번,3번,4번은 기준과 일치하는 cluster의 모든 topics 혹은 partitions을 찾는 것이므로 --topic 인수를 지정하여 topic을 한정하면 안 됨. 또한 이 옵션들은 --list 명령에서는 작동하지 않으므로 --describe 명령과만 써야함

* output에 포함되는 information

  * partition count
  * configuration overrides
  * a listing of each partition with its replica assignments

* Example

  ```
  # kafka-topics.sh --zookeeper zoo1.example.com:2181/kafka-cluster --describe
  Topic:other-topic PartitionCount:8 ReplicationFactor:2 Configs:
  Topic:other-topic Partition: 0 ... Replicas: 1,0 Isr: 1,0
  Topic:other-topic Partition: 1 ... Replicas: 0,1 Isr: 0,1
  Topic:other-topic Partition: 2 ... Replicas: 1,0 Isr: 1,0
  Topic:other-topic Partition: 3 ... Replicas: 0,1 Isr: 0,1
  Topic:other-topic Partition: 4 ... Replicas: 1,0 Isr: 1,0
  Topic:other-topic Partition: 5 ... Replicas: 0,1 Isr: 0,1
  Topic:other-topic Partition: 6 ... Replicas: 1,0 Isr: 1,0
  Topic:other-topic Partition: 7 ... Replicas: 0,1 Isr: 0,1
  #
  ```

  ```
  # kafka-topics.sh --zookeeper zoo1.example.com:2181/kafka-cluster
  --describe --under-replicated-partitions
  Topic: other-topic Partition: 2 Leader: 0 Replicas: 1,0
  Isr: 0
  Topic: other-topic Partition: 4 Leader: 0 Replicas: 1,0
  Isr: 0
  #
  ```

  ​

### Consumer Groups

* Kafka의 Consumer groups의 2가지 type
  * older consumers  : 해당 consumer의 정보는 Zookeeper에서 maintain
  * new consumers : 해당 consumer의 정보는 Kafka brokers에서 maintain
* 사용하는 script
  * kafka-consumer-group.sh
  * 두 유형의 consumer group을 모두 list(나열)하고 describe 가능
* Older consumer VS New consumer
  * Consumer Groups tool을 사용할 때 Kafka cluster에 액세스하기 위한 argument
    * Older consumer : --zookeeper
    * New consumer : --bootstrap-server
  * consumer groups과 offset information의 삭제는 Zookeeper에서 관리되는 older consumer만 가능

#### List Groups

> -- Old consumer
>
> kafka-consumer-groups.sh --zookeeper <zookeeper connect> -- list
>
> -- New consumer
>
> kafka-consumer-groups.sh  --bootstrap-server <broker connect> -- list

* New consumer deprecated argument

  * --new-consumer argument는 0.10.1.0 이후로 사라지고 --botstrap-server argument를 넘겨주면 New consumer로 인식함. (추후 old consumer는 제거될 예정)

* Example (Old consumer)

  ```
  # kafka-consumer-groups.sh --zookeeper
  zoo1.example.com:2181/kafka-cluster --list
  console-consumer-79697
  myconsumer
  #
  ```

* Example (New consumer)

  ```
  # kafka-consumer-groups.sh --new-consumer --bootstrap-server
  kafka1.example.com:9092/kafka-cluster --list
  kafka-python-test
  my-new-consumer
  #
  ```

#### Describe Groups

> -- Old consumer
>
> kafka-consumer-groups.sh --zookeeper <zookeeper connect> --describe --group <string>
>
> -- New consumer
>
> kafka-consumer-groups.sh  --bootstrap-server <broker connect> --describe --group <string>

* Example (Old Consumer) 기준버전 Kakfa 0.10.0

  ```
  # kafka-consumer-groups.sh --zookeeper zoo1.example.com:2181/kafka-cluster
  --describe --group testgroup
  GROUP TOPIC PARTITION
  CURRENT-OFFSET LOG-END-OFFSET LAG OWNER
  myconsumer my-topic 0
  1688 1688 0
  myconsumer_host1.example.com-1478188622741-7dab5ca7-0
  myconsumer my-topic 1
  1418 1418 0
  myconsumer_host1.example.com-1478188622741-7dab5ca7-0
  myconsumer my-topic 2
  1314 1315 1
  myconsumer_host1.example.com-1478188622741-7dab5ca7-0
  myconsumer my-topic 3
  2012 2012 0
  myconsumer_host1.example.com-1478188622741-7dab5ca7-0
  myconsumer my-topic 4
  1089 1089 0
  myconsumer_host1.example.com-1478188622741-7dab5ca7-0
  myconsumer my-topic 5
  1429 1432 3
  myconsumer_host1.example.com-1478188622741-7dab5ca7-0
  myconsumer my-topic 6
  1634 1634 0
  myconsumer_host1.example.com-1478188622741-7dab5ca7-0
  myconsumer my-topic 7
  2261 2261 0
  myconsumer_host1.example.com-1478188622741-7dab5ca7-0
  #
  ```

* Fields provided for old consumer group (Kafka 0.10.0)

  * GROUP : consumer group의 이름
  * TOPIC : consume되는 topic의 이름
  * PARTITION : cnsume되는 partition의 id
  * CURRENT-OFFSET : 지정된 topic partition에 대한 consumer group에 의해 commit된 last offset (partition내에서 consumer의 위치)
  * LOG-END-OFFSET : cluster에 produce되고 commit된 last message의 offset
  * LAG : 지정된 topic partition에 대한 consumer의 Current-Offset과 broker의 Log-End-Offset의 차이
  * OWNER : 지정된 topic partition을 comsume하는 consumer group의 member. group member이 제공한 arbitrary(임의의) ID이며 consumer의 hostname을 반드시 포함하지는 않음

* Example (New Consumer) 기준버전 1.0.0 

  *Kafka docs보고 정리. 출력 format이 조금 달라짐*

  ```
  bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-group
   
  Note: This will only show information about consumers that use the Java consumer API (non-ZooKeeper-based consumers).
     
  TOPIC                          PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG        CONSUMER-ID                                       HOST                           CLIENT-ID
  my-topic                       0          2               4               2          consumer-1-029af89c-873c-4751-a720-cefd41a669d6   /127.0.0.1                     consumer-1
  my-topic                       1          2               3               1          consumer-1-029af89c-873c-4751-a720-cefd41a669d6   /127.0.0.1                     consumer-1
  my-topic                       2          2               3               1          consumer-2-42c1abd4-e3b2-425d-a8bb-e1ea49b29bb2   /127.0.0.1                     consumer-2
  ```

* Fields provided for new/old consumer group (Kafka 1.0.0)

  - GROUP : consumer group의 이름
  - TOPIC : consume되는 topic의 이름
  - PARTITION : cnsume되는 partition의 id
  - CURRENT-OFFSET : 지정된 topic partition에 대한 consumer group에 의해 commit된 last offset (partition내에서 consumer의 위치)
  - LOG-END-OFFSET : cluster에 produce되고 commit된 last message의 offset
  - LAG : 지정된 topic partition에 대한 consumer의 Current-Offset과 broker의 Log-End-Offset의 차이
  - CONSUMER-ID 
  - HOST :  지정된 topic partition을 comsume하는 consumer group의 membe의 hostname
  - CLINET-ID

#### Delete Group

>-- Old Consumer
>
>kafka-consumer-groups.sh --zookeeper <zookeeper connect> --delete --group <string> --topic <string>

* old consumer clients의 삭제

  * Zookeeper에 metadata가 저장되어 있는 old consumer는 command-line으로 Consumer group을 삭제할 수 있음

  * 삭제하기 위해서는 group의 대상 consumers가 shutdown 되어야 함 (shutdown하지 않으면 group의 metadata가 사용중일 때 삭제되므로 consumer가 undefined behavior를 할 수 있음)

  * argument

    * --topic (optional)
      * 생략하면 group이 consume하는 모든 topics에 대해 저장된 모든 offsets을 포함하여 Zookeeper로부터 전체 group이 제거됨
      * 지정하면 group이 consume하는 단일 topic에 대해 저장된 offset을 삭제하고 전체 group은 삭제되지 않음

  * Example

    ```
    # kafka-consumer-groups.sh --zookeeper
    zoo1.example.com:2181/kafka-cluster --delete --group testgroup
    Deleted all consumer group information for group testgroup in
    zookeeper.
    #
    ```

    ```
    # kafka-consumer-groups.sh --zookeeper
    zoo1.example.com:2181/kafka-cluster --delete --group testgroup
    --topic my-topic
    Deleted consumer group information for group testgroup topic
    my-topic in zookeeper.
    #
    ```

* new consumer clients의 삭제

  * broker에 metadata가 저장되어 있고, broker가 partition 처리 및 rebalance를 coordination하는 new consumer는 command-line으로 Consumer group을 삭제할 수 없음
  * group에 대한 offset이 expires(만료될 때) Consumer group이 삭제됨

#### Offset Management

* Introduce
  * offsets을 retrieve(검색)하고, snew offsets을 store(저장)할 수 있음
  * messages를 다시 읽어야 하는 문제가 있거나, wrong format과 같은 문제가 있어서 처리될 수 없는 past message때문에 offsets를 앞으로 이동해야 할 때 유용함
* Managing Offsets Committed to Kafka
  * ​

> kafka-consumer-groups.sh --zookeeper <zookeeper connect> --delete --group <string> --topic <string>

* ​





### Dynamic Configuration Chnages

#### Override Topic Configuration Defaults

#### Overriding Client Configuration Defaults

#### Describing Configuration Overrides

#### Removing Configuration Overrides





### Partition Management

#### Preferred Replica Election

#### Changing a Partition’s Replicas

#### Changing Replication Factor

#### Dumping Log Segments

#### Replica Verification







### Consuming and Producing

#### Console Consumer

#### Console Producer







### Client ACLs





### Unsafe Operations

#### Moving the Cluster Controller

#### Killing a Partition Move

#### Removing Topics to Be Deleted

#### Deleting Topics Manually







### Summary

