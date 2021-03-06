## Variance Initialization

#### Variance Initialization

* 멤버변수(클래스변수, 인스턴스변수)와 배열은 초기화를 하지 않아도 자동적으로 변수의 자료형에 맞는 기본값으로 초기화가 이뤄짐
* 지역변수는 자동으로 초기화되지 않으므로 지역변수의 초기화는 필수





#### 변수의 자료형별 기본값

| 자료형           | 기본값        |
| :--------------- | :------------ |
| boolean          | false         |
| char             | '\u0000'      |
| byte, short, int | 0             |
| long             | 0L            |
| float            | 0.0f          |
| double           | 0.0d 또는 0.0 |
| 참조형 변수      | null          |



#### 멤버 변수 초기화 방법

1. 명시적 초기화(explicit initialization)

   * 변수를 선언과 동시에 초기화

   ```java
   class Test {
       int a = 3;
       Test2 test2 = new Test2();
       
       //...
   }
   ```

2. 생성자(constructor)

   ```java
   class Test {
       int a;
       Test test2;
       Test() {
           this.a = 3;
           this.test2 = new Test2();
       }
   }
   ```

3. 초기화 블럭(initialization block)

   * 초기화블럭은 {}를 이용하며, 초기화 블럭 내에서는 메서드와 같이 조건문, 반복문, 예외처리구문 등을 자유롭게 사용할 수 있으므로, 초기화 작업이 복잡하여 명시적 초기화만으로는 부족한 경우 초기화 블럭을 사용


   * 인스턴스 초기화 블럭
     * 인적스턴스변수를 초기화하는데 사용
     * 클래스가 메모리에 처음 로딩될 때 한번만 수행 (클래스가 처음 로딩될 때 클래스변수들이 자동적으로 메모리에 만들어지고, 곧바로 클래스 초기화블럭이 클래스변수들을 초기화)
   * 클래스 초기화 블럭
     * 클래스변수를 초기화하는데 사용
     * 인스턴스를 생성할 때 마다 수행

   ```java
   class Test {
       static {
           //initialize class variances
       }
       {
           //initialize instance variance
       }
   }	
   ```

> 인스턴스 변수의 초기화는 주로 생성자를 사용하고 인스턴스 초기화 블럭은 모든 생성자에서 공통으로 수행돼야 하는 코드를 넣는데 사용하여, 코드의 중복을 제거 -> 코드의 신뢰성을 높여주고 오류의 바랭가능성을 줄여줌



#### 멤버변수의 초기화 시기와 순서 : 클래스 변수

* 초기화 시점 : 클래스가 처음 Method영역에 로딩될 때 단 한번 초기화

  > 클래스는 언제 로딩될까?
  >
  > * 클래스의 로딩 시기는 JVM의 종류에 따라 다를 수 잇는데, 클래스가 필요할 때 바로 메모리에 로딩하도록 설계가 되어있는 것도 잇고, 실행효율을 높이기 위해서 사용될 클래스들을 프로그램이 시작될 때 미리 로딩하도록 되어잇는 것도 있음
  > * 해당 클래스가 이미 메모리에 로딩되어 잇다면, 또다시 로딩하지 않으며 초기화역시 다시 수행되지 않음

* 초기화 순서 : 기본값 -> 명시적초기화 -> 클래스 초기화 블럭





#### 멤버변수의 초기화 시기와 순서 : 인스턴스 변수

* 초기화 시점 : 인스턴스가 생성될 때마다 각 인스턴스별로 초기화
* 초기화 순서 : 기본값 -> 명시적초기화 -> 인스턴스 초기화 블럭 -> 생성자