## Python Coding Convention

#### Code Layout

* 들여쓰기는 공백 4칸
* 한줄은 최대 79자
* top-level 함수와 클래스 정의는 2줄 띄기
* 클래스 내의 메소드 정의는 1줄 띄기

아래와 같이 top-level 함수 끼리는 2줄의 공백이, class 안의 함수는 1줄의 공백을 갖는 것이 권장사항이다.
```
def top_level_method():
    print("top level method")
 
    
def top_level_method():
    print("top level method")
    
    
class TopLevelClass(object):
    def __init__(self):
        pass
        
    def class_method(self):
        pass
```


#### Whitespace in Expressions and Statements

* 공백을 쓰지 않는 경우
  * 대괄호([]), 중괄호({}), 소괄호(())안
  * 컴마(,), 콜론(:), 세미콜론(;) 앞
  * 키워드 인자(keyword argument)와 인자의 기본값(default parameter value)의 = 



#### Comments

* 주석의 종류는 2가지로, 한 줄 작성 시 `#` 으로 시작하며 여러 줄 작성 시에는 quote 혹은 double quote 3개를 주석에 해당하는 부분에 감싸준다.
* 코드와 모순되는 주석은 있어서는 안되며 항상 코드에 따라 갱신
* 불필요한 주석은 제거
* 한 줄 주석은 신중히 작성
* [문서화 문자열(Docstring)](https://en.wikipedia.org/wiki/Docstring)에 대한 컨벤션은 [PEP 257](https://www.python.org/dev/peps/pep-0257/)을 참고



#### Naming Conventions

* snake 표현 방식 사용
  * \_single\_leading\_underscore: 내부적으로 사용되는 변수
  * single\_trailing\_underscore\_: 파이썬 기본 키워드와 충돌을 피하려고 사용
  * \_\_double\_leading\_underscore: 클래스 속성으로 사용되면 그 이름을 변경 (ex. FooBar에 정의된 \_\_boo는 \_FooBar\_\_boo로 바뀝니다.)
  * \_\_double\_leading\_and\_trailing\_underscore\_\_ : [마술(magic)](https://en.wikipedia.org/wiki/Magic_(programming))을 부리는 용도로 사용되거나 사용자가 조정할 수 있는 네임스페이스 안의 속성. 이런 이름을 새로 만들지 마시고 오직 문서대로만 사용해야 함
* 변수명으로 사용하지 않는 문자
  * 소문자 L, 대문자 O, 대문자 I (폰트에 따라 가독성이 매우 떨어짐)
* Module (모듈명)
  * 짧은 소문자로 구성되며 필요하다면 밑줄로 구분
  * 모듈은 파이썬 파일(.py)에 대응하기 때문에 파일 시스템의 영향을 받으니 주의
  * C/C++ 확장 모듈은 밑줄로 시작
* Class (클래스명)
  * 클래스 명은 카멜케이스(CamelCase)로 작성
  * 내부적으로 쓰이면 밑줄을 앞에 붙임
* Sub-Class (서브-클래스)
  * 이름충돌을 막기 위해서는 밑줄 2개를 앞에 붙임
* Exception (예외)
  * 실제로 에러인 경우엔 “Error”를 뒤에 붙임
* 함수명
  * 함수명은 소문자로 구성하되 필요하면 밑줄로 나눔
  * 대소문자 혼용은 이미 흔하게 사용되는 부분에 대해서만 하위호환을 위해 허용
* Method 
  * 함수명과 같으나 비공개(non-public) 메소드, 혹은 변수면 밑줄을 앞에 붙임
  * 인스턴스 메소드의 첫 번째 인자는 언제나 self
  * 클래스 메소드의 첫 번째 인자는 언제나 cls
* Constant (상수)
  * 모듈 단위에서만 정의하며 모두 대문자에 필요하다면 밑줄로 나눔



#### Programming Recommendations

* 코드는 될 수 있으면 어떤 구현(PyPy, Jython, IronPython등)에서도 불이익이 없게끔 작성되어야 함

* `None`을 비교할때는 `is`나 `is not`만 사용

* 클래스 기반의 예외를 사용

* 모듈이나 패키지에 자기 도메인에 특화된(domain-specific)한 기반 예외 클래스(base exception class)를 빌트인(built-in)된 예외를 서브클래싱해 정의하는 것을 권장. 이 때 클래스는 항상 문서화 문자열을 포함

  ```python
  class MessageError(Exception):
      """Base class for errors in the email package."""
  ```

- `raise ValueError('message')`가 (예전에 쓰이던) `raise ValueError, 'message'`보다 권장
- 예외를 `except`로 잡기보단 명확히 예외를 명시 (ex. `except ImportError`)
- `try` 블록의 코드는 필요한 것만 최소한으로 작성
- `string` 모듈보다는 `string` 메소드를 사용. 메소드는 모듈보다 더 빠르고, 유니코드 문자열에 대해 같은 API를 공유
- 접두사나 접미사를 검사할 때는 `startswith()`와 `endswith()`를 사용
- 객체의 타입을 비교할 때는 `isinstance()`를 사용
- 빈 시퀀스(문자열, 리스트(list), 튜플(tuple))는 조건문에서 거짓(false)
- 불린형(boolean)의 값을 조건문에서 `==`를 통해 비교금지 (ex `if stmt` 혹은 `if not stmt` 형식으로 작성)



#### Import

* 하나의 import에는 모듈 하나
* import는 표준 라이브러리, 서드파티, 로컬 라이브러리 순서로 묶기



#### Etc Rules

* 파일은 UTF-8 또는 ASCII로 인코딩


* assignment operator 주위에 하나이상의 space는 금지

  * 변수가 추가되는 경우에 공백을 유지하기 위한 불필요한 변경이 생겨 source merge시 혼란 발생

  ```python
  # Yes
  x = 1
  y = 2
  long_variable = 3

  # No
  x             = 1
  y             = 2
  long_variable = 3
  ```

* Imports는 각 line으로 분리

  * 대부분의 변경 추적 도구가 행 기반임을 고려하여 코드의 변경을 쉽게 확인하기 위함

  ```python
  # Yes
      import os
      import sys
  # No
      import sys, os
  ```



#### PEP8 체크하기

방법1) PyCharm 이용

File - Settings - Editor - Inspections - PEP8 검색하여 체크



방법2)

파이썬 외부패키지인 [pep8](https://pypi.python.org/pypi/pep8) 이용



#### 참고 자료

* PEP
  * [PEP(Python Enhance Proposal) 8 - Style Guide for Python Code](https://www.python.org/dev/peps/pep-0008/)
  * [GhostOnNetwork Blog - PEP8 Korean Translation](http://kenial.tistory.com/902)
  * [PEPs Korean](https://bitbucket.org/sk8erchoi/peps-korean)
* Google
  * [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html)
* Melange
  * [Melange Python Style Guide](https://code.google.com/archive/p/soc/wikis/PythonStyleGuide.wiki)
* Etc
  * [Spoqa Blog - Python Coding Convention](https://spoqa.github.io/2012/08/03/about-python-coding-convention.html)
  * [KeiChoi Blog - Python Coding Stype Guide](http://www.hanul93.com/kicomav-pep8/)



