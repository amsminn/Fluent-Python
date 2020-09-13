# chapter 3 정리 : 딕셔너리와 집합 - 1
***
### Concept 
* 딕셔너리
    dict형은 모듈 네임스페이스, 클래스 밒 인스턴스 속성, 함수의 키워드 인수 등   
    사용되는 파이썬 구현의 핵심 부분이다  
    내장 함수들은 \_\_builtins\_\_.\_\_dict\_\_에 들어있다    
    
---    
    
### Contents
**1. 공통적으로 사용되는 딕셔너리 메서드**  

    - 일반적인 매핑형
    - 지능형 딕셔너리
      
      
**2. 없는 키에 대한 특별 처리**    

    - 존재하지 않는 키를 setfault()로 처리하기
    - defaultdict : 존재하지 않는 키에 대한 또 다른 처리
    - __missing__() 메서드  
      
      
**3. 표준 라이브러리에서 제공하는 다양한 딕셔너리 클래스**    

    collections.OrderedDict
    collections.ChainMap
    collections.Counter
    
---

### Usage    
  
**1. 일반적인 매핑형** 
  
collections.abc 모듈은 dict 및 이와 유사한 자료형의 인터페이스를 정의하기 위해   
Mapping 및 MutableMapping 추상 베이스 클래스(ABC)를 제공한다
  
특화된 매핑은 dict나 collections.UserDIct 클래스를 상속하기도 한다  
추상 베이스 클래스는 매핑이 제공해야 하는 최소한의 인터페이스를 정의하고 문서화하기 위한것이고  
넓은 의미의 매핑을 지원해야 하는 코드에서 isinstance() 테스트를 하기 위한 기준으로 사용된다


```python
import collections
my_dict = {}
print(isinstance(my_dict, collections.abc.Mapping))
```

    True
    

**"해시가 가능하다"는 무슨 의미일까?**  
수명 주기 동안 결코 변하지 않는 해시값을 갖고 있고(\_\_hash\_\_() 메서드가 필요)  
다른 객체와 비교할 수 있으면(\_\_eq\_\_() 메서드가 필요), 객체를 해시 가능하다고 한다  
  
따라서 원자적 불변형(str, byte, 수치형)은 모두 해시 가능하다
튜플은 들어있는 모든 항목들이 해시 가능하다면 해시 가능하다.


```python
tt = (1,2,(3,4)) # tt = (1,2,[30,40]) 리스트는 해쉬가 안됨
print(hash(tt))
tl = (1,2,frozenset([30,40]))
print(hash(tl))
```

    3794340727080330424
    5149391500123939311
    

**다양한 딕셔너리 구현 방법**


```python
a = dict(one = 1, two = 2, three = 3)
b = {'one' : 1, 'two' : 2, 'three' : 3}
c = dict(zip(['one', 'two', 'three'], [1,2,3]))
d = dict([('two', 2), ('one', 1), ('three', 3)])
e = dict({'three' : 3, 'one' : 1, 'two' : 2})
print(a==b==c==d==e)
```

    True
    

**지능형 딕셔너리**  
파이썬 2.7부터는 지능형 리스트와 제너레이터 표현식 구문이 가능한 딕셔너리에도 적용된다  
(지능형 집합도 마찬가지) 지능형 딕셔너리는 모든 반복형 객체에서 키-값 쌍을 생성함으로써   
딕셔너리 객체를 만들 수 있다


```python
DIAL_CODES = {
    ('China', 86),
    ('India', 91),
    ('United States', 1),
    ('Indonessia', 62),
    ('Brazil', 55)
}
country_code = {country : code for country, code in DIAL_CODES}
print(country_code)
```

    {'China': 86, 'United States': 1, 'Brazil': 55, 'India': 91, 'Indonessia': 62}
    

**2. 없는 키에 대한 특별 처리**    
  
**존재하지 않는 키를 setdefault()로 처리하기**  

**조기 실패**철학에 따라, 존재하지 않는 키 k로 d\[k\]를 접근하면 dict는 오류를 발생시킨다  
KeyError를 처리하는 것보다 기본값을 사용하는 것이 더 편리한 경우에는  
d\[k\] 대신 d.get(k, default)를 사용한다는 것은 파이썬 개발자라면 누구나 알고 있다  
그렇지만 발견한 값을 갱신할 때, 해당 객체가 가변 객체라면 \_\_getitem\_\_()이나 get() 메서드는  
보기 어색하며 효율성도 떨어진다  
  
**\_\_missing\_\_() 메서드**  

매핑형 이름으로도 쉽게 추측할 수 있는 \_\_missing\_\_() 메서드를 이용해서   
존재하지 않는 키를 처리할 수 있다 기본 클래스인 dict에는 정의되어 있지 않지만  
dict는 이 메서드를 알고 있다 따라서 dict 클래스를 사속하고 \_\_missing\_\_() 메서드를 정의하면   
dict.\_\_getitem\_\_() 표준 메서드가 키를 발견할 수 없을 때 KeyError를 발생시키지 않고 \_\_missing\_\_()  
메서드를 호출한다  
  
**3. 표준 라이브러리에서 제공하는 다양한 딕셔너리 클래스**       
  
      collections.OrderedDict
      키를 삽입한 순서대로 유지함으로써 항목을 반복하는 순서를 예측할 수 있다  
      
      collections.ChainMap
      매핑들의 목록을 담고 있으며 한꺼번에 모두 검색할 수 있다   
      각 매핑을 차례대로 검색하고 그중 하나에서라도 키가 검색되면 성공한다  
      인터프리터를 구현하는데 많이 사용한다
        
      collections.Counter
      모든 키에 정수형 카운터를 갖고 있는 매핑, 기존 키를 갱신하면 카운터가 늘어난다  
      다중 집합에서 객체의 수를 세기 위해 사용할 수 있다
