## Install kafka-manager

[github/yahoo/kafka-manager](https://github.com/yahoo/kafka-manager)



#### Requirements

1. Kafka
2. Java 8+



#### Configuration

kafka-manager/conf/application.conf

* register zookeeper host

  `kafka-manager.zkhosts="${zookeeper_name}:${port},${zookeeper_name}:${port},.."`



#### Deploy

1. `kafka-manager/sbt clean dist`




#### Start

1. `unzip target/unizversal/kafka-manager-xxx.zip` (Deploy 후 생기는 xxx.zip파일의 압축 해제)
2. `bin/kafka-manager -Dconfig.file=${config_file_path} -Dhttp.port=${port}`



#### 참고자료

https://github.com/yahoo/kafka-manager

https://www.gooper.com/ss/?mid=bigdata&document_srl=3353

http://edbaker.weebly.com/blog/install-and-evaluation-of-yahoos-kafka-manager



