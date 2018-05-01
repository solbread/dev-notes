## Multi Thread : Atomic Class

#### Atomic Class

* CAS(Compare - And - Swap) 기반으로 작성됨

  > CAS? 특정 메모리 위치의 값이 주어진 값을 비교하여 같으면 새로운 값으로 대체
  >
  > [wiki - compare and swap](https://en.wikipedia.org/wiki/Compare-and-swap)
  >
  > ```C
  > int compare_and_swap(int* reg, int oldval, int newval)
  >
  > {
  >   int old_reg_val = *reg;
  >   if (old_reg_val == oldval)
  >      *reg = newval;
  >   return old_reg_val;
  > }
  > ```

* Java의 Atomic Class

  ```java
  int current;
  do {
  	current = get();
  } while(!compareAndSet(current, current + 1));
  ```

  > 참고 compareAndSet (native method)
  >
  > ```java
  > public final boolean compareAndSet(V expect, V update)
  > ```
  >
  > Atomically sets the value to the given updated value if the current value `==` the expected value.
  >
  > - Parameters:
  >
  >   `expect` - the expected value
  >
  >   `update` - the new value
  >
  > - Returns:
  >
  >   true if successful. False return indicates that the actual value was not equal to the expected value.

  compareAndSet 메소드에 의해 current값이 현재 current값이면 +1을 하고, 아니면 loop를 다시 반복하여 새로운 current값을 받아와서 연산을 시도한다.