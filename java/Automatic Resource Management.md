## Automatic Resource Management

#### try-with-resources문

* 보통 자원은 finally문 안에서 close()를 이용하여 닫았는데, close()도 예외를 발생시킬 수 있다. 그래서 finally 블럭 안에 try-catch문을 추가해서 close()에서 발생할 수 있는 예외를 처리하도록 하였다. 그러나 이렇게 하면 1. 코드가 복잡해져서 가독성이 떨어지며 2. try블럭과 finally블럭 모두에서 예외가 발생하면 try블럭 예외는 무시된다.

* try-with-resources문을 사용하면 try블럭과 리소스를닫는 부분 모두에서 예외가 발생하면 try블럭의 예외에 'suppressed(억제된)'이라는 의미의 머리말과 함께 출력

  > Throwable에는 억제된 예외와 관련된 아래와 같은 메서드가 존재
  >
  > `void addSuppressed(Throwable exception)` : 억제된 예외를 추가
  >
  > `Throwable[] getSuppressed()` : 억제된 예외(배열)을 반환

