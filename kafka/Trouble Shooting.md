## Trouble Shooting

#### Situation1

> ./bin/zookeeper-server-start.sh ./config/zookeeper.properties
>
>  Classpath is empty. Please build the project first e.g. by running './gradlew jar -PscalaVersion=2.11.12'

kafka를 다운받은 후 zookeeper와 kafka의 properties를 설정 한 후 zookeeper를 start하자 위와 같은 에러가 발생하였다. 원인은 kafka source version을 다운받았기 때문이며 kafka binary 파일을 다운받은 이후 해결되었다.