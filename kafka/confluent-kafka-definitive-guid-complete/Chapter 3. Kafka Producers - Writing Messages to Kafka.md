## Chapter 3. Kafka Producers: Writing Messages to Kafka

###Introduce

* Kafka를 queue or message bus or data storage platform으로 사용하는 경우 아래의 3개의 clinets를 사용하게 됨
  * Kafka에 데이터를 쓰는 producer
  * Kafka로부터 데이터를 읽는 consumer
  * producer와 cnsumer의 역할을 동시에 하는 application
* Apache Kafka는 개발자가 Kafka와 상호작용하는 applications을 개발할 때 사용 가능한 client API가 내장되어 있음
* Third-Party Clients
  * 내장된 clients 외에도 Kafka는 binary wire protocol을 가짐
  * binary wire protocol은 Kafka의 network port에 올바른 byte sequences를 보냄으로써 간단하게 Kafka에서 messages를 읽거나 Kafka에 messages를 쓸 수 있게 함
  * Kafka의 wire protocol을 구현하는 multiple clients는 Java, C++, Python, Go 등 다양한 언어의 application으로 구현 가능
    * Java가 아닌 clients는 project wiki에서 maitained되며 Apache Kafka project의 일부가 아님
  * 이 chapter 에서는 wire protocol과 external clients에서 다루지 않음



### Producer Overview

* applicatioon이 Kafka에 messages를 쓰면 database에 writing하기 전에 아래와 같은 다양한 작업을 수행할 수 있다는 이점이 존재
  * auditing(감시) 혹은 analysis(분석)를 위한 recoording user activities(사용자 활동 저장)
  * recording metrics(metrics 저장)
  * storing log messages(log messages 저장)
  * recording information from smart appliances(스마트가전으로부터 정보를 기록)
  * communicating asynchronously with other applications(다른 applications와 비동기적인 통신)
  * buffering information(정보 버퍼링)
* 위처럼 다양한 use cases에서는 아래와 같은 다양한 requirements가 포함됨
  * 모든 message가 중요한지? or messages의 손실을 허용하는지?
  * message가 accidentally(우연히) duplicating(중복)되어도 괜찮은지?
  * 지켜야할 엄격한 latency(지연시간)가 있는지?
  * 지켜야할 엄격한 throughput이 있는지?
    * 이러한 요구사항들은 producer API를 이용하여 Kafka에 messages를 쓰는 방법이나 사용하는 configuration에 영향을 줌
