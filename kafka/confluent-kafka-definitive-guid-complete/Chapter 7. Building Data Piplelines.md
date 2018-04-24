# Chapter 7. Building Data Piplelines



### Introduce

* Apache Kafka를 이용하여 data pipelines을 build할 때 언급되는 2가지 use case

  1. Apache Kafka의 2 end points 중 하나인 곳에 data pipeline을 만드는 것
     * e.g) Kafka에서 S3로 또는 MongoDB에서 Kafka로 data를 가져오는 것
  2. 두개의 서로 다른 Systems간에 pipeline을 build하지만 Kafka를 intermediary(매개체)로 사용하는 것
     * e.g) Twitter에서 Kafka로 data를 보내고 Kafka에서 Elasticsearch로 data를 보냄으로써 Twitter에서 Elasticsearch로 data를 보냄

* Apache Kafka 0.9 version에 Kafka Connect를 추가했을 때는 Kafka가 LInkedIn 및 다른 대규모 조직에서 위 두가지 use cases로 사용되는것을 확인한 후였음

  * 모든 조직이 data pipelines으로 Kafka를 integrate하는데 있어서 해결해야 하는 어려움이 있는것을 확인하였고, 이러한 어려움을 해결하기 위하여 Kafka APIs를 추가하였음

* Kafka가 data pipelines에 제공하는 주된 가치

  * pipleline 단계 사이에서 매우 큰 reliable buffer역할을 수행
  * data pipeline내의 producers와 consumers를 effectively(효과적으로) decoupling(분리)
  * 이러한 decupling, reliability security, efficiency는 Kafka를 대부분의 data pipelines에 적합하게 만듦

* Putting Data Integrating in Context

  * 일부 organizations는 Kafka를 pipeline의 end point로 생각함

    * 어떻게 Kafka에서 Elastic으로 data를 가져올까? 와 같은 문제를 만남
    * data가 Elastic에 필요하고 현재 Kafka에 data가 있는 경우에는 Kafka에서 Elastic으로 data를 가져올 방법에대해 생각하게 됨



  * Kafka를 Kafka 그 자체가 아닌 적어도 두개의(아마도 더 많은) end points를 포함하는 larger context안에서 사용할 수 있음

  * data-integeration problem에 직면한 모든사람들은 immediate(즉각적인) end points에만 집중하지 말고 bigger picture(큰 그림)을 고려하기를 권장함

  * short-term(단기간)의 integrations에만 집중하면 complex하고 expensive한 data integration mess로 끝나게 됨

  * 이 section에서 다루는 내용들


      * data pipeliens을 build할 때 고려해야 할 몇가지 공통적인 문제
        - Kafka에만 국한된 것이 아니라 일반적인 data integration problem
        - 이러한 문제가 있음에도 왜 Kafka data integration use cases에 적합하고, 이러한 문제들을 address(다루다)하는 방법에 대해서 보여줌
    * Kafka Connect API
      - 일반 producer clients, consumer clients와의 차이점
      - 언제 각 client type을 사용해야 할 지
      - API에 대한 세부사항 (Kafka Connect에 대한 전체적인 설명은 이 장의 범위를 벗어나지만, 간단한 usage example을 제시하고, 더 많은 것을 배울 것을 알려줄 것임)
    * 다른 data integration systems와 Kafka와의 itegrate 방법

    ​



### Considerations When Building Data Pipelines

*모든 data pipeline의 build에 대한 세부사항을 다룰수는 없지만, mutiple system을 integrate(통합)하려는 의도로 software architectures를 design(설계)할 때 고려해야 할 가장 중요한 사항 중 일부를 강조하려고 함*

#### Timeliness

* 일부 시스템은 데이터가 하루에 한 번 큰 bulks로 도착하며, 다른 시스템은 data가 생성된후 a few milliseconds에 도착할 것으로 예상됨. 대부분의 data pipeline은 이 두 extremes 사이의 어딘가에 적합함
* 좋은 data integration systems은 다른 pipeliens에 대해 다른 timeliness requirements를 지원하고, business requirements가 변함에 따라 다른 timetables간에 migration을 쉽게 할 수 있음
* scalable 하고 reliable storage를 갖춘 streaming data platform인 Kafka는 near-real-time(거의실시간) pipeline에서 부터 hourly batches(시간별 배치)까지 모든것을 지원할 수 있음
  * Producer는 필요에 따라 Kafka에 frequently(자주) 혹은 infrequently(가끔씩) write할 수 있음
  * consumers는 도착한 latest events를 read하고 deliver(전달)하거나, 매 시간마다 실행하여 Kafka에 연결하고 이전 시간에 누적된 events를 read하여 batches 처리 할 수 있음
* Kafka는 prducers와 consumers 사이에서 time-sensitivity requirements를 decuple(분리)하는 giant buffer 역할을 함
  * Producer는 evnets를 real-time으로 write할 수 있으며, consumers는 events를 batch처리 할 수 있으며 이 반대의 상황도 가능함
  * Consumers에 의해 consumption reate가 전적으로 좌우되므로, Kafka 자체는 필요할 경우 acks을 delay함으로써 producer에 back-pressure을 apply함

#### Reliable

* single points of failure를 피하고 모든 종류의 failure evetns로부터 자동적으로 복구살 수 있기를 바람
  * data pipeline은 종종 business critical systems에 data가 도착하는 방식
  * timeliness requirement가 few-milliseconds end fo the spectrum에 가까울 때 a few seconds(몇 초)이상 failure하면 커다란 disruptive(혼란/장애)를 야기함
* delivery gurantees
  * 몇몇 systems에서는 데이터를 잃을 수 있지만, 대부분의 경우 at-least-once delivery가 requirement이므로, soure system(출발지 system)의 모든even가 destination에 도달하지만 때때로 retries로 인한 duplicate가 발생할 수 있음
  * source system(출발지 system)의 모든 event가 destination에 loss(손실) 또는 duplication(중복) 없이 도달하는 종종 exactly-once delivery가 필요한 경우도 있음
* Kafka Connect API
  * offsetd을 handle할 때 external systems(외부 시스템들)과 integrate하기 위한 APIs를 제공해줌으로써 Kafka's Connect APIs가 end-to-end exactly-once pipeline을 build
  * 사실 사용 가능한 많은 open source connectors가 exactly-once가 지원됨

#### High and Varying Throughput

* data pipeline은 modern data system(최신 데이터 시스템)에서 요구하는 high thrughputs로 scale(확장)될 수 있어야 함
  * throughput이 갑자기 증가하면 adapt(적응)할 수 있어야 함
  * Kafka가 producers와 consumers 사이의 buffer역할을 하기 때문에, 더이상 consumer throughput과 producer throughput이 couple(결합)되지 않음
  * producer throughput이 consumer의 throughput을 exceed(초과)하면, data는 consumer가 따라잡을 떄 까지 Kafka에 누적되므로 복잡한 back-pressure mechanism을 구현할 필요가 없음
  * consumers나 producers를 independently(독립적으로) 추가하여 확장하는 Kafka의 ability(능력)은 변화하는 requirements에 match하기 위해 pipeline의 어느한쪽을 dynamically(동적으로) and independently(독립적으로) scale(확장)할 수 있게 해줌
