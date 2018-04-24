## Story18 GC가 어떻게 수행되고 있는지 보고싶다



### 자바 인스턴스 확인을 위한 jps

* 해당 머신에서 운영 중인 JVM의 목록을 보여줌

* JDK의 bin 디렉터리에 위치

* 사용

  * 커맨드 프롬프트나 유닉스의 터미널에서 아래 명령어 입력

    ```cli
    jps [-q] [-mlvV] [-Joption] [<hostid>]
    ```

    * 옵션없이 입력하면 현재 서버에서 수행되고 있는 자바 인스턴스의 pid(process id)와 porcess name이 출력 됨

    * -q : 클래스나 JAR 파일명, 인수 등을 생략하고 단지 프로세스 id만 나타냄

    * -m : main 메서드에 지정한 인수들을 나타냄

    * -l : 애플리케이션의 main 클래스나 애플리케이션 JAR 파일의 전체 경로 이름을 나타냄

    * -v : JVM에 전달된 자바 옵션 목록을 나타냄

      * 톰캣 프로그램의 경우 process name이 'Bootstrap'이므로 구분하기가 쉽지 않으므로 해당 옵션을 사용하여 자바 옵션을 출력하여 어떤 시스템인지를 판단 가능

    * -v : JVM의 플래그 파일을 통해 전달된 인수를 나타냄

      > 플래그 파일
      >
      > .hotspotrc의 확장자를 가지거나 자바 옵션에 -XX:Flags=\<file name\>로 명시한 파일
      >
      > 이 파일을 이용하여 JVM의 옵션을 지정할 수 있음

    * -Joption : 자바 옵션을 이 옵션 뒤에 지정 가능

      ​

### GC 상황을 확인하는 jstat

* GC가 수행되는 정보를 확인하기 위한 명령어

* 라인 단위로 결과를 보여줌

* 사용

  ```cli
  jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
  ```

  * -t : 자바 인스턴스가 생성된 시점(=서버가 기동된 시점)부터의 수행 시간 

  * -h:lines : 각 열의 설명을 지정된 라인 주기로 표시

  * interval : 로그를 남기는 시간의 차이(millisecond 단위)

  * count : 로그를 남기는 횟수

  * \<option\>의 종류

    *각 옵션에 따라 나타나는 내용은 책의 부록에 수록*

    * class : 클래스 로더에 대한 통계
    * compiler : 핫스팟 JIT 컴파일러에 대한 통계
    * gc : GC 힙 영역에 대한 통계
    * gccapacity : 각 영역의 허용치와 연관된 영역에 대한 통계
    * gcnew : 각 영역에 대한 통계
    * gcnewcapacity : Young 영역과 관련된 영역에 대한 통계
    * gcold : Old와 Perm 영역에 대한 통계
    * gcoldcapacity : Old 영역 크기에 대한 통계
    * gcpermcapacity : Perm 영역의 크기에 대한 통계
    * gcutil : GC에 대한 요약 정보
    * printcompilation : 핫 스팟 컴파일 메서드에 대한 통계

  * Example

    ```cli
    $ jstat -gcnew -t -h10 2624 1000 20 > jstat_WAS1.log
    ```

    * -gcnew : 각 영역에 대한 통계를 보여줌
    * -t : 수행 시간을 나타냄
    * -h10 : 10줄에 한 번씩 각 열의 설명(타이틀)을 나타냄
    * 2624 : pid
    * 1000 : 1초(1000ms)에 한 번씩 정보를 보여줌
    * 20 : 20회 반복 수행
    * Jstat_WAS1.log 파일에 결과를 저장

  * 활용

    * jstat에서 출력되는 결과를 사용하여 그래프를 그리면 GC가 처리되는 추이를 알아볼 수 있으므로 편리함
    * 결과를 파일로도 남길 수 있어 나중에 분석 할 때 사용할 수 있음
    * jstat결과만으로는 어떻게 해석을 하면 좋을지 알기 어렵지만 JVM 파라미터 튜닝을 할 떄나, GC를 수행하는 데 소요된 모든 시간을 보고 싶을 때 유용하게 사용 가능



  * 한계

    * 로그를 남기는 주기 동안 GC가 한번 발생 할 수도 있고, 여러번 발생 할 수도 있으므로 정확하게 분석하기가 어려움 -> verbosegc 옵션 사용을 권장 



### jstat 명령에서 GC 튜닝을 위해서 가장 유용한 옵션 두 개

-gcutil, -gccapacity

#### -gcutil

* GC에 대한 요약 정보(힙 영역의 사용량을 %로 나타냄)

* 출력 내용의 의미

  * S0 : Survivor0 영역의 사용량 백분율
  * S1 : Survivor1 영역의 사용량 백분율
  * E : Eden 영역의 사용량 백분율
  * O : Old 영역의 사용량 백분율
  * P : Perm 영역의 사용량 백분율
  * YGC : Young 영역의 GC 횟수
  * YGCT : Young 영역의 GC(minorGC)가 수행된 누적 시간(초)
  * FGC : Old 영역과 Perm 영역의 GC(majorGC) 횟수 합
  * FGCT : Old 영역과 Perm 영역의 GC(majorGC)가 수행된 누적 시간(초)
  * GCT : YGCT+FGCT

* 활용

  * YGCT / YGC : minor GC가 한 번 수행될 때의 시간의 평균
  * FGCT / FGC : major GC가 한 번 수행될 때의 시간의 평균

* 주의사항

  * CMS GC를 사용할 경우에는 Full GC의 단계에 따라서 수행되는 시간이 다르기 때문에 평균값이 낮다고 그냥 무시해서는 안 됨

  ​

#### -gccapacity

* 각 영역의 허용치와 연관된 영역에 대한 통계 (각 영역에 할당되어 있는 메모리를 kb단위로 나타냄)
* 출력 내용 의미
  * NGC로 시작하면 New(Young) 영역 크기 관련
  * OGC로 시작하면 Old 영역 크기 관련
  * PGC 로 시작하면 Perm 영역 크기 관련
  * MN으로 끝나면 최소값
  * MX로 끝나면 최대값 
  * C로 끝나면 Commited값
  * YGC : Minor GC 횟수
  * FGC : Full GC 횟수
* 활용
  * 어떤 영역의 크기를 늘리고 줄여야 할지를 알 수 있음
  * 자바 프로세스 메모리 점유 상황을 쉽게 활용 가능



### 원격으로 JVM 상황을 모니터링 하기 위한 jstatd





### verbosegc 옵션을 이용하여 gc 로그 남기기

* Java 수행 옵션에 추가하고 JVM 재시작



### 어설프게 아는 것이 제일 무섭다