* Main steps involved in sending data to Kafka (Kafka에 데이터를 보내는 주요 과정

![figure3-1](C:\Users\jungsol\Documents\kafka\pic\figure3-1.PNG)

1. topic, record, value를 포함하는 ProducerRecord를 creating하여 Kafka로 messages를 producing (선택적으로 key, partition도 지정 가능)
2. ProducerRecord를 보낸 후에는, producer는 key와 value object를 ByteArrays로 serialize하여 network를  통해 보낼 수 있또록 함
3. data는 partitioner에 보내짐
   * ProducerRecord에 partitin을 지정했다면, partitioner는 아무것도 하지 않고 지정한 partition을 반환
   * ProducerRecrd에 partition을 지정하지 않았다면, partitioner는 ProducerRecord의 key를 기반으로 partition을 선택
   * Parition이 선택되면 producer는 record가 전송될 topic가 partition을 알고 있게 됨
4. partition이 선택되면 producer는 record가 기록될 topic과 partition을 알고 있으므로, 해당 record를 같은 topic과 partition에 보내게되는 batch of records(records 일괄 처리)에 추가
5. seprated thread(별도의 thread)는 batches of records를 적절한 Kafka broker에 보냄
6. broker가 messages를 받았을 때 response를 보냄
   * messages가 성공적으로 Kafka에 written되었다면 RecordMetadata object를 반환
     * RecordMetadata object는 해당 record의 topic, partition, offset 정보를 가짐
   * broker가 messages를 writing하는 것이 실패하였다면, error를 반환
     * producer가 error를 받으면 포기하고 error를 반환하기 전까지 몇번 더 retry sending을 함 (configuration으로 retry 횟수 정할 수 있음)



### Constructing a Kafka Producer

* Kafka Producer의 3가지 mandatory configurations

  * bootstrap.servers

    * \$\{host\}:\$\{port\}, \$\{host\}:\$\{port\},....
    * producer가 Kafka cluster에 초기 연결할 때 사용하는 brokers list
    * producer는 초기 연결 후 더 많은 정보를 얻으므로 이 list에는 모든 brokers가 포함될 필요 없음
    * 최소 2개의 brokers를 포함하는것이 권장됨 (하나의 broker가 down되더라도 producer가 cluster에 연결될 수 있도록)

  * key.serializer

    * producer가 Kafka로 produce할 records의 key를 직렬화하는데 사용되는 class
    * org.apache.kafka.common.serialization.Seriallizer interface를 implements한 class의 이름으로 설정되어야 함
    * Kafka brokers는 messages의 keys와 values를 byte arrays로 받기를 원함
    * 그러나 producer interface를 사용하면 모든 Java object를 key와 value로 사용할 수 있음
      * readable(읽기쉬운) code를 만들지만
      * producer는 이러한 objects를 byte arrays로 변환하는 방법을 알아야 함
    * Kafka client package에는 ByteArraySerializer(많은 일을 하지 않음), StringSerializer, IntegerSerializer이 포함되어 있음 -> common types의 keys와 values을 사용할 경우, 자체적인 serializers를 구현할 필요가 없음
    * keys없이 values만 보내는 경우에도 key.serializer는 필수조건

  * value.serializer

    * producer가 Kafka로 produce할 records의 value를 직렬화하는데 사용되는 class
    * message의 value object를 byte array로 serialize하는 class로 설정

  * Simple Example

    ```java
    private Properties kafkaProps = new Properties();
    kafkaProps.put("bootstrap.servers", "broker1:9092,broker2:9092");
    kafkaProps.put("key.serializer", 
    	"org.apache.kafka.common.serialization.StringSerializer");
    kafkaProps.put("value.serializer", 
    	"org.apache.kafka.common.serialization.StringSerializer");
    producer = new KafkaProducer<String, String>(kafkaProps);
    ```

    1. Properties object를 만듦
    2. message의 key와 value를 string형으로 사용하기 때문에 내장된 StringSerializer을 사용하여 Properties object에 설정
    3. appropriate(적절한) key와 value type을 setting하고 Properteis object를 넘겨주어 새로운 producer를 만듦

  ​

### Sending a Message to Kafka

* 더 나은 throughput을 원한다면 같은 producer를 사용하는 threads를 더 추가하거나, application에 더 많은 producers를 추가하면 됨
* producer를 instantiate(인스턴스화)한 후에는 sending messages를 보냄
* message를 보내는 방법에는 크게 3가지가 있음
  * Fire-and-forget(보내고 잊기)
  * Synchronous send
  * Asynchronous send

1. Fire-and-forget(보내고 잊기)

   * server에 message를 보내고, 성공적으로 도착했는지의 여부를 신경쓰지 않음
   * Kafka는 highly available 하고 producer는 messages를 automatically(자동적으로) retry sending을 하므로, 대부분의 경우에 messages는 성공적으로 도착함
   * 일부 messages는 이러한 method를 사용하여 손실됨

   ```java
   ProducerRecord<String, String> record =
   	new ProducerRecord<>("CustomerCountry", "Precision Products", "France");
   try {
   	producer.send(record);
   } catch (Exception e) {
   	e.printStackTrace();
   }
   ```

   1. Producer는 ProducerRecord object를 accept하기 때문에 multiple constructors(생성자)를 가지는 하나의 ProducerRecord를 생성
      * ProducerRecord 의 구성
        * sending data를 보낼 String의 Topic
        * Object 형태의 Key (Serializer와 match 되어야함)
        * Object 형태의 Value (Serializer와 match 되어야함)
   2. send() method를 이용하여 ProducerRecord를 보냄
      * message는 buffer에 placed되고 별도의 thread에서 broker로 send됨
      * send() mehtod는 RecordMetadata를 가지는 Java future object를 반환하지만 반환된 값을 무시하기 때문에, message가 성공적으로 전송되었는지의 여부를 알 수 없음
      * Use case
        * The method of sending messages can be used when dropping a message silently is acceptable. (message를 자동으로 삭제하는 경우에 사용) (대부분의 production application에는 해당되지 않음)
   3. Kafka brokers에게 혹은 broekrs들 자신들에게 messages를 보내는 동안 발생하는 errors는 무시하지만, 만약 Kafka에 message를 sending하기 전에 발생하는 error는 exception을 얻음
      * SerializationException : message의 serialize에 실패
      * BufferExhaustedException : buffer가 가득 찬 경우
      * TimeoutException : buffer가 가득 찬 경우
      * InterrupException : sending thread가 interrupted 된 경우

2. Synchronous send

   * messages를 보내고, send() methd는 Future object를 return하고 get()으로 future를 기다리고 send()가 성공했는지 확인

   ```java
   ProducerRecord<String, String> record =
   	new ProducerRecord<>("CustomerCountry", "Precision Products", "France");
   try {
   	producer.send(record).get();
   } catch (Exception e) {
   	e.printStackTrace();
   }
   ```
   1. prducer는 ProducerRecord를 sending한 후 Future.get()을 사용하여 Kafka로부터의 회신을 기다림
      * 만약 record가 성공적으로 Kafka에 전송되지 않으면 throw exception
      * 만약 errors가 없다면 RecrodMetadata object를 얻음
   2. Kafka가 data를 sending하기 전에 errors가 발생하거나, Kafka brokers가 nonretriable exceptions를 return하거나, 가능한 retries를 다 써버린 경우 excepton이 발생

   * KafkaProducer의 두가지 유형의 errors
     1. Retriable errors
        * message를 다시 sending하면 해결 될 수 있음
        * e.g) connection이 끊킨 경우 connection을 reestablished하면 connection으 ㄹ얻을 수 있으므로 error가 해결됨
        * e.g) no leader error는 해당 partitin에 대한 leader가 다시 선출되면 해결됨
        * KafkaProducer는 이러한 errors에 대해 자동으로 retry하도록 구성되어 있으므로 retries 횟수를 다 소모하고, error가 해결되지 않았을 때만 Retriable Exceptions를 얻을 수 있음
     2. Non Retriable errors
        * retrying하여도 해결되지 않는 errors
        * retrying을 시도하지 않고 바로 exceptin을 반환
        * e.g) message size too large

