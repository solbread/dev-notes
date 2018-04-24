## Chapter 6. Reliable Data Delivery

### Introduce

* Reliable data delivery는 system의 attributes 중 하나
* performance와 마찬가지로 whiteboard diagram에서 system으로 design되어야 함
* reliability는 single component가 아닌  system 요소이므로 Apache Kafka의 reliability guaramtees에 대해 논의할 때 Kafka를 사용하는 전체 system과 use case를 고려해야 함
  * application architecture, application에서 사용하는 producer와 consumer APIs의 방법, producer와 consuer의 configuration, topic configuration, broker configuration을 포함하여 전체적으로 reliable system을 만들어야 함
  * application complexity, performance, availability, disk-space usage를 trade-offs해야 함
* reliability 측면에서 Kafka와 integrate(통합)되는 system은 Kafka 자체만큼이나 중요함
* Reliability는 system concern(관심사)이기 때문에, 한사람만의 책임이 아닌 Kafka administrators, Linux administrators, network and storage administrators, and aaplication developers 모두가 reliable system을 위해 협력해야 함
* Apache Kafka는 reliable data delivery(신뢰성 있는 데이터 전달)에 대해 very flexible
  * Kafka는 많은 use cases가 있으며, 일부 use cases는 많은 reliability를 필요로 하지만 다른것들은 reliability보다 spped와 simplicity를 더 우선시함. 
  * Kafka는 충분히 configurable하도록 작성되어 있으며, client API는 모든 종류의 reliability를 trads-off할 정도로 flexible
  * flexibility로 인해 Kafka를 사용할 때 실수로 shoot yourself in the foot(제 발등 찍는) 경우도 있음. 



### Reliability Guarantees

* ACID

  * 가장 잘 알려진 reliability guarantee로, relational database가 universally(보편적으로) support하는 standard reliability guarantee (표준 신뢰성 보장)
  * Atomicity(원자성), Consistency(일관성), Isolation(격리), Durability(내구성)을 의미
  * database가 ACID를 compliant(준수)한다고 하면, database가 transaction behavior과 관련하여 특정 behaviors를 guarantee
    * 이러한 guarantee는 system이 다른 conditions(조건)에서도 system이 promise(약속)한 것과 어떻게 behave할지를 알고 있으므로 사람들이 critical applications의 relational database를 신뢰하는 이유임
  * guarantees를 이해하고 guarantees에 rely on(의존)하여 safe application을 write할 수 있음

* Kafka가 제공하는 guarantees에 대한 이해는 reliable applications을 build하는 사람들에게 중요하며, 이러한 이해는 System developers가 different failure conditions에서 어떻게 system이 작동할지를 파악하게 해 줌

* Apache Kafka가 guarantee하는 것들

  1. order messages in a partitions (partitin내의 messages들의 순서)
  2. produce된 messages는 partition의 모든 in-sync replicas에 write되었을 때 commit되었다고 간주
     * 반드시 disk로 flush되는 것은 아님
     * producer는 보낸 message의 acknowledgements의 수신을 아래의 선택지에서 선택 가능
       1. message가 모두 commit되었을 때 (ack=all)
       2. message가 leader에 wrtie되었을 때 (ack=1)
       3. message가 network상에 전송되었을 때 (ack=0)
  3. 하나 이상의 replica가 살아있는 한 commit된 messages는 손실되지 않음
  4. conumsers는 commit된 messages만 read 가능

  * 이러한 기본적인 gurantees는 reliable system을 build하는데 사용할 수 있지만 그 자체로는 system을 완전히 reliable할 수 없음
  * reliabe system을 build하기 위해서는 trade-offs(절충안)가 필요하며, Kafka는 administrators와 developers에게 configurate할 수 있는 parameters를 제공하여 필요한 reliability를 결정할 수 있도록 함
  * reliably(안정/신뢰적)이고 , cnsistently(일관적)이게 message를 저장하는 것이 availability, high throughput, low latency, hardware costs와 비교하여 얼마나 중요한지를 trade-off(절충)해야 함



### Replication

* Introduce Replication mechanism
  * partition당 multiple relplicas를 사용하는 Kafka의 replication mechanisum은 Kafka의 reliability guarantees에 핵심
    * multiple replicas에 write된 message를 갖는 것은 Kafka가 crash되었을 때도 messages의 durability(내구성)를 제공
* recap the highlights of replication mechanism(복제 메커니즘의 주요사항 요약)
  * kafka tpic은 partitions으로 나뉨
  * partition을 single disk에 저장
    * kafka는 partition내의 evnets의 순서를 guarantee하고, partition의 상태는 online(available) of offline(unavailable)일 수 있음
  * 각 partition에는 multiple replicas이 있을 수 있으며, 그 중 하나는 leader로 지정됨
  * 모든 evnets는 leader replica에서 produce되고 consume됨
  * 다른 replicas는 leader와 in sync상태를 유지하고 최신 events를 replicate해야 함
  * leader가 unavailable 상태가 되면, in-sync replicas 중 하나가 new leader가 됨 
* In-Sync Replicas
  * partitions의 leader는 in-sync
  * follower는 in-sync하기 위해 아래의 조건을 만족해야 함
    * Zookeeper가 configurational time(defulat=6s)안에 heartbeat를 보냄
    * Fetched messages from the leader in the last 10 seconds (configurable). (configurational time(default=10s)안에 messages를 leader로부터 fetch했었음)
    * Fetched the most recent messages from the leader in the last 10 seconds. (configurational time(default=10s)안에 leader로부터의 가장 최신 messages를 fetch했었음)
      * follower가 leader로부터 messages를 가져오고있다는 것만으로는 충분하지 않으며 lag(밀린데이터)가 없어야 함
