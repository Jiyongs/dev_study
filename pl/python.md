### 파이썬 버전알기
```python
python --version
```
파이썬 버전별로 지원하는 문법과 기능이 다르므로 버전을 확실히 알고 시작하자

### PEP 8 스타일 가이드를 따르자
PEP8는 파이썬 개선 제안서(Python Enhancement Proposal) #8를 의미하며, 파이썬 코드의 스타일 가이드이다.
일관성 있는 스타일을 사용하여 유지보수의 용이성과 가독성을 높이자. 또한 협력을 위해서도 중요하다.  
① whitespace
- 공백은 space를 사용
- 문법적으로 의미있는 들여쓰기는 space 4개를 사용
- 한 줄의 문자 길이가 79자 이하가 되도록
- 길어서 다음 줄로 넘어가면 추가로 space 4개를 사용
- 함수와 class는 개행 2개로 구분
- class에서 method는 개행 1개로 구분
- list index, function 호출, keword 인수 할당에는 space를 사용하지 않기
- 변수 할당 앞뒤에 space 1개만 사용

② naming
- 함수, 변수, 속성은 lowercase_underscore 형식
- protected instance 속성은 _leading_undercase 형식
- private instance 속성은 __double_leading_unsercase 형식
- class와 exception은 CapitalizedWord 형식
- module 수준 상수는 ALL_CAPS 형식
- class의 instance method는 첫 번째 parameter(해당 객체를 참조)의 이름을 self로 지정
```python
class ClassExample:
    def method_example(self):
        print('A')
```
- class method는 첫 번째 parameter(해당 calss를 참조)의 이름을 class로 지정
```python
class ClassExample:
    @classmethod
    def method_example(cls):
        print('A')
```
※class method는 class를 인스턴스화 하지 않아도 호출 가능
※self, cls는 모두 class의 속성에 접근하기 위한 방법
③ 표현식과 문장
- 긍정 표현식의 부정(if not A is B) 대신, 인라인 부정(if A is not B) 을 사용
- 길이를 통해 빈 값을 확인하지 않는다. if not somlist를 사용하고, 빈 값은 암시적으로 False가 된다고 가정한다.
- 한 줄로 된 if, for, while, except 문을 쓰지 않는다.
- 항상 파일의 맨 위에 import를 선언한다.
- module을 import할 때는 module의 절대이름을 사용하며, 상대 경로를 사용하지 않는다.
- 상대 경로를 사용해야 한다면 명시적 구문을 쓰자
```python
# x
import foo
# o
from bar import foo
# 상대경로 사용
from . import foo
```
- import는 표준 라이브러리 모듈, 서드파티 모듈, 자신이 만든 모듈 순서로 구분하자. 각각 알파벳 순서로 정렬한다.