3. Asynchronous send

   * sned() method를 callback function과 함께 call을 하고 Kafka broker으로부터 response를 받을 때 callback function이 triggered됨
   * applicatin과 Kafka 사이의 network roundtrip time이 존재하므로, message를 보낸 후 reply를 기다리면 시간이 오래 소요됨 (반면 reply를 기다리지 않으면 시간이 거의 소요되지 않음)
   * 대부분의 경우 Kafka가 응답으로 보내는 record가 쓰여진 topic, partition, offset 정보가 필요 없으며, message 전송이 실패했는지 성공했는지의 여부만 알 면 됨 -> message를 asynchronously sending하고 error scenarios를 다루기 위해 prducers는 records를 보낼 때 callback function을 추가할 수 있음

   ```java
   private class DemoProducerCallback implements Callback {
   	@Override
   	public void onCompletion(RecordMetadata recordMetadata, Exception e) {
       	if (e != null) {
   			e.printStackTrace();
   		}
   	}
   }

   ProducerRecord<String, String> record =
   	new ProducerRecord<>("CustomerCountry", "Biomedical Materials", "USA");
   producer.send(record, new DemoProducerCallback());
   ```

   1. callback을 사용하기 위해서는 onCompletion()의 singel function을 가지는 org.apache.kafak.clients.producer.Callback interface를 implements해야 함
   2. Kafka가 error를 반환하면 onCompletion()의 e는 null이 아닌 exception을 가지며 onCompletion() method 내에서 강력한 error handling을 할 수 있음
   3. message는 Fire-and-Forget 방식이나 Synchronous send 방식과 동일하게 ProducerRecord를 생성
   4. ProducerRecord를 보낼 때 Callback object를 넘겨줌 (그러면 sending 이후 callback function이 호출됨)



### Configuring Producers

* producer는 많은 configuration parameters를 가지며, Apache Kafka documentation에 문서화 되어 있으며, configuration parameters는 reasonable(합리적인) default value를 가지므로, 모든 parameters를 변경할 필요가 없음

* 일부 configuration parameters는 memory 사용, performace, reliability(안정성)에 중요한 영향을 미침 (해당 configurations을 아래에서 다룸)

* acks

  * default = 1
  * producer가 성공적으로 write했다고 여기기 위해, record를 수신해야 하는 partitin replica의 개수
  * message의 손실과 큰 관계가 있음
  * acks parameter로 가능한 value는 4가지(0, 1, -1, all)
    * 0
      * producer는 broker로부터의 reply를 기다리지 않음
      * 무언가가 잘못되어 broker가 message를 받지 못하면 producer는 해당 사실을 알지 못하고 message가 손실됨
      * producer는 server로부터 어떠한 response도 기다리지 않으므로 network에서 지원하는 만큼 message를 빠르게 보낼 수 있으므로 매우 높은 throughput을 얻음
    * 1
      * producer는 broker로부터 leader replica가 message를 받는 순간 response를 받음
      * leader가 crashed되고 새로운 leader가 선출되지 않아 message가 leader에 쓰여지지 않으면, producer는 error를 수신하고 data 손실을 방지하기 위해 sending message를 retry할 수 있음
      * The message can still get lost if the leader crashes and a replica without this message gets elected as the new leader (via unclean leader election). (leader가 crashes되고 message를 가지고 있지 않은 replica가 새로운 leader가 되면 message는 여전히 손실될 수 있음)
      * acks가 1일때의 throughput은 message를 synchronously sending인지, asynchronously sending인지에 달림
        * synchronously sending이면, client는 get() method를 call하여 server로부터 Future object를 얻을 때 까지 대기하므로, 최소 network roundtrip time만큼 latency가 발생하며 latency(지연/대기시간)이 상당히 길어짐
        * asynchronously sending이면 latency는 hidden되지만 throughput은 in-flight messages의 개수만큼 제한됨
          * the number of in-flight messages : producer가 server로부터 응답을 받기 전에 보낼 수 있는 messages의 개수 (하나의 connectin에 보낼 답을 받지 못한(unacknowledged) 요청 수)
    * all (= -1)
      * 모든 in-sync replica가 message를 받으면 producer는 broker로부터 성공 응답을 받음
      * 하나 이상의 broker가 message를 가지고 있다는 것이 보장되고, 어떠한 crash의 경우에도 message가 살아남을 수 있게 되므로 가장 안전한 방법
      * 하나 이상의 broker가 message를 받기를 기다리므로 latency가 길어짐

* buffer.memory

  * default = 33554432
  * producer가 brokers에게 보내려고 대기중인 messages의 buffer로 사용할 memory의 양
  * server에 전달될 수 있는 것보다 빠르게 message를 보내려고 하면, prducer의 공간이 부족할 수 있으며, 추가적인 send() calls이 block되거나 block.on.buffer.full parameter에 의해 exception이 throw될 수 있음
    * block.on.buffer.full (0.9.0.0이후 max.block.ms)

