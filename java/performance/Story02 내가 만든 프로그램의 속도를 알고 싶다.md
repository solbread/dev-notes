## Story02 내가 만든 프로그램의 속도를 알고 싶다

### Intro

시스템의 성능이 느릴 때 가장 먼저 해야 하는 작업은 **병목지점**을 확인하는 것

애플리케이션 속도 문제 분석 프로그램 : 프로파일링 툴, APM 툴

하지만 대부분의 프로젝트나 운영 사이트에서는 예산상의 이유로 분석 툴을 사용하지 않음



### 프로파일링 툴이란?

#### 프로파일링 툴

* Profiler <<美>> (수사 기관 등의) 범죄 심리 분석관
* 시스템 문제 분석 툴

#### APM(Application Performance Monitoring or Management) 툴

* 운영용 서버를 진단 및 모니터링 하기 위해서 사용
* WebTune, Pharos, Introscpe, dynaTrace
  * WebTune은 개발 장비에서 사용하는 라이선스가 무료 ([WebTune](http://www.misoinfo.co.kr/solutions/webtune/webtune-overview/))

#### 프로파일링 툴 VS APM 툴

*정리해야함*



### System 클래스

모든 System 클래스의 메서드는 static으로 되어있고, 그 안에서 생성된 in, out, err와 같은 객체들도 static으로 선언되어 있으며, 생성자(Constructor)도 없음

#### Property VS Environment

* 공통점 :  JVN에서 사용할 수 있는 설정
* 속성(Property)
  * JVM에서 지정된 값
  * Properties
* 환경(Environment)
  * 장비(서버)에 지정되어 있는 값
  * env

#### 복사 메소드

- static void arraycopy(Object src, int srcPos, Object dest, int destPos, int length)
  - src배열을 dest배열로 복사(shallow copy)

#### Property 관련 메소드

- static Properties getProperties()
  - 현재 자바 속성값들을 받아옴
- static String getProperty(String key)
  - key에 지정된 자바 속성 값을 받아옴
- static String getProperty(String key, String def)
  - key에 지정된 자바 속성을 받아옴
  - 해당 key가 존재하지 않을 경우 def에 지정된 기본값으로 받아옴
- static void setProperties(Properties props)
  - props 객체에 담겨 있는 내용을 자바 속성에 지정
- static String setProperty(String key, String value)
  - 자바 속성에 있는 지정된 key의 값을 value로 변환
  - 이전 property값을 반환 (없었으면 null)

#### Environment 관련 메소드

* static Map\<String, String\> getenv()
  * 현재 시스템 환경 값 목록을 Map으로 반환
  * cmd 혹은 terminal에서 'set'명령어를 치는 결과와 동일
* static String getenv(String name)
  * name에 지정된 환경변수의 값을 얻음

#### 네이티브 라이브러리 관련 메소드

* static void load(String filename)
  * 파일명을 지정하여 네이티브 라이브러리를 로딩
* static void loadLibrary(String libname)
  * 라이브러리의 이름을 지정하여 네이티브 라이브러리를 로딩
* Java Native Interface(Jni) 참고링크
  * [Jni Java9](https://docs.oracle.com/javase/9/docs/specs/jni/index.html)
  * [Jni Java8](https://docs.oracle.com/javase/8/docs/technotes/guides/jni/)

#### 운영 중 절대 사용해서는 안되는 메소드

* static void gc()
  * 자바에서 사용하는 메모리를 명시적으로 해제하도록 GC를 수행하는 메소드
* static void exit(int status)
  * 현재 수행중인 자바 VM을 멈춤
* static void runFinalization()
  * garabge collector가 알아서 해당 객체를 더 이상 참조할 필요가 없을 때 Object 객체에 있는 finalize()라는 메서드는 자동으로 호출는데, 해당 메서드를 실행하면 참조 해제 작업을 기다리는 모든 객체의 finalize() 메서드를 수동으로 수행해야 함



### System.currentTimeMillis와 System.nanoTime

* static long currentTimeMillis()

  * 현재의 시간을 ms로 반환 (1/1000초)
  * UTC(Coordinated Universal Time)을 따름
    * 높은 정확도를 가지는 시간 표준 체계
    * 한국은 UTC+9 지역

  > milliseconds를 seconds로 읽는 가장 쉬운 방법은 콤마(,)를 찍어서 표현하였을 때 4번쨰 자리수를 기준으로 읽는 것
  >
  > 20ms -> 0.02s
  >
  > 1,200ms -> 1.2s
  >
  > 6,000,000ms -> 6000s



*정리해야함*