* Out-of-Sync Replicas
  * 하나이상의 replicas가 in-sync 와 outof-sync 상태를 빠르게 전환되면 이는 cluster에 문제가 있다는 신호
    * 이러한 현상의 가장 흔한 원인은 broker의 Java's garbage collection이  misconfiguration(잘못구성)하고있는 경우
    * Misconfiguration garbage collection은 broker가 few seconds(수 초)동안 일시중지 될 수 있으며, 그 동안 Zookeeper와의 connectivity가 끊어짐
    * broker가 Zookeeper와의 connectivity가 끊어지면 cluster와 out-of-sync 상태로 간주됨
* In-sync replicas의 수에 따른 성능이슈?
  * 조금 뒤쳐저 있는 in-sync replica는 prducers와 consumers를 느리게 할 수 있음
    * producers와 consumers는 commit하기 전에 in-sync replicas가 message를 다 얻기를 기다리기 때문
  * out-of-sync replica가 뒤쳐져 있다면 producer와 consumer의 performance에 영향이 없음
    * producer와 consumer가 out-of-sync replica가 messages를 얻기까지 기다리지를 않기 때문
  * in-sync replicas의 수가 적으면 partition의 effective(유효한) replication factor가 낮아지므로 downtime이나 data loss(손실)에 대한 위험이 높아짐



### Broker Configuration

* Introduce
  * reliable message storage와 관련하여 Kafka의 behavior를 변경하는 broker의 3가지 configuration이 paramters가 있음
    * Replication Factor, Unclean Leader Election, Minimum in-Sync Replicas
  * 대부분의 broker cnfiguration variables와 마찬가가지로, broker level, topic level에서 제어 가능
    * topic level에서 reliability trade-offs를 제어할 수 있으므로, 같은 Kafka cluster에서 reliable topics과 onnreliable topics을 별도 configuration 할 수 있음
* Replication Factor
  * configuration parameter
    * broker level configuration parameter : default.replication.factor
    * topic level configuration parameter : replication.factor
    * default value =1
    * topic을 생성할 때 replication의 개수
    * topic을 생성한 후에도 replication.factor parameter를 변경하여 replicas를 추가 혹은 삭제 할 수 있음
  * n개의 replication factor를 사용하면 n-1개의 broker가 손상되더라고 topic의 data를 여전히 reliable(신뢰적)으로 read하고 write할 수 있음
  * 1개 이상의 replicas가 in-sync 상태로 남아있다면 Kafka가 data loss를 보장
  * higer replicatin factor
    * higher availability
    * higher reliability
    * fewer disaster
    * lower availability for hardware
      * trade-off availability for hardware
      * replication factor이 n이면 n개의 brokers가 필요하며 n개의 data 사본을 저장해야 하며, n배의 disk space가 필요
  * replication factor의 결정
    * topic이 high availability를 위해
    * replication factor =1
      * cluster의 일부인 single borker가 다시시작할 때, 특정 topic을 unavailable해도 괜찮을 때
    * replication factor = 2
      * 하나의 broker가 손상되어도 괜찮음
      * but keep in mind that losing one broker can sometimes send the cluster into an unstable state, forcing you to restart another broker—the Kafka Controller. (하나의 broker는 가끔 cluster의 unstable state를 만들며, 이 경우 Kafka Controller인 another broker를 강제로 restart해야 함)
      * replication factor가 2인 경우 operational issue에서 recover하기 위해 unavailability가 될 수 있음
    * replication factor = 3
      * availability가 issue가 되는 topic에서 권고하는데, 드문case에서 충분히 안전하지 못하므로 5개를 사용하기도 함
  * Placement of replicas
    * 기본적으로 Kafka는 partitin의 replica가 separate broker에 있는지를 확인
      * 어떤 경우에는 이것으로만 충분히 safe하지 않음 (rack level에서의 availiability가 필요)
    * rack-level abailability
      * partition의 모든 replicas가 동일한 rack에 있고, top-of-rack(최상위) swtich를 misbehaves(잘못작동)하면, 모든 replicas가 availability를 잃게됨
      * broker를 multiple racks에 배치하고, broker.rack의 broker configuration parameter를 사용하여 각 broker의 rack name을 configure하는 것이 좋음
        * rack name을 configure하면 Kafka는 더 높은 availability를 guarantee하기 위하여 partitin의 replicas가 여러 racks에 분산되어 있는지를 확인
      * broker.rack
        * broker level configuration parameter
        * defualt value = null
        * rack aware replication assignment for fault tolerance (내결함성을 위한 렉 인식 복제 할당에 사용됨)


