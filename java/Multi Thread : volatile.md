## Multi Thread : volatile

#### volatile?

* 메인 메모리에 저장하는 변수 : volatile 변수는 CPU 캐시(레지스터)가 아니라 메인메모리로부터 읽어들이며, volatile 변수를 쓸 때도 CPU 캐시(레지스터)가 아닌 메인메모리에 기록

* 멀티스레딩 환경에서 변수의 가시성(Visibility)을 보장

  * 메인메모리만 이용하므로 read/write작업에 대해서 동기화를 해줌

* 경쟁상태 문제를 해결해주지는 않음

  * 여전히 동시에 변수를 읽어들이는 문제가 있으므로 경쟁상태 문제를 해결해 주지 않음

* JDK 1.5 이전버전에서는  Happens-Before가 지원되지 않음

  > Happnes-Before
  >
  > - It Thread A writes to a volatile variable and Thread B subsequently reads the same volatile variable, then all variables visible to Thread A before writing the volatile variable, will also be visible to Thread B after it has read the volatile variable.
  > - The reading and writing instructions of volatile variables cannot be reordered by the JVM (the JVM may reorder instructions for performance reasons as long as the JVM detects no change in program behaviour from the reordering). Instructions before and after can be reordered, but the volatile read or write cannot be mixed with these instructions. Whatever instructions follow a read or write of a volatile variable are guaranteed to happen after the read or write.
  >
  > 한 쓰레드가 volatile 변수를 수정할 때, 단지 이 volatile 변수만이 메인 메모리로 저장되는 것이 아니라, 이 쓰레드가 volatile 변수를 수정하기 전에 수정한 모든 변수들이 함께 메인 메모리에 저장(flushed)된다. 그리고 쓰레드가 volatile 변수를 메인 메모리에서 읽어들일 때, volatile 변수를 수정하면서 메인 메모리로 함께 저장된 다른 모든 변수들도 메인 메모리로부터 함께 읽어들여진다.



#### synchronized와의 차이점

* volatile은 가시성 작업에 대해서만 동기화가 되며 원자성을 가지므로, 경쟁상태 문제를 해결해주지 못한다.
* 반면 synchronized는 lock을 이용하여 하나의 thread만 접근할 수 있게 하므로 경쟁상태 문제도 해결 해 준다.

```java
long b = 324l;
int a = b + 10;
```

a에 volatile 선언을 해도 멀티스레딩시 위험하다. 스레드의 접근 순서에 따라 a의 값을 volatile로 선언해줘도, 스레드의 접근 순서에 따라 가져가는 a의 값이 달라질 것이다. 위 코드는 b를 int로 캐스팅하고 10을 더하고 다시 a에 할당하는 3가지 작업을 거치기 때문이다. 

1. volatile을 사용하지 않은 변수: 마구 최적화가 될 수 있다. 재배치(reordering)이 될 수있고, 실행중 값이 캐쉬에 있을 수 있다.	
2. volatile을 사용한 변수 (1.5미만): 그 변수 자체에 대해서는 최신의 값이 읽히거나 쓰여진다.
3. volatile을 사용한 변수 (1.5이상): 변수 접근까지에 대해 모든 변수들의 상황이 업데이트 되고, 변수가 업데이트된다.
4. synchronziation을 사용한 연산: sync블락 전까지의 모든 연산이 업데이트 되고, sync안의 연산이 업데이트된다.



#### volatile의 한계

multi thread가 공유 변수에 대한 update 연산이 있을 경우 volatile로는 충분하지 않으며, 이 경우 synchronization을 통해 변수의 연산의 원자성(atomic)을 보장해주어야 한다.



#### volatile의 사용처 

하지만 한 thread에서 volatile 변수의 값을 읽고 쓰고, 다른 스레드에서는 오직 변수의 값을 읽기만 할 경우, 읽는 스레드에서는 volatile 변수의 가장 최근에 쓰여진 값을 보는 것을 보장할 수 있다.



#### volatile 변수와 성능

volatile 변수에 대한 읽기와 쓰기는 변수를 메인 메모리로 부터 읽거나 쓰게 되는데, 메인 메모리에 읽고 쓰는것은 CPU 캐시보다 더 비싸다. 또한 volatile 변수는 리오더링을 방지하기 때문에 변수의 가시성을 강제할 필요가 있는 경우에만 volatile 변수를 사용하는 것이 좋다.	



#### Happens-Before

* Java1.5 이상에서 보장되는 성질로 한 쓰레드가 volatile 변수를 수정할 때, 단지 이 volatile 변수만이 메인 메모리로 저장되는 것이 아니라, 이 쓰레드가 volatile 변수를 수정하기 전에 수정한 모든 변수들이 함께 메인 메모리에 저장(flushed)



#### 추가적으로 읽어볼 자료

[jenkov : volatile](http://tutorials.jenkov.com/java-concurrency/volatile.html)

[박철우의 블로그 : 자바 volatile 키워드(윗글 번역)](http://parkcheolu.tistory.com/16)

[Dream Repository : Java의 volatile 키워드에 대한 이해](http://kwanseob.blogspot.kr/2012/08/java-volatile.html)

