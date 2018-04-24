## Chapter10. Monitoring Kafka

### Introduce

* Apache Kafka applications은 수많은 measurements for their operation(작업에 대한 측정값)을 가짐 
  * 범위는 reate of traffic(트래픽 속도) 부터 모든 request type에 대한 detailed timig metric(세부적인 타이밍 메트릭), topics별 및 partitions별 metrics까지 다양함
  * 중요한 것이 무엇이고 별도 설정 가능한 것이 무엇인지에 대해 혼란스러울 수 있음
  * broker의 모든 operation이 세부정보를 제공하지만, monitoring system 관리 책임자에게 골칫거리 일 수 있으므로 적절한 metrics에 대해 monitroing을 하는 것이 중요
  * available한 metrics는 자주 변경되고 많은 것들이 Kafka developer의 hardcode에 informative함
* 이 Chpater에서 다룰 부분
  * monitor할 수 있는 가장 중요한 metrics(측정 항목)
  * 문제를 debug할 때 사용할 수 있는 중요한 metrics



### Metric Basics

* Introduce

  * Kafka broekr 및 clients가 제공하는 metrics에 대해 살펴보기 전에, Java applications을 monitor하는 방법과 monitoring과 alerting에 대한 모범사례에 해대 살펴봄
  * 이를 통해 application을 monitor하는 방법과 이 장에서 설명되는 specific metrics가 가장 중요한 metric으로 선정되는지의 이유를 알 수 있음

* Where Are the Metrics?

  * Kafka가 공개한 모든 metrics는 Java Management Extensions(JMS) interface를 통해서 access가능
  * 외부 monitoring system에서 JMS를 사용하는 가장 쉬운 방법은, monitroing system에서 제공하는 collection agent를 사용하여, Kafka process에 attach
    * 이는 system에서 실행되고 Nagios XI check_jmx plugin or jmxtrans와 같은 JMS interface에 연결되는 separate process일 수도 있음
  * Monitoring agent
    * monitoring agents를 설정하는 방법에 대한 depth dicussion은 이 chapter의 범위를 벗어나며, monitoring agent의 정의에 대한 선택이 너무 많음
      * 조직에서 Java applications의 monitoring 경험이 없는 경우, service형 monitoring을 고려하는 것이 좋음  (monitoring agents, metrics collection points, storage, graphing, alerting을 service pachage에 제공하는 회사가 많기 때문에 이들은 필요한 monitoring agents를 설정하는데 도움이 됨)
  * Finding the JMX Port
    * monitoring systems같이 Kafka broker의 JMX에 연결하는 application을 configure하기 위해, broker는 configure되어 Zookeeper에 저장된 broker 정보인 JMX port를 설정
    * /brokers/ids/<ID> znode는 JSON 포맷의 hostname과 jmx_port keys를 포함하는 broker에 대한 data가 포함됨

* Internal or External Measurements

  * Internal Metrics
    * JMX와 같은 interface를 통해 제공되는 metrics
    * monitor되는 application에 의해 생성되고 제공됨 (application 자체 이외에는 세부 수준이 없음)
    * metrics 예시
      * timing of individual request stages
  * External Metrics
    * 외부에서 measure할 수 있는 metrics
    * Kafka client 혹은 일부 third-party(타사) application이 broker server에 제공하는 metircs
    * merics 예시
      * overall time for a request
      * availability of a particular request type
      * availability (broker recachable)
      * latency (how long does a request task)
    * external measuremetns의 value(가치)에 대한 예시 : monitoring the health of a website
      * web server가 제대로 실행되고 있으며, 보고되는 모든 metrics가 작동하고 있지만 web server와 external users간의 network 문제가 있어서 모든 users가 web server에 연결할 수 없음. 이럴 때 외부 network에서 실행중인 External monitoring은 website의 accessibility를 확인하고 감지하여 상황을 alter(알람)