* Unclean Leader Election

  * configuration parameter

    * broker level의 parameter이며 실제로는 cluster-wide level
    * unclean.leader.election.enable
    * default value = false
      * Kafka 0.11.0 이전에는 true이며, 0.11.0 이후로 availability보다 durability를 선호하여 false
    * out-of-sync replicas가 leader가 되는 것(unclean leader election)을 허용할지 허용하지 않을지를 제어하는 configuration
      * true이면 unclean leader election을 허용

  * Clean Leader Election

    * partition의 leader가 더이상 available할 수 없을 때, in-sync replicas 중 하나가 leader로 선출되는 경우
    * leader election은 commit된 data는 모든 in-sync replicas에 존재하므로 손실이 없음을 guarantee하기 때문에 clean

  * Leader가 아닌 in-sync replica가 존재하지 않는 상황이 발생하는 2가지 시나리오

    1. partition이 3개의 replicas가 있었고 2개의 followers가 unavailable(crashed) 되었을 때, producers가 계속해서 leader에게 write를 하면 leader도 하나의 in-sync replica이기 때문에 모든 messages는 acknowledge되고 commit됨. 이 상황에서 leader가 unavailable하게 됨. 이 상황에서 out-of-sync followers 중 하나가 먼저 start되면 out-of-sync replica는 partition에 대한 유일한 available replica
    2. partition에 3개의 replicas가 있었고, 2개의 followers가 network issue로 뒤쳐져서 replicate하고 있지만 in-sync하지 않음. 그래서 leader는 유일하게 in-sync replica로 유지되는데 만약 leader가 unavailable하게 되면 2개의 available replicas는 더이상 in-sync가 아님
       * out-of-sync replica가 new leader가 될 수 있도록 허용하면 replica가 out-of-sync인 상태에서 leader replica에 쓰여진 messages가 다 손실되며, 이는 consumers에게 inconsistency(불일치)가 발생

  * Leader가 아닌 in-sync replica가 존재하지 않는 경우에 내릴 수 있는 2가지 결정

    1. leader가 available될 때 까지 대기

       * lower availability (기존의 leader가 available해 질 때까지 기다려야 하므로)
       * out-of-sync replica는 new leader로 허용하지 않으면, in-sync replica인 old leader가 online으로 될 때 까지 partition은 offline으로 남아있음. (memory chip의 교체가 필요한 경우 등의 일부 cases에는 시간이 많이 걸릴 수 있음)
    2. out-of-sync인 replica를 new leader로 선출 (Unclean Leader Election)
       * data loss(손실) 및 inconsistency(불일치)가 발생
       * follower인 replica0, replica1을 available하는 동안 leader인 replica2에 offsets100~200의 messages를 write하였다고 가정하고, 3개의 replica가 unavailable하게 되고, replica0이 online으로 돌아오면 replica0은 offsets0~100의 messages만 가지며 100~200은 없음. 만약 replica0이 new leader가 되면, producer는 new messages를 write하고 consumers는 new messages를 read함. 이 new leader는 완전히 새로운 offsets 100~200의 messages를 얻음. 몇 consumers는 old offsets100~200의 messages를 read하였었고, 몇 consumers는 new offsets 100~200의 messages를 read하였고, 또 일부 consumers는 old offsets 100~200의 messages와 new offsets 100~200의 messages를 섞어서 read하였을 수 있음. 이는 downstream reports와 같은 것을 볼 때 매우 나쁜 결과를 초래함. 그 후 replica2가 online으로 돌아와서 new leader가 되면, 이 시점에서 it will delete any messages it got that are ahead of the current leader. Those messages will not be available to any consumer in the future.  (new leader는 current leader 앞에 있는 모든 messages를 삭제. 이러한 messages는 앞으로 어떠한 consumers에게도 전달이 되지 않음) 



  * Unclean Leader Election

    * disable
      * data의 quality와 consistency의 durability가 중요한 경우
      * 은행 시스템 (거래가 잘못되는것이 중단되는것보다 더 위험)
    * able
      * availability가 중요한 경우
      * real-time clickstream analysis

* Minimum In-Sync Replicas

  * configuration parmeter
    * broker level : min.insync.replicas
    * topic level : min.insync.replicas
    * default vaue : 1
    * topic의 partition에서 in-sync 되어야 하는 replica의 최소 개수
  * configure 된 개수보다 적은 in-sync replica를 가질 경우
    * Producer는 NotEnoughReplicasException을 수신
      * broker는 produce requests를 accept하지 않음
    * consumer는 존재하는 data를 계속 read할 수 있음
      * in-sync replicas가 read-only가 됨
      * This prevents the undesirable situation where data is produced and consumed, only to disappear when unclean election occurs. (unclean election occurs가 발생하면 data가 사라지므로, data가 produce되고 consume되는 undesirable(바람직하지않은) situation을 막음)
    * unavailalbe partitions을 available하게 만든 후 (아마 broker를 재시작하는 방법), 해당 replica가 in-sync될 때까지 기다려서 해당 partition이 ISR(in-sync replica)의 최소 개수를 충족하여 read-only에서 복구되게 해야 함



### Using Producers in a Reliable System

* Introduce

  * most reliable cnfiguration으로 broker를 configure하더라도, producer가 reliable하지 않으면 system이 data를 잃을 수 있음 (아래에 해당 예시 2가지)
    * 3개의 replicas를 configure하였으며, unclean leader election을 disble로 하여 broker의 availability를 높였기 때문에, kafak cluster에 commit된 message를 잃어서는 안됨. 그러나 producer가 acks=1로 하여 messages를 보내도록 configure되었다면, producer가 보낸 messages가 leader에는 write되었지만 in-sync replicas에는 write되지 않았을 때 leader는 "Message was written sucessfully"라고 respond하며, 이 즉시 다른 leader replica가 crash되어 다른 replicas에게 messages가 replicate되지 않음. out-of-sync replica로 declare(선언)하기 전까지 시간이 걸리므로 여전히 다른 replicas는 in-sync상태이므로 이 중 하나가 leader replica가 됨. messages는 replicas에 write되지 않았으므로 messages는 손실되지만, producer는 성공적으로 write되었다고 생각함. consumer는 messages를 보지 못하지만 replica에 messages가 commit된적이 없기 때문에 system의 consistency(일관성)은 유지되지만 producer의 입장에서는 data가 손실됨
      * consystency는 broker에 쓰인 모든 messages를 모든 consumer가 read하는 것
      * reliability requirements를 충족시키는 올바른 ack configuration을 사용해야 함
    * 3개의 replicas를 configure하였으며, unclean leader election을 disble로 하여 broker의 availability를 높였기 때문에, kafak cluster에 commit된 message를 잃어서는 안됨. 첫번째 예제로인해 acks=all로 producer가 messages를 보내기 시작함. Kafka에 messages를 write하려고 시도하고 있지만 partitin에 대한 leader는 crash되었고, 새로운 leader를 선출하고 있음. 이 경우 Kafka는 "Leader not Available"로 respond하며 이 시점에서 producer가 error를 올바르게 처리하지 못하고, 성공적으로 write를 할 때 까지 retry를 하지 않으면 messages가 손실될 수 있음. broker가 messages를 받지 못하였기 때문에 broker reliability 문제가 아니며, consumer도 messages를 받지 못했기 때문에 consistency 문제가 아님. 그러나 producer가 error를 제대로 처리하지 못하여 message의 손실이 발생한 것
      * configuration과 code에서 error를 올바르게 처리해야 함
  * reliable message storage와 관련하여 Kafka의 behavior를 변경하는 producer의 3가지 고려사항이 있음
    * Send Acknowledgments, Configuring Producer Retries,  Additional Error Handling

