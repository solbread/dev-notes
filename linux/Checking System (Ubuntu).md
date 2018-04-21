## Checking System (Ubuntu)

#### 우분투 아키텍처 확인

```uname -m``` 또는 ```uname -p```

* 32Bit : i386, i586, i686, x86
* 64Bit : x64, x86_64, AMD64



#### 카프카 버전 확인

```find ./libs/ -name \*kafka_\* | head -1 | grep -o '\kafka[^\n]*'```

#### 메모리 사용률 확인

`sar -r 1`

`free`

`top -n1 | grep Mem:`

`cat /proc/meminfo | grep Mem`

#### CPU 사용률 확인

`mpstat | tail -1 | awk '{print 100-$11}'`

`top -n 1 | grep -i cpu\(s\)| awk '{print $5}' | tr -d "%id," | awk '{print 100-$1}'`



