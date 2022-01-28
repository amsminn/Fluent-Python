# chapter 5 정리 : 일급 함수
***
### Concept 
* 일급 객체
    프로그래밍 언어 이론가들은 다음과 같은 작업을 수행할 수 있는 프로그램 개체를
    '일급 객체'로 정의한다
        * 런타임에 생성할 수 있다
        * 데이터 구조체의 변수나 요소에 할당 할 수 있다
        * 함수 인수로 전달할 수 있다
        * 함수 결과로 반환할 수 있다
            ex) 정수, 문자열, 딕셔너리 타입  
            
이 장에서는 일급 객체로서의 함수를 처리하는 실용적인 방법과 영향에 대해 집중적으로 살펴본다  

---    
    
### Contents  

**1. 고위 함수**    

    -고위 함수의 정의와 예시      
    
**2. 익명 함수**  

    -lamda

**3. 일곱 가지 콜러블 객체**  
  
    -callable() 의 용도와 분류
      
**4. 함수형 프로그래밍을 위한 패키지**  
 
    -operator 모듈
    -functools.partial()로 인수 고정하기
      
**5. 요약**    
    
    
---

### Usage    
  
**1. 고위 함수** 
      
    함수를 인수로 받거나 함수를 결과로 반환하는 함수를 **고위 함수**라고 한다
    대표적으로 map() 함수가 있다
    list.sort()와 sorted() 또한 일급 함수이다
        -sorted()는 선택적인 key 인수를 받는다


```python
fruits = ['strawberry', 'fig', 'apple', 'cherry', 'raspberry', 'banana']

print(sorted(fruits))
print(sorted(fruits, key = len))
```

    ['apple', 'banana', 'cherry', 'fig', 'raspberry', 'strawberry']
    ['fig', 'apple', 'cherry', 'banana', 'raspberry', 'strawberry']
    

**map(), filter(), reduce()의 대안**
  
이름이 다른 경우도 있지만 함수형 언어는 모두 map(), filter(), reduce() 고위 함수를 제공한다
map()과 filter() 함수는 여전히 파이썬3에 내장되어 있지만
지능형 리스트와 제너레이터 표현식이 소개된 후로는 이 함수들의 중요성이 떨어졌다
지능형 리스트와 제너레이터 표션식이 map(), filter()이 처리하는 작업을 대체 할 수 있고
가독성도 더 좋기 때문이다  
  
아래 코드를 보면 가독성에서 큰 차이를 볼 수 있다 


```python
print(list(map(int, range(6)))) # map()
print([n for n in range(6)]) # 지능형 리스트

a = list(map(int, filter(lambda n : n % 2, range(6)))) 
print(a)
a = [n for n in range(6) if n%2]
print(a)

from functools import reduce
from operator import add
print(reduce(add, range(100)))
print(sum(range(100)))
```

    [0, 1, 2, 3, 4, 5]
    [0, 1, 2, 3, 4, 5]
    [1, 3, 5]
    [1, 3, 5]
    4950
    4950
    

---

**2. 익명 함수**
  
lambda 키워드는 파이썬 표현식 내에 익명 함수를 생성한다
그렇지만 파이썬의 단순한 구문이 람다 팜수의 본체가 순수한 표현식으로만 구성되도록 제한한다
즉 람다 본체에서는 할당문이나 while, try 등의 파이썬 문장을 사용할 수 없다
  
익명 함수는 인수 목록 안에서 아주 유용하게 쓰인다 
예를 들어 reverse() 함수 대신 람다를 사용하도록 할 수 있다
  
  아래는 그 코드이다


```python
fruits = ['strawberry', 'fig', 'apple', 'cherry', 'raspberry', 'banana']
print(sorted(fruits, key = lambda word : word[::-1]))
```

    ['banana', 'apple', 'fig', 'raspberry', 'strawberry', 'cherry']
    

---