* Send Acknowledgments

  * acks = 0

    * producer가 network를 통해 message를 보내기만 하면, Kafka에 message를 성공적으로 write한 것으로 간주
      * 전송중인 object를 serialize할 수 없거나, network card에 결함이 있을 경우 error가 발생되지만, partition이 offline이거나, 전체 Kafka cluster가 긴 vacation중이라면 error가 발생되지 않음
    * very fast
    * higher throughput
    * utilize most of bandwidth (대부분의 대역폭을 사용)
    * lower reliability
      * leader가 unavailable한 것을 알지 못하므로 clean leader election을 하더라도 message가 유실될 수 있음

  * acks = 1

    * message를 받고 partition data file에 write data를 write한 순간에 leader가 acknowledgement 또는 error를 보냄
      * disk에 sync된다는 것을 의미하지는 않음
    * leader election중이면, producer는 LeaderNotAvailableException을 받으며, producer가 이 error를 정상적으로 처리한다면 message를 retry하므로 message는 안전하게 new leader에 도착
    * message가 성공적으로 leader에 write되고, acknowledge되었으나, followers에 replicate되기 전에 leader가 crash되면 data가 유실됨

  * acks = all(-1)

    * configurable value만큼의 in-sync replicas에서 message를 replicate 한 후에 leader가 acknowledgment 혹은 error를 respond

    * min.insync.replica configuration parameter의 value를 조절하여, acknowledgement를 보내기 전에 messages를 받는 replicas의 개수를 제어 가능

    * prducer는 완전히 commit되기 전에 sending message를 종료하지 않음

    * higher reliability

    * most slowest

      * prducer는message betch를 "done"으로 하고 이어나가기 전에, 모든 replicas가 messages를 가져오기를 기다림
      * producer를 async mode로 사용하고 큰 batches를 보내면 완화할 수 있음

    * lower throughput

      ​

> * producer의 error handling은 2가지 부분이 있음
>
>   * producer가 자동적으로 handle  -> Configuring Producer Retries
>   * developer가 producer library를 사용하여 처리 -> Additional Error Handling
>
> * Producer가 broker에게 messages를 반환하면 broker는 success 또는 error code를 반환
>
> * error code의 2가지 종류
>
>   1. retriable error
>
>      - 재시도하면 해결될 수 있는 error
>
>
>      - LEADER_NOT_AVAILABLE
>
>     2. nonretriable error
>
>      - 재시도해도 해결되지 않는 error
>
>
>      - INVALID_CONFIG 



* Configuring Producer Retries

  * producer가 broker에게 messages를 send하면 broker는 success 또는 error code를 반환
  * messages를 손실하지 않기 위해서는, retriable error가 발생했을 때, messages를 계속해서 다시 send하도록 producer를 configure 하는 것이 좋음
    * leader 또는 network connectivity 문제는 해결되는데 a few second(몇 초)정도 걸릴 수있으며, 만약 prducer가 이 시간동안 계속해서 try를 할 경우, 이 시간 후에는 자동으로 data가 재전송 되므로, 이러한 문제를 직접 처리할 필요가 없음
  * retries
    * producer configuration
    * default value = 0
    * value range = [0,...,2147483647]
    * Retriable Error가 발생했을 때, producer가 자동으로 retry하는 횟수
  * retry 횟수의 선정 방법
    * producer가 n번 retry 후에 throw exception을 하였을 때, 수행할 계획에 따라 달라짐
      * exception을 catch한 후에 또다시 retry할 것이라면 retries를 늘려야 함
      * exception을 catch한 후에 message를 그냥 버리거나 별도로 저장한 후 나중에 처리할 것이라고 한다면 retries를 늘릴 필요 없음
      * e.g) Kafka의 cross-DC replication tool(MirrorMaker)는 매우 reliable replication tool로서, messages를 drop하면 안되기 때문에, 끝없이 retry하도록 retries=MAX_INT로 configure됨

* Additional Error Handling

  * built-in된 producer retries를 사용하면 messages의 손실 없이 다양한 종류의 errors를 처리할 수 있지만 아래와 같이 developer가 직접 처리해야 하는 errors도 존재
    * Nonretriable broker errors (message size, authorization errros 등)
    * broker에 mssages를 보내기 전에 발생하는 errors (serialization error 등)
    * producer가 retry attempt를 모두 소비했을 때
    * when the available memory used by the producer is filled to the limit due to using all of it to store messages while retrying (available meomry가 limit까지 차는 경우)
  * 위와 같은 error handling
    * error handlers를 작성하여 application의 목표에 맞도록 error를 처리
      * messages를 drop? error를 log? local disk의 directory에 message를 store? another application으로 callback을 trigger? 등 system architecture에 따라서 결정
    * 모든 error hanlder에서 message를 재전송하려면, producer's의 retry functionality에 의존하는 것이 좋음

