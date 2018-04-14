## Kafka Command

#### Start the Server

```bin/zookeeper-server-start.sh -daemon config/zookeeper.properties```

`bin/kafka-server-start.sh -daemon config/server.properties`



#### Create a Topic

`bin/kafka-topic.sh --create --zookeeper ${host_name}:${port} --replication-factor ${factor_number} --partition 2 --topic ${topic_name}`



#### Send some messages

`bin/kafka-console-producer.sh --broker-list ${host_name}:${port} --topic ${topic_name}`



#### Start a consumer

`bin/kafka-console-consumer.sh --bootstrap-server ${host_name}:${port} --topic ${topic_name} --from-geginning`



#### 참고자료

[Apache Kafka QuickStart](https://kafka.apache.org/quickstart)