* Kafka는 high-throughput distributed system(높은 처리량의 분산 시스템)
  * Kafka는 modest(그다지 크지 않은/적당한) clusters에서도 초당 수백mb를 처리할 수 있으므로, 수요가 늘어남에 따라 data pipeline이 확장되지 않을수가 없음
* Kafka Connect API는 scale out(확장)하는 것 뿐 아니라 work(작업)을 peralleize(병렬화)하는 것에도 focuse
* platform이 data sources와 sinks가 multiple threads of execution간에 작업을 분할하고, single machine에서 running할 때에도 available CPU resources를 사용하는 방법에 대해 다음 section에서 설명
* Kafka는 여러 유형의 compression(압축)을 지원하므로, users와 adminsd는 throughput requirements가 증가함에 따라 network와 storage resources 사용을 제어할 수 있음

#### Data Format

* data pipeline에서 가장 중요한 고려사항 중 하나는 다양한 data formats과 data types을  reconcile(조정)하는 것
  * 지원되는 data types는 database와 다른 storage systems에 따라 다름
  * Kafka의 Avro를 사용하여 XML 과 relational data를 Kafka에 load하고, Elasticsearch에 write할때는 JSON으로, HDFS에 write할 때는 Parquet으로, S3에 write할 때는 CSV로 convert해야 함
* Kafka 자체와 Connect API는 data formats과 관련하여 완전히 agnostic(독립적)
  * producers와 consumers는 모든 format의 데이터를 나타내는 어떠한 serializer도 사용가능
  * Kafka Connect에는 data types과 schemas를 포함하는 자체적인 in-memory objects가 있지만 pulggable(플러그 가능한) converters(변환기)가 모든 format의 records를 저장할 수 있음
  * 따라서 Kafka에 어떤 data 형식을 사용하든 connectors의 선택을 restrict(제한)하지 않음
* 많은 sources와 sinks는 schema를 가짐
  * data의 source에서 schema를 읽고, 저장하고, compatibility(호환성)을 validate(검증)하거나 sink database에서 schema를 update하는데 사용할 수 있음
  * e.g) MySQL에서 Hive로 가는 data pipeline에서, 누군가가 MySQL에 column을 추가하였을 때, 좋은 pipeline은 new data를 load할 때 Hive에도 column이 추가되는지를 make sure(확실하게 해야함)
* Kafka에서 external systems으로 data를 write할 때 Sink connectors는 data가 external system에 write 되는 format을 responsible(담당)
  * Some connectors choose to make this format pluggable. (몇몇 connector는 형식을 선택할 수 있음)
  * e.g) HDFS connector는 Avro와 Parquet formats중 하나를 선택할 수 있음
* 다른 type의 data를 지원하는 것만으로는 충분하지 않으며, generic data integration framework는 다양한 sources와 sinks 사이의 behavior 차이를 handle(처리)해야 함
  * e.g) Syslog는 data를 push하는 surce이고, relational database는 framework에서 data를 pull(가져와야)함. HDFS는 append-only이며, data를 write할 수 만 있지만, 대부분의 system에서는 data를 append하고 기존의 data를 update할 수 있음

#### Transformations

* Transformations(변환)은 다른 requirements보다 controbersial(논쟁의 여지가 있음)
* two schools of building data pipeline : ETL, ELT

1. ELT (Extract-Transform-Load)
   * The data pipeline is responsible for making modifications to the data as it passes through. (data pipeline은 data가 통과할 때 data를 수정할 책임이 있음)
   * data를 저장하고 수정하고 다시 저장할 필요가 없기 때문에, time과 storage를 절약할 수 있음
     * transformation에 따라, 때로는 computation과 storage의 burden이 data pipeline 자체로 옮겨지므로 desirable(바람직)하지 않을 수 있음 
   * The main drawback of this approach is that the transformations that happen to the data in the pipeline tie the hands of those who wish to process the data farther down the pipe. (data pipeline의 data에 발생하는 transformations이 pipe를 따라 멀리 data를 처리하려는 사람들의 손에 묶여있음)
     * MongoDB와 MySQL 사이의 pipeline을 만든 사람이 특정 events를 filter하거나, records로부터 fields를 remove하기로 결정한 경우, MySQL의 데이터에 access하는 모든 users와 applications은 partial data에만 access할 수 있음
     * missing fields에 대한 access가 필요하면, pipeline을 rebuild야 하며, historical data(과거 데이터)는 reprocess(재처리)가 필요함
2. ETL (Extract-Load-Transform)
   * high-fidelity pipeline 또는 data-lake architecture 라고도 함
   * 대상에 도착하는 data가 source data와 가능한 유사하다는 것을 확인하기 위하여 data pipeline은 data type conversion을 중심으로 최소한의 transformation만 수행함을 의미
   * target system은 "raw data"를 collect하고, 모든 필요한 처리가 target system에서 수행됨
   * system이 모든 data에 access할 수 있기 때문에, target system의 users에게 최대한 flexibility(유연성)을 제공 
   * 모든 data 처리가 pipeline과 추가적인 applications으로 split되지 않고 하나의 system에 제한되기 때문에 troubleshoot(문제해결)이 쉬움
   * transformation은 target system의 CPU와 storage resrouces를 사용한다는 것
     * 어떤 경우에 이러한 system은 expensive하고, 가능하면 해당 systems에서 computation을 move(제거)하려는 strong motivation이 있음 

#### Security

* Security(보안)은 항상 concern 해야 하는데, data pipeline의 측면에서 주요 보안문제는 아래와 같음
  * Can we make sure the data going through the pipe is encrypted? (pipe를 통과하는데 data가 encrypt(암호화)되어 있는지 확인할 수 있나요?)
    * 주로 datacenter boundaries(경계)를 cross(넘나드는)하는 data pipeline의 관심사
  * Who is allowed to make modifications to the pipelines? (누가 pipeline을 수정할 수 있나요?)
  * If the data pipeline needs to read or write from access-controlled locations, can it authenticate properly? (data pipeline이 access-controlled 위치에서 read 혹은 write가 필요하다면, ㅇ로바르게 인증할 수 있나요?)
* encrypt
  * sources에서 Kafka로 Kafka에서 sinks로 연결되어 있기 떄문에 Kafka는 wire상의 data를 encrypt(암호화)할 수 있음
  * It also supports authentication (via SASL) and authorization—so you can be sure that if a topic contains sensitive information, it can’t be piped into less secured systems by someone unauthorized. (SASL을 통한 Auathentication(인증)과 authorization(권한부여)를 지원하므로, opic에 sensitive(중요한) information이 포함되어 있으면, 권한이 없는 사람이 보안이 약한 systems에 pipe(연결) 할 수 없음)
  * Kafka가 제공하는 audit(감사) log로 unauthorized access(인증되지 않은 접근)과 authorized access(인증된 접근)을 추적할 수 있음 
    * 약간의 coding을 추가하면, 각 topic의 events가 어디서 왔는지, 누가 수정했는지 추적할 수 있으므로 각 record에 대해 entire lineage(계보)를 제공함

#### Failure Handling

