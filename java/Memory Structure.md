## Memory Structure

#### Java Memory Area

- PC 레지스터, JVM 스택, 힙(Heap), 메서드 영역, 런타임 상수(constant) 풀, 네이티브 메서드 스택

  ![java memory area](./picture/java%20memory%20area.png)

- Class Loader Subsystem

  - 클래스나 인터페이스를 JVM으로 로딩

- Execution Engine(실행 엔진)

  - 로딩된 클래스의 메서드들에 포함되어 있는 모든 instruction(명령어) 정보를 실행

- Heap Memory (=Shared Memory)

  - 여러 스레드에서 공유하는 데이터들이 저장
  - JVM이 시작될 때 생성
  - gc가 발생하는 영역
  - 저장되는 데이터 : class instacne, array

- Non-Heap Memory

  - 자바 내부 처리를 위해서 필요한 영역

  - Method Area

    - JVM이 시작될 때 생성

    - 모든 JVM 스레드에서 공유

    - 저장되는 데이터 : Runtime Constant Pool(런타임 상수 풀), 메서드 데이터, 메서드와 생성자 코드, 클래스에 대한 정보(클래스변수를 포함)

      > Runtime Constant Pool : 자바 클래스 파일에는 constant_pool이라는 정보가 포함되어 있는데, 실행 시에 이를 참조하기 위한 영역
      >
      > 실제 상수 값, 실행시에 변하게 되는 참조 정보, 메서드 데이터, 메서드와 생성자 코드

      장점

      - 기동시간의 단축
      - 끝까지 사용되지 않는 클래스(필드, 메서드)가 있을 경우 메모리를 절약 (메서드 영역은 프로그램이 시행되기 시작되기 시작할 때부터 끝날 때까지 계속 존재)

  - Java Threads (JVM stack)

    - 스레드가 시작할 때 생성

    - 저장되는 데이터 : 메서드가 호출되는 정보인 프레임(frame), 지역 변수, 임시 결과, 메서드 수행과 리턴에관련된 정보(매개변수, 리턴값 등)

      > * Stack의 크기는 고정하거나 가변적일 수 있음
      > * 만약 연산을 하다가 jvm의 스택 크기의 최대치를 넘어섰을 경우에는 StackOverflowError 발생
      > * 가변적일 경우 stack의 크기를 늘이려고 할 때 메모리가 부족하거나, 스레드를 생성할 때 메모리가 부족한 경우에는 OutOfMemoryException 발생
      > * 메서드 호출 스택
      >   1. 메서드가 호출되면 수행에 필요한 만큼의 메모리를 스택에 할당받음 (이 메모리는 메서드가 작업을 수행하는 동안 지역변수(매개변수 포함)들과 연산의 중간결과 등을 저장)
      >   2. 메서드가 수행을 마치고나면 사용했던 메모리를 반환하고 스택에서 제거
      >   3. 각 메서드를 위한 메모리상의 작업공간은 서로 구별되며, FILO로 호출스택에 쌓임 (호출스택 제일 위에 있는 메서드가 현재 실행 중인 메서드이며, 아래에 있는 메서드가 바로 위의 메서드를 호출한 메서드)

  - Program Counter(PC) Register

    - 자바의 스레드들은 각자의 Program Counter(PC) register를 가짐
    - native code를 제외한 모든 java code들이 수행될 때 JVM의 instruction 주소를 pc register에 보관

  - Native Internal Threads (Native method stack)

    - java code가 아닌 다른 언어로 된(보통은 C) 코드들이 실행하게 될 떄의 스택 정보를 관리




#### Method VS Heap VS Stack

* Method Area
  * 클래스 변수
  * 메서드 영역은 프로그램이 시행되기 시작되기 시작할 때부터 끝날 때까지 계속 존재하는 정적인 공간
* Heap Area
  * 인스턴스 변수
  * 실행하면서 객체 등에 의해 공간이 확보되고, GC에 의해 축소
* Stack Area
  * 지역 변수
  * 스레드 별로 존재, 필요에 의해 임시적으로 생성 및 삭제




#### Heap Area

![heap memory (./picture/heap%20memory%20(previous%20java8).jpg)](./picture/heap memory (previous java8).jpg)

