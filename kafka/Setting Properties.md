## Setting Properties

### Zookeeper

#### 파일 위치

config/zookeeper.properties

#### properties

* `dataDir` : zookeeper가 사용할 데이터 디렉토리
* `clientPort` : zookeeper가 사용할 포트
* `maxClientCnxns` : 하나의 클라이언트에서 동시접속하는 개수 제한. 0은 무제한


* `iniLimit` : 하나의 zookeeper 리더에 연결해서 동기화하는 시간
* `syncLimit` : 리더가 아닌 zookeeper 노드가 리더와 동기화하는 시간
* `sever.${id}=${host}:${port}:${port}` : zookeeper 클러스터 server list

#### Set Zookeeper ID

`echo ${id} > myid` : dataDir안의 myid파일에 zookeeper id값을 명시

#### Example

```properties
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
# 
#    http://www.apache.org/licenses/LICENSE-2.0
# 
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# the directory where the snapshot is stored.
dataDir=/tmp/zookeeper
# the port at which the clients will connect
clientPort=2181
# disable the per-ip limit on the number of connections since this is a non-production config
maxClientCnxns=0

initLimit=5
syncLimit=2
server.1=localhost:2888:3888
```



### Kafka

#### 파일 위치

config/server.properites

#### Properties

* `broker.id` : broker의 id
* `zookeeper.connect` : zookeeper 인스턴스 정보
* `log.retention.hours` : 메세지 저장 시간
* `log.retention.bytes` : partition의 물리적 최대 크기. -1이면 무제한 (log.retention.bytes * partition count = physical size of topic)
* `log.dirs` : log directory
* `num.partitions` : partiton 기본개수
* `min.insync.replica` : isr의 최소 개수
* `listeners` : listener names

#### 