* 모든 data가 완벼갛다고 가정하면 위험함
* 사전에 failure handling을 계획하는것이 중요함
  * faulty(결함있는) records가 pipeline에 들어가는 것을 막을 수 있나?
  * parse할 수 없는 records를 복구할 수 있나?
  * bad records가 (아마 인간에 의해) 고쳐지고 재처리될 수 있나?
  * bad event가 normal event와 똑같이 보이고, a few days later(며칠 후에) 문제를 발견하면 어떻게 되나?
    * Kafka는 모든 events를 오랜 기간동안 저장하기 때문에, 필요한 경우 시간을 되돌아가서 errors를 recover할 수 있음

#### Coupling and Agility (결합도와 민첩성)

* data pipelines의 가장 중요한 목표중 하나는 data sources(원본)과 data targets을 decouple(분리)하는 것
* colupling이 발생할 수 있는 3가지 경우
  * Ad-hoc pipelines (임시 파이프라인)
    * 일부 회사는 applications 쌍 각각에 대해 custom pipeline(사용자 자정 파이프라인)을 build
      * e.g) Logstash를 사용하여 Elasticsearch에 log를 dump하고, HDFS에 logs를 dump하는 Flume, Oracle에서 HDFS로 dta를 가져오는 GoldenGate, MySQL과 XMLs에서 Oracle로 data를 가져오는 Informatica등을 사용함
    * 이는 data pipeline을 특정 end points(종단점)에 tightly(긴밀하게) couple(결합)하고, deploy(배포), maintain(유지관리), monitor(모니터링)에 많은 노력이 필요한 intergration points의 혼란을 만듦
    * 회사가 채택하는 모든 new system은 추가 pipeline을 build하고, 신기술을 채택비용이 올라가며, innovation(혁신)을 inhibit(억제)하게됨
  * Loss of metadata (metadata 손실)
    * data pipeline이 schema metadata를 perserve(보존)하지 않고, schema evolution이 허용되지 않는다면, source에서 data를 produce하는 software와 destination에서 사용하는 software가 tightly(긴밀하게) couple됨
    * schema information이 없으면, 두 software 모두 data를 parse하고 interpret(해석)하는 방법에 대한 information을 포함해야 함
    * eg) Oracle에서 HDFS로 data가 흐르며, DBA가 schema information을 preserve(보존)하지 않고 schema evolution을 허용하지 않고, new Field를 Orcle에 추가하면, HDFS에서 data를 read하는 모든 application은 중단되거나 모든 developer가 동시에 application을 upgrade해야 함
      * 어떤 옵션도 agile(민첩)하지 않음
    * schema eveloution을 지원하는 pipeline은 각 team이 그들의 application을 상황이 악화되는 것을 염려하지 않고 그들의 속도에 맞춰서 수정할 수 있음
  * Extreme processing (극한 처리)
    * 일부 데이터 처리는 data pipelines에 ingerent(내재)되어 있음
    * 결국 다른 data formats이 make sense(받아들여지고)하고 다른 use cases가 지원되는 서로 다른 systems간에 data를 이동함
    *  too much processing ties all the downstream systems to decisions made when building the pipelines. (pipelines을 build할 때 너무 많은 처리량은 모든 downstream system 결정에 연결됨)
      * 결정 : 보존할 필드와 데이터 aggregate(집계)방법 등에 대한 결정
    * agile하지 않고, efficient하지 않고 safe 하지 않은 downstream application의 requirements의 변화에 따라 pipeline은 지속적으로 변경됨
    * 보다 agile(민첩한) 방법은 최대한 많은 raw data를 보존하고 downstream apps이 data처리와 aggregation 관련한 자체 결정을 할 수 있도록 하는 것



### When to Use Kafka Connect Versus Producer and Consumer

(Producer와 Consumer를 사용할 때 VS Kafka Connect를 사용할 때)

#### Introduce

* Kafka에 write하거나 Kafka로부터 read할 때 producer와 consumer를 사용할 수도 있으며, Connect API와 connectors를 사용할 수도 있음
* Kafka clients (Kafka producer, Kafka Consumer)
  * Kafka clients는 application에 포함된 clients
  * application이 Kafka에 data를 write하거나 Kafka로 부터 data를 read하게 함
  * application을 connect하려는 application의 code를 수정할 때, Kafka로 데이터를 push하거나 Kafka에서 data를 pul할 때 Kafka clients를 사용
* Kafka Connect
  * Connect를 사용하여 Kafka를 write하지 않은 datastores와 수정할  수 없거나 수정 하지 않을 code에 connect(연결)
  * Connect는 external datastore에서 Kafka로 데이터를 pull(가져오거나), Kafka에서 external datastore로 data를 push(보냄)
  * connector가 이미 있는 datastores의 경우에는, connectors를 configure해야 하는 nondevelopers가 Connect를 사용할 수 있음
* Kafka를 datasurce에 연결해야 하지만 connector가 아직 존재하지 않으면 Kafka clients 또는 Connect API를 사용하여 application을 write할 수 있음
  * Connect는 아래와 같은 out-of-the-box(격이다른) features를 제공하므로 권장됨
    *  configuration management (구성 관리)
    * offset storage (오프셋 저장)
    * parallelization (병렬 처리)
    * error handling (에러 처리)
    * support for different data types (다른 데이터 유형 지원)
    * standard management REST APIs (표준 관리 REST API)
* Kafka를 datastore에 연결하는 작은 application을 write하는것은 간단하게 들리지만, data types과 configuration과 관련하여 처리해야 하는 세부사항은 many little(많음)
  * Kafka Connect는 대부분 이 작업을 처리하므로 external stores로부터 data를 transport(전송)하는데에 집중할 수 있음

#### Kafka Connect

* Apache Kafka의 일부

* Kafka와 다른 databases간에 data를 move(이동)할 수 있는 scalable(확장가능)하고 reliable(신뢰가능)한 방법을 제공

* Kafka Connect가 실행하고 moving data(데이터 이동)을 담당하는 libraries인 connector plugins을 개발하고 실행하기 위한 API와 runtime을 제공

* Kafka Connect 사용법

  * Kafka Connec는 cluster of worker processes로 실행됨

  1. wokers에 connector plugins을 설치
  2. 특정 configuration으로 실행되는 connectors를 configure하고 manage하기 위하여 REST API를 사용
  3. Connectors는 많은 양의 data를 병렬로 이동시키고 worker nodes에서 더 efficient(효과적)하게 available resources를 사용하기 위하여 추가적인 작업을 시작
  4. Source connector tasks는 source system으로 부터 data를 읽고 Connect data objects를 worker processes에 제공
  5. Sink connector tasks는 workers로부터 connector data object를 가져와서 target data system에 write
  6. Kafka Connect는 Kafka에서 다양한 formats의 data objects를 저장하는 것을 지원하기 위해 convertors를 사용
     * JSON format의 지원은 Apache Kafka의 일부이며, Confluent Schema Registry는 Avro converters를 제공
  7. users는 사용하는 connectors에 관계 없이data가 Kafka에 저장되는 format을 선택할 수 있음

#### Running Connect

* Kafka Connect는 Apache Kafka와 함께 제공되므로 별도로 설치할 필요가 없음

