## Keyword : this, super

#### this VS this()

* this
  * 인스턴스 자신을 가리키는 참조변수
  * 인스터스의 주소가 저장되어 있음
  * 클래스메서드에서는 인스턴스와 관련 없는 작업을 하기 때문에 인스턴스에 대한 정보가 불필요. 또한 클래스메서드는 호출된 시점에 인스턴스가 존재하지 않을 수 있음 -> 인스턴스 자신을 가리키는 참조변수인 'this'를 사용할 수 없음
  * 인스턴스메서드는 특정 인스턴스와 관련된 작업을 하기 때문에 자신과 관련된 인스턴스의 정보가 필요 -> 모든 인스턴스메서드에서는 참조변수 this가 지역변수로 숨겨진 채로 존재
* this() or this(매개변수)
  * 생성자
  * 같은 클래스의 다른 생성자를 호출할 때 사용



#### super

* 자손 클래스에서 조상 클래스로부터 상속받은 멤버를 참조하는데 사용되는 참조 변수
* super VS this
  * 멤버변수와 지역변수의 이름이 같을 때 this를 사용하여 구분했듯이, 상속받은 멤버와 자신의 클래스에 정의된 멤버의 이름이 같을 때 super를 사용하여 구분
  * 조상클래스로부터 상속받은 멤버도 자손클래스 자신의 멤버이므로 super대신 this를 사용할 수 있음. 하지만 조상클래스의 멤버와 자손클래스의 멤버가 중복 정의되어 서로 구분해야 하는 경우에만 super를 사용하는 것이 좋음 
  * 모든 인스턴스 메서드에는 자신이 속한 ㅇ니스턴스의 주소가 지역변수로 저장되는데, 이것이 참조변수인 this와 super가 됨