* message 중복처리 문제 (Kafka 0.11.0 version 이전)

  * 실패한 messages를 다시 보내는 시도는, 두 메세지가 모두 broker에 성공적으로 write되어 중복으로 이어질 위험이 있음 ()
    - e.g) network 문제로 broker의 acknowledgmetn가 producer에 전달되지 못했지만, message가 성공적으로 write되고 replicate되었을 때, producer는 임시적인 network문제로 acknowledgmetn를 받지 못하고, sending message를 retry함
  * Retries 및 careful error handling으로 각 message가 적어도 한 번 저장되는것은 guarantee되지만, Apache Kafka(0.10.0)의 version에서는 message가 정확하게 한 번 저장되는 것을 보장할 수 는 없음
    - 이로인해, 많은 application은 각 message마다 unique한 identifier를 사용하여 messages를 consume할 때 duplicates(중복)을 감지하고 clean(정리)할 수 있음
    - 위의 방법을 사용하지 않는 다른 application은 message를 idempotent(멱등원)으로 만들어, 같은 message가 2번 보내지더라도 correctness(정확성)에 부정적인 영향을 미치지 않도록 함
      - idempotent(멱등원) : 연산을 여러번 적용하더라도 결과가 달라지지 않는 성질
      - "Account value is 110$"는 여러번 전송하더라도 결과가 달라지지 않으므로 idempotent(멱등원)
      - "Add $10 to the account"는 보낼떄마다 결과가 달라지므로 idempotent(멱등원)이 아님

* idempotent delivery (Kafka 0.11.0 version 이후)

  *(Kafak docs 4.6 Message Delivery Semantics 보고 정리함)*

  * message delivery guarantees의 종류
    1. At most once
       * Messages may be lost but are never redelivered. (mesage가 재배달 되지 않지만 손실될 수 있음)
    2. At least once
       * Messages are never lost but may be redelivered. (message가 손실되지 않지만 재배달될 수 있음)
    3. Exactly once
       * this is what people actually want, each message is delivered once and only once. (message는 한 번만 배달됨)
  * 0.11.0.0 이전 version에서는 producer가 message가 commit되었다는 response를 받지 못한 경우, message를 다시 보낼 수 밖에 없었음 -> At-Least-Once delivery semantics
  * 0.11.0.0 이후 version에서는 producer가 idempotent delivery option을 지원하여 전체 log에 중복이 발생하지 않도록 함 -> Exactly-Once delivery semantics
    * broker는 각 producer에게 ID를 assing하고, producer가 모든 message를 sequence number와 함께 전송하여 duplicate를 제거
    * producer는 transcation과 같은 semantics를 이용하여 multiple topic partitions에 messages를 보내는 기능을 지원하므로, 모든 messages가 write되거나 write되지 않음



### Using Consumers in a Reliable System

* Introduce

  * Kafka는 모든 in-sync replicas에 write하여 commit이 끝난 후
    * consumer는 data를 available할 수 있음
    * consumer가 consistent(일관성)이  guarantee(보장)되는 data를 얻음
  * Consumer는 batch of events를 fetch할 때, batch의 마지막 offset을 확인하고, 이 offset에서 시작하는 또 다른 batch of event를 request하여 어떠한 event도 놓치지 않고 올바른 순서로 새로운 data를 얻을 수 있음
  * Commit Offsets
    * Consumer가 멈춘다면 해당 consumer가 처리하던 partitions을 처리할  consumer는(다른 consumer 일수도 있고, 재시작된 기존 consumer일 수도 있음) 어디에서 시작해야 할 지를 알아야 하므로, 이전 consumer가 중단되기 전에 마지막으로 처리한 offset에 대한 정보가 필요함
    * Consumer는 consume하는 각 partition에 대해 현재 위치를 저장하므로 다시 시작한 후에 어디에서 시작하면 되는지 알 수 있음
  * Consumer의 data 손실
    * read하였지만 아직 처리되지 않은 events에 대해 offsets를 commit한 경우에 data 손실 발생
    * offsets을 commit하는 시기와 방법에 주의해야 함
  * Committed Message VS Committed Offsets
    * Committed Message
      * produce requests에 의해 Broker가 commit
      * 모든 in-sync replicas에 write되어 consumer가 available할 수 있는 messages
    * Committed Offsets
      * Consumer가 commit
      * Consumer가 partition의 특정 offset까지의 모든 messages를 receive하고 처리했음을 acknowledge(확인)하기 위하여 Kafka에게 보내는 offset