- Young

  - Young = Eden + Survivor1 + survivor2
  - 두 개의 Survivor 영역 존재
    - 두 영역 사이에는 우선순위는 존재하지 않음
    - 두 개의 영역 중 한 영역은 반드시 비어있어야 함

- Old

- Perm

  - JDK 8부터 사라짐

  - 거의 사용이 되지 않는 영역

  - 클래스와 메서드 정보, intern된 String 정보

    > intern된 String 정보
    >
    > String 클래스에는 intern() 이라는 메서드가 존재
    >
    > 이 메서드를 호출하면 해당 문자열의 값을 바탕으로 한 단순 비교가 가능
    >
    > 참조 자료형은 equals() 메서드로 비교해야 하지만, intern() 메서드가 호출된 문자열들은 == 비교가능
    >
    > 따라서, 값 비교 성능은 빨라지지만, 문자열 정보들이 Perm 영역에 들어가기 때문에 Perm 영역의 GC가 발생하는 원인이 되기도 함

#### Heap Area (after JDK 8)

![heap memory (./picture/heap%20memory%20(after%20java8).jpg)](./picture/heap memory (after java8).jpg)

![heap memory2 (./picture/heap%20memory2%20(after%20java8).jpg)](./picture/heap memory2 (after java8).jpg)

- JDK8 이후는 perm 영역이 사라지고 Meta space 영역으로 대체됨
- 참고자료
  - https://yckwon2nd.blogspot.kr/2015/03/java8-permanent.html
  - https://dzone.com/articles/java-8-permgen-metaspace
  - http://netframework.tistory.com/entry/Java8-PermGen%EC%97%90%EC%84%9C-Metaspace%EB%A1%9C





#### 메모리에 객체가 할당되는 과정

1. 메모리에 객체가 생성되면, Eden에 할당됨
2. Eden 영역에 데이터가 꽉 차면 비어있는 Survivor 영역으로 GC 후 살아남아 있는 객체가 할당됨
   - 객체의 크기가 Survivor 영역의 크기보다 큰 경우 바로 Old 영역으로 할당됨
3. Survivor 영역이 차면 GC 후 살아남은 Eden 영역과 꽉 찬 Survivor 영역의 객체가 비어있는 Survivor 영역으로 할당됨 
4. Survivor 1과 2를 왔다 갔다 하던 객체들은 Old 영역으로 이동됨 





#### 배열의 선언과 생성 과정

1. int[] score;
   * 데이터를 저장할 수 있는 공간은 아직 마련되지 않았으며 int형 배열 참조변수 score를 선언
2. score = new int[5];
   * 연산자 'new'에 의해서 메모리의 빈 공간에 5개의 int형 데이터를 저장할 수 있는 공간이 마련
   * 각 배열요소는 자동으로 int의 기본값인 0으로 초기화
   * 대입연산자 '='에 의해 배열의 주소가 int형 배열 참조변수 socre에 저장





#### 객체의 선언과 생성 과정

1. Tv v;
   * Tv클래스 타입의 참조변수 t를 선언
   * 메모리에 참조변수 t를 위한 공간이 생성되며, 인스턴스가 생성되지 않앗으므로 참조변수로 아무것도 할 수 없음
2. t = new Tv();
   * 연산자 'new'에 의해 Tv클래스의 인스턴스가 메모리의 빈 공간에 생성
   * 멤버변수는 각 자료형에 해당하는 기본값으로 초기화됨
   * 인스턴스 초기화 블럭이 실행됨
   * 생성자 Tv()가 호출되어 수행됨
   * 대입연산자 '='에 의해 생성된 객체의 주소값이 참조변수 t에 저장



#### Stack Size

* Java의 Stack Size는 -Xss 옵션에 의해서 결정
  * -Xss : Java 스레드에 대한 최대 스택 크기
* 관련 자료
  * [Odi's astoundingly incomplete notes](http://www.odi.ch/weblog/posting.php?posting=411)
  * [What is the maximum depth of the java call stack?](https://stackoverflow.com/questions/4734108/what-is-the-maximum-depth-of-the-java-call-stack)
  * [How to know about OutOfMemory or StackOverflow errors ahead of time](https://stackoverflow.com/questions/794227/how-to-know-about-outofmemory-or-stackoverflow-errors-ahead-of-time)