* max.block.ms

  - default = 60000
  - 해당 parameter에 지정된 ms시간동안 blocking되면 throwing exception
  - send()를 호출할 때와 partitionsFor()를 통해 metadata를 explicity(명시적으로) requesting할 때 producer가 block되는 시간을 제어
    - producer의 send buffer가 꽉 차거나 metadata가 available하지 않을 때 block됨
  - max.block.ms에 도달하면 timeout exception 발생

* compression.type

  * default = producer (0.9.x version에서는 none)
  * default value는 none (압축하지 않은 채로 전송)
  * compression 알고리즘이 지정되면 brokers로 messages를 sending하기 전에 compress
  * 장점) network utilization(사용) 및 storage를 줄일 수 있음
  * 단점) 종종 Kafka에 messages를 보낼 때 bottleneck(병목현상) 발생
  * 가능한 value : uncompressed / producer / snappy / gzip / lz4
    * uncompressed
      * 0.9.x version에서는 'none'
      * compression을 사용하지 않음
    * producer
      * 0.9.x version에서는 존재하지 않음
      * producer가 set한 original compression codec을 유지
    * snappy
      * google에서 low CPU overhead와 좋은 performance로 decent(적절한) compression ratios을 제공하도록 만들어짐
      * performace와 bandwidth 모두 문제가 되는 경우 권장
    * gzip
      * 더 많은 CPU와 시간을 사용하지만 compression ratio가 좋음
      * network bandwidth가 제한적인 경우 권장

* retries

  * default = 0
  * producer가 server로부터 error message를 수신하면 해당 error는 일시적일 수 있음 (e.g. partition의 leader의 부재)
  * retries parameter의 value는 producer가 일시적인 error에 의해 전송 실패한 message의 sending을 retry하고 client에게 이러한 issue를 알리는 횟수를 제어
    * 모든 error가 아닌 일시적인 retriable errors에 대해서만 retry 하므로 nonretriable errors나 retry 횟수를 exhausted한 경우에 대한 처리를 logic에서 해줘야 함
  * retry.backoff.ms
    * producer가 재시도간에 기다리는 ms 단위의 시간
    * default value는 100
  * crashed broker가 recover되기 까지 걸리는 시간을 test 하여 retries횟수와 retires간의 delay time을 설정하여 retrying에 걸리는 전체 시간이 Kafka cluster가 crash로부터 복구되는데 걸리는 식나보다 길도록 설정해야 함

* batch.size

  * default = 16384
  * 같은 partition에 mutiple records가 전송되면 producer는 이들을 batch 처리
  * 이 parameter는 각 batch에 사용될(message가 아님) bytes 단위의 memory양을 제어
  * batch size가 꽉 차면 batch안의 모든 messages들이 보내짐
  * producer는 batch가 가득 찰 때까지 기다리지만은 않음
    * producer는 반만 찬 batch나 single message만 있는 batch도 보냄
    * batch size가 크면 batches에 더 많은 memory를 사용 할 뿐 sending message의 delay를 유발하지 않음
    * batch size를 너무 작게 설정하면 producer가 message를 너무 자주 send 하므로 overhead가 발생

* linger.ms

  * default = 0
  * ms단위의 현재 batch를 sending하기 전에 추가적인 messages를 기다리는 시간
  * KafkaProducersms 현재 batch가 꽉 차거나, linger.ms limit에 도달하면 batch of messages를 send
  * default value는 0
    * batch에 하나의 message만 있더라도 available한 sender thread가 있을 경우 바로 message를 보냄
  * 0이상의 값으로 설정하면 broker로 batch를 보내기 전에, producer는 batch에 추가적인 messages를 몇 milliseconds동안 기다림
    * increases latency
    * increases throughput (한번에 더 많은 message를 보내므로 message당 overhead가 적음)

* client.id

  * default = ""
  * 어떠한 string도 가능
  * client에서 보낸 message를 broker에서 식별하는데 사용됨
  * logging, metrics, quotas에서 사용됨

* max.in.flight.requests.per.connection

  * default = 5
  * producer가 response를 받지 않고 server에 보낼 messages의 개수
  * 높은 값을 가지면
    * increase memory usage
    * increase throughput
    * batch처리의 efficient가 감소하여 reduce throughput
  * Ordering Guarantees(순서 보증)
    * Apache Kafka는 하나의 partitin내의 messages의 순서를 보존
      * meesages가 producer로부터 특정 order로 보내진 경우, broker는 그 순서로 partition에 write하며, 모든 consumers는 그 순서로 read함
      * Use case에 따라 Ordering Guarantees가 중요한 경우가 있음
    * retries parameter를 0이 아닌 값으로 설정하고 max.in.flight.requests.per.connection을 1보다 큰 값으로 설정하면, broker가 첫번째 batch of messages의 write를 실패하고, 두번째 batch of messages의 wrtie를 성공한 후, 첫번째 batches의 write를 retry하고 성공하면 순서가 바뀜
    * retries parameter를 0으로 setting하는 것은 reliable system의 옵션이 아니므로, guaranteeing order가 중요하다면 in.flight.requests.per.connection의 값을 1로 지정하는 것을 권장
      * 이 값을 1로 설정하면 retries가 발생하더라도, retry하는 동안 추가적인 batch 0f messages가 send되지 않으므로 message가 전송된 순서대로 broker에 write되는 것이 보장됨
      * producer의 throughput을 심하게 제한하므로 순서가 중요할 때만 이 옵션을 사용