* Consumer Configuration

  * reliable consumer를 위한 4가지 configuration
    * group.id, auto.offset.reset, enable.auto.commit, auto.commit.interval.ms

  1. group.id
     * 동일한 group.id를 갖고 같은 topic을 subscribe하는 consumers는 해당 topic의 partitions의 subset이 assign됨
       * 해당 topic의 모든 messages는 group전체에 의해 나뉘어져서 읽어짐
     * consumer가 subscribe하는 topics의 모든 messages를 읽어야 하는 경우, unique한 group.id 사용
  2. offset.reset
     * commit된 offsets이 없거나, broker에 존재하지 않는 offsetes을 요청하였을 때 consumer가 읽을 offset을 제어
     * earliest를 선택하면
       * valid offset이 없을 때마다 partition의 첫번째 data부터 시작
       * consumer는 많은 양의 data를 두번 처리할 가능성이 있음 (중복처리)
       * data 손실이 최소화됨
     * latest를 선택하면
       * valid offset이 없을 때 마다 partition의 마지막 data에서 시작
       * consumer는 중복 처리하는 것을 최소화
       * data의 손실이 발생할 가능성이 높음
  3. enable.auto.commit
     * auto commit을 사용하면
       * consumer가 schedule에 따라 자동으로 commit됨
       * consumer 구현이 간단
       * consumer poll loop안에서 consume된 모든 record를 처리하면 처리하지 않은 offset이 commit되지 않음
       * 중복 처리되는 데이터가 발생할 수 있음
         * consumer가 일부 records를 처리하고 중단이 되면 commit이 되기 전이기 때문에 처리된 일부 records가 재처리됨
       * cnsumer poll lop안에서 records를 처리하지 않을 경우, commit된 offset의 records에 대해 처리되었음이 보장되지 않음
         * poll loop에서는 백그라운드에서 처리되는 다른 thread에 records를 전달하여 records의 처리는 thread안에서 할 때, 아직 처리되지 않은 record에 대해 offset이 commit됨
  4. auto.commit.interval.ms
     * commit frequence가 증가하면
       * overhead가 발생
       * consumer가 중단되었을 때 발생하는 중복처리 데이터 개수 감소

* Explicitly Committing Offsets in Consumers (Consumer에서의 명시적인 Commit Offset)

  * auto offset commit을 사용하면 explicityly commit offset에 대해 고려할 필요가 없지만, 중복처리를 최소화하거나 poll loop 밖에서 event를 처리하기 위하여 offset을 commit하는 것에 대한 더 많은 제어가 필요하다고 결정한 경우 offset을 manually and explicityly commit할 필요가 있음

    *(4장에서 manual commit offset mechanism은 다뤘으므로 여기서는 data reliable측면에서 접근)*

  * Always commit offsets after events were processed

    * 항상 events가 처리된 후에 offsets을 commit해야 함
    * poll loop내에서 모든 처리를 수행한다면 poll loops사이에 state를 유지하지 않으면 됨
    * poll loop가 끝나면 auto commit configuration 또는 commit event를 사용할 수 있음

  * Commit frequency is a trade-off between performance and number of duplicates in the event of a crash

    * Commit의 빈도는 performance와 crash 발생 시 performance와 중복 처리간의 trade-off
      * Commit frequency가 높아지면 :  performance↓ crash시 중복처리되는 데이터의 개수↓
      * Commit frequency가 낮아지면 : performance↑ frash시 중복처리되는 데이터의 개수↑
    * poll loop안에서 여러번 commit하거나, 여러개의 loops에대해 한번만 commit하도록 구성할 수 있음

  * Make sure you know exactly what offsets you are committing

    * 어떤 offsets을 commit하는지 정확하게 알아야 함
    * 처리된 messages에 대해 항상 commit해야 함. 읽고나서 처리되지 않은 messages의 commit은 consumer가 messages를 손실할 수 있음
    * polling loop 중간에서 commit할 때 흔히 하는 실수는,  read한 last offset을 실수로 commit하고 처리된 last offset을 commit해서는 안 됨

  * Rebalance

    * partitions이 revoke(제거)되기 전에 offsets을 commit하고, new partitions이 assign될 때 maintain한 state를 celan
      * *consumer가 종료하기 전에 partitions의 offset을 commit하고, 해당 partitions이 reassign될 때는 기존의 상태를 가져와야 한다는 것을 의미하는건가?*

  * Consumers may need to retry

    * Consumers는 retry가 필요할지도 모름
    * 몇몇의 경우에서 poll을 call하고, processing records를 call한 후에, 몇몇의 records가 완전히 처리되지 않고 나중에 처리해야 할 수 있음
      * e.g) Kafka에서 database로 records를 wrtie하는데, 그 순간 database가 available하지 않아서 나중에 다시 시도해야 하는 경우가 있음 
    * Kafka는 traditional pub/sub messaging system과 달리 offects을 commit하고 개별 messages를 ack(확인)하지 않음
      * offset 30의 record를 처리하지 못하고 offset 31의 record를 처리하면 offset 31을 commit하는 것은 offset 30을 포함하여 offset 31의 records까지 모두 처리가 되었다는 것을 의미하므로 offset 31을 commit해서는 안 됨
    * retriable error가 발생했을때 가능한 2가지 행동
      1. 성공적으로 처리한 last record를 commit 하고 재처리해야 하는 records는 buffer에 저장하고 records의 처리를 이어나감
         * 모든 recrods를 처리하는 동안 poll을 계속 해야 할 수도 있음
         * consuemr의 pause() method를 이용하여 추가적인 poll이 추가적인 data를 반환하지 않도록 할 수 있음
      2. 별도의 topic에 records를 쓰고 계속 진행
         * 별도의 consumer group을 이용하여 retry topic의 records를 retries를 처리하거나 하나의 consumer가 main topic과 retry topic모두를 subscribe하여 두 topics간에 pause를 할 수 있음
         * 이 pattern은 많은 messaging systems에서 사용하는 deadletter-queue system과 유사함

  * Consumers may need to maintain state

    * 일부 applications에서 Consumers는 poll하기 위한 multiple calls 간에 state를 유지해야 할 수도 있음
      * e.g) moving average를 계산하려는 경우, Kafka에서 새로운 events를 poll할 때 마다 average를 update해야 함. process가 재시작되면 last offsets에서 consume하기 시작해야 할 뿐 아니라 match되는 moving average도 복구해야 함
    * state를 유지하는 방법
      * offset을 commit하는 동시에 accmulated value와 같은 state를 result topic에 write
      * thread가 시작될 때 result topic에서 state를 가져와서 중단된 부분부터 재시작 할 수 있음
      * Kafak 0.11.0 이전에서는 transaction을 제공하지 않기 때문에 문제를 완전히 해결하지는 못함
        * 최신 결과를 topic에 write하고 offsets을 commit하기 전에 crash될 수 있으며 이 반대도 일어날 수 있음
        * 일반적으로 이 문제는 해결하기가 복잡하며, 직접 해결하기 보다는 aggregation, joins, windows, 기타 analytics를 위한 고급 DSL과 유사한 APIs를 제공하는 Kafka Streams와 같은 library를 살펴보는 것이 좋음
      * Kafka 0.11.0이후에서는 transaction을 제공

  * Handling long processing times (긴 processing times의 처리)

    * 때때로 records를 처리하는데 오랜 시간이 걸릴 수 있음
      * e.g) 매우 복잡한 계산을 차단하거나 수행할 수 있는 service와 interact(상호작용)하는 것
    * Kafka 0.10.1 이전 version에서는 추가적인 records를 처리하지 않으려고 하더라도, broker에게 heartbeats를 보내기 위해 poll을 계속 해야 함 -> 데이터 처리시간이 너무 오래 걸리면 broker에게 heartbeats를 보낼 수 없어 rebalance가 trigger됨
    * 이런 상황에서의 해결방안
      * multiple threads로 처리할 수 있는 경우 thread pool로 data를 전달하여 병럴 처리로 속도를 높일 수 있음
      * worker threads로 records를 전달한 후, worker threads가 끝날 때 까지 consumer를 pause하고 additional data의 fetch 없이 poll을 유지하고, worker threads까 끝나면 consumer를 resume

  * Exactly-once delivery (정확하게 하나만 배달)

    * 일부 applications는 데이터 손실이 없는 at-least-once semantics 뿐 아니라 exactly-once semantics를 요구함

    * Kafka 0.11.0 이전 version에서는, exactly-once를 지원을 제공하지는 않지만 Kafka가 정확하게 한 번 외부 시스템에 쓰여지도록 보장하는 few trick을 사용할 수 있음

      * 데이터가 Kafka로 produce되는 동안 발생하는  중복은 처리하지 않음

      1. unique keys를 지원하는 system에 results를 write (idempotent(멱등원) write)
         * 이 system에는 key-value store, relational database, Elasticsearch 등의 많은 data store가 포함됨
         * result를 data store에 write할 때, record 자체가 unique key를 포함하거나 topic, partition, offset의 combination(조합)을 이용하여 unique key를 만들어서 Kafka Record를 식별하는 unique key로 사용
         * record를 unique key와 함께 value로 쓰고, 나중에 실수로 같은 record를 다시 consume하면 정확하게 같은 key와 value를 write하게 됨. data store는 기존의 것이 override되고 중복처리되지 않은 경우와 동일한 결과를 얻음
      2. transactions이 있는 system에서 records와 offsets을 write
         * 이 system은 relational database가 가장 쉬운 예시이지만, atomic renames를 가지는 HDFS도 자주 쓰임
         * 동일한 transcation에서 records와 offsets을 write하여 in-sync되도록 함
         * 시작 시, external store에 write되어 있는 최신 records의 offsets를 retrieve(검색)한 후 consumer.seek()를 사용하여 해당 offset에서 consume을 시작



