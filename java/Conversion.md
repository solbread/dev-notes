## Conversion

#### 산술 변환(Usual Arithmetic Conversion)

피연산자 타입의 일치를 위해 자동 형변환

1. 두 피연산자의 타입을 같게 일치시킨다. (보다 큰 타입으로 일치, 피 연산자의 값 손실을 최소화 하기 위함)
   * log + int -> long + long -> long
   * float + int -> float + float -> float
   * double + float -> double + double -> double

2. 피연산자의 타입이 int보다 작은 타입이면 int로 변환된다. (연산 중 overflow가 발생할 가능성이 높기 때문)
   * byte + short -> int + int -> int
   * char + short -> int + int -> int

   ```java
   byte a = 10;
   byte b = 30;
   byte c = (byte)(a * b); // 결과는 44
   ```

   a * b는 a와 b가 모두 int형으로 변환되어 연산되므로 1 0010 1100 인데 이를 byte형으로 형변환하면 8bit까지만 유효므로 0010 1100이 되어 결과는 44가 된다.

3. 삼항연산자에서 두개의 피연산자의 타입이 다른 경우 발생

   ```java
   x = x + (mod < 0.5 ? 0 : 0.5); // 0과 0.5의 타입이 다르므로,아래와 같은 conversion 발생
   x = x + (mod < 0.5 ? 0.0 : 0.5); //0이 0.0으로 변환되며, 삼항연산의 결과도 double타입
   ```

   ​




#### char 타입의 형변환

char타입의 문자는 실제로 해당 문자의 유니코드(부호없는 정수)로 바뀌어서 저장된다.

```java
char c1 = 'a'
c1++ // 'b'
c1 + 1 // 97
'a' + 1 // 'b'
```

1. `c1++` 형변환 없이 c1에 저장된 값을 1증가시킴
2. `c1+1` c1이 int형으로 변환된 후 덧셈이 실행
3. `'a'+1` 상수또는 리터럴 간의 연산은 실행과정동안 변하는 값이 아니기 때문에, 컴파일 시에 컴파일러가 계산해서 그 결과로 대체함으로써 코드를 보다 효율적으로 만듦