* timeout.ms / request.timeout.ms / metadata.fetch.timeout.ms

  * timout.ms
    * Kafka 1.0.0에서 removed
    * broker가 acks configuratin을 충족시키기 위해 in-sync replicas가 message를 acknowledge(확인)하기를 기다리는 시간을 제어
    * 만약 필요한 acknowledgments없이 시간이 경과하면 error를 return
  * request.timeout.ms
    * default = 30000
    * producer가 data를 sedning할 때 server로부터 reply를 얼마나 기다릴지를 제어
  * metadata.fetch.timeout.ms
    * Kafka 1.0.0에서 removed
    * writing중인 partitions에 대한 현재 leaders와 같은 metadata를 요청할 때 기다리는 시간
  * reply 없이 timeout이 발생하면 prducer는 sending을 retry하거나 throw exception 혹은 send callback을 통해 error를 응답함

* max.request.size

  * default = 1048576
  * prducer에 의해 보내지는 produce request의 size를 제어
  * 보내질 수 있는 message의 최대 크기와 producer가 하나의 request에서 보낼 수 있는 mesasge의 개수를 모두 포함
    * max.request.size가 1mb일 경우, 하나의 request에서 보낼 수 있는 최대 메세지는 1mb거나, 1k의 message 1000개를 batch처리 할 수 있음
  * max.request.size와 message.max.bytes는 일치시키는 것이 좋음
    * message.max.bytes
      * default value = 1000012
      * broker configuration option
      * 허용할 수 있는 최대 message의 크기
    * 두 configuration parameter를 일치시켜야 producer가 broker에서 rejected될 size의 message를 보내려고 시도하지 않음

* receive.buffer.bytes / send.buffer.bytes

  * default value = 32768 / 131072
  * data의 writing, reading 시 sockets에 읳 사용 되는 TCP 송수신 buffers의 size
  * -1로 설정하면, OS의 defaults값이 사용됨
  * producers나 consumres가 brokers와 다른 datacenter의 broker와 통신할 때 network links가 일반적으로 latency가 높고 bandwidth가 낮기 때문에 해당 parameters를 늘리는게 좋음

  ​

### Serializer

* Kafka client package에는 ByteArraySerializer, StringSerializer, IntegerSerializer이 포함되어 있음

* ByteArraySeralizer와 IntegerSerializer는 자주 쓰이지 않으며 더 generic records에 대한 serialize를 해야 함

  * Custom Serailizer 사용
  * Generic Serializer Library 사용 (recommended)
    * Avro, Thrift, Protobuf

