## Chapter 4. Kafka Consumers: Reading Data from Kafka

### Intorudce

* Kafka로부터 data를 읽는 applications는 KafkaConsumer를 사용하여 Kafka topics을 subscribe하고 topics로부터 messages를 receive

* Kafka로 부터 data를 Reading하는 것은 다른 messaging systems로부터 data를 Reading하는 것과 약간 다르며 포함된 unique concepts과 ideas가 거의 없음

  ​

### Kafka Consumer Concepts

* Kafka에서 data를 read하는 방법을 이해하기 위해서는 consuemrs 자체와 consumer grups에 대한 이해가 필요

* Consumers and Consumer Groups

  * Consumer Groups의 필요성
    * Kafka topic으로부터 message를 읽고 검증하여 다른 data store로 write하는 application이 있다고 가정하면 이 application은 consumer object를 create하고 appropriate tpic을 subscribe하고 message를 receive하고 이를 validating, writing the results
    * 이 application이 잘 작동하다가 producers가 messages를 write하는 속도가 application에서 validate할 수 있는 속도를 초과하면?
      * topics의 consumption을 확장해야 함
      * multiple producers가 하나의 topic에 write할 수 있는 것 처럼, multiple consumers가 같은 topic내에서 data를 splitting하여 읽을 수 있어야 함
  * Kafka Consume와 Consumer group의 관계 
    * Kafka consumers는 일반적으로 consumer group의 일부
    * 같은 consumer group의 multiple consumers가 하나의 topic을 subscribe하고, group의 각 consumer가 topic의 subset of the partition으로 부터 messages를 receive
      * 각 consumer가 receive하는 subset of partitin은 consumer group내에서 consumer마다 다름

* Consumer group을 이용한 Single application의 Scale(확장)

  * 2가지 방법
    1. adding consumers
    2. adding applications

  1. Adding Consumers

     * Consumer group에 consumers를 추가하여 Kafka topic의 data consumption을 scale

     * Kafka Topic의 data consumption 확장의 필요성

       * Kafka consumers는 database에 data를 쓰거나 data에 time-consuming computation을 하는 등의 high latency operations를 수행하는 것이 일반적임
         * single consumer로는 topic으로 유입되는 data를 따라갈 수 없으며, multiple consumers가 같은 topic을 read해서 각 consumer가 partitions 및 messages의 subset만 갖도록 하여 부하를 공유하는 더 많은 consumers를 추가해야 함

     *  the number of partitions >= the number of consumers

       * 하나의 partition의 data를 여러 consumers가 receive할 수 없음
       * topic의 partitions의 개수 보다 더 많은 consumers를 추가하면 idle(유휴상태) consumer가 발생

     * load(부하)가 increase되면 더 많은 consumers를 추가할 수 있으므로 topic을 생성할 때 partitons의 수를 많이 하는 것이 좋음 *(chapter2에는 partition의 개수를 선택하는 방법에 대한 제안이 있음)*

     * Example

       ![figure4-1](C:\Users\jungsol\Documents\kafka\pic\figure4-1.PNG)

       * 4개의 partitions을 가지는 topic인 T1이 있고, 하나의 consuemr를 가지는 consumer group G1의 consumer인 C1이 topic T1을 subscribe
       * consumer C1은 T1의 4개의 partitions으로부터 messages를 수신

       ![figure4-2](C:\Users\jungsol\Documents\kafka\pic\figure4-2.PNG)

       * cnsumer gorup G1에 consumer C2를 추가
       * consuemr C1, C2는 T1의 2개의 partitions에서만 messages를 receive
       * figure4-2에 의하면 C1은 partition0, partition2에서 messages를 receive, C2는 partition1, partition3에서 messages를 receive

       ![figure4-3](C:\Users\jungsol\Documents\kafka\pic\figure4-3.PNG)

       * G1에 4개의 consumers가 있는 경우 각 consumer는 single partition에서 messages를 receive

       ![figure4-4](C:\Users\jungsol\Documents\kafka\pic\figure4-4.PNG)

          * single topic을 subscribe하고 있는 single consumer group에 topic이 가지는 partitions의 개수보다 더 많은 consumers를 추가하면, 일부 consumers는 idle(유휴상태)가 되며 messages를 receive하지 못함
          * figure4-4에서 G1의 consumers의 개수가 T1의 partitions의 개수보다 많으므로 C5에는 할당된 partition이 없으며 mesasges를 receive하지 못함


  2. Adding applications (adding consumer groups)

     * 같은 topic으로 부터 data를 read하는 multiple applications을 가짐

     * Kafka의 주요 design goals 중 하나는 Kafka topics에 produce된 data를 oganization 전체에서 다양한 use cases로 사용할 수 있게 하는 것

     * 각 application은 messages의 subset이 아닌 전체 messages를 가져오기를 원함

       * application이 topic에서 모든 messages를 가져오려면 해당 application 고유의 consumer group을 정의하여 사용하면 됨

     * traditional messaging systems과 달리 Kafka는 performance의 reduce 없이 다수의 consumers와 consumer gourps로 확장

     * Example

       ![figure4-5](C:\Users\jungsol\Documents\kafka\pic\figure4-5.PNG)

       * 새로운 consumer group G2를 추가하면 G1의 수행작업에 independent하게 T1의 모든 messages를 가져옴

  * Summarize
    * create new consumer
      * 하나 이상의 topic에서 모든 messages를 필요로 하는 경우
    * add consumer to an existing consumer grup
      * topic에 대한 messages의 reading과 prcessing의 scale이 필요하는 경우
      * 추가된 consumer는 messages의 subset을 읽음

