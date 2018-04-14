## Checking System (Ubuntu)

#### 우분투 아키텍처 확인

```uname -m``` 또는 ```uname -p```

* 32Bit : i386, i586, i686, x86
* 64Bit : x64, x86_64, AMD64



#### 카프카 버전 확인

```find ./libs/ -name \*kafka_\* | head -1 | grep -o '\kafka[^\n]*'```