* Production 용도로, Connect를 사용하여 많은 양의 data를 이동하거나 많은 connectors를 실행하려는 경우 Connect를 별도의 servers에 실행해야 함

  * 모든 machines에 Apache Kafka를 설치하고, 일부 servers에는 brokers를 시작하고 다른 servers에서는 Connect를 시작하면 됨

* Connect worker를 시작하는 것은 broker를 시작하는것과 매우 비슷하게, properties file을 이용하여 start script를 호출하면 됨

  > bin/connect-distributed.sh config/connect-distributed.properties

* Connect workers에 대한 Configurations

  * bootstrap.servers
    * Connect가 사용하는 Kafka brokers 목록
    * Connectors는 brokers로 부터의 data 혹은 brokers로 보내는 data를 pipe
    * cluster의 모든 brokers를 지정할 필요는 없지만 적어도 3개 이상을 지정하는 것이 권장사항
  * group.id
    * 같은 group ID를 가진 모든 workers는 Connect cluster의 일부임
    * cluster에 시작된 connector는 모든 workers와 workers의 tasks에 대해 실행됨 
  * key.converter / value.converter
    * Connect는 Kafka에 저장된 여러 data format을 처리할 수 있음
    * 해당 configurations은 Kafka에 저장될 messge의 key와 value에 대한 converter를 설정
    * 기본값은 Apache Kafka에 포함된 JSONConverter를 사용하는 JSON format
    * Confluent Schema Registry의 일부인 AvroConverter로 설정할 수도 있음

* 일부 converters는 converter-specific(conveter 관련/중심) configuration parameters가 포함됨

  *  key.converter.schema.enable / value.converter.schema.enable
    * *Kafka docs에 나와있지 않은 configuration*
    * message에 schema를 포함할지(true) 포함하지 않을지(false)를 제어
  *  key.converter.schema.registry.url / value.converter.schema.registry.url
    * *Kafka docs에 나와있지 않은 configuration*
    * Avro messages는 schema가 포함되어 있는데, Schema Registry의 위치를 구성
  * rest.host.name
    * default value : null
    * REST API의 hostname으로, 이 configuration parameter가 걸정되면 이 interface에 bind됨
  * rest.port
    * default value : 8083
    * REST API의 port

* REST API

  - Kafka Connect의 REST API를 통해 Connectors는 configure되고 monitor됨

  - wokers가 가동되고 cluster가 있으면, REST API를 점검하여 cluster가 작동중인지를 확인

    > gwen$ curl http://localhost:8083/
    >
    > {"version":"0.10.1.0-SNAPSHOT","commit":"561f45d747cd2a8c"}

  - 기본 REST URI에 access하면 실행중인 current version이 return됨

  - 사용가능한 connector plugins을 확인

    > gwen$ curl http://localhost:8083/connector-plugins
    > [{"class":"org.apache.kafka.connect.file.FileStreamSourceConnector"},
    > {"class":"org.apache.kafka.connect.file.FileStreamSinkConnector"}]

    - palin Apache Kafka를 실행하면, 사용가능한 connector plugins은file source와 file sink

* Standalone Mode (독립 실행 형 모드)

  * Kafka Connect에는 standalone mode도 있음
  *  distributed mode와 비슷함
    * distributed mode는 bin/connect-distributed.sh 을 사용
    * standalone mode는  bin/connectstandalone.sh 을 사용
  * REST API 대신command line에서 configuration file을 connector에 전달할 수 있음
  * standalone mode에서는 모든 connectors와 tasks이 하나의 standalone(독립 실행형) worker에서 실행됨
  * Connectors와 tasks를 특정 machine에서 실행해야 하는 경우(e.g. syslog connector는 port에서 listen(수신대기)하므로, 어떤 machine이 실행중인지를 알아야함)와, development 및 troubleshooting을 위해 standalone mode에서 Connect를 실행하는 것이 더 쉬움

#### Connector Example: File Source and File Sink

* 이 예제는 file connectors와 Apache kafka의 일부인 JSON converter를 사용

1.  Zookeeper와 Kafka가 준비되어 있고 작동하는지를 확인해야 함

2. 시작하기 위하여 distributed(분산된) Connect worker를 실행

3. 실제 production environment에서는, high availability를 제공하기 위하여 최소 2~3개를 실행해야 하지만 이 예제에서는 하나의 connect만 시작

   > bin/connect-distributed.sh config/connect-distributed.properties &