* Application Health Checks

  * Kafka에서 metrics를 수집하는 방법에 관계 없이, simple healthcheck를 통해서 application process의 전반적인 상태 monitor를 할 수 있는 방법이 있는지를 확인해야 함 *(아래와 같은 2가지 방법으로 수행될 수 있음)*

    1. broker가 살아있는지 죽었는지를 보고하는 external process (health check)
    2. Kafka broker에 의해 보고되는 metrics 결함에 대한 alert (stale metrics)

    * 2번 방법이 효과적이지만, Kafka broker의 failure(결함)과 monitoring system자체의 failure(결함)을 구별하기가 어려울 수 있음

  * Kafka broker의 경우 client가 borker에 연결하는데 사용하는 것과 동일한 port인 external port에 연결하여 Kafka broker가 response하는지를 간단하게 확인할 수 있음

  * Clinet applications의 경우, process가 실행중인지를 간단하게 확인하는 것에서부터, application health(상태)를 결정하는 internal method를 확인하는 것 까지 더 복잡할 수 있음 

* Metric Coverage

  * Kafka에 의해 expose(노출)된 measurements(측정)횟수를 고려할 때, 검토할 것을 선정하는것이 매우 중요함 (특히 measurements를 기반으로 alert을 할 때는 더 중요함)
    * 너무 많은 alerts로 인해 문제가 얼마낮 심각한지 알기가 어려운 "alert fatigue"가 발생하지 않게 함
    * 모든 metrics를 적절하게 정의하고 최신 상태로 유지하는데 드는 resources를 절약
  * alerts가 overwhelming 하거나 종종 incorrect(부정확)할 때, alerts이 applications의 상태를 정확하게 설명하지 않는다고 봄 -> 이 경우 hig-level converage(높은 수준의 적용범위)를 가지는 a few(몇가지) alerts를 가지는 것이 더 advantageous(유리)
    * 큰 문제가 있음을 나타내는 하나의 alert만 표시하면, 문제의 특성을 파악하려면 추가 데이터를 수집해야 할 수 있다는 점이 있음
    * e.g) 자동차의 Engine확인 계기판을 생각해보면, 공기필터, 오일, 배기장치 등의 개별적인 문제를 보여주는 많은 지표들을 계기판에 표시하는 것은 혼란스러울 수 있으므로, 대신 하나의 표시기에 문제저이 있음을 알 수 있도록 표시한 후, 자세한 정보를 찾아서 문제를 정확하게 파악하면 됨
  * highest amount of coverage
    * alerting을 간단하게 유지하기 위해서 가장 높은 coverage를 제공하는 metrics을 살펴볼 에정



### Kafka Broker Metrics

#### Introduce

* 많은 Kafka broker metrics가 존재
  * Kafka broker metrics는 대부분 low-level measurements
  * dvelopers가 특정 이슈를 조사하거나, 추후에 debug 목적으로 정보를 필요로할 때 추가되는 metrics
  * broker내의 모든 기능에 대한 정보를 제공하는 metrics가 있지만, 가장 일반적인 정보는 매일 Kafka를 운영하는데 필요한 정보를 제공
* Who Watches the Watchers? 
  * 많은 organizations에서 Kafka를 사용하여 central monitoring system에서 application metrics, system metrics, logs for consumption을 수집함
    * 이는 monitoring systems에서 applications을 decouple하는 좋은 방법이지만 Kafka 자체에 대한 sepecific concern이 나타남
      * Kafka 자체를 monitoring하는데 system을 사용하는 경우, Kafka가 손상되었을 때 monitoring system의 data flow 또한 손상되므로 이를 알지 못할 가능성이 높음
  * 위와 같은 현상을 해결할 수 있는 방법
    1. Kafka에 대한 별도의 monitoring system을 사용하여 Kafka에 dependency하지 않는 것
    2. multiple datacenter가 있는 경우, 각 data center의 Kafka cluster에 대한 metric을 교차해서 생성하는 것
       * A datacenter의 Kafka cluster에 대한 metrics은 B datacneter에 생성되고, B datacenter의 Kafka cluster에 대한 metrics는 A datacenter에 생성
       * 이 경우 Kafka monitoring 및 alerting이 Kafka의 working에 의존적이지 않은지를 확인해야 함

#### Under-Replicated Partitions

* Kafka broker metrics에서 가장 중요한 metrics
* cluster의 각 broker에서 leader replica인 partitons의 세서 공급하여measurement
  * 이 단일 measurement를 통해 broker에서 부터 resource exhaustion(자원 고갈)까지 Kafka sluter의 여러 문제를 파악할 수 있음

#### Broker Metrics

#### Topic and Partition Metrics

#### JVM Monitoring

#### OS Monitoring

#### Logging



### Client Monitoring

#### Producer Metrics

#### Consumer Metrics

#### Quotas







### Lag Monitoring





### End-to-End Monitoring





### Summary