* Consumer Groups and Partition Rebalance

  * consumer group의 consumers들은 subscribe하는 topic의 partition의 ownership을 공유하므로 consumer group의 consumers에 상태변경이 있었을 때 Rebalance됨

  * Rebalance

    * partition의 ownership을 다른 consumer로 이동시키는 것
    * consumer group에 consumers를 쉽고 안전하게 add and remove할 수 있게 함
    * consumer group에 high availability(가용성) 및 svalability(확장성)을 제공
    * normal course of events (일반적인 이벤트 과정)에서는 fairly undesirable(상당히 바람직 하지 않음)

  * Rebalance되는 경우

    * group에 new consumer가 추가되는 경우
      * new consumer는 another consumer가 consume하던 partitions의 messages를 consuming
    * consumer가 shuts down or crashes되는 경우
      * 해당 consumer는 group을 leave
      * 해당 consumer가 consume하던 partitions은 남은 consumers가 consume
    * consumer group이 consume하는 topic이 변경되는 경우
      * administrator가 new partitions을 추가하는 등의 작업으로 인해 topic이 변경될 수 있음
      * consuemrs에게 Reassignment of partitions이 발생

  * Rebalance의 undesirable 측면 (단점)

    * Rebalance하는 동안 consumers는 message를 consume할 수 없으므로 전체 consumer group을 사용할 수 없음
    * partitions이 이동하면, consumer는 현재 state(상태)를 잃게되며, caching된 data가 있다면 caches를 refresh해야 하므로 consumer가 state를 다시 설정할 때 까지 application이 느려짐

  * 안전하게 Rebalance를 처리하고 unnecessary ones를 피하는 방법

    * Heartbeats
      * consumer가 consumer group의 member자격을 유지하고, assign된 partition의 ownership을 유지할 때 heartbeats를 이용
      * 해당 consumer group의 coordinator로 지정된 Kafka broker에게 consumer는 heartbeats를 보냄
        * consumer가 regular intervals로 heartbeat를 sending하는 한, 해당 consumer는 alive하고, partitions의 message를 처리하는것으로 간주
        * Heartbeats는 consumer가 poll(즉 retrieves(검색) records)할 때와, consume된 records를 commit할 때 전송됨
      * consumer가 heartbeats를 오랫동안 보내지 않으면, 해당 session(connection)이 timeout되며 group coordinator가 해당 consumer를 dead로 간주하고, rebalance를 trigger
    * uncleanly closing
      * consumer가 crash되고 messages processing이 stop되면, group coordinator는 a few seconds(몇 초)가 걸려서 heartbeats가 없기 때문에 dead로 간주하고 rebalance를 trigger
      * 이 a few seconds동안 해당 consumer에 assign된 partitions은 어떤한 message도 처리가 되지 않음
    * cleanly closeing
      * consumer가 cleanly close되면 consumer는 group coordinator에게 자신이 consumer group을 leave할 것이라고 notify
      * group coordinator는 즉시 rebalance하여 processing의 gap을 줄임
    * hearbeat의 frequency, seesion timeout configuration의 설정이 필요

  * Changes to Hearbeat Behavior in Recent Kafka Versions(최신 Kafak Version에서의 Heartbeat 동작 변경)

    * 기준 recent kafka version : 0.10.1

    * consumer에서 poll 사이사이에 heartbeats를 보내는 별도의 heartbeat thread를 도입

    * heartbeat frequency와 frequency of polling을 분리할 수 있게됨

      * hearbeat frequency
        * consumer group이 특정 consumer가 crash되고 더이상 heartbeats를 보내지 않는 다고 판단하는 시간
        * session.timeout.ms
          - consumer crash를 감지하고 heartbeats 전송을 중지하는데 걸리는 시간을 제어
      * frequency of polling
        * brokers에서 return된 data를 process하는데 걸리는 시간
        * max.poll.interval.ms
          * group을 떠나고 rebalance를 trigger하기 전에 polling이 없는 상태로 얼마나 있을 수 있는지를 configure
          * 처리하는데 오래걸리는 records를 처리하기 위한 configuration option
          * livelock을 막음
            * livelock : application이 crash되지는 않았지만 some reason으로 pregress하지 못하는 상태
          * 해당 configuration option의 값을 늘리면 new records를 polling 하기까지 더 길게 delay됨

    * 이 장의 나머지 부분에서는

      1. older Kafka version의 behavior의 문제점과 이를 처리하는 방법
      2. records 처리에 오래걸리는 application을 처리하는 방법

      * 에 대해서 disscuess하는데 Apache Kafka 0.10.1 이상을 사용하는 독자들에게는 적합하지 않음
        * Apache Kafka 0.10.1 이상에서는 records 처리가 오래걸리면 max.poll.interval.ms option을 조정하면 됨

  * How Does the Process of Assigning Partitions to Brokers Work? (Brokers에게 Partitions을 Assign하는 프로세스의 작동 방식)

    * consumer group내의 assigning partitions는 group leader consumer가 함
    * consumer group의 leader 선출 방법
      * Consumer가 consumer group에 join하려고 하면 group coordinator에게 JoinGroup request를 보냄 
      * consumer group에 처음으로 join한 consumer가 group leader가 됨
    * consumer에게 partition을 assign하는 방법
      1. consumer group의 leader consumer가 group coordinator로부터 최근에 heartbeat를 보내어 alive로 간주되는 consumers 명단을 수신
      2. leader consumer가 각 consumer에게 subset of partitions을 assign
         * PartitionAssignor를 implements하여 어떤 consmer가 어떤 partitions을 처리할지 결정
         * built-in partition assignment policies(파티션 할당 정책)이 있음
      3. leader consumer가 partition assignment를 한 후, consumer leader는 group coordinator에게 모든 consumers의 정보가 들어있는 assignments list를 보냄
    * 각 consumer는 자신의 assignment만 알 수 있으며, leader는 group내의 전체 consumers 목록과 그들의 assignments 목록을 가진 유일한 client
    * rebalance가 발생할 때 마다 1~3을 반복



### Creating a Kafka Consumer

* KafkaConsumer instance는 Consumer configuration options 정보를 담는 properties로 생성

* Consumer의 mandatory configuration 3가지

  * bootstrap.servers, key.deserializer, value.deserializer

* Sample Code

  ```java
  Properties props = new Properties();
  props.put("bootstrap.servers", "broker1:9092,broker2:9092");
  props.put("group.id", "CountryCounter");
  props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
  props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
  KafkaConsumer<String, String> consumer = new KafkaConsumer<String,String>(props);
  ```

  1. key와 value로 모두 String object를 가지는 Records를 consume하는 KafkaConsumer instance



### Subscribing to Topics

* subscribe(java.util.Collection\<java.lang.String\> topics);

  ```java
  consumer.subscribe(Collections.singletonList("customerCountries"));
  ```

  1. ustomer는 customerCountries라는 single topic을 subscribe

* subscribe(java.util.regex.Pattern pattern);

  ```java
  consumer.subscribe("test.*");
  ```

  1. 모든 test topics을 subscribe

  * regular experssion을 사용하여 subscribe 가능
  * regular expression과 mache되는 new topic을 create하면, rebalance가 거의 즉시 발생하고 consumers가 new topic을 consum하기 시작
  * multiple topic으로부터 consume해야 하는 applications에 유용하며 topics이 포함하는 여러 유형의 data를 처리할 수 있음
    * Kafka와 another system간에 데이터를 replicate하는 application에서 많이 씀



### The Poll Loop

* consumer API의 핵심은 server를 polling하여 data를 얻는 simple loop

* consumer가 topics을 subscribe하고 poll loop는 세부사항(coordination, partition, rebalances, heartbeats, data fetching)을 처리하여, assign된 partitions에서 available data를 return

* Sample code

  ```java
  try {
    while (true) {
      ConsumerRecords<String, String> records = consumer.poll(100);
      for (ConsumerRecord<String, String> record : records) {
        log.debug("topic = %s, partition = %s, offset = %d, customer = %s, country = %s\n",record.topic(), record.partition(), record.offset(), record.key(), record.value());
        int updatedCount = 1;
        if (custCountryMap.countainsValue(record.value())) {
          updatedCount = custCountryMap.get(record.value()) + 1;
        }
        custCountryMap.put(record.value(), updatedCount);
        JSONObject json = new JSONObject(custCountryMap);
        System.out.println(json.toString(4))
      }
    }
  } finally {
    consumer.close();
  }
  ```

  1. infinite loop

     * 보통 Consumer는 더 많은 data를 얻기 우해 Kafka를 끊임없이 pll하는 long-running applications
  2. poll (long timeout)

     * timeout
       * consumer buffer에서 data를 available할 수 없는 경우 poll()이 block하는 기간
       * timeout이 0일 경우 poll()은 즉시 return
       * timeout이 0이 아닐 경우 brker로부터 data가 arrive할 때 까지 특정 millisecond동안 대기
       * 이 parameter 값은 application이 얼마나 빠른 response를 원하는지에 따라 결정됨
     * 상어가 계속 움직이거나 죽는것과 같은 방식으로 Consumers는 Kafka를 계속 poll하거나, dead로 간주되며 dead로 간주된 Consumer가 consume하던 partitions은 group내의 another consumer에게 전달되어 consume을 이어감
  3. poll()은 ConsumerRecords\<K,V\>의 records 목록을 반환
     * 각 record에는 record가 있었던 tpic, partition, offset, key, value가 포함됨
     * iterate하며 records 각각을 처리함
  4. 보통 polling한 데이터의 처리는 저장하거나 저장된 데이터를 update하는것으로 끝이남
     * 이 code에서는 각 country의 customers 수를 기록하기 위해 hashtable을 update하고 결과를 JSON으로 print
     * 보다 현실적인 example은 result를 data store에 저장하는 것
  5. close()
     *  종료 전에 항상 consumer를 닫아야 함
     *  network connection과 socket을 닫음
     *  rebalance를 즉시 trigger
       * group coordinator가 consumer가 heartbeats를 send하는 것을 멈추고 dead하였다는것을 발견 한 후rebalance를 하면, 발견하기 까지의 기간동안 해당 consumer가 consume하던 parttions는 message가 consume되지 않게 되는데, close()를 호출하면 이러한 과정없이 즉시 rebalance가 trigger됨


