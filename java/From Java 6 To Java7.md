## From Java6 To Java7

#### Diamond Operator

* 제네릭을 사용하여 객체를 생성/초기화 할 때, Java7에서는 왼쪽에 선언한 것을 바탕으로 컴파일러가 타입을 추측할 수 있도록 지원하여 Diamond Operator를 사용 가능

  ```java
  // Before Java7
  List<Integer> list = new ArrayList<Integer>();
  
  //In Java7
  List<Integer> list = new ArrayList<>();
  ```

#### Using String in switch statements 

* Java7 이전에는 Primitive, Enumerated 자료형만 statements에 사용할 수 있었지만, Java7에서 String이 추가되었다.

#### Automatic Resource Management

* Closable interface를 구현한 리소스는 try문안에 리소스를 선언하면 자동으로 리소스가 관리된다.
* 리소스가 여러개면 세미콜론(;)으로 구분하여 선언한다.

#### int형 자릿수 표현 (Using Underscores)

```java
int million = 1_000_000;
```

#### Multi Multi - Catch

```java
  public void newMultiMultiCatch() {
    try {
      methodThatThrowsThreeExceptions();
    } catch (ExceptionOne e) {
      // log and deal with ExceptionOn
    } catch (ExceptionTwo | ExceptionThree e) {
      // log and deal with ExceptionTwo and ExceptionThree
    }
  }
```

#### Java.nio.file 2.0

* Path, Paths, FileSystem, FIleSystems
* File Change Notification (WatchService API) 추가
  * 변화에 따라 Notification 이벤트 수신
  * 참고) git - File Handling



#### Fork and Join Framework

* Parallel Core를 효과적으로 사용하기 위한 Framework
* 작업을 여러개의 Core에 분산시키고 Result Set을 리턴하기 위해 그것들을 Join하는 Framework
* 원리
  * Task를 더이상 나누지 않아도 해결할 수 있을 때까지 나눈다. Divide-and-Conquer 알고리즘과 비슷하다. 이 Framework 의 핵심 개념 한 가지는 이상적인 상태에서 Worker Thread 들은 더이상 유휴상태가 되지 않는다는 것이다. 일감이 떨어진 Worker 가 바쁜 Worker 로부터 작업을 “훔치는” Work-Stealing Algorithm 이 구현되어 있다.



#### Supporting dynamism

> Java는 변수, 메서드와 리턴값의 타입이 Compile 단계에서 결정되는 정적 언어이다. JVM 은 Runtime 단계에서 타입 정보를 찾을 필요없이 철저하게 정의된 바이트코드를 실행한다.
>
> 타입이 동적으로 정해지는 언어들도 있다. Ruby, Python, Clojure 가 그렇다. 이 언어들은 타입이 Runtime 단계에서 결정된다.
>
> 자바 진영에서 동적 언어를 효율적으로 임시 처리하는 것에 대한 수요가 커졌다. 그동안 JVM 에서 동적 언어를 실행할 수는 있었지만, 상당한 제약과 제한이 있었다.
>
> Java 7 에서 동적 호출이라는 새로운 기능이 추가되었다. 이것은 자바 이외의 언어의 요구사항을 처리할 수 있도록 VM 을 변화시킨다. java.lang.invoke 라는 새로운 패키지는 MethodHandle, CallSite 등과 같은 클래스로 구성되어 있고, 동적 언어에 대한 지원을 확장하기 위해 만들어졌다.

* 



#### 	Reference

[JunoLee - 자바7와 자바8에 대하여](https://johanneslee.github.io/articles/page7/)