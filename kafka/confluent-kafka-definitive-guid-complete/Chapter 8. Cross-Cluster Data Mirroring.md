## Chapter 8. Cross-Cluster Data Mirroring

### Introduce

* 대부분의 경우 single Kafka cluster의 setup(설정), maintenance(유지관리), use(사용)에 대해 다룸
* 그러나 architecture에 둘 이상의 cluster가 필요할 수 있는 몇 가지 시나리오가 존재
  * 경우에따라, clusters는 완전히 분리되며, clusters는 다른 departments(부서)나 use cases에 속하며, 하나의 cluster에서 another cluster로 data를 copy할 이유가 없음. 때로는 SLAs가 다르거나 workloads가 많아 하나의 cluster를 조정하여 multiple use cases를 처리하는 경우도 있고, security requirements가 다른 경우도 있음
  * 다른 use caes에서는 다른 clusters가 interdependent(의존적)하고 administrators가 clusters간에 data를 계속적으로 copy해야 하는 경우도 있음.  대부분의 databases에서 database server간에 data를 계속 copy하는 것을 replication(복제)라고 함.
* MirrorMaker
  * 동일한 cluster의 일부인 Kafka nodes간에 data의 movement(이동)을 설명하기 위해 "replication"을 사용한 후에, Kafka clusters mirroring 간의 data copy를 call
  * Apache Kafka의 내장된 cross-cluster replicator(클러스터 간 복제기)
* 이 Chapter에서 다룰 내용
  * data의 전체 혹은 일부에 대한 cross-cluster mirroring
  * cross-cluster mirroring의 use case를 다룸
  * use cases를 implement하고, architecture pattern의 pros and cons(장단점)을 discuss하는데 사용되는 몇가지 architenctures를 보여줌
  * MirrorMaker 자체의 사용 방법
  * deployment와 performance tuning과 같은 opertaion tips(운영 팁)
  * MirrorMaker에 대한 a few(몇가지) alternaties



### Use Cases of Cross-Cluster Mirroring

1. Regional and central clusters (지역 및 중앙 클러스터)
   * 경우에 따라서 회사는 여러 지리적 regions(지역) 또는 city(도시) 또는 continents(대륙)에 하나 이상의 datacenter를 보유
   * 각 datacenter는 자체 Kafka cluster가 있을 때
     * 일부 applications은 local cluster와의 communication에 의해 작동될 수 있지만,
     * 일부 application은 multiple datacenters의 data가 필요할 수 있으며 이 경우 cross data-center replication solutions을 고려할 수 있음
       * e.g) 수요와 공급에 따라 가격을 수정하는 company : 이 회사는 현존하는각 도시에 datacenter를 가질 수 있으며, 지역의 공급과 수요에 대한 정보를 모으고 그에따가 가격을 조정. 이 모든 정보는 business analysts(비즈니스 분석가)가 회사 전체의 매출 보고서를 실행할 수 있는 center clusetr에 mirror(반영)됨
2. Redundancy (중복)
   * application은 단 하나의 Kafka cluster에서만 실행되며 다른 locastions의 data가 필요하지 않지만, 몇몇의 이유로 전체 cluster가 unavailable해지는 경우에 대해 걱정할 수 있음
   * 위의 상황을 위해 첫번째 cluster에 있는 모든 데이터를 가지는 두번째 Kafka cluster를 두고, emergency(긴급상황)이 발생할 때 두번째 cluster에 application을 연결하고 평소대로 이어가기를 원함
3. Cloud migrations
   * 많은 회사들이 on-premise(사내 구축형) datacenter와 cloud provider 모두에서 business를 운영하고 있음
   * 종종 applications은 redundancy를 위해 cloud provider의 multiple regions(여러 영역)에서 실행되며 때때로 multiple cloud providers가 사용됨
   * 위의 경우 on-premise(사내 구축형) datacetner와 각 cloud region에 적어도 하나의 Kafka cluster가 있는 경우가 많음. 이러한 Kafka cluster는 각 datacenter와 region에서 datacenters간에 data를 efficient하게 trasfer(전송)하기 위해 application에 의해 사용됨
     * e.g)  새로운 application이 cloud에 배포되어있지만, on-premise database에서 실행되고 on-premise database에 저장된 application에 의해 update되는 data의 일부가 필요한 경우, Kafka Connect를 사용하여 local Kafka cluster의 변경사항을 capture한 후에, 새로운 application이 사용할 수 있는 Kafka cluster로 변경사항을 mirror. 이를 통해 crossdatacenter traffic 비용을 제어하고, traffic의 governance(관리)와 security(보안)을 향상시킬 수 있음