### Validating System Reliability

* Introduce
  * process의 reliability requirement를 figure out(확인)하고, borker를 configure하고, clients를 configure하고, use case에 따라 가장 좋은 방식의 API를 사용을 통해, 안심하고 모든 event가 손실되지 않는 다는 확신으로 run할 수 있나요? -> 그럴 수 있지만 먼저 validation(유효성검사)를 하는 것이 좋음
  * 유효성 검사는 3가지 단계로 이뤄짐 : validate the configuration, validate the application, monitor the application in production

1. Validating Configuration

   * broker와 client의 configuration을 application logic과 분리하여 쉽게 test할 수 있으며 이것이 아래와 같은 이유로 권장됨

     * 선택한 configuration이 requirements(요구사항)을 충복할 수 있는지 테스트 가능
     * system의 예상되는 behavior을 추론하는것은 좋은 exercise

   * Kafka가 포함하는 validation(유효성 확인)을 돕는 2가지 toll이 있음

     * org.apache.kafka.tools pacakage에 포함
     * 해당 class들은 command-line tool로 실행되거나, automated testing framework에 포함됨

     1. VerifiableProducer
        * verifiable producersms 1에서 선택한 값까지의 숫자를 포함하는 일련의 messages를 생성
        * 생성자를 configure하는 것과 동일한 방법으로 configure할 수 있으며, acks, retries, produce되는 messages의 비율을 설정할 수 있음
        * 실행하면 수신한 acks에 따라 broker로 전송된 각 message의 success 또는 error가 print됨
     2. VerifiableConsumer
        * complementary(상호보완적인) check를 수행함
        * events를 consume하고 consume한 event를 순서대로 print함
        * commit과 rebalances와 관련된 information도 print함

   * 테스트할 때 고려해야 할 사항들

     * Leader election (리더 선출)
       * leader를 죽이면 어떻게 되는가?
       * producer와 cnsumer가 평소처럼 다시 작동하는데까지 얼마나 걸리는가?
     * Controller election (컨트롤러 선출)
       * controller를 다시 시작한 후 system이 다시 시작되는데 걸리는 시간은 얼마나 되는가?
     * Rolling restart
       * messages를 잃지 않고 broker를 하나씩 재시작 할 수 있는가?
     * Unclean leader election test
       * 하나의 partition에 대한 모든 replicas를 하나씩 kill하고 out-of-sync broker를 시작하면 어떻게 되는가? operation을 재시작 하기 위해서는 무엇이 필요한가? 이게 받아들여지는가?

   * Validation Configuration 절차

     1. 테스트할 scenario를 선택
     2. verifiable producer를 시작
     3. verifiable consumer를 시작
     4. scenario를 실행
        * e.g) data를 produce하는 partition의 leader를 kill
     5. 검증
        * e.g) short pause 후 정상적으로 messages의 손실 없이 resume되는 것을 기대한다면, producer가 produce한 messages의 개수와 consumer가 consume한 messages의 개수가 일치해야 함

