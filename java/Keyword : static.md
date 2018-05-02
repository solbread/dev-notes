## Keyword : static

#### class method(static method) VS instance method

1. 클래스를 설계할 때, 멤버변수 중 모든 인스턴스에 공통값을 가지는 변수는 클래스변수로 만든다.
   * 클래스 변수(static 변수)는 인스턴스를 생성하지 않아도 사용할 수 있다.
2. 메서드 내에서 인스턴스 변수를 사용하지 않는다면 클래스 메서드로 만드는 것을 고려한다.
   * 클래스 메서드(static메서드)는 인스턴스 변수를 사용할 수 없다. 




#### class method

* class method에서는 instance variance에 접근 불가
* method내에서 instance method를 호출 불가




#### class method의 성능

인스턴스메서드는 실행 시 호출되어야할 메서드를 찾는 과정이 추가적으로 필요한 반면 클래스메서드는 필요없으므로, 메서드 호출시간이 짧아져서 성능이 향상된다.