# chapter 2 정리 : 시퀀스
***
### Concept 
* 내장 시퀀스
    컨테이너 시퀀스와 균일 시퀀스 또는 가변 시퀀스와 불변 시퀀스로 구분한다  
    
    1. - 컨테이너 시퀀스
          서로 다른 자료형 저장 가능 : list, tuple, collections.deque
       - 균일 시퀀스
          단 하나의 자료형만 담을 수 있음 : str, bytes, bytearray, memoryview, arrary.array
    2. - 가변 시퀀스
          list, bytearray, array.array, collections.deque, memoryview    
          
          
* 지능형 리스트와 제너레이터
    지능형 리스트를 사용한다면 코드의 가독성에 도움이 된다    
    
    
* 튜플 
    튜플은 불변리스트로 사용할 수도 있지만 필드명이 없는 레코드로도 사용 가능    
    
    
* 슬라이싱
    사용자 정의 클래스에 슬라이싱을 해보자!    
    
    
* sort()와 sorted()의 차이
    sort()는 리스트 내부를 변경하고 sorted()는 새로운 리스트를 만들어서 반환한다      
    
    
* Bisect search
    정렬된 상태의 배열에 대해서 새 원소가 삽입될 위치를 반환  
    
    
* Memory View
    메모리 뷰 내장 클래스는 공유 메모리 시퀀스형으로서 bytes를 복사하지 않고
    배열의 슬라이스를 관리할 수 있도록 해준다
***

### Usage  

**지능형 리스트**


```python
symbols = "*@#&"
codes =[]
for symbol in symbols:
    codes.append(ord(symbol))
print(codes)
```

    [42, 64, 35, 38]
    

위 코드는 문자열 symbols의 아스키 코드를 추출해 codes라는 리스트에 담은 코드다
리스트와 반복문에 대한 기본적인 지식만 있다면 누구나 이해할 수 있다


```python
symbols = "*@#&"
codes = [ord(symbol) for symbol in symbols]
print(codes)
```

    [42, 64, 35, 38]
    

지능형 리스트를 사용한다면 위와 같이 가독성이 좋은 코드를 짤 수 있다
물론 지능형 리스트 구문을 남발한다면 역효과가 날 수 있으니 주의한다  
-지능형 리스트 구문은 map()과 filter()의 조합보다 빠르다고 한다  

---

**제너레이터 표현식**  
튜플, 배열 등의 시퀀스형을 초기화하려면 지능형 리스트를 이용할 수도 있지만
제너레이터 표현식은 다른 생성자에 전달할 리스트를 통째로 만들지 않고 
반복자 프로토콜(iterator protocol)을 이용해서 항목을 하나씩 생성하기 때문에 
메모리를 절약할 수 있다


```python
#제너레티러 표현식을 이용한 튜플 초기화
symbols = "*@#&"
tup = tuple(ord(symbol) for symbol in symbols)
print(tup)

# 제너레이터 표현식을 이용한 배열 초기화
import array
symbol = "*@#&"
arr = array.array('I', (ord(symbol) for symbol in symbols))
print(arr)
```

    (42, 64, 35, 38)
    array('I', [42, 64, 35, 38])
    

---  
**튜플**  
튜플은 단순한 불변 리스트가 아니다   
tip)   
가변 항목을 튜플에 넣는 것은 좋은 생각이 아니다  
복합 할당은 원자적인 연산이 아니다
         
* 레코드로서의 튜플
      튜플은 레코드를 담고 있다 
      튜플의 각 항목은 레코드의 필드 하나를 의미하며 항목의 위치가 의미를 결정한다  
      레코드로서 살짝 부족한 감이 있는 튜플을 보완하기 위해 namedtuple()이 고안되었다  
      
* 불변 리스트로서의 튜플
      튜플을 불변리스트로 사용할 때 \_\_reversed\_\_()를 제외하고
      리스트가 제공하는 모든 메서드를 지원한다  
      
    
