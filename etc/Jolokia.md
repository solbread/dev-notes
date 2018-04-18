## Jolokia

#### Jolokia?

HTTP 프로토콜을 이용해서 JMX 값을 JSON 형식으로 받아볼 수 있도록 해주는 JMX-HTTP Connector

>  Jolokia is remote JMX with JSON over HTTP. It is fast, simple, polyglot and has unique features. It's JMX on Capsaicin.



#### Jolokia QuickStart

1. download [jolokia-jvm-${version}-agent.jar](https://jolokia.org/download.html)

   > tar.gz파일을 다운받을 경우 agents 폴더에 jolokia-jvm.jar 파일

2. `java -jar jolokia-jvm-${version}-agent.jar --port ${jolokia_port} ${jmx_process_pid}`

> Tomcat에 붙여서 사용하는 방법
>
> https://jolokia.org/tutorial.html



#### 조회 가능한 Mbean 출력

`curl "${jolokia_host}:${jolokia_port}/jolokia/list"`



#### 특정 Mbean 출력

`curl "${jolokia_host}:${jolokia_port}/read/${object_name}"`



#### 참고자료

[jolokia(JMX on Capsalcin)](https://jolokia.org/)