## Exception

#### Throwable 클래스

Exception 클래스의 조상



####exception re-throwing (예외 되던지기) 

* catch블럭 내에서 예외를 throw하여 호출한 메서드에서 처리하도록 함

* exception re-throwing을 하는 경우에는 메서드의 선언부에 발생할 예외를 throws에 지정해주어야 한다.

* 사용하는 경우

  1.  한 메서드에서 발생할 수 있는 예외가 여럿인 경우, 일부는 try-catch문을 통해서 메서드 내에서 자체적으로 처리하고, 나머지는 선언부에 지정하여 호출한 메서드에서 처리하도록 함으로써 양쪽에 나눠서 처리될 수 있도록 함
  2. 하나의 예외에 대해서 예외가 발생한 메서드와 호출한 메서드, 양쪽에서 처리할 수 있도록 함

* Example

  ```java
  class ExceptionEx17 {
      public static void main(String[] args) {
          try {
              method1();
          } catch (Exception e) {
              System.out.println("occur exception in main method");
          }
      }
      static void method1() throws Exception {
          try {
              throw new Exception();
          } catch (Exception e) {
              System.out.println("occur exception in method1");
          }
      }
  }

  /*
  출력결과
  occur exception in method1
  occur exception in main method
  */
  ```

  ​

#### chained exception (연결된 예외)

예외A가 예외B를 발생시켰다면, A를 B의 '원인 예외(cause exception)'라고 한다.

* `Throwable initCause(Throwable cause)` : 지정한 cause예외를 원인 예외로 등록

* `Throwable getCause()` : 원인 예외를 반환

* 효과

  * 예외를 조상클래스예외로 해서 catch블럭을 작성하면, 실제로 발생한 예외가 어떤 것인지를 알 수 없다는 문제가 발생하며, 조상클래스의 자식클래스들만 처리가 가능하므로 상속관계가 중요해지는 문제가 있었다. 하지만 chained exception을 사용하여 예외가 예외를 포함하도록 하면, 발생한 예외가 어떤것인지 정확하게 알 수 있고 예외들의 상속관계가 불필요해진다.

* Example

  ```java
  try {
      startInstall();
      copyFIles();
  } catch (SpaceException e) {
      InstallException ie = new InstallException("설치 중 예외 발생"); //create exception
      ie.initCause(e); //InstallException의 원인 exception을 SpaceException으로 지정
      throw ie; //InstallException을 발생시킴
  } catch (MemoryException e) {
      ... //위와동일한 방식으로 chained exception
  }
  ```

  ​