* Poll loop의 역할

  1. get data
  2. new consumer로 처음 poll()하면, group coordinator를 찾아 consumer group에 join하고 partition assignment를 받음
  3. rebalance가 trigger되었을 때, rebalance의 처리도 poll loop안에서 수행
  4. send heartbeat

* poll loop는 위처럼 다양한 역할을 하므로, loop의 iterations 사이의 모든 작업이 신속하고 효율적으로 처리되도록 make sure(확실하게)해야 함

* Thread Safety

  * KafkaConsumer는 not thread safe
  * thread 당 하나의 consumer가 rule
    * 하나의 thread에서 하나의 group에 속하는 multiple consumers를 가질 수 없으며, multiple threads가 같은 consumer를 가질 수 없음
    * 하나의 application에서 동일 group의 multiple consumers를 실행하려면 각각의 thread에서 각각 실행해야 함
    * consumer logic을 object로 wrap한 후, Java의 ExecutorService에 각각 하나의 consumer를 가지는 multiple threads를 시작하는 것이 좋음
      * [Confluent blog](https://www.confluent.io/blog/)는 이런 방식의 tutorial이 존재 



### Configuring Consumers

*설명은 책을 정리했고, default value는 kafka documentation 보고 직접 기록* 

* 대부분의 paramters는 reasonable defaults를 가지며 수정이 필요하지 않지만 몇몇은 consumers의 performance와 availability(가용성)에 영향을 줌

1. bootstrap.server (mandatory)

   * \$\{host\}:\$\{port\}, \$\{host\}:\$\{port\},....


   * consumer가 Kafka cluster에 초기 연결할 때 사용하는 brokers list
   * consumer는 초기 연결 후 더 많은 정보를 얻으므로 이 list에는 모든 brokers가 포함될 필요 없음
   * 최소 2개의 brokers를 포함하는것이 권장됨 (하나의 broker가 down되더라도 producer가 cluster에 연결될 수 있도록)

2. key.deserializer (mandatory)

   * consumer가 consume한 records의 key를 역직렬화하는데 사용되는 class
   * org.apache.kafka.common.serialization.Deserializer interface를 implements한 class의 이름으로 설정되어야 함
   * byte array 형태의 meesage의 key를 java object로 deserialize

3. value.deserializer

   * consumer가 consume한 records의 value를 역직렬화하는데 사용되는 class
   * byte array형태의 message의 value를 java object로 deserialize

4. group.id (optional | default="")

   * KafkaConsumer instance가 속하는 consumer group을 지정
   * consumer group에 속하지 않는 consumers를 만드는 것은 가능하지만, 거의 드물기 때문에 optional configuration이지만 mandatory configuration처럼 쓰임

5. fetch.min.bytes (optional | default=1)

   * broker에서 data를 receive하는 data의 minimum amount
   * broker가 consumer로부터 records 요청을 받더라도, new records 사이즈가 해당 option value보다 작은 경우, records를 consumer에게 보내기 전에 더 많은 message를 기다림
   * This reduces the load on both the consumer and the broker as they have to handle fewer back-and-forth messages in cases where the topics don’t have much new activity (or for lower activity hours of the day). (새로운 활동이 많지 않거나 활동이 적은 시간일 때, consumer와 broker는 더 적은 앞뒤메세지를 처리해야 하므로, consumer와 brokder의 load를 reduce함)
   * 해당 configuration option의 value를 default보다 높여야 하는 경우
     * data available이 많지 않을 때, consumer가 CPU를 너무 많이 쓰는 경우
     * consumers가 많을 경우에 value를 높이면 broker의 load(부하)를 감소시킴

6. fetch.max.wait.ms (optional | default=500)

   * fet.min.btytes configuration option에 의해 Kafka는 consumer에게 response하기 전에 충분한 data를 wait하는데 해당 option은 이 wait time을 제어 
   * kafka topic에 return할 데이터의 minumum amoun를 만족하지 못했을 경우 data는 flow되지 않고 해당 option time(ms)만큼 추가적인 latency 발생
     * lattency를 limit하려면 해당 option 값을 낮게 잡으면 됨
   * fetch.max.wait.ms / fetch.min.bytes
     * 두 configuration optin이 모두 설정되어 있다면, Kafka는 consumer로부터 fetch request를 받고 data가 두 option중 하나라도 충족이되면 data를 response

7. max.partition.fetch.bytes (optional | default=1048576)

   * server가 partition당 return하는 최대 bytes 수
   * consumer가 poll()하여 ConsumerRecords가 반환되었을 때 record object는 consumer에게  assign된 partition당 최대 해당 option의 value bytes만큼 사용
   * 해당 option의 기준은 per peratition
     * 하나의 topic에 20개의 partitions이 있고 5개의 consumer가 있을 경우 각 consumer는 ConsumerRecords에 4mb까지 사용할 수 있음
     * consumer group의 consumers가 fail하여 각 consumer가 더 많은 partition을 처리해야 하는 경우 더 많은 memory가 필요하게 됨
   * value를 결정하는 요인
     1. max.partition.fetch.bytes > max.message.bytes(broker configuration option)
        * broker에 cnsumer가 consume하지 못하는 message가 발생하여 consumer가 그 message를 읽으려고 계속 try하며 hang이 걸릴 수 있음
     2. consumer가 data를 처리하는데 걸리는 시간
        * consumer는 session timeout과 subsequent rebalance를 피하기 위해 poll()을 자주 call해야 함
        * single poll()의 data가 매우 큰 경우 -> consumer는 process시간이 길어지며 -> session timeout을 피하기 위해 next iteration of poll loop을 얻지 않음
        * cnsumer가 data를 처리하는 시간이 길 경우에는 max.partition.fetch.bytes를 낮추거나, seession.timeout.ms을 높여야 함

8. session.timeout.ms (optional | default=10000)

   * consumer가 brokers와 접촉에서 벗어나있을 수 있는 time
     * consumer가 heartbeats를 보내지 않고 지낼 수 있는 시간
   * consumer가 group coordinator에게 heartbeat를 전송하지 않고 해당 value time이 지나면, 해당 consumer는 dead로 간주되고 grup cordinator는 consumer group의  rebalance를 trigger하여, dead consumer의 partittions을 다른 consumers에게 assign
   * hearbeat.interval.ms와 밀접한 관련이 있음
   * lower session.timeout.ms
     * consumer groups이 farilure를 빠르게 감지하여 복구할 수 있음
     * 원치않는 rebalances가 발생할 수 있음
       * consumer가 dead된게 아니라 poll loop이나 garbage collection을 complete하는데 시간이 오래걸린것만으로도 session.timeout.ms을 넘어가서 rebalance가 발생할 수 있음
   * higher session.timeout.ms
     * accidental rebalance가 줄어들지만 실제 failure를 탐지하는데 시간이 오래걸림

9. haertbeat.interval.ms (optional | defualt=3000)

   * KafkaConsumer poll()가 group coordinator에게 heartbeat를 보내는 frequently를 제어
   * seesion.timeout.ms와 heartbeat.interval.ms의 관계
     * heartbeat.interval.ms < seesion.timeout.ms
     * 일반적으로 session.timeout.ms의 1/3 time으로 heartbeat.interval.ms를 지정

10. auto.offset.reset (optional | default=latest)

  * commit된 offset이 없거나  consumer가 오랫동안 down되어 있어서 offset의 record가 이미 broker에서 removed되는 등의 이유로 commit된 offset이 invalid할 경우에 partition을 read하려고 할 때 broker의 behavior를 제어
  * 3가지 값을 가질 수 있음 : earliest | latest | none
    * 이외의 값이 들어오면 consumer에 throw exception
    * earlieset
      * 자동으로 첫번째 offset으로 reset
    * latest
      * 자동으로 마지막 offset으로 reset
      * lat offset으로 reset되므로 consumer는 consumer가 실행된 후에 write된 newest records에서부터 읽기 시작함
    * none
      * consumer가 previous offset정보를 가지도 있지 않으면 throw exception

11. enable.auto.commit (optional  | default=true)

    * consumer가 offsets을 자동으로 commit할지 여부를 제어
    * offsets을 commit할 때 중복을 최소화하고 missing data를 방지하기위해 제어하는것을 선호한다면 false로 설정
    * enable.auto.commit이 true일 때 offset을 commit하는 frequency는 auto.commit.inteval.ms configuration option으로 설정 가능

12. auto.commit.interval.ms (optional | default=5000)

    * consumer가 offset을 commit하는 시간 간격

13. partition.assignment.strategy (optional | default=org.apache.kafka.clients.consumer.RangeAssignor)

    * Partition Assignmnet Strategy의 class name
    * PartitionAssignor는 consumers와 subscribed topics을 고려하여 consumer에게 assign할 partitions을 결정하는 class
    * 기본적으로 2가지 assignment strategy가 있음: Range, RoundRobin
      1. Ragnge
         * org.apache.kafka.clients.consumer.RangeAssignor
         * 구독하는 각 topic의 partition의 consecutive subset을 각 consumer에게 assign
           * 2개의 consumers C1, C2가 2개의 topics T1, T2를 subscribe하고 각 topic은 3개의 partitions를 가질 때, C1은 각 topic의 partition0, partition1이 assign되며 C2는 각 topic의 partition2가 assign됨
         * partitions의 개수가 consumers의 개수로 나눠떨어지지 않을 경우 더 많은 partitions이 assign된 consumers가 발생
           * (위의 예제에서) 각 topic은 partition의 수가 홀수개이며 각 topic을 assign하는 것은 independently (독립적으로) 수행되므로 첫번째 consumer는 두번째 consumer보다 더 많은 partitions을 가짐
      2. RoundRobin
         * org.apache.kafka.clients.consumer.RoundRobinAssignor
         * subscribe되는 모든 partitions을 가져와서 consumers에게 sequentially(순차적으로) assign
           * Range는 각 topic의 partitions을 consumers에게 assing하는 작업이 topic별로 independency하게 수행되므로 균등분배가 이뤄지지 않을 가능성이 높았지만 RoundRobin은 모든 partitions을 가져와서 sequentially assign을 하기 때문에 Range보다 균등적으로 분배됨
           * 2개의 consumers C1, C2가 2개의 topics T1, T2를 subscribe하고 각 topic은 3개의 partitions를 가질 때, C1은 topic T1의 partition0, partition2, topic2의 partition1이 assign되며 C2는 topic T1의 partition1과 topic2의 partition0, partition2이 assign됨
         * partitions의 개수가 consumers의 개수로 나눠떨어지지 않더라도 각 consumer에 assign된 partitions 개수의 차이는 최대 1
    * Custom Partition Assignment Strategy
      * org.apache.kafka.clients.consumer.internals.AbstractPartitionAssignor의 추상 class를 상속하여 구현한 class이름을 configuration option value로 사용하면 됨


14. clinet.id (optional | default="")

    * client에서 보낸 messages를 identify하기 위해서 brokers에서 사용되는 문자열
    * logging, metrics, quotas에 사용됨

15. max.poll.records (optional | default=500)

    * poll()의 single call에서 return하는 records의 최대 개수
    * polling loop에서 application이 처리해야 하는 data의 amount를 조절하는데 유용

16. receive.buffer.bytes / send.buffer.bytes (optional | default=65536 / 131)

    * data를 write하고 read할 때,  sockets에서 사용하는 TCP 송/수신 buffer의 size
    * 이 value가 -1로 설정되면 OS defaults값을 사용함
    * producers나 consumers가 다른 datacenter에 있는 broker와 communication할 떄는 higher latency, lower bandwidth netwrk links이기 때문에 해당 configuration option의 value를 높이는 것이 좋음


### Commits and Offsets

* poll()을 call할때 마다 grup내의 consumer가 아직 read하지 않은 Kafka에 write된 records가 반환

  * 이를 위해서는 group내의 consmer에 의해 read된 records를 track(추적)할 수 있어야 함
  * Kafka의 고유한 특성 중 하나는 많은 JMS queue와 처럼 acknowledgements를 track하지 않음
  * 대신 consumers는 Kafka를 사용하여 각 partition에서의 positin(offset)을 알 수 있음

* Commit

  * partition의 current position(oppset)을 cmmit하는 update 작업이 필요

  * Kafka에 message를 prduce하면 \_\_consumer\_offsets이라는 spectial topic이 각 partition에 offset을 commit

    * consumer가 up, running, churning away 상태인 경우 아무런 영향이 없지만 crash되거나 new consumer가 consumer group에 join되어 rebalance가 trigger된 경우, rebalance 이후에는 각 consumer가 새로운 partition set를 assign될 수 있음. 이 경우 consumer는 각 partiton의 최근에 commit된 offset을 read하여 그 offset에서 consume을 이어감
    * 마지막으로 commit된 offset과 마지막으로 처리된 offstet이 다를 경우 data 처리의 정합성이 깨질 수 있음

  * Last committed offset  / Last Processed offset

    ![figure4-6](C:\Users\jungsol\Documents\kafka\pic\figure4-6.PNG)

    * commit된 offset이 client가 처리한 마지막 message의 last offset보다 작으면 마지막에 처리되던 offset의 message와 commit된 offset 사이의 messages는 2번 처리될 수 있음

    ![figure4-7](C:\Users\jungsol\Documents\kafka\pic\figure4-7.PNG)

    * commit된 offset이 client가 실제로 처리한 마지막 message의 offset보다 큰 경우 마지막으로 처리된 message의 offset과 commit된 offset 사이의 모든 messages는 consumer group에서 처리가 되지 않음
    * offset을 manage하는 것을 client application에 큰 영향을 주며 KafkaConsumer API는 다양한  offset을 commit하는 방식을 제공



### Automic Commit

* consumer가 poll()해서 받은 records 중 가장 큰 offset을 자동으로 commit
* enable.auto.commit configuration option을 true로 하여 automic commit을 사용할 수 있으며 auto.commit.interval.ms option의 value를 조정하여 automic commit 주기를 조정할 수 있음
* consumer의 다른것과 마찬가지로 automic commit도 poll loop에 의해 실행됨
  * poll을 할 때마다, consumer는 commit할 시간인지 확인
  * commit할 시간이라면 last poll에서 반환한 offset을 commit
* 한계점 : 중복 데이터 처리 발생
  * automic commit은 편리하지만 개발자가 중복된 메시지를 피하도록 충분히 control할 수 없음
  * automic commit 주기 사이에 kafka가 rebalance가 발생할 경우 마지막 commit이후 처리된 data는 2번 처리됨
  * commit interval을 줄임으로써 중복 데이터 처리가 발생하는 것을 줄일 수 있지만 가능성을 완전히 제거하는것은 불가능
  * poll()을 호출 한 후 다음 poll()을 호출 하기전에 데이터 처리를 완료하는 것이 중요하며, 일반적인 경우에는 문제가 되지않지만 exception처리나, poll lop을 종료할 때 주의해야 함

### Manual Commit

* 대부분의 개발자들은 messages의 손실가능성을 제거하고 중복처리되는 messages의 개수를 줄이기 위해 offset이 commit되는 시간에 더 많은 제어를 함
* Consumer는 timer를 기반으로 한 것이 아니라, developer가 원하는 point에서 current ffset을 commit할 수 있는 API를 제공 
  * public void commitSync()
  * public void commitSync(java.util.Map\<TopicPartition,OffsetAndMetadata\> offsets)
  * public void commitAsync()
  * public void commitAsync(OffsetCommitCallback callback)
  * public void commitAsync(java.util.Map\<TopicPartition,OffsetAndMetadata\> offsets, OffsetCommitCallback callback)
* auto.commit.offset=false
  * 해당 conifugration option을 fasle로 설정하면 application이 명시적으로 commit하라고 할 때만 commit



### Commit Current Offset (Synchronous commit)

* commitSync()

  * poll()에서 반환된 최신 offfset을 commit하고 어떠한 reason에 의해 commit이 실패하면 caller에게 throw exception
  * 모든 records를 처리한 후에 commitSync()를 call하여 last offset을 commit해야하며, 그렇지 않으면 messages가 손실 될 수 있음
  * rebalance가 trriger되면 최근 batch의 시작부터 rebalance time까지의 모든 messages가 두번 처리됨
  * synchronous commit이며 commit이 성공하거나 복구할 수 없는 error가 발생할 때 까지 block

* broker가 commit request에 respond하기 전까지 application이 block되므로 application의 throughput이 제한됨

  * throughput은 commit frequence를 낮추면 향상시킬 수 있지만, commit frequence를 낮추면 rebalance가 발생했을 때 중복 처리되는 데이터의 개수가 늘어날 수 있음
  * Asynchronous commit API로 해당 문제 해결 가능

* code sample

  ```java
  while (true) {
    ConsumerRecords<String, String> records = consumer.poll(100);
    for (ConsumerRecord<String, String> record : records) {
      System.out.printf("topic = %s, partition = %s, offset = %d, customer = %s, country = %s\n", record.topic(), record.partition(), record.offset(), record.key(), record.value());
    }
    try {
      consumer.commitSync();
    } catch (CommitFailedException e) {
      log.error("commit failed", e)
    }
  }

  ```

  1. 이 예제에서는 record의 contents를 print하여 data의 처리를 완료했다고 가정
     * application을 records를 이용하여 프로그램을 수정하거나 풍부하게하거나 집계하거나 dashboard에 표시하거나, 중요한 event를 알리는 등의 많은 작업을 수행 할 수 있으므로 use cases에 따라 완료한 시점을 결정해야 함
  2. 현재 batch의 모든 records를 처리하고 나면 추가적인 messages를 polling하기 전에 commitSyn()를 호출하여 last offset을 commit
  3. commitSync()는 nonretriable(복구할수없는) 오류가 발생하지 않는 한 commit을 rery하므로 commit이 Exception이 발생했을 경우는 exception을 logging하는 것 외에는 할 수 있는 일이 많지 않음



### Asynchronous Commit

* commitAsync()

  * commit request를 보내고 작업을 이어감

* Simple code

  ```java
  while (true) {
    ConsumerRecords<String, String> records = consumer.poll(100);
    for (ConsumerRecord<String, String> record : records) {
      System.out.printf("topic = %s, partition = %s,offset = %d, customer = %s, country = %s\n",record.topic(), record.partition(), record.offset(),record.key(), record.value());
    }
    consumer.commitAsync();
  }
  ```

  * last offset을 commit하고 carry on(계속 수행함)

* nonretriable failure가 발생하더라도 commit을 retry하지 않음

  * retry를 하지 않는 이유는 commitAsync()가 server에서 response를 받을 때 까지 이미 성공한 commit이 있을 수 있기 때문
  * offset을 2000으로 commit하려고 request하였는데 일시적인 communication문제가 발생하여서 broker는 결국 request를 받지 못하여 response하지 않았음. 그런데 이 사이에 다른 batch를 처리하고 offset을 3000으로 성공적으로 commit하였는데, commitAsync()가 이전에 실패한 commit을재시도하면 다시 offset이 2000이 되어버리므로, 1000개의 data가 중복처리되며, rebalance의 경우에는 더 많은 중복이 발생

* commitAsync()는 broker가 respond할 때 callback이 trigger될 수 있는 option을 제공하기 때문에, commit 순서의 complication과 importance를 신경써야 함

  * 일반적으로는 callback을 사용하여 commit error를 log하거나, metric을 계산하지만 retry를 위해 callback을 사용하려면 commit 순서와 관련한 문제점을 인식하고 있어야 함

* Sample Code : Using callback

  ```java
  while (true) {
    ConsumerRecords<String, String> records = consumer.poll(100);
    for (ConsumerRecord<String, String> record : records) {
      System.out.printf("topic = %s, partition = %s,offset = %d, customer = %s, country = %s\n",record.topic(), record.partition(), record.offset(),record.key(), record.value());
    }
    consumer.commitAsync(new OffsetCommitCallback() {
      public void onComplete(Map<TopicPartition, OffsetAndMetadata> offsets, Exception exception) {
        if (e != null) {
          log.error("Commit failed for offsets {}", offsets, e);
        }
      }
    });
  }
  ```

  * commit을 하고 계속 수행하다가, commit이 실패하면 exception과 offset을 log

* Retring Async Commits  

  * asynchronous retry를 위해 commit order를 올바르게 만드는 pattern은 monotonically increasing sequence number(단조 증가 시퀀스 번호)를 사용

  1. 매 commit마다 sequence number를 늘리고, commitAsync callback에 sequence number를 추가
  2. retry를 보낼 준비가 되면 callback에서 가져온 sequence number가 instance variable의 sequence number와 같은지 확인
     * 같으면, 새로운 commit이 없었으므로 retry해도 safe함
     * instacne sequence number가 더 높은 경우 newer commit이 이미 전송되었으므로 재시도 하지 않으면 됨



### Combining Synchronous and Asynchronous Commits

* retry 없는 commit의 실패는 가끔씩 발생하는 경우에는 다음 commit이 성공하기 때문에 큰 문제가 되지 않음

* 그러니, consumer를 닫기전에 혹은 rebalance되기 전에 last commit은 commit의 성공을 보장해야 함

  * 이를 위해 shutdown전에는 commitAsync()와 commitSync()를 combine하여 사용

* Sampele Code 

  ```java
  try {
    while (true) {
      ConsumerRecords<String, String> records = consumer.poll(100);
      for (ConsumerRecord<String, String> record : records) {
        System.out.printf("topic = %s, partition = %s, offset = %d,customer = %s, country = %s\n",record.topic(), record.partition(),record.offset(), record.key(), record.value());
      }
      consumer.commitAsync();
    }
  } catch (Exception e) {
    log.error("Unexpected error", e);
  } finally {
    try {
      consumer.commitSync();
    } finally {
      consumer.close();
    }
  }
  ```

  *rebalance listeners section에서 rebalance전에 commit하는 방법에대해 discuss. 여기서는 consumer close전에 commit하는 방법에대해서만 다룸*

  1. while loop안에서는 commitAsyn()를 사용
     * one commit이 실패하면 next commit으로 retry의 효과를 볼 수 있음
  2. consumer를 닫을 때는 next commit이 없으므로 commitSync()를 호출하여 성공 or unrecoverable exception이 발생할 때 까지 retry



### Commit Specified Offset

* 최신 offset을 commit하면 processing batches(일괄처리)를 끝낼때마다 commit이 발생하지만, 이보다 더 자주 commit을 하고 싶다면? poll()이 거대한 batch를 return하고 rebalance가 발생했을 때 처리된 데이터를 다시 처리하지 않도록 하기 위해서 batch의 중간에 offsets을 commit하려고 한다면? 

  * return된 last offset을 commit하는 commitSync()와 commitAsync()는 사용할 수 없음
  * commitSync()와 commitAsync()에 commit하고자 하는 map of partitions and offsets을 전달할 수 있음

* Sample Code

  ```java
  private Map<TopicPartition, OffsetAndMetadata> currentOffsets = new HashMap<>();
  int count = 0;

  ....

  while (true) {
    ConsumerRecords<String, String> records = consumer.poll(100);
    for (ConsumerRecord<String, String> record : records) {
      System.out.printf("topic = %s, partition = %s, offset = %d,customer = %s, country = %s\n",record.topic(), record.partition(), record.offset(),record.key(), record.value());
      currentOffsets.put(new TopicPartition(record.topic(), record.partition()), new OffsetAndMetadata(record.offset()+1, "no metadata"));
      if (count % 1000 == 0) {
        consumer.commitAsync(currentOffsets, null);
        //consumer.commitSync(currentOffsets, null);
      }
      count++;
    }
  }
  ```

  1. Map\<TopicPartition, OffsetAndMetadata\> currentOffsets = new HashMap\<\>()
     * 이 map은 manually(수동으로) offset을 track(추적)하는데 사용
  2. consumer한 records의 처리를 println문으로 stand-in(대체)
  3. 각 record를 읽은 후 offsets map을 처리하고자 하는 next message의 offset으로 update. 이 offset은 다음번에 read하고자하는 지점
  4. sample code에서는 1000개의 record마다 current offset을 commit하기로 결정
     * application에서는 time 혹은 records의 content을 기반으로 commit 가능
  5. 특정 offset으로 commit
     * public void commitSync(java.util.Map\<TopicPartition,OffsetAndMetadata\> offsets)
     * public void commitAsync(java.util.Map\<TopicPartition,OffsetAndMetadata\> offsets, OffsetCommitCallback callback)
     * 특정 offsets으로 commit할 때, error handling을 해야 함



### Rebalance Listeners

* consumer는 종료하기 전과 parttion rebalancing을 하기 전에 offset을 commit하기를 원함

* consumer가 종료나 partition rebalancing으로 인해 partition에 대한 ownership을 상실하려고 할 때,

  * 마지막으로 처리한 event의 offset을 commit 해야 함
  * consumer가 가끔씩만 처리하는 buffer를 관리하는 경우 해당 buffer에 누적된 events를 처리해야 함 (close file handles, close database connection 등)

* consumer API를 사용하면 partitions을 add하거나 remove할 때 code를 실행할 수 있음

  * subscribe() 메서드를 호출할 때 ConsumerRebalanceListener를 전달하여 작업을 수행 가능

* ConsumerRebalanceListener

  * implements할 수 있는 2개의 method 존재

  1. public void onPartitionsRevoked(Collection\<TopicPartition\> partitions)
     * consumer가 messages를 consume하는 것을 멈추고, rebalance가 시작되기 전에 호출됨
     * offset을 commit하며, 다음번에 이 partition을 가져오는 consumer는 어디서 시작해야하는지를 알 수 있음
  2. public void onPartitionsAssigned(Collection\<TopicPartition\> partitions)
     * partition이 broker에 reassign(재할당)되었지만, consumer가 messages를 consume하기 전에 호출됨

  * Sample Code 

    * partition의 ownership을 상실하기 전에 onPartitionRevoked()를 이용하여 offset을 commit
    * onPartitionsAssigned을 사용하는 예제는 다음 section

    ```java
    private Map<TopicPartition, OffsetAndMetadata> currentOffsets = new HashMap<>();
    private class HandleRebalance implements ConsumerRebalanceListener {
      public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
        
      }
      public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
        System.out.println("Lost partitions in rebalance. Committing current offsets:" + currentOffsets);
        consumer.commitSync(currentOffsets);
      }
    }
    try {
      consumer.subscribe(topics, new HandleRebalance());
      while (true) {
        ConsumerRecords<String, String> records = consumer.poll(100);
        for (ConsumerRecord<String, String> record : records) {
          System.out.printf("topic = %s, partition = %s, offset = %d, customer = %s, country = %s\n", record.topic(), record.partition(), record.offset(), record.key(), record.value());
          currentOffsets.put(new TopicPartition(record.topic(), record.partition()), new OffsetAndMetadata(record.offset()+1, "no metadata"));
        }
        consumer.commitAsync(currentOffsets, null);
      }
    } catch (WakeupException e) {
      // ignore, we're closing
    } catch (Exception e) {
      log.error("Unexpected error", e);
    } finally {
      try {
        consumer.commitSync(currentOffsets);
      } finally {
        consumer.close();
        System.out.println("Closed consumer and we are done");
      }
    }
    ```

    1. ConsumerRebalanceListener를 implements
    2. 이 예제에서는 new partition을 만들 때 해야할 일이 없으며 단지 consuming messages를 시작할 뿐임
    3. rebalancing으로 인해 partitin을 lose하는 경우에는, onPartitionsRevoked에서 현재 offsets을 commit해야 함
       * batch의 처리 중간에 partition이 revoke(제거)될 수 있기 떄문에 처리중인 batch의 가장 latest offsets이 아닌 현재 처리한 latest offset을 commit
       * offsets은 이미 처리된 events에 대한 것이기 때문에 lose할 partitions뿐 아니라 모든 partitions에 대해 offsets을 commit. (이는 전혀 harm하지 않음)
       * commit method는 commitSync()를 사용하여 rebalance되기 전에 offset이 확실히 commit되게 해야 함
    4. ConsumerRebalanceListener를 subscribe() method에 전달하면 consume에 의해 invoke(적용)됨



### Consuming Records with Specific Offsets

* Offsets을 지정하여 Consuming Records하도록 하는 API

  * public void seekToBeginning(java.util.Collection\<TopicPartition\> partitions)
    * partition의 시작부분에서 시작하여 messages를 consume
  * public void seekToEnd(java.util.Collection\<TopicPartition\> partitions)
    * partition을 끝까지 skip하고 new messages에서 시작하여 messages를 consume
  * public void seek(TopicPartition partition, long offset)
    * 특정 partitin의 특정 offset에서 시작하여 messgaes를 consume
    * 사용 예시
      * 몇개의 messages를 go back
      * time-sensitive application의 경우에는 더 relevant(관련높은) messages로 skip
      * Kafka 이외의 system에 offsets을 저장하는 경우

* public void seek(TopicPartition partition, long offset) Example

  * 예시 상황

    * Kafka에서 website에서 users의 clickstream와 같은 events를 read하고, users가 아닌 automated programs에서 clicks을 나타내는 records를 remove하는 것과 같이 data를 처리하여 결과를NOSQL or Hadoop등의 database에 저장하는 scenario가 존재

    * 어떠한 data도 잃고 싶지 않고, 동일한 result를 data를 database에 두 번 저장하고 싶지 않는다고 가정

    * Code Exmple1

      ```java
      while (true) {
        ConsumerRecords<String, String> records = consumer.poll(100);
        for (ConsumerRecord<String, String> record : records) {
          currentOffsets.put(new TopicPartition(record.topic(), record.partition()), record.offset());
          processRecord(record);
          storeRecordInDB(record);
          consumer.commitAsync(currentOffsets);
        }
      }
      ```

      * 위의 code의 한계점
        * record를 편집한 후 offsets를 commit
        * record가 database에 저장되었지만 offset이 commit되기 전에 application이 crash(중단)되어 record가 다시 처리되고 dtabase에도 중복되어 저장될 가능성이 존재
        * 하나의 atomc action에서 record와 offset을 저장하는 방법이 있다면 이와같은 문제를 피할 수 있음
          * records는 database에 저장되고, offsets은 kafka에 저장되는 한, records와 offsets이 둘 다 commit되거나 둘 다 commit되지 않는것이 보장되는 것은 불가능
          * record와 offset을 하나의 transaction에서 작성한다면 위와 같은 문제를 해결할 수 있음

    * record와 offset을 모두 database에 저장한다면

      * record와 offset이 모두 commit되거나 둘다 commit되지 않아 재처리하면 되는 것이 보장됨
      * 이 경우에는 consumer가 partition을 assign받았을 때 시작 offset의 위치를 알아내는 방법이 필요한데 이 때 seek()를 사용하면 됨

    * Sample Code

      ```java
      public class SaveOffsetsOnRebalance implements ConsumerRebalanceListener {
        public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
          commitDBTransaction();
        }
        public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
          for(TopicPartition partition: partitions) {
            consumer.seek(partition, getOffsetFromDB(partition));
          }
        }
      }

      consumer.subscribe(topics, new SaveOffsetOnRebalance(consumer));
      consumer.poll(0);
      for (TopicPartition partition: consumer.assignment()) {
        consumer.seek(partition, getOffsetFromDB(partition));
      }
      while (true) {
        ConsumerRecords<String, String> records = consumer.poll(100);
        for (ConsumerRecord<String, String> record : records) {
          processRecord(record);
          storeRecordInDB(record);
          storeOffsetInDB(record.topic(), record.partition(), record.offset());
        }
        commitDBTransaction();
      }
      ```

      * ConsumerRebalanceListener를 이용하여 database에 저장된 offset에서 process를 시작

      1. onPartitionsRevoked

         * commitDBTransaction()
           * database에 transaction을 commit하는 method로 가정
         * partition을 lose하려고 할 때, transactions을 commit하여 records와 offsets이 database에 insert되어 information이 유지되도록 하면 됨

      2. onPartitionsAssigned

         * getOffsetFromDB : database에서 특정 partition의 offset을 가져오는 method로 가정
         * new partitions에 대한 ownership을 가졌을 때, database로부터 해당 partition의 commit된 offset을 가져와서 처리할 record를 seek

      3. consumer가 시작되면, topics을 subscribe한 후에 poll()을 한번 호출하여 consumer group에 join하고 partitions을 assign받은 다음에, assign된 partitions의 correct offset을 seek

         * seek()는 consume할 position을 update하므로 poll()을 하여 messages를 fetch
         * offset이 존재하지 않는등의 seek()에 error가 있는 경우 poll()에 의해 throw exception

      4. storeRecordInDB(record);

         storeOffsetInDB(record.topic(), record.partition(),record.offset())

         commitDBTransaction()

         * database에 records와 offset을 저장하는 table을 update
         * records의 update가 빠르다고 가정하고, 매번 record를 update하지만 commits은 느리기때문에 batch가 끝날때만 commitDBTransaction
           * 이는 다른 방법으로 최적화 될 수 있음

      * 외부 저장소에 offsets과 data를 저장하여 정확하게 한번만 수행되도록 구현하는 방법에는 여러가지가 있지만, 모두 ConsumerRebalance Listener와 seek()를 사용하여 offstes을 시간내에 저장하고, consumer가 올바른 위치의 messages를 읽기 시작할 수 있도록 보장함




### But How Do We Exit?

* poll loop을 cleanly exit 하는 방법

  1. poll loop을 exit하기로 결심했다면, consumer.wakeup()을 call하는 another thread가 필요
     * main thread에서 consumer loop이 run중이라면, ShutdownHook에서 이를 수행 가능
     * consumer.wakeup()는 다른 thread에서 safe하게 호출할 수 있는 유일한 consumer method
  2. wakeup을 call하면 WakeupException과 함께 poll()이 exit되거나, thread가 poll()을 wait하지 않는 동안 consumer.wakeup()이 호출되었다면 enxt iteration에서 poll()이 호출될 때 throw Exception
  3. WakeupException은 처리할 필요가 없지만 consumer thread를 종료하기전에 consumer.close()를 반드시 호출해야 함
  4. consumer를 close하면 offsets이 commit되고, group coordinator에게 consumer가 group을 leave한다고 message를 보냄
  5. consumer coordinator는 message를 받자마자 즉시 rebalance를 trigger
     * close되는 consumer의 partition의 session이 timeout되기까지 기다릴 필요 없이 해당 consumer group의 다른 cnsumer에게 assign됨

* Sample Code

  * main application thread에서 consumer가 run중일 때, 종료되는 code
  * 아래의 code는 일부이며, 전체 코드는 아래 link에서 확인 가능
    * http://bit.ly/2u47e9A

  ```java
  Runtime.getRuntime().addShutdownHook(new Thread() {
    public void run() {
      System.out.println("Starting exit...");
      consumer.wakeup();
      try {
        mainThread.join();
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
  });

  ...
    
  try {
    // looping until ctrl-c, the shutdown hook will cleanup on exit
    while (true) {
      ConsumerRecords<String, String> records = movingAvg.consumer.poll(1000);
      System.out.println(System.currentTimeMillis() + "-- waiting for data...");
      for (ConsumerRecord<String, String> record : records) {
        System.out.printf("offset = %d, key = %s, value = %s\n", record.offset(), record.key(), record.value());
      }
      for (TopicPartition tp: consumer.assignment()) {
        System.out.println("Committing offset at position:" + consumer.position(tp));
      }
      movingAvg.consumer.commitSync();
    }
  } catch (WakeupException e) {
    // ignore for shutdown
  } finally {
    consumer.close();
    System.out.println("Closed consumer and we are done");
  }
  ```

  1. shutdownHook은 별도의 thread에서 run되기 때문에, 우리가 취할 수 있는 유일한 safe action은 wakeup()을 call하여 poll loop를 끝내는 것
  2. Another thread calling wakeup will cause poll to throw a WakeupException. (wakeup을 call하는 another thread가 poll에 throw WakeException을 유발)
     * application이 예기치않게 종료되지 않도록 exception을 catch하지만, 아무런 처리를 할 필요가 없음
  3. consumer를 종료하기 전에, cleanly 하도록 close해야 함



### Deserialize

* Introduce

  * Kafka producers는 object를 byte arrays로 serialize하여 Kafka로 보내야 했듯이,

    Kafka Consumer는 Kafka에서 받은 byte arrays를 Java objects로 deserialize해야 함

  * Serializer와 마찬가지로 기본 자료형에 대한 Deserializer는 built-in되어 있으나, Cutom type의 객체로 deserialize 하기 위해서는 Custom Deserilaizer 필요

  * Serializer와 Deserializer는 match되어야 함

    * 각 topic에 쓰인 serializers를 track해야 하며, 각 topic에는 사용하는 deserializer가 interpret(해석)할 수 있는 data만 포함되어있다는 것이 보장되어야 함
    * serialize와 deserialize에 Avro 및 Schema Repository을 사용하면, 특정 topic에 쓰여진 모든 data가 topic의 schema와 compatible(호환되는)하도록 보장. 즉, schema와 deserializer가 match되어 deserialize 할 수 있음
    * producer와 consumer의 compatibility(호환성) error는 적절한 error message로 쉽게 caught되므로, serialization errors에 대해 byte arryas를 debug할 필요 없음

* Custom Deserializers

  * 이는 덜 일반적인 방법으로 아래 section에서 다룰 Avro를 사용하여 deserialize하는 방식이 훨씬 많이 쓰임

  * org.apache.kafka.common.serialization.Deserializer interface를 implements

  * Customer object

    ```java
    public class Customer {
      private int customerID;
      private String customerName;
      public Customer(int ID, String name) {
        this.customerID = ID;
        this.customerName = name;
      }
      public int getID() {
        return customerID;
      }
      public String getName() {
        return customerName;
      }
    }
    ```

  * Sample Code : Custom Deserializer

    ```java
    import org.apache.kafka.common.errors.SerializationException;
    import java.nio.ByteBuffer;
    import java.util.Map;

    public class CustomerDeserializer implements Deserializer<Customer> {
      @Override
      public void configure(Map configs, boolean isKey) {
        // nothing to configure
      }
      @Override
      public Customer deserialize(String topic, byte[] data) {
        int id;
        int nameSize;
        String name;
        try {
          if (data == null) {
            return null;
          }
          if (data.length < 8) {
            throw new SerializationException("Size of data received by IntegerDeserializer is shorter than expected");
          }
          ByteBuffer buffer = ByteBuffer.wrap(data);
          id = buffer.getInt();
          String nameSize = buffer.getInt();
          byte[] nameBytes = new Array[Byte](nameSize);
          buffer.get(nameBytes);
          name = new String(nameBytes, 'UTF-8');
          return new Customer(id, name);
        } catch (Exception e) {
          throw new SerializationException("Error when serializing Customer to byte[] " + e);
        }
      }
      @Override
      public void close() {
        // nothing to close
      }
    }
    ```

    1. Customer class의 구현이 필요하며, producing applications과 consuming applications 모두에서 Cutomer class와 serializer class는 match 되어야 함
       * 많은 consumers와 producers가 data에 대한 access를 공유하는 대규모 organization에서는 이러한 match가 힘들 수 있음 (Avro 사용의 필요성)
    2. serializer의 logic을 reverse하여 byte array에서 customer ID와 name을 추출하여 object를 construct(생성)

  * Sampel Code : Consumer (Using custom deserializer)

    ```java
    Properties props = new Properties();
    props.put("bootstrap.servers", "broker1:9092,broker2:9092");
    props.put("group.id", "CountryCounter");
    props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
    props.put("value.deserializer", "org.apache.kafka.common.serialization.CustomerDeserializer");
    KafkaConsumer<String, Customer> consumer = new KafkaConsumer<>(props);
    consumer.subscribe("customerCountries");
    while (true) {
      ConsumerRecords<String, Customer> records =
        consumer.poll(100);
      for (ConsumerRecord<String, Customer> record : records) {
        System.out.println("current customer Id: " + record.value().getId() + " and current customer name: " + record.value().getName());
      }
    }
    ```

    1. key는 String type을 사용하고 value는 Customer ojbect를 사용

  * Conclusion

    * Custom serializer와 deserializer의 implementing은 권장되지 않음
      * producers와 consumers이 강하게 couple(결합)
      * fragile(허약)
      * error-prone(오류를 발생시키기 쉬움)
    * JSON, Thrift, Protobuf, Avro와 같은 standard message format을 사용하는 것을 추천

* Using Avro deserialization with Kafka consumer

  * Chapter3에서 구현한 Avro를 이용한 Customer class의 implementation을 사용하고 있다고 가정

  * Kafka에서 해당 object를 consume하기 위해서는 KafkaAvroDeserializer를 구현해야 함

    ```java
    Properties props = new Properties();
    props.put("bootstrap.servers", "broker1:9092,broker2:9092");
    props.put("group.id", "CountryCounter");
    props.put("key.serializer", "org.apache.kafka.common.serialization.StringDeserializer");
    props.put("value.serializer", "io.confluent.kafka.serializers.KafkaAvroDeserializer");
    props.put("schema.registry.url", schemaUrl);
    String topic = "customerContacts";
    KafkaConsumer consumer = new KafkaConsumer(createConsumerConfig(brokers, groupId, url));
    consumer.subscribe(Collections.singletonList(topic));
    System.out.println("Reading topic:" + topic);
    while (true) {
      ConsumerRecords<String, Customer> records = consumer.poll(1000);
      for (ConsumerRecord<String, Customer> record: records) {
        System.out.println("Current customer name is: " + record.value().getName());
      }
      consumer.commitSync();
    }
    ```

    1. props.put("value.serializer", "io.confluent.kafka.serializers.KafkaAvroDeserializer")
       * KafkaAvroDeserializer를 이용하여 Avro messages를 deserialize
    2. props.put("schema.registry.url", schemaUrl)
       * shemas를 저장하는 위치를 가리킴
       * consumer가 producer에 의해 register된 schema를 사용하여 message를 deserialize
    3. ConsumerRecords\<String, Customer\> records = consumer.poll(1000);
       * deserializer에 의해 생성된 Customer type을 record의 value로 지정
    4. record.value().getName()
       * record.value()는 Customer instance이므로, 이를 적절하게 사용 가능

  ​

### Standalone Consumer: Why and How to Use Consumer Without a Group

* 특정 topic의 모든 partitions 혹은 특정 partition에서 항상 data를 읽어야 하는 single consumer가 있을 수 있음

  * 이 경우에는 때때로 groups 혹은 rebalances가 consumer 에게 특정 topic, partitions일 assign하고 messages를 consume하고 offsets을 commit할 필요가 없음
  * rebalance가 안되며, partitions을 수동으로 find해야 하는 것 외의 다른 모든 business는 usal(보통과같음)

* consumer가 어떤 partitions을 read해야하는지 정확하게 알고있다면, tpic을 subscribe하지 말고  몇개의 partitions을 assign

* topic에 new partitions을 추가하더라도 consumer는 알 수 없음

  * consumer.partitionsFor()을 주기적으로 또는 partition이 추가될때마다 application을 bouncing 함으로써 간단하게 check하여 topic에 new partitions이 추가되는 것을 처리해야 함

* consumer는 topics을 subscribe하여 consumer group에 속하거나, partitions자체를 assign할 수 있지만, 둘 다 동시에 할 수는 없음

* Sample Code

  * 특정 topic의 모든 partitions을 assign하고 consumer

  ```java
  List<PartitionInfo> partitionInfos = null;
  partitionInfos = consumer.partitionsFor("topic");
  if (partitionInfos != null) {
    for (PartitionInfo partition : partitionInfos) {
      partitions.add(new TopicPartition(partition.topic(), partition.partition()));
    }
    consumer.assign(partitions);
    while (true) {
      ConsumerRecords<String, String> records = consumer.poll(1000);
      for (ConsumerRecord<String, String> record: records) {
        System.out.printf("topic = %s, partition = %s, offset = %d, customer = %s, country = %s\n", record.topic(), record.partition(), record.offset(), record.key(), record.value());
      }
      consumer.commitSync();
    }
  }
  ```

  1. partitionInfos = consumer.partitionsFor("topic")
     * clsuster에게 topic의 available partitions를 요청
       * 특정 partition을 사용하려고 한다면 이 부분은 skip가능
  2. consumer.assign(partitions)
     * assign하고 싶은 TopicPartition object의 list와 함께 assign()을 call하여 assign



### Older ConsumerAPIs

* Older Consumer (=SimpleConsumer, high-level consumer, ZookeeperConsumerConnector)

  * New Consumer(Current Consumer)
    * org.apache.kafka.clients package에 있는 Java KafkaConsumer client
  * Apache Kafka는 2개의 older cleints 존재
    * kafka.consumer package의 부분인 Scala로 작성된 older client
    * core Kafka module인 older client
  * SimpleConsumer
    * 실제로 simple하지는 않음
    * SimpleConsumer는 특정 partitions과 offsets으로부터 consume할 수 있는 Kafka APIs의 thin wrapper
  * high-level consumer or ZookeeperConsumerConnector
    * consumer groups을 사용하고, consumer group이 partitions을 rebalances한다는 점에서 high-level consumer는 current consumer와 비슷하지만, Zookeeper를 이용하여 consumer progups을 관리하고, commit과 rebalances를 current consumer처럼 control할 수 없음
  * current consumer는 두가지 behaviors를 모두 지원하고, developer에게 훨씬 더 많은 reliavility(안정성)과 control(제어기능)을 제공하기 떄문에 older consumers에 대해서는 다루지 않음

  ​

### Summary

* in-depth explanation of Kafka's consumer groups
* the way they allow multiple consumers to share the work of reading events grom tpics
* example of a consumer subscribing a tpic and continuously reading evetns
* consumer configuration parameters
* how consumer cnfiguration parameters affect consumer behavior
* discuss offset
* how consumers keep track of offsets
* how consumers commit offsets for reliable consumers
* handling rebalances
* closing the consumer

