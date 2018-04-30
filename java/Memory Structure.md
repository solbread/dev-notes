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

    - 저장되는 데이터 : Runtime Constant Pool(런타임 상수 풀), 메서드 데이터, 메서드와 생성자 코드

      > Runtime Constant Pool : 자바 클래스 파일에는 constant_pool이라는 정보가 포함되어 있는데, 실행 시에 이를 참조하기 위한 영역
      >
      > 실제 상수 값, 실행시에 변하게 되는 참조 정보, 메서드 데이터, 메서드와 생성자 코드

  - Java Threads (JVM stack)

    - 스레드가 시작할 때 생성

    - 저장되는 데이터 : 메서드가 호출되는 정보인 프레임(frame), 지역 변수, 임시 결과, 메서드 수행과 리턴에관련된 정보 

      > Stack의 크기는 고정하거나 가변적일 수 있음
      >
      > 만약 연산을 하다가 jvm의 스택 크기의 최대치를 넘어섰을 경우에는 StackOverflowError 발생
      >
      > 가변적일 경우 stack의 크기를 늘이려고 할 때 메모리가 부족하거나, 스레드를 생성할 때 메모리가 부족한 경우에는 OutOfMemoryException 발생

  - Program Counter(PC) Register

    - 자바의 스레드들은 각자의 Program Counter(PC) register를 가짐
    - native code를 제외한 모든 java code들이 수행될 때 JVM의 instruction 주소를 pc register에 보관

  - Native Internal Threads (Native method stack)

    - java code가 아닌 다른 언어로 된(보통은 C) 코드들이 실행하게 될 떄의 스택 정보를 관리



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