2. Validating Applications

   * broker와 client의 cnfiguration이 requirements를 충족하면, application이 필요한 guarantees를 제공하는지를 test해야 함
   * Check List
     - error-handling code
     - offset commits
     - rebalance listeners
     - Kafka's client libararies와 interact(상호작용)하는 application logic
   * application에 대한 integration test(통합 테스트)도 이뤄져야 함
   * application의 validate(유효성 검사를 수행)하더라도, 아래와 같은 다양한 failure conditions(오류 조건들)에서 test를 하는것이 권장됨
     * Clinets가 server에 대한 cnnectivity를 잃음 (system administrator가 simulating network failures에 도움을 줄 수 있음)
     * Leader election
     * Rolling restart of brokers
     * Rolling restart of consumers
       * e.g) consumers를 rolling restart할 때, consumers의 rebalance를 위해 short pause후에 1,000개 이하의 duplicate values로 consumption을 계속하기로 기대하고 test를 진행할 수 있음. 테스트에서는 application이 실제로 offset을 commit하고 rebalance가 작동되는 방식을 확인할 수 있음
     * Rolling restart of producers

3. Monitoring Reliability in Production

   * 데이터가 예상한대로 flow하는지를 확인하기위해서 production system을 지속적으로 monitor해야 함
   * Chapter9에서 Kafka cluster를 monitor하는 방법에 대해 세부내용을 다루지만, cluster의 상태를 monitor하는 것 외에도 system을 통한 flow of data(data흐름)을 monitor하는 것이 중요함
   * JMX metrics
     * Kafka's Java clinets에 포함됨
     * client측의 status(상태)와 events를 monitor할 수 있게 함
   * monitoring producer
     * reliability에 가장 중요한 2가지 metrics는
       * error-rate per record
       * retry-rate per record
       * error-rate와 retry-reate가 올라가면 system에 문제가 있음을 나타냄
     * producer logs에서 WARN level로 events를 send하는 동안 발생하는 "Got error produce response with correlation id 5689 on topic-partition [topic-1,3], retrying (two attempts left). Error: …"와 같은 errors를 monitor해야 함
       * 0 attempts left인 events가 나타나면 prducer의 retries가 부족한것이므로, retries의 횟수를 늘리거나 errors를 발생시키는 부분을 해결할 수 있음
   * monitoring consumer
     * 가장 중요한 metric는 consumer lag
       * 이 metric은 broker의 partition에 commit되어 있는 latest message로부터 consumer가 얼마나 떨어져 있는지를 의미
       * 이상적으로 lag는 항상 0이고 consumer는 항상 latest message를 읽어야 함
       * 실제로 poll()를 호출하면 multiple messages가 return되고 consumer는 더 많은 messages를 fetch하기전에 이 messages를 처리하는데 시간을 소비하기 때문에, lag는 조금씩 fluctuate(변화함)
         * 그러나 결과적으로는 consumer가 뒤쳐지기 보다는 따라잡아야 함
     * Burrow
       * LinkedIn이 제작
       * consumer lag checker
       * consumer lag의 fluctuation(변동)때문에 traditional alerts(전통적인 알람)을 설정하는 것은 어려울 수 있는데 이를 쉽게 하도록 함
   * monitoring data flow
     * data flow(데이터 흐름)을 monitor하는 것은 produce된 data가 timely manner(적시에) consume되는 것을 확인하는 것 (requirements에 따라 "timely manner"의 의미가 결정됨)
     * 적시에 data가 consume되려면 data produce 시기를 알아야 함
       * Kafka 0.10.0 이후부터는 messages가 event가 produce된 시간을 나타내는 timestamp를 포함
       * Kafka 0.10.0 이전버전의 client는, 각 event에 대해서 message가 create되었을 때, timestamp, name of app producing the message(message를 produce하는 app 이름), hostname을 기록하는것이 권장됨
     * monitoring data flow 방법
       1. produce된 message가 적절한 시간 내에 consume되도록 하려면 produce된 evetns의 초당 개수를 기록하는 application이 필요
       2. consumer는 consume되는 events의 초당 개수를 기록하고, event가 produce되는 시간부터 consume되는 time까지의 lags를 event timestamp를 이용하여 기록해야 함
       3. producer와 consumer의 초당 events 개수를 reconcile(조정)하여 messages가 손실되지 않았는지 확인하고 events사이의 시간이 합리적인지를 확인
     * end-to-end monitoring system
       * 중요한 topics에 monitoring consumer를 추가하여 produce된 events를 count하고 compare하여, 특정 시점에 events를 consume하는 consumers가 없더라도 producer를 monitoring 할 수 있도록 함
       * 구현하기가 어렵고 시간이 오래걸릴 수 있음
       * 이러한 유형의 open source implementation system은 없음
       * Confluent의 commericial implementation(상업적 구현)이지만Confluent Control Center의 일부가 제공을 함



### Summary

* 모든 option과 공통 pattern과 use case에 대한 requirements(요구사항)을 이해함으로써, application이 얼마나 reliable해야 하고, trade-offs가 얼마나 의미있는지를 고려하여 결정해야 함





