## Kafka Monitoring 

카프카 모니터링 시스템을 구축하면서 겪은 삽질에 대한 기록이다.



#### 모니터링 방식은?

모니터링 방식을 어떻게 할지 정하는데 굉장히 오래걸렸다.

1. JMX 사용
   - Kafka Cluster에 JMX Port를 열어서 JMX Metrics를 이용해서 모니터링 하는 방법이다. Kafka에서 제공하는 많은 metrics가 있기 때문에 이 값을 직접 조회해서 직접 모니터링을 제어하는 것이다. 별도의 모니터링 툴을 사용하지 않고 직접 모니터링을 구현하기 때문에 가장 자유도가 높다
   - 가장 큰 문제점은 Consumer 혹은 Producer가 죽었을 때 모니터링 하기가 어렵다는 것이다. 해당 부분에 대한 내가 생각하는 해결책은 
     1. Producer와 Consumer를 만들 때 client-id를 다 지정해주어 모니터링 하고자하는 client들의 metric의 최근 업데이트시간을 체크하는 것이다. 
     2. 살아있는 Consumer list를 가져와서 별도의 health-check 루틴을 만드는 것이다.
     3. Consumer 모니터링에 대해 JMX를 사용하지 않고 Burrow를 사용하는 것이다.
2. Kafka-Manager를 사용
   - Kafka Manger에서 rest api형태로 metrics를 조회할 수 있도록 되어있어 이를 활용하는 방안도 고민해보았다. ([conf/route 파일에서 rest api 조회가능](https://github.com/yahoo/kafka-manager/blob/master/conf/routes))
   - 하지만 kafka-manager daemon의 활성여부에 영향을 받으며 daemon refresh 속도가 느릴때가 있으므로 이보다는 JMX를 이용한 직접 제어가 더 좋다고 판단되었다.
3. Burrow 사용
   - Consumer 전용 모니터링 도구로 Consumer에 대한 상태체크를 할 수 있도록 도와준다.
   - 레퍼런스
     - [Burrow - git](https://github.com/linkedin/Burrow)
     - [Burrow - wiki](https://github.com/linkedin/Burrow/wiki)
     - http://seoeun25.tistory.com/550



최종적으로 JMX를 이용하여 모니터링을 구현하였으며,

Consumer가 살아있는지의 여부는 client-id를 consumer group-id와 동일하게 하여 update date를 체크해주도록 하였다.



#### JMX를 사용하기까지의 삽질기

JMX를 사용하겠다고 마음을 먹어서 JMX를 사용한 모니터링 시스템 구현을 하는데 굉장한 많은 삽질을 하였다.

1. JMX metric 값을 받아오는 방식은?

   * 받아올 수 있는 방법에는 굉장히 많은 방식이 있다. jmxterm, jolokia 등의 툴을 이용할 수도있는데, Java application에서도 받아올 수 있는 방법이 있다. 나는 java application에서 JMXConnector를 이용하여 받아오도록 구현하였다. (로컬에서 테스트할때는 jolokia를 사용하여 바로바로 jmx 값을 조회하기도 하였다.)

2. Consumer의 JMX metric 값을 받아올 수가 없었던 문제

   * Broker의 JMX metric값은 바로 받아왔지만, Consumer의 JMX metric 값을 받아오려고 하면 InstanceNotFoundException이 발생하는 문제가 있었다. 왜 발생하는지 계속 찾아보았는데 `getDomains()` 메소드를 이용하여 받아올 수 있는 mBean domain list를 보니  kafka.consumer가 아예 빠져있는 것을 확인할 수 있었다. 

     JMX 개념이 부족해서, Kafka를 시작할때 JMX 포트를 열어주면 모든 Kafka Cluster의 JMX metric을 받아올 수 있는 줄 알았다. 그런데 그것이 아니었고 각 metric을 받아오기 위해서는 해당 daemon이 떠있는 프로세스에 JMX port를 열어주어야 했다. 그래서 Consumer가 떠있는 프로세스에 JMX port를 열어주었고 이렇게 하니까 kafka.consumer이 domain list에 나타났다.

   * JMX port는 열었지만 계속해서 InstanceNotFoundException이 발생하는 문제가 있었다. 계속 고민하였는데 mBean의 파라미터로 주는 client-id에 나는 계속 consumer의 group-id를 입력하고 있었는데, 알고보니까 group-id랑 client-id는 다른 것이었다. 하나의 consumer group에 여러개의 consumer가 있을 수 있는데, 각 consumer는 고유의 client-id를 갖는다. 이는 configuration option으로 설정해줄 수 있고, 따로 설정하지 않으면 기본값이 된다. 그래서 kafka-manager 혹은 kafka 제공 script등으로 consumer 정보를 확인하면 client id에 Consumer-1 이런식으로 있는것을 확인할 수 있다. 그래서 client-id를 group id가 아닌 client id로 지정해주니까 정상적으로 metric을 받아오게 되었다.

3. Consumer의 client list를 얻는 방법은?

   * consumer의 jmx metric을 조회할 때 client-id를 넘겨주어야 한다는 것을 알았는데, 그렇다면 client list를 받아오는건 어떻게 해야할지 고민되었다. 방법은 아래와 같이 2가지가 있을 수 있는데,
     1. consumer를 만들 때 configuration option인 client.id에 id를 지정해주고, 해당 client id 정보를 전부 가지고 있어 jmx metric을 조회할때 파라미터로 넘겨주는 것이다.
     2. wildcard를 이용해서 parameter에 `client-id=*`로 넘겨주어 queryNames 메소드를 실행하여 mbean의 object name set을 받아와서 iterate하여 모든 client에 대해 jmx metric을 조회하는 방법이다. (이 방법을 사용하였다.)

4. 죽어있는 Consumer를 확인하는 방법은?

   * Consumer를 띄울때 client-id를 group-id와 동일하게 주어서 metrics update 시간을 통해서 Consumer가 죽어있는지를 확인하는 것으로 결정하였다. 이 부분에 대해서는 계속해서 고민이 필요할 것 같다.



