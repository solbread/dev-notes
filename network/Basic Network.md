## Basic Network

#### 세마포어(Semaphore)

* [위키백과 - 세마포어](https://ko.wikipedia.org/wiki/%EC%84%B8%EB%A7%88%ED%8F%AC%EC%96%B4)

* 정의

  * 두 개의 원자적 함수로 조작되는 정수 변수로서, 멀티프로그래밍 환경에서 공유 자원에 대한 접근을 제한하는 방법으로 사용

* 한계

  * 철학자들의 만찬 문제의 고전적인 해법이지만 모든 교착상태를 해결하지는 못함
  * P함수와 V함수의 동작은 독립적이기 때문에 잘못 사용하는 경우 문제가 발생 -> 고급 언어에서 동기화를 제공해야 함
    * P - 임계 구역 - P : 현재 프로세스가 임계 구역에서 빠져나갈 수 없게 된다. 또한 다른 프로세스들은 임계 구역에 들어갈 수 없으므로 교착 상태(Deadlock)가 발생한다.
    * V - 임계 구역 - P : 2개 이상의 프로세스가 동시에 임계구역에 들어갈 수 있으므로 상호 배제(Mutual Exclusion)를 보장할 수 없게 된다.

* 구성

  * 세마포어는 S라는 정수값을 가지는 변수이며, P와 V라는 명령에 의해서만 접근할 수 있음
  * P는 임계구역에 들어가기 전 수행되고, V는 임계구역에서 나올 때 수행
  * 이 때 변수 값을 수정하는 연산은 모두 원자성을 만족해야 함. 즉, 한 프로세스(또는 스레드)에서 세마포어 값을 변경하는 동안 다른 프로세스가 동시에 이 값을 변경해서는 안 됨

* 방법1 : 바쁜 대기(busy waiting을 이용)

  ```
   P(S) {
       while S <=0; // 아무것도 하지 않음 (반복문)
       S--;
   }
  
   V(S) {
       S++;
   }
  ```

  * 임계 구역에 들어갈 수 있을 때 까지 빈 반복문을 수행하기 때문에, 단일처리기 다중프로세스 환경에서 처리기 효율이 떨어짐. 또한 대기 중인 프로세스들 중 어느 것을 임계구역에 진입시킬지를 결정할 수 없음

* 방법2 : 재움 큐를 활용하여 프로세스를 재움

  ```
   P(S) {
       S--;
       if S < 0
           // 이 프로세스를 재움 큐에 추가 (잠 듦)
   }
  
   V(S) {
       S++;
       if S <= 0
           // 재움 큐로부터 프로세스를 제거 (깨어남)
   }
  ```

  

#### Example

Java의 Semaphore class를 상속하여 구현

```java
public class SemaphoreResource extends Semaphore {

    private static final int INTERVAL = 1000;

    public SemaphoreResource(int permits) {
        super(permits);
    }

    public void use(String threadName) throws InterruptedException {

        while(this.availablePermits() == 0) {
            Thread.sleep(INTERVAL);
                System.out.println(threadName + " sleep for waiting... ");
        }
        acquire(); // 세마포어 리소스 확보
        System.out.println(threadName + " do task : remained permits : " + this.availablePermits());

    }

    public void release(String threadName) throws InterruptedException {
        release(); // 세마포어 리소스 해제
        System.out.println(threadName + " task end : remained permits : " + this.availablePermits());
    }

}
```

* acquire : 세마포어 리소스 확보
* release : 세마포어 리소스 해제
* this.availablePermits() : 사용가능한 프로세스 수