* Custom Serializers

  ```java
  // Suppose that instead of recording just the customer name, you create a simple class to represent customers
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

  //Custom serializer for Customer class
  import org.apache.kafka.common.errors.SerializationException;
  import java.nio.ByteBuffer;
  import java.util.Map;
  public class CustomerSerializer implements Serializer<Customer> {
  	@Override
  	public void configure(Map configs, boolean isKey) {
  		// nothing to configure
  	}
  	@Override
  	/**
  	We are serializing Customer as:
  	4 byte int representing customerId
  	4 byte int representing length of customerName in UTF-8 bytes (0 if name is Null)
  	N bytes representing customerName in UTF-8
  	*/
  	public byte[] serialize(String topic, Customer data) {
  		try {
  			byte[] serializedName;
  			int stringSize;
  			if (data == null)
  				return null;
  			else {
  				if (data.getName() != null) {
  					serializeName = data.getName().getBytes("UTF-8");
  					stringSize = serializedName.length;
  				} else {
  					serializedName = new byte[0];
  					stringSize = 0;
  				}
  			}
  			ByteBuffer buffer = ByteBuffer.allocate(4 + 4 + stringSize);
  			buffer.putInt(data.getID());
  			buffer.putInt(stringSize);
  			buffer.put(serializedName);
  			return buffer.array();
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

  * org.apache.kafka.common.serialization.Seriallizer interface를 Implements한 Custom serailizer를 구현
  * CustomerSerializer로 Configuring한 producer는 ProducerRecord\<String, Customer\>를 정의할 수 있으며, Customer objects를 directly send 가능
  * Custom Serailizer의 구현은 매우 간단하지만 fragile(허술한,취약한) code
    * 많은 customers를 가지게 되어 customerID를 long type으로 바꿔야하거나 startDate field를 추가해야하는 등 object의 변화가 필요하게 되는 경우, old messages와 new messages 간의 compatibility(호환성)을 유지하는데 문제가 있음
    * serializers와 desrializers 간의 다른 version compatibility issues를 debugging 하는것은 arrays of raw bytes의 compare가 필요하기 때문에 상당히 어려움
    * 만약 Kafka에 data를 writing하는 multiple producer가 있다면, 모든producers는 같은 serializers를 사용해야 하며 object가 변경되었을 때 producers는 정확하게 동시에 code 수정이 이뤄져아함
  * 위와 같이 Custom Serailizer의 단점으로 인해 JSON, Apache Avro, Thrift, Protobuf와 같은 existing serializers와 deserializers를 사용하는 것을 권장

* Serializing Using Apache Avro

  * Apache Avro

    * Doug Cutting이 create
    * language-neutral data serialization formcat(언어 중립적인 data serialization)
    * data files을 많은 audience(사람들)과 share하는 방법을 제공
    * language-indepedent schema (lanuage에 독립적인 schema)

  * Avro의 schema는 보통 JSON으로 describe(표현)되고, JSON으로 serializing하는 것을 지원하더라도 보통 binary files로 serialization

  * messages를 writing하는 application이 new schema로 변경하였을 때, data를 reading하는 application은 어떠한 변화도 없이 messages 처리를 계속 할 수 있음

  * Example

    * original schema

      ```json
      {
        "namespace": "customerManagement.avro",
        "type": "record",
        "name": "Customer",
        "fields": [
          {"name": "id", "type": "int"},
          {"name": "name", "type": "string""},
          {"name": "faxNumber", "type": ["null", "string"], "default": "null"}
        ]
      ```

      1. id와 name는 mandatory fields
      2. fax number는 optional field, default는 null

      * original schema format으로 몇 달동안 몇 terabytes의 data를 generate 하였는데, fax number field를 email field로 대체하여 new version schema를 만들게 됨

    * new schema

      ```json
      {
        "namespace": "customerManagement.avro",
        "type": "record",
        "name": "Customer",
        "fields": [
          {"name": "id", "type": "int"},
          {"name": "name", "type": "string"},
          {"name": "email", "type": ["null", "string"], "default": "null"}
        ]
      }
      ```

      * new version으로 upgrad한 이후에는, old records는 faxNumber를 포함하고, new records에는 email을 포함
      * multiple prducers에서 각 producer들이 천천히 new version의 schema로 upgrade 한다고 하면 -> 여전히 fax numbers를 사용하는 preupgrade applications과 email을 사용하는 postupgrade applications가 Kafka에서 모든 event를 처리할 수 있는 방법을 고려해야 함
      * data를 reading하는 application은 getName(), getId(), getFaxNumber()과 비슷한 methods를 포함
        * new schema로 written된 message를 만나면, getName()과 getId()는 수정없이 작동하지만, fax number를 포함하지 않으므로 getFaxNumber()는 null을 반환
        * reading application을 etFaxNumber()가 아닌 getEmai()을 사용하도록 upgrade 한 후에 original schema로 written된 message를 만나면, getEmail()을 호출할 것이고, 해당 message는 email을 포함하지 않으므로 getEmail()은 null 반환
      * Avro는 모든 reading applications의 변경 없이 message의 schema가 바뀌더라도 exception, breaking errors, 기존 data의 expensive updates가 필요 없음
      * 2가지 caveats(주의사항)
        1. data writing에 쓰이는 schema와 reading application에서 expect하는 schema는 compatible해야 함
           * Avro documentation은 compatibility rules(호환성 규칙)을 포함
        2. The deserializer will need access to the schema that was used when writing the data, even when it is different than the schema expected by the application that accesses the data. (data에 access하는 application이 expect 하는 schema와 다르더라도  deserializer는 data를 writing할 때 사용되었던 schema에 access해야 함)
           * Avro files에 writing schema가 file 안에 포함되어 있지만, Kafka messages를 처리하는 더 좋은 방법이 있음 -> Using Avro Recrds with Kafka

* Using Avro Records with Kafka

  * data file에 전체 schema를 저장하여 fairly reasnable overhead를 유발하는 Avro files과 달리, 각각의 record에 전체 schema를 저장하면 record size가 두배가 됨

  * Avro는 여전히 data를 read할 때 entire schema가 필요하므로 schema를 elsewhere(다른곳)에 배치해야 함 -> 이를 위해 common architecture pattern을 따르고 Schema Registry를 사용

  * Schema Registry

    * Apache Kafka의 부분이 아니라 선택 할 수 있는 몇가지 open source
    * Registry에 schema를 저장하고 필요한 경우에 schema를 pulling하는 작업이 serializers와 desrializers에서 수행됨. 
    * Confluent Schema Registry (책에서 사용하는 Schema Registry)
      * GitHub에서 code를 찾을 수 있거나 Confluent Platform의 일부로 설치 가능

  * Schema Registry를 이용한 data produce, consume

    1. Kafka에 data를 write하는데 사용되는 모든 schemas를 registry에 저장
    2. record에는 schema identifier만 저장하여 Kafka에 produce
    3. consumers는 pull한 record의 schema identifier를 이용하여 schema registry에서 schema를 얻고, data를 deserialize

  * Example

    * Kafka에 prducer하는 data는 Avro serializer를 사용
    * Generated Avro objects 이용
    * Schema Registry를 이용

    ![figure3-2](C:\Users\jungsol\Documents\kafka\pic\figure3-2.PNG)

    ```java
    Properties props = new Properties();
    props.put("bootstrap.servers", "localhost:9092");
    props.put("key.serializer", "io.confluent.kafka.serializers.KafkaAvroSerializer");
    props.put("value.serializer", "io.confluent.kafka.serializers.KafkaAvroSerializer");
    props.put("schema.registry.url", schemaUrl);
    String topic = "customerContacts";
    int wait = 500;
    Producer<String, Customer> producer = new KafkaProducer<String, Customer>(props);
    // We keep producing new events until someone ctrl-c
    while (true) {
      Customer customer = CustomerGenerator.getNext();
      System.out.println("Generated customer " + customer.toString());
      ProducerRecord<String, Customer> record = 
        new ProducerRecord<>(topic, customer.getId(), customer);
      producer.send(record);
    }
    ```


    1. Key와 Value를 KafkaAvroSerializer를 사용하여 Avro 객체로 Serialize
       * AvroSerializer는 primitives(정수형)도 처리할 수 있으므로, Key는 String으로 value는 Customer object로 사용 가능
    2. shema.registry.url parmeter는 단순하게 schemas를 저장하는 장소를 가리킴
    3. key로 String, value는 Customer object로 가지는 ProducerRecord를 처리하는 Producer를 생성
    4. Customer로 ProducerRecord를 instantiate(인스턴스화)하고, 새로운 record를 만들 때 Customer object를 넘겨줌
    5. Customer object를 value로 가지는 record를 send하면 KafkaAvroSerializer가 나머지를 처리함


  * Example

    * generic Avro objects 이용

    ```java
    Properties props = new Properties();
    props.put("bootstrap.servers", "localhost:9092");
    props.put("key.serializer", "io.confluent.kafka.serializers.KafkaAvroSerializer");
    props.put("value.serializer", "io.confluent.kafka.serializers.KafkaAvroSerializer");
    props.put("schema.registry.url", url);
    String schemaString = "{\"namespace\": \"customerManagement.avro\"," + 
      "\"type\": \"record\", " +
      "\"name\": \"Customer\"," +
      "\"fields\": [" +
      "{\"name\": \"id\", \"type\": \"int\"}," +
      "{\"name\": \"name\", \"type\": \"string\"}," +
      "{\"name\": \"email\", \"type\": [\"null\",\"string\"], \"default\":\"null\" }" + 
      "]}";
    Producer<String, GenericRecord> producer = new KafkaProducer<String, GenericRecord>(props);
    Schema.Parser parser = new Schema.Parser();
    Schema schema = parser.parse(schemaString);
    for (int nCustomers = 0; nCustomers < customers; nCustomers++) {
      String name = "exampleCustomer" + nCustomers;
      String email = "example " + nCustomers + "@example.com"
        GenericRecord customer = new GenericData.Record(schema);
      customer.put("id", nCustomer);
      customer.put("name", name);
      customer.put("email", email);
      ProducerRecord<String, GenericRecord> data = new ProducerRecord<String, GenericRecord>("customerContacts", name, customer);
      producer.send(data);
    }
    ```

    1. Key와 Value를 KafkaAvroSerializer를 사용하여 Avro 객체로 Serialize *(위와 동일)*
    2. shema.registry.url parmeter는 단순하게 schemas를 저장하는 장소를 가리킴 *(위와 동일)*
    3. Avro generated object에 의해 제공되지 않으므로, Avro schema를 schemaString 변수에 정의하여 제공
    4. value object type은 Avro GenericRecord이며, 이는 schema와 write하려는 data로 initialize됨
    5. ProducerRecord의 value는 schema와 value를 포함하는 GenericRecord이며, serializer는 record에서 schema를 가져오고, schema registry에 저장하고, object serialize
       * 이렇게 하면 consumer에서 data를 read할 때, record에 있는 schema identifier를 보고 schema registry에서 schema를 가져와서 deeserialize함



### Partitions

* Introduce

  * ProducerRecord는 topic name, key, value로 구성됨
  * Kafka message는 key - value의 pairs
    * key가 null이어도 topic name과 value만으로 ProducerRecord를 만들 수 있지만 대부분의 applications에서는 keys가 있는 records를 생성
  * Key의 two goals
    1. message와 함께 저장되는 additional information 
    2. message를 write할 topic의 partition을 결정

* Partition에서 Key의 역할

  * 동일한 key를 가진 모든 messages는 같은 partition으로 보내짐
    * process가 topic의 하나의 subset of partitions를 read하는 경우에, 하나의 key는 같은 process에 의해서만 read
  * key가 null이고 default partitioner가 사용되면 record는 topic의 available partitions 중 하나에 랜덤으로 전송됨

* Create Key-value Record

  * key is not null

    ```java
    ProducerRecord<Integer, String> record = new ProducerRecord<>("CustomerCountry", "Laboratory Equipment", "USA");
    ```

  * key is null

    ```java
    ProducerRecord<Integer, String> record =
    new ProducerRecord<>("CustomerCountry", "USA");
    ```

* Round-Robin Algorithm

  * paritions간의 messages의 balance를 유지하는데 사용됨

* Partitioner

  * Default Partitioner
    * Kafka는 key를 hash하여, hash값으로 message를 특정 partition에 mapping
    * hash는 Kafka의 자체 hash algrithm을 사용하므로, Java가 upgrade 되더라도 결과가 달라지지 않음
    * key가 항상 같은 partition에 mapping되는건 중요하므로, availble partition 뿐 아니라 topic의 전체 partition을 이용하여 mapping을 계산
      * 따라서 data를 쓸 때 특정 partitin이 unavailable하면 error가 발생 (그러나 Kafka의 replication과 availability(가용성)으로 인해 거의 드묾)

* Consistency of Paritioning (paritioning의 일관성)

  * key의 partition에 대한 mapping은 topic당 partition의 개수가 변경되지 않는 한 일관적
  * partitions으로 부터 data를 읽을 때 모든 종류의 optimization이 가능
  * 그러나 새로운 partition을 topic에 추가하는 순간 더이상 일관성이 보장되지 않음
    * 같은 key에 대해 partition을 추가하기 전과 partition을 추가한 후에 해당 data가 저장되는 partition 번호가 달라질 수 있음
  * partitioning keys가 중요할 때는 충분한 partitions을 갖는 topic을 만들고 더이상 partitions을 추가하지 않는것이 좋음

* Implementing a custom partitioning strategy (custom 파티셔닝 정책 구현)

  * Kafka는 hash paritions로만 제한하지 않으며 data를 다른 방법으로 partition해야 좋은 경우도 있음

  * Example

    * B2B vender(공급업체)가 있는데, Bananas라고 불리는 handheld device를 제조하는 회사가 가장 큰 고객이며 하루 거래량의 10%이상이 Banana
    * 이런경우에 hash partitioning을 사용하면 Banana records는 다른 accounts과 함께 모두 같은 partition에 할당되어 하나의 partition이 나머지 partition의 약 2배가 됨
      * servers의 space가 부족하거나 처리속도가 느려질 수 있음
      * Banana에는 자체적인 하나의 partition을 부여하고 나머지 accounts들에는 hash partitioning을 이용하여 partition을 mapping하는 것이 바람직

  * Custom partitioner code

    ```java
    import org.apache.kafka.clients.producer.Partitioner;
    import org.apache.kafka.common.Cluster;
    import org.apache.kafka.common.PartitionInfo;
    import org.apache.kafka.common.record.InvalidRecordException;
    import org.apache.kafka.common.utils.Utils;

    public class BananaPartitioner implements Partitioner {
      public void configure(Map<String, ?> configs) {}
      public int partition(String topic, Object key, byte[] keyBytes, 
                           Object value, byte[] valueBytes, Cluster cluster) {
        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
        int numPartitions = partitions.size();
        if ((keyBytes == null) || (!(key instanceOf String))) {
          throw new InvalidRecordException("We expect all messages to have customer name as key")
        }
        if (((String) key).equals("Banana")) {
          return numPartitions; // Banana will always go to last partition
        }
        // Other records will get hashed to the rest of the partitions
        return (Math.abs(Utils.murmur2(keyBytes)) % (numPartitions - 1))
      }
      public void close() {
        
      }
    }
    ```

    1. org.apache.kafka.clients.producer.Partitioner interface를 implements하여 Custom Partitioner를 create
       * 해당 interface에 포함된 abstract class는 3개
         * public void configure(Map\<String, ? \>) configs;
         * public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster);
         * public void close();
    2. 위의 예제에서는 partition method에 hard-coding하였지만, 실제로는 configure method를 이용하여 특정 custmer name을 넘겨줘야함
    3. String key만을 expect하기 때문에 String type의 key가 아닐 경우 throw Exception
    4. key가 Banana일 경우에는 Banana의 partition number를 return하며 그 외의 accounts에 대해서는 hash partitioner를 사용하여 partition을 결정



### Old Producer

* New(Current) Producer
  * org.aphace.kafka.clients package에 있는 Java producer client
* Apache Kafka는 2개의 older clients가 존재
  1. kafka.producer package의 부분인 Scala로 쓰여진 older client
  2. core Kafka module인 older client
* SyncProducer와 AsyncProducer
  * SyncPrducer
    * acks parameter의 value에 따라 server가 추가적인 messages를 보내기 전에 message 혹은 batch of messages의 ack을 기다림
  * AsyncProducer
    * background에서 messages를 batches하여 별도의 thread로 batch of messages를 보낸 후, client에게 성공에 관한 feedback을 하지 않음
* Current Producer가 SyncProducer와 AsyncProducer를 모두 지원하며, 더 reliability(안정성)을 가지며 개발자에게 contrl(제어기능)을 제공하므로 Current Producer를 사용하는것을 권장



### Summary

* send events to Kafka.
* adding error handling
* synchronous and asynchronous producing
* serializers (control format of events we write to Kafka)
* Avro
  * one of many ways to serialize events
  * Avro is very commonly used with Kafka
* partitioning













