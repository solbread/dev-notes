## Fork and Join Framework

- Parallel Core를 효과적으로 사용하기 위한 Framework

- 작업을 여러개의 Core에 분산시키고 Result Set을 리턴하기 위해 그것들을 Join하는 Framework

- 원리

  - Task를 더이상 나누지 않아도 해결할 수 있을 때까지 나눈다. Divide-and-Conquer 알고리즘과 비슷하다. 이 Framework 의 핵심 개념 한 가지는 이상적인 상태에서 Worker Thread 들은 더이상 유휴상태가 되지 않는다는 것이다. 일감이 떨어진 Worker 가 바쁜 Worker 로부터 작업을 “훔치는” Work-Stealing Algorithm 이 구현되어 있다.

- 구현

  - Fork-Join 메커니즘을 지원하는 Core Class
    - `ForkJoinPool`  : Work-Stealing Algorithm을 구현한 ExecutorService를 특화시켜서 구현
    -  `ForkJoinTask`

  ```java
   int numberOfProcessors = Runtime.getRunTime().availableProcessors();
   ForkJoinPool pool = new ForkJoinPool(numberOfProcessors)
  ```