**3. 일곱가지 콜러블 객체**  
  
  호출 연산자인 ()는 사용자 정의 함수 이외의 다른 객체에도 적용할 수 있다
  호출 할 수 있는 객체인지 알아보려면 callable() 내장 함수를 사용한다
  파이썬 데이터 모델 문서는 다음 일곱 가지 콜러블을 나열하고 있다
    
    사용자 정의 함수 : def문이나 람다를 통해 생성
      
    내장 함수 : len()이나 time.strftime()처럼 C언어로 구현된 함수(Cpython의 경우)  
      
    내장 메서드 : dict.get()처럼 C언어로 구현된 메서드
      
    메서드 : 클래스 본체에 정의된 함수  
      
    클래스 : __new__() 메서드로 생성, __init__() 메서드로 초기화  
     
    클래스 객체 : 클래스가 __call__() 메서드를 구현하면
                   이 클래스의 객체는 함수로 호출될 수 있다   
                     
    제너레이터 함수 : yield 키워드를 사용하는 함수나 메서드
                       이 함수가 호출되면 제너레이터 객체를 반환한다


```python
a = [callable(obj) for obj in (abs, str, 12)]
print(a)
```

    [True, True, False]
    

---

**4. 함수형 프로그래밍을 위한 패키지**
  
**operator 모듈**  
  
함수형 프로그래밍을 할 때 산술 연잔자를 함수로 사용하는 것이 편리할 때가 종종 있다
예를 들어 팩토리얼을 계산하기 위해 재귀적으로 함수를 호출하는 대신 숫자 시퀀스를
곱하는 경우를 생각해보자
합계를 구할 때는 sum()이라는 함수가 있지만 곱셈에 대해서는 이에 해당하는 함수가 없다
reduce() 함수를 사용할 수 있지만, reduce()는 시퀀스의 두 항목을 곱하는 함수를 필요로 한다


```python
from functools import reduce

def fact(n): 
    return reduce(lambda a,b : a*b, range(1,n+1))
print(fact(3))
```

    6
    


```python
from functools import reduce
from operator import mul

def fact(n):
    return reduce(mul, range(1,n+1))
print(fact(3))
```

    6
    

operator 모듈은 시퀀스에서 항목을 가져오는 람다를 대체하는 itemgetter() 함수와 
객체의 속성을 읽는 람다를 대체하는 attrgetter() 함수를 제공한다  
  
아래 코드는 특정 필드의 값을 기준으로 튜플의 리스트를 정렬할 때 일반적으로 사용하는
itemgetter()를 보여준다
예제에서는 1번 필드인 국가 코드로 정렬된 도시들을 출력한다
본질적으로 itemgetter(1)은 lambda fields : fields[1]과 동일하다


```python
!pip install operator

from operator import itemgetter
metro_data = [
    ('Tokyo', 'JP', 36.933),
    ('Delhi NCR', 'IN', 21.935),
    ('Mexico', 'MX', 20,142),
    ('New York-Newark', 'US', 20,104)
]

for city in sorted(metro_data, key = itemgetter(1)):
    print(city)
```

    ('Delhi NCR', 'IN', 21.935)
    ('Tokyo', 'JP', 36.933)
    ('Mexico', 'MX', 20, 142)
    ('New York-Newark', 'US', 20, 104)

    

**functools.partial()로 인수 고정하기**
  
    
functools 모듈은 몇가지 고위 함수를 통합한다
그중 가장 널리 알려진 함수가 reduce() 함수이다
functools에서 제공하는 나머지 함수중 
partial() 및 이외의 변형인 partialmethod() 함수가 매우 유용하다  
  
functools.partial()은 함수를 부분적으로 실행 할 수 있게 해주는 고위 함수이다
어떤 함수가 있을때 partial()을 적용하면 원래 함수의 일부 인수를 고정한 콜러블을 생성한다
이 기법은 하나 이상의 인수를 받는 함수를 그보다 적은 인수를 받는 콜백 함수를 사용하는
API에 사용하고자 할 때 유리하다


```python
from operator import mul
from functools import partial
triple = partial(mul, 3)
print(triple(7))
print(list(map(triple, range(1,10))))
```

    21
    [3, 6, 9, 12, 15, 18, 21, 24, 27]
    

---

**5. 요약**  
  
지능혈 리스트 및 제너레이터 표현식과 내장된 리덕션 함수의 등장으로 
map(), filter(), reduce()함수의 사용빈도가 떨어지기는 했지만
함수형 프로그래밍의 기본적인 특징인 고위 함수를 파이썬에서 쉽게 볼 수 있다
sorted(), min(), max() 내장 함수와 functools.partial()이 자주 사용되는 고위 함수의 예다