아래 코드는 네임튜플 사용의 예다


```python
from collections import namedtuple 
city = namedtuple('city', 'name country population')
tokyo = city('Tokyo', 'JP', 36.933)
print(tokyo)
```

    city(name='Tokyo', country='JP', population=36.933)
    

namedtuple의 사용법을 정리하면
    1. 클래스명과 필드명의 리스트 등 총 2개의 매개변수가 필요한데
       필드명의 리스트는 반복형 문자열이나 공백으로 구분된 하나의 묹열로 지정한다
    2. 데이터는 위치를 맞추고 콤마로 구분해서 생성자에 전달해야 한다
    3. 필드명이나 위치를 이용해서 필드에 접근한다  
    
---

**슬라이싱**  
s[a : b :c]의 형태로 표현  
a에서 시작하여 b까지 c의 폭으로 참조한다

**s[a : b : c]에서 c를 생략하는 경우**  
s[a : b]의 형태로 표현하면 c는 디폴트값인 1로 정해진다

(s[a : b : c]을 평가하기 위해 파이썬 인터프리터는   
s.\_\_getitem\_\_(slice(a,b,c))를 호출한다)
  
아래는 슬라이싱의 예다


```python
#생략없는 슬라이싱
s = 'bicycle'
print(s[::3])

#폭을 생략한 슬라이싱
print(s[:3]);
print(s[3:])
```

    bye
    bic
    ycle
    


```python
l = list(range(10))
print(l)
l[2:5] = [20,30] # l[2:3]나 l[2:5]를 해도 똑같은 결과가 나온다 
                 # 우항의 리스트에 20,30만이 있기 때문
print(l)
del l[2:5]
print(l)
```

    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
    [0, 1, 20, 30, 5, 6, 7, 8, 9]
    [0, 1, 6, 7, 8, 9]
    

---
**Bisect Search**



```python
import bisect

def grade(score, breakpoints=[60, 70, 80, 90], grades='FDCBA'):
    i = bisect.bisect(breakpoints, score)
    return grades[i]

if __name__ == "__main__":
    print([grade(score) for score in [33, 99, 77, 89, 90, 100]])
```

    ['F', 'A', 'C', 'B', 'A', 'A']
    

---
**Memory View**
array 비슷한 표기법을 사용하는 memoryview.cast() 메서드는 바이트를 이동시키지  
않고 C언어의 형변환 연산자처럼 여러 바이트로 된 데이터를 읽거나 쓰는 방식을 
바꿀 수 있개 해준다
(복사 없는 메모리 공유가 특징)


```python
import array
numbers = array.array('h', [-2, -1, 1, 2])
memv = memoryview(numbers)

print(memv) 
print(len(memv))
print(memv[0])

memv_oct = memv.cast('B')
print(memv_oct.tolist())
print(numbers)
```

    <memory at 0x000001FB18DE3940>
    4
    -2
    [254, 255, 255, 255, 1, 0, 2, 0]
    array('h', [-2, -1, 1, 2])
    

---
**Summary**  
  
표준 라이브러리에서 제공하는 시퀀스형을 제대로 파악하고 있어야 간결하고  
효율적이며 파이썬스러운 코드를 작성할 수 있다
  
균일 시퀀스는 작고 빠르고 사용하기 쉽지만 원자적인 데이터만을 다룰 수 있다
반면 컨테이너 시퀀스는 융퉁성이 있지만 가변 객체를 저장할 때에는 조심할 필요가 있다
  
지능형 리스트와 제너레이터 표현식은 시퀀스를 생성하고 초기화하는 강력한 표기법이다
  
    
파이썬에서 제공하는 튜플은 익명 필드를 가진 레코드 및 불변 리스트로 사용할 수 있다
튜플을 레코드로 사용할 때에는 튜플 언패킹이 필드에 접근하는 가장 안전하고 가독성이 
좋은 방법이다 