4. file source를 시작. 기본적으로 Kafka's configuration을 Kafka topic으로 pipe하여 Kafka configuration을 read하여 configure

   > echo '{"name":"load-kafka-config", "config":{"connector.class":"FileStream-
   > Source","file":"config/server.properties","topic":"kafka-config-topic"}}' |
   > curl -X POST -d @- http://localhost:8083/connectors --header "content-
   > Type:application/json"
   >
   > {"name":"load-kafka-config","config":{"connector.class":"FileStream-
   > Source","file":"config/server.properties","topic":"kafka-configtopic","
   > name":"load-kafka-config"},"tasks":[]}

   * connector를 만들기 위해 cnnector name인 load-kafkaconfig와, connector class와 load하려는 file 및 file을 load하고자 하는 topic을 포함하는 connector configuration map을 포함하는 JSON을 작성

5. Kafka Console Consumer를 사용하여, topic에 configuration을 load하였는지 확인

   > gwen$ bin/kafka-console-consumer.sh --new --bootstrap-server=localhost:9092 --
   > topic kafka-config-topic --from-beginning

6. Kafka Console Consumer에서 아래와 같은 출력이 나옴

   > {"schema":{"type":"string","optional":false},"payload":"# Licensed to the
   > Apache Software Foundation (ASF) under one or more"}
   >
   > \<more stuff here\>
   >
   > {"schema":{"type":"string","optional":false},"payload":"#############################
   > Server Basics
   >
   > #############################"}
   > {"schema":{"type":"string","optional":false},"payload":""}
   > {"schema":{"type":"string","optional":false},"payload":"# The id of the broker.
   > This must be set to a unique integer for each broker."}
   > {"schema":{"type":"string","optional":false},"payload":"broker.id=0"}
   > {"schema":{"type":"string","optional":false},"payload":""}
   >
   > \<more stuff here\>

   * 이는 config/server.properties 파일에 있는 내용
     * JSON을 line by line으로 변환하고 connector에서 topic에 배치
   * 기본적으로 JSON converter는 각 record에 schema를 배치
     * 특별한 경우, schema는 매우 간단한데, string type의 payload라는 single column만 있고, 각 record에대한 file에서 single line을 포함함

7. topic의 내용을 file에 dump하여 file sink converter를 사용

   * 결과 file은 원래 server.properties file과 완전히 동일해야 하며, JSON converter가 JSON records를 아래와 같은 simple text lines로 다시 변환함

   > echo '{"name":"dump-kafka-config", "config":
   > {"connector.class":"FileStreamSink","file":"copy-of-serverproperties","
   > topics":"kafka-config-topic"}}' | curl -X POST -d @- http://localhost:
   > 8083/connectors --header "content-Type:application/json"
   > {"name":"dump-kafka-config","config":
   > {"connector.class":"FileStreamSink","file":"copy-of-serverproperties","
   > topics":"kafka-config-topic","name":"dump-kafka-config"},"tasks":
   > []}

   * source configuration의 변경사항에 주의해야 함
     * connector.class는 FileStreamSource가 아닌 FileStreamSink를 사용
     * file property을 여전히 가지지만, source of records이 아닌 destination file을 refer(가리킴)하므로 하나의 topic을 specify(지정)하기 보다는 topics를 specify
       * sink가 있는 하나의 file에 multiple topics을 write할 수 있으며, source는 하나의 topic에만 write할 수 있음
   * 결과적으로 copy-of-server-properties 파일을 가지게 됨
     * topic을 populate(이주)하는데 사용하는 config/server.properties와 동일함

8. connector의 삭제

   > curl -X DELETE http://localhost:8083/connectors/dump-kafka-config

   * connector를 삭제한 후 Connect worker log를 보면, 다른 connectors가 삭제된 connector의 tasks를 재시작
   * connector가 제거된 후에 workers간에 남아있는 tasks를 rebalance하고 equivalent workloads(동등한 작업부하)를 ensure(보장)하기 위하여 restart

#### Connector Example: MySQL to Elasticsearch

* MySQL table을 가져와서 Kafka topic으로 stream하고, Kafka topic에서 load하여 Elasticsearch에 load하고 contents를 index(색인)

1. MacBook에서 MySQL과 Elasticsearch의 설치

   > brew install mysql
   > brew install elasticsearch

2. connectors가 있는지를 make sure(확인)

   * Confluent OpenSource를 실행중이라면, connectors가 platform의 일부로 install되어 있어야 함


   * Confluent OpenSource를 실행중이지 않다면, GitHub에서 connector를 build

     > 1. Go to https://github.com/confluentinc/kafka-connect-elasticsearch
     > 2. Clone the repository
     > 3. Run mvn install to build the project
     > 4. Repeat with the JDBC connector

3. build된 각 connector의 target directory 아래에 만들어진 jar 파일을 가져와서 Kafka Connect의 class path에 복사

   > gwen$ mkdir libs
   > gwen$ cp ../kafka-connect-jdbc/target/kafka-connect-jdbc-3.1.0-SNAPSHOT.jar
   > libs/
   > gwen$ cp ../kafka-connect-elasticsearch/target/kafka-connectelasticsearch-
   > 3.2.0-SNAPSHOT-package/share/java/kafka-connect-elasticsearch/*
   > libs/

4. Kafka Connect workers이 아직 run되고 있지 않다면 start해야 하며, new connector plugins이 list되어 있는지를 확인

   > gwen$ bin/connect-distributed.sh config/connect-distributed.properties &
   >
   > gwen$ curl http://localhost:8083/connector-plugins
   > [{"class":"org.apache.kafka.connect.file.FileStreamSourceConnector"},
   > {"class":"io.confluent.connect.elasticsearch.ElasticsearchSinkConnector"},
   > {"class":"org.apache.kafka.connect.file.FileStreamSinkConnector"},
   > {"class":"io.confluent.connect.jdbc.JdbcSourceConnector"}]

   * Connect cluster에서 사용할 수 있는 추가적인 connector plugins이 있음
   * MySQL을 사용하기 위해서 JDBC source는 MYSQL driver를 필요로 함
     * 이를 위해 connectors를 copy할 때 Oracle website에서 MySQL용 JDBC driver를 download하고, package를 unzip하고, mysql-connector-java-5.1.40-bin.jar를 lib/ 디렉토리에 copy 했었음

5. JDBC connector를 사용하여 Kafka로 stream할 수 있는 table을 MySQL에 생성

   > gwen$ mysql.server restart
   >
   > mysql> create database test;
   > Query OK, 1 row affected (0.00 sec)
   >
   > mysql> use test;
   > Database changed
   >
   > mysql> create table login (username varchar(30), login_time datetime);
   > Query OK, 0 rows affected (0.02 sec)
   >
   > mysql> insert into login values ('gwenshap', now());
   > Query OK, 1 row affected (0.01 sec)
   >
   > mysql> insert into login values ('tpalino', now());
   > Query OK, 1 row affected (0.00 sec)
   >
   > mysql> commit;
   > Query OK, 0 rows affected (0.01 sec)

   * datagase와 table을 create하고, 예제로 a few rows(몇개의 행)을 insert

6. JDBC source connector를 configure

   * documentation을 보고 available한 configuration options을 find out(알아낼)할 수 있지만, REST API를 사용하여 available한 configuration options을 find out할 수도 있음

   ~~~JSON
   gwen$ curl -X PUT -d "{}" localhost:8083/connector-plugins/JdbcSourceConnector/
   config/validate --header "content-Type:application/json" | python -m json.tool

   {
     "configs": [
       {
         "definition": {
           "default_value": "",
           "dependents": [],
           "display_name": "Timestamp Column Name",
           "documentation": "The name of the timestamp column to use to detect new or modified rows. This column may not be nullable.",
           "group": "Mode",
           "importance": "MEDIUM",
           "name": "timestamp.column.name",
           "order": 3,
           "required": false,
           "type": "STRING",
           "width": "MEDIUM"
         },
         <more stuff>
   ~~~

   * 기본적으로 REST API에 connector의 configuration을 validate하고, empty configuraton을 보내도록 요청함 -> 요청에 대한 응답으로 available configurations에 대한 JSON 정의를 얻음
   * Python을 통해 output을 pipe하여 JSON을 보다 읽기 쉽게 만듦

7. 위의 information을 기반으로, JDBC connector를 configure하고 create해야 함

   > echo '{"name":"mysql-login-connector", "config":{"connector.class":"JdbcSource-
   > Connector","connection.url":"jdbc:mysql://127.0.0.1:3306/test?
   > user=root","mode":"timestamp","table.whitelist":"login","validate.
   > non.null":false,"timestamp.column.name":"login_time","topic.prefix":"
   > mysql."}}' | curl -X POST -d @- http://localhost:8083/connectors --header
   > "content-Type:application/json"
   >
   > ​
   >
   > ```json
   > {"name":"mysql-login-connector","config":{"connector.class":"JdbcSourceConnector","connection.url":"jdbc:mysql://127.0.0.1:3306/test?user=root","mode":"timestamp","table.whitelist":"login","validate.non.null":"false","timestamp.column.name":"login_time","topic.prefix":"mysql.","name":"mysqllogin-connector"},"tasks":[]}
   > ```

8. mysql.login topic에서 data를 read함으로써 작동하는지를 확인

   > gwen$ bin/kafka-console-consumer.sh --new --bootstrap-server=localhost:9092 --
   > topic mysql.login --from-beginning
   >
   > \<more stuff\>
   >
   > ```json
   > {"schema":{"type":"struct","fields":[{"type":"string","optional":true,"field":"username"},{"type":"int64","optional":true,"name":"org.apache.kafka.connect.data.Timestamp","version":1,"field":"login_time"}],"optional":false,"name":"login"},"payload":{"username":"gwenshap","login_time":1476423962000}}
   >
   > {"schema":{"type":"struct","fields":[{"type":"string","optional":true,"field":"username"},{"type":"int64","optional":true,"name":"org.apache.kafka.connect.data.Timestamp","version":1,"field":"login_time"}],"optional":false,"name":"login"},"payload":{"username":"tpalino","login_time":1476423981000}}
   >
   > ```

   * data를 read하였는데 topic이 존재하지 않거나, data가 표시되지 않는다는 errors가 발생하면, Connect worker logs에서 아래와 같은 errors를 확인

     > [2016-10-16 19:39:40,482] ERROR Error while starting connector mysql-loginconnector
     > (org.apache.kafka.connect.runtime.WorkerConnector:108)
     > org.apache.kafka.connect.errors.ConnectException: java.sql.SQLException: Access
     > denied for user 'root;'@'localhost' (using password: NO)
     > at io.confluent.connect.jdbc.JdbcSourceConnector.start(JdbcSourceConnector.
     > java:78)

     * connection string을 올바르게 가져오기 위해 여러번 시도했음. 다른 문제는 class path에 driver가 있거나 table을 read하는 permission이 포함됨
     * connector가 실행중인동안, login table에 추가적인 rows를 insert하면, mysql.login topic에 반영되는 것을 즉시 확인해야 함
     * MySQL data를 Kafka로 가져오는 것은 그 자체로는 유용하지만, Elasticsearch에 data를 write하여 더 재미있게 example을 확장

9. Elasticsearch를 시작하고, local port에 access하여 작동 여부를 확인

   > gwen$ elasticsearch &
   > gwen$ curl http://localhost:9200/
   >
   > ```json
   > {
   >   "name" : "Hammerhead",
   >   "cluster_name" : "elasticsearch_gwen",
   >   "cluster_uuid" : "42D5GrxOQFebf83DYgNl-g",
   >   "version" : {
   >     "number" : "2.4.1",
   >     "build_hash" : "c67dc32e24162035d18d6fe1e952c4cbcbe79d16",
   >     "build_timestamp" : "2016-09-27T18:57:55Z",
   >     "build_snapshot" : false,
   >     "lucene_version" : "5.5.2"
   >   },
   >   "tagline" : "You Know, for Search"
   > }
   > ```

10.  connector를 시작 

    > echo '{"name":"elastic-login-connector", "config":{"connector.class":"ElasticsearchSinkConnector","
    > connection.url":"http://localhost:
    > 9200","type.name":"mysql-data","topics":"mysql.login","key.ignore":true}}' |
    > curl -X POST -d @- http://localhost:8083/connectors --header "content-
    > Type:application/json"
    >
    > ​
    >
    > ```json
    > {"name":"elastic-login-connector","config":{"connector.class":"Elasticsearch-SinkConnector","connection.url":"http://localhost:9200","type.name":"mysqldata","topics":"mysql.login","key.ignore":"true","name":"elastic-loginconnector"},"tasks":[{"connector":"elastic-login-connector","task":0}]}
    > ```

    * connection.url
      * 간단히 이전에 configure된 local Elasticsearch server의 URL
    * topics
      * Each topic in Kafka will become, by default, a separate Elasticsearch index, with the same name as the topic. (기본적으로,  Kafka의 각 topic은 같은 이름의 토픽을 가지는 별도의 Elasticsearch index가 됨)
      * Example에서는 Elasticsearch에 write하는 유일한 topic은 mysql.login
    * type.name
      * topic내에서 write중인 data의 type을 정의
      * 하나의 topic에 있는 모든 events가 same type으로 가정하므로, type.name=mysql-data로 hardcode
    * key.ignore
      * MySQL에서 table을 정의할 때, primary key를 제공하지 않았음 -> Kafka의 evnets는 null keys를 가짐 
      * Kafka의 events는 key가 없기 때문에, Elasticsearch connectors는 topic name, partition ID, offset을 각 event의 key로 사용하도록 지시해야 함

11. mysql.login data를 가지는 index가 create 되었는지 확인

    > gwen$ curl 'localhost:9200/_cat/indices?v'
    > health status index pri rep docs.count docs.deleted store.size
    > pri.store.size
    > yellow open	mysql.login	5	1	3	0	10.7kb
    > 10.7kb

    * index가 없다면, Connec worker log에서 error를 확인

      * missing configuration 또는 libraries는 errors의 일반적인 원인인데, 둘다 정상이라면 records에 대해 index를 검색 할 수 있음

        >gwen$ curl -s -X "GET" "http://localhost:9200/mysql.login/_search?pretty=true"
        >
        >​
        >
        >```json
        >{
        >  "took" : 29,
        >  "timed_out" : false,
        >  "_shards" : {
        >    "total" : 5,
        >    "successful" : 5,
        >    "failed" : 0
        >  },
        >  "hits" : {
        >    "total" : 3,
        >    "max_score" : 1.0,
        >    "hits" : [ {
        >      "_index" : "mysql.login",
        >      "_type" : "mysql-data",
        >      "_id" : "mysql.login+0+1",
        >      "_score" : 1.0,
        >      "_source" : {
        >        "username" : "tpalino",
        >        "login_time" : 1476423981000
        >      }
        >    }, {
        >      "_index" : "mysql.login",
        >      "_type" : "mysql-data",
        >      "_id" : "mysql.login+0+2",
        >      "_score" : 1.0,
        >      "_source" : {
        >        "username" : "nnarkede",
        >        "login_time" : 1476672246000
        >      }
        >    }, {
        >      "_index" : "mysql.login",
        >      "_type" : "mysql-data",
        >      "_id" : "mysql.login+0+0",
        >      "_score" : 1.0,
        >      "_source" : {
        >        "username" : "gwenshap",
        >        "login_time" : 1476423962000
        >      }
        >    } ]
        >  }
        >}
        >```

        * new records를 MySQL table에 추가하면, Kafka의 mysql.login topic과 crresponding(해당하는) Elasticsearch index에 자동으로 나타남

#### Build Your Own Connectors

* Introduce
  * 지금까지 JDBC source 및 Elasticsearch sink를 build하고 install하는 방법을 살펴보았으므로, use case에 적합한 connectors 쌍을 build하고 use할 수 있음
  * Confluent는 회사와 community에 의해 write되고 support된 알고 있는 connectors 목록을 maintain(유지관리)함
  * list에서 사용하려는 connector를 선택하고, GitHub repositry에서 build하고, documentaion 혹은 REST API에서 pull한 configuraion을 기준으로 configure하고, Connect worker cluster에서 실행할 수 있음
* Connector API는 public(공용)이며 누구나 new connector를 만들 수 있음
  * In fact, this is how most of the connectors became part of the Connector Hub—people built connectors and told us about them. (대부분의 connector는 Connector Hub의 일부가 되는 방식임. Connector Hub는 사람들이 connectors를 build하고 connectors에 대해 이야기를 함)
  * integrate(통합)하려는 datastore를 hub에서 available하지 않는 경우, 직접 write하는 것이 좋음
  * connectors를 직접 write하면 다른 사람들이 발견하고 사용할 수 있도록 community에 기여할 수 있음
* 기존 connectors를 시작점으로 보고, maven archtype을 사용하여 jumpstart하는 것이 권장됨
* *Connector를 build하는 것과 관련된 세부사항을 논의하는 것은 이 장의 범위를 벗어나지만, official documentation에서 관련 내용을 배울 수 있음*
  * Apache Kafka community mailing list(users@kafka.apache.org)에 최신 connectors를 보여주거나 도움을 요청할 것을 권장

#### A Deeper Look at Connect

* Introduce

  * Connect가 작동하는 방식을 이해하려면, 3가지 기본 개념과 이들이 상호작용 하는 방식을 이해해야 함
  * Connect를 사용하려면 clust of workers를 실행하고 connectr를 start/stop해야 함
  * converter에 의해 data가 처리됨
    * e.g) MySQL의 row를 JSON reocrds로 convert하며, 이를 connector는 Kafka에 write

* Connectors and tasks

  * Connector plugin은 Connectors와 tasks 2부분으로 구성된 connectorAPI를 implement(구현)

  * Connectors

    * connector는 3가지 중요한 사항을 담당

    1. connector에 대해 실행할 tasks의 수를 결정
       * e.g)  JDBC source connector는 database에 연결하고 복사할 기존 table을 찾고, max.tasks configuration의 값과 tables의 개수 중 작은 것을 선택하여 필요한 tasks의 개수를 결정
    2. tasks간의 data-copying 작업을 분할하는 방법 결정
       * e.g) 실행될 tasks의 수를 결정하면, connector configuration과 각 task에 assign된 복사할 tables의 목록을 사용하여 각 task에 대한 configuration을 generate
       * taskConfigs() method는 실행하고자 하는 task의 configuration으로 구성된 list of maps을 return
    3. workers에서 tasks의 configuration을 가져와서 전달
       * e.g) workers는 tasks를 시작하고 각각의 unique configuration을 제공하여 database에서 tables의 unique subset을 copy

    * REST API를 통해 connector가 시작될 떄, 모든 노드에서 시작될 수 있으며 이후에 시작하는 tasks도 모든 node에서 실행될 수 있음

  * tasks

    * tasks는 실제로 data를 Kafka에서 가져오고 보내는 일을 담당

    1. 모든 tasks는 worker로부터 context를 receive함으로써 initialize
       * Source context는 source task가 source records의 offset을 저장할 수 있도록 하는 object가 포함됨 (e.g. file connector에서 offsets은 file의 현재 위치이고, JDBC source connector에서 offsets은 table의 primary key id가 될 수 있음)
       * Sink connector의 Context는 connector가 Kafka로 부터 받은 records를 control하도록 하는 methods가 포함되어 있음
       * 이 method는 back-pressure을 apply 하고 retry 하고 exactly-once delivery를 위해서 외부에서 offsets을 store하는 등의 작업에 사용됨
    2. tasks가 initialize된 후에는 connector가 task에 대해 생성한 configuration을 포함하는 Properties object와 함께 시작됨
    3. tasks가 시작되면 source tasks는 external system에서 poll하고, worker가 Kafka brokers에게 send하는 records 목록을 return
    4. Sink tasks는 Kafka에서 worker를 통해 records를 수신하며 외부 시스템에 기록을 write함

* Workers

  * 역할
    * Kafka Connect의 worker process는 connectors와 tasks를 실행하는 "containier" processes
    * connectors와 connectors의 configuration을 정의하는 HTTP requests 처리
    * connector configuration 을 저장
    * connectors와 connectors의 tasks를 시작
    * appropriate configuration을 전달
    * source connectors와 sink connectors의 offsets을 자동으로 commit
    * tasks에서 errors가 throw 되었을 때 이를 retry
  * connectors와 tasks가 reassign 되는 경우
    * worker process가 중단되거나 crash되는 경우, Connect cluster안의 다른 workers는 Kafka's consumer protocol의 heartbeat를 사용하여 인지하고, connectors와 해당 worker에서 실행되던 tasks를 남아있는 workers에게 reassign
    * new worker가 Connect cluster에 join한 경우, 다른 workers는 이를 인식하고, connectors 또는 tasks를 assign하여 모든 workers 사이에 부하가 분산되도록 함
  * connectors and tasks VS workers
    * 역할의 차이
      * connectors와 tasks는 data integration의 "moving data(데이터 이동)"부분을 담당
      * workers는 REST API, configuration management, reliability, high availability, scaling, load balaning을 담당
    * 이러한 concern(관심사/역할)의 분리는 기본 consumer/producer APIs와 비교되는 Connect APIs의 장점
      * 숙련된 developer는 Kafka로부터 data를 read하고 database에 insert하는 code를 write하는 데에는 하루 이틀 정도 소요되지만, configuration, errors, REST APIs, monitoring, deployment, scaling up/down, handling failures를 처리해야 하는 경우 몇달이 걸릴 수 있음
      * 그러나 connector로 data copy를 implement하면, connector는 complicated operational issue(복잡한 운영 문제)를 처리하는 workers에 plugin되어 걱정할 필요가 없음

* Converters and Connect’s data model

  * Kafka Connect API에는 data를 describe(설명)하는 data objects와 schema가 모두 포함된 data API가 포함되어 있음
  * Connector의 전체 절차
    1. source connector가 data를 read하여 Connector schema object를 construct(만듦)
       * e.g) JDBC source는 database에서 column을 read하고 database에서 return된 columns의 data type을 기반으로 Connector schema object를 construct(만듦) 
    2. schema를 사용하여 database record의 모든 field를 포함하는 Struct(구조체)를 생성
    3. 각 clumn에 대해 column의 이름과 값을 clumn에 저장
    4. 모든 source connector는 source system에서 event를 read하고,  schema와 value의 한 쌍을 generate 
    5. Sink connectors는 scheman와 value 쌍을 가져와서 schema를 사용하여 value를 parse하고 target system에 insert
  * Connet wrokers가 otbjects를 Kafka에 store하는 방법
    1. user가 worker 혹은 connector를 configure할 때, Kafka에 data를 store하는데 사용할 converter를 선택
       * 현재 사용 가능한 converter는 Avro, JSON, Strings
       * JSON converter는 result record에 schema를 포함하거나 포함하지 않도록 configure될 수 있으며, structured data(구조화된 data)와 semistructured(반 구조화 된 data) 를 모두 지원할 수 있음
    2. Connector가 Data API reocrd를 worker에게 return하면, worker는 configure된 converter를 사용하여 record를 Avro object 또는 JSON object 또는 string으로 convert하고, Kafka에 결과를 store
  * Kafka에 store된 data를 objects로 read하는 방법
    1. connect worker가 Kafka에서 record를 read하면, configure된 converver를 이용하여 Avro 또는 JSON 또는 string의 kafka의 format에서 Connect Data API record로 record를 conver하고, 이를 sink connector에 전달
    2. sink connector는 destination system에 insert
  * Connect API는 connector implementation에 독립적으로, Kafka에 store된 다양한 유형의 data를 지원할 수 있음. 즉, 모든 connector는 converter가 available하는 경우, 모든 record type에 사용할 수 있음

* Offset management

  * Offset management는 workers가 connector에 대해 수행하는 convenient services 중 하나
    * REST API를 통하여 deployment(배포) 및 configuration management
  * connectors는 connectors가 이미 처리한 data를 알아야 하며, Kafka가 제공한 APIs를 사용하여 이미 처리된 events에 대한 정보를 유지할 수 있음
  * source connectors에는 connector가 Connect workers에 return하는 records에 logical partition과 logical offset이 포함됨
    * Kafka partiton과 Kafka offsets이 아닌 source system의 필요에 따른 partitions과 offsets
    * e.g) file source에서 partition은 file이 될 수 있으며, offset은 file의 line number(행 번호) 또는 character number(문자 번호)가 될 수 있음
    * e.g) JDBC source에서 partition은 database table이 될 수 있으며, offset은 table의 record의 ID가 될 수 있음
  * source connector를 write하는데 있어서 가장 중요한 design decisions(설계 결정들) 중 하나는, source system의 data를 partition하고, offsets을 track하는 좋은 방법을 결정하는 것
    * connector가 달성할 수 있는 level of parallelism(병렬 처리 수준)과 at-least-once 또는 exactly-once semantics로 deliver 할 수 있는지의 유무에 영향을 미침
  * Source Connector가 Offset을 관리하는 절차
    1. source connector가 각 record의 source partition과 offset을 포함하는 records list를 return하면, worker는 Kafka broker에 records를 send
    2. broker가 records를 성공적으로 acknowledge하면, worker는 Kafka에게 보낸 records의 offset을 store
       * storage mechanism은 plugin 가능하며, 보통 Kafka topic
       * storage mechanism을 통해 restart 혹은 crash 이후에, connectors는 가장 최근에 저장된 offset에서 event를 처리하기 시작할 수 있음
  * Sink Connector가 Offset을 관리하는 절차
    1. 이미 가지고 있는 topic, partition, offset idnetifier(식별자)가 있는 Kafka records를 read
    2. destination system에 recods를 저장하는 connector put() method를 호출
       * connector가 success를 report하면, usual consumer commit methods(일반 consumer commit 방법)을 사용하여 connector에 제공한 offset을 다시 Kafka로 commit
  * framework 자체에서 제공하는 Offset tracking은 developers가 connectors를 write하고 다른 connectors를 사용할 때 일관된 level의 behavior을 보장하도록 함



### Alternatives to Kafka Connect

* Introduce
  * 지금까지 Connect API에 대해 자세히 살펴보았는데, Connect API가 제공하는 convenience(편리함)과 reliability(안정성)은 좋지만, Kafka에서 data를 가져오는 유일한 방법은 아님
  * Connect API의 다른 alternatives(대안)과 보편적으로 사용되는 경우를 해당 section에서 다룸
* Ingest Frameworks for Other Datastores
  * 어떤 사람들은 data architectures를 Hadoop이나 Elasticsearch와 같은 systme을 중심으로 build(구축)
    * 이러한 시스템에는 자체적인 data ingestion tools(데이터 처리 도구)인 Flume for Hadoop과 Logstash 또는 Fluentd for Elasticsearch가 있음
  * Connect API VS Ingest Framework
    * Kafka가 architecture의 핵심이고 수많은 surces와 sinks를 연결하는 것이 목표인 경우에는 Kafka Connect API를 사용하는 것이 좋음
    * Hadoop 중심 혹은 Elasticsearch 중심 system을build하고 Kafka가 해당 system에 ㄷ한 수많은 inputs 중 하나라면 Flume이나 Logstash를 사용하는 것이 좋음 
* GUI-Based ETL Tools
  * infrmatica와 같은 old-school(구식) system,Talend와 Pentaho와 같은 open source alternatives(대안), Apache Nifi와 StreamSets와 같은 newer alternatives는 data source와 destination 모두로써 Apache Kafka를 suppor(지원)
    * 위와 같은 systems을 이미 사용하고 있다면 이러한 stystems을 사용하는 것이 좋음
      * e.g. 이미 Pentaho를 사용하여 모든 작업을 수행하고 있다면, Kafka를 위한 another data integration system을 추가하는 것에 관심이 없을 수 있음
  * ETL(extract, transform, load) pipelines을 build하기 위하여 GUI-based approach(GUI기반 접근방식)을 사용하는 경우에도 의미가 있음
    * 이 시스템의 가장 큰 단점은 일반적으로 involved(복잡한) workflows를 위해 build되었으며, Kafka 안팎으로 data를 가져오고 싶을 때 다소 heavy하고 involved(복잡한)
    * 'Tranformations'절에서 언급했듯이, 디부분의 ELT 도구는 불필요한 complexity를 추가하는 반면 data integration(데이터 통합)은 모든 조건에서 faithful한 message 전달에 초점을 맞춰야 함
  * Kafka는 Connect를 이용하여 data integration, prducers와 consumers를 이용하여 application integration, stream processing 모두를 처리할 수 있는 platform이며 권장됨
    * 오로지 data stores를 integrate하는 ETL tool은 Kafka로 대체될 수 있음
* Stream-Processing Frameworks
  * 거의 모든 stream-processing frameeworks에는 Kafka로부터 events를 read하고 다른 systems에 write할 수 있는 기능이 포함되어 있음
  * destination system이 support 되고, 이미 Kafka로부터 events를 처리하기 위해서 stream-processing framwork를 사용하고 있따면, data integration에도 동일한 framwork를 사용하는것이 합리적임
    * Kafka에 처리되는 event를 저장하지 않고 단순이 read하고 다른 system에 write하므로 이로 인해 stream-processing workflow의 한 단계가 절약됨
    * 단점은 messages의 lost(손실)과 corrupted(손상)과 같은 문제의 troubleshoot를 하기가 어려울 수 있음



### Sumaary

* use of Kafka for data integration
* reasons to use Kafka for data itegration
* general considerations for integration solutions
* why Kafka and its Connect APIs are a good fit
* several examples of how to use Kafka Connect in different scenarios
* How Connect works
* few algernatives to Kafka Connect

- 추가내용 : Reliable
  - 결론적으로 어떤 data integration solution을 사용하든지 가장 중요한 것은 모든 failure condition(오류 조건들)에서 모든 messages를 전달할 수 있는 기능
  - Kafka Connect는 Kafka의 검증되고 진정한 reliability featrues의 integration을 기반으로하여 매우 reliable하다고 믿지만, 선택한 system을 아래와 같은 test하는 것이 중요함
    - 선택한 data integration system이 message를 누락하지 않고 stopped processes(중지된 프로세스), crashed machines(손상된 시스템), network delays(네트워크 지연), high loads(높은 부하)에서도 견딜 수 있는지를 확인해야 함
  - data integration system은 messages를 deliver하는 하나의 job을 가짐
  - data systems을 integrate할 때, reliability는 가장 중요하고 유일한 requirement
  - reliability를 확인하는 방법
    - data system을 선택할 때는, "Considerations When Building Data Pipeline" section에 소개된 requirements를 review(검토)하고, 선택한 시스템이 해당 requirements를 만족하는지를 확인하는 것이 중요함
    - requiremnets를 지원하는 방식으로 data integration solution을 사용하고 있는지 확인해야 함
    - Kafka가 at-least-once semantics를 지원하는 것만으로는 충분하지 않음 (exactly-once를 지원해야 함)

