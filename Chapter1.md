# chapter 1 정리 : 특별 메서드
***
### Concept 
기본적인 객체 연산을 수행하는 메서드로 파이썬 인터프리터가 호출한다   
-언제나 앞뒤로\_\_len\_\_()과 같이 이중 언더바를 갖고 있다  
***

### Usage  
특별 메서드를 구현하면 사용자 정의 객체도 내장형 객체처럼 작동하게 되어  
보다 파이썬스러운 코딩 스타일을 구사할 수 있다  
-C++에 비유하자면 연산자 오버로딩과 비슷한 느낌이다
***
### How it works internally   
* len(x)와 for in i t에서 호출되는  iter(y)를 예시로 설명하면 각각 x.\_\_len\_\_()와 y.\_\_iter\_\_()를 호출한다
* list, str, bytearray와 같은 내장 자료형이라면 PyVarObject C 구조체의 ob_size 필드의 값을 반환한다  
(이 경우에는 \_\_len\_\_() 메서드 호출보다 ob_size의 리턴이 빠르기 때문) 



***
### Example Code  
**1) FrenchDeck 클래스 구현**


```python
import collections
from random import choice 
Card = collections.namedtuple('Card', ['rank', 'suit'])
class FrenchDeck:
    ranks = [str(n) for n in range(2,11)] + list('JOKA')
    suits = 'spades diamonds clubs hearts'.split()
    def __init__(self):
        self._cards = [Card(rank, suit) for suit in self.suits
                                        for rank in self.ranks]
    def __len__(self):
        return len(self._cards)
    def __getitem__(self, position):
        return self._cards[position]
deck = FrenchDeck()
```

***
**2) \_\_len\_\_()과 \_\_getitem\_\_() 사용해보기**


```python
print(len(deck), deck[1], sep = '\n')
```

    52
    Card(rank='3', suit='spades')
    

여기서 len(deck)은 \_\_len\_\_(deck)을 호출하고 
deck[1]은 deck.\_\_getitem\_\_(1)을 호출해 다음과 같은 결과가 출력된다
***

**3) \_\_getitem\_\_()을 사용한 random.choice와 슬라이싱**


```python
print('Random choice :',choice(deck))
```

    Random choice : Card(rank='J', suit='diamonds')
    


```python
deck[12::13]
```




    [Card(rank='A', suit='spades'),
     Card(rank='A', suit='diamonds'),
     Card(rank='A', suit='clubs'),
     Card(rank='A', suit='hearts')]



***
### 특별 메서드 개요 (연산자 제외)

| 범주 | 메서드명 |
|----------------------------------|---------------------------------------------------------------------|
|문자열/바이트 표현|\_\_repr\_\_, \_\_str\_\_, \_\_format\_\_, \_\_bytes\_\_|
|숫자로 변환|\_\_abs\_\_, \_\_bool\_\_, \_\_complex\_\_, \_\_init\_\_, \_\_float\_\_, \_\_hash\_\_, \_\_index\_\_|
|컬렉션 에뮬레이션|\_\_len\_\_, \_\_getitem\_\_, \_\_setitem\_\_, \_\_delitem\_\_, \_\_contains\_\_, \_\_iter\_\_, \_\_reserved\_\_, \_\_next\_\_|
|콜러블 에뮬레이션|\_\_call\_\_|
|콘텍스트 관리|\_\_enter\_\_, \_\_exit\_\_|
|객체 생성 및 소멸|\_\_new\_\_, \_\_init\_\_, \_\_del\_\_|
|속성 관리|\_\_getattr\_\_, \_\_getattribute\_\_, \_\_setattr\_\_, \_\_delattr\_\_, \_\_dir\_\_|
|속성 디스크립터|\_\_get\_\_, \_\_set\_\_, \_\_delete\_\_|
|클래스 서비스|\_\_prepare\_\_, \_\_instancecheck\_\_, \_\_subclasscheck\_\_|

|범주|메서드명|연산자|
|---------------------|----------------------------------------------------------|---------------------------------|
|단항 수치 연산자|\_\_neg\_\_<br>\_\_pos\_\_<br>\_\_abs\_\_|-<br>+<br>abs()|
|향상된 비교 연산자|\_\_lt\_\_<br>\_\_le\_\_<br>\_\_eq\_\_<br>\_\_ne\_\_<br>\_\_gt\_\_<br>\_\_ge\_\_|<<br><=<br>==<br>!=<br>><br>>=|
|산술 연산자|\_\_add\_\_<br>\_\_sub\_\_<br>\_\_mul\_\_<br>\_\_truediv\_\_<br>\_\_floordiv\_\_<br>\_\_mod\_\_<br>\_\_divmod\_\_<br>\_\_pow\_\_<br>\_\_round\_\_|+<br>-<br>*<br>/<br>//<br>%<br>divmod()<br>** 또는 pow()<br>round()|
|역순 산술 연산자|\_\_radd\_\_<br>\_\_rsub\_\_<br>\_\_rmul\_\_<br>\_\_rtruediv\_\_<br>\_\_rfloordiv\_\_<br>\_\_rmod\_\_<br>\_\_rdivmod\_\_<br>\_\_rpow\_\_||
|복합 할당 산술 연산자|\_\_iadd\_\_<br>\_\_isub\_\_<br>\_\_imul\_\_<br>\_\_itruediv\_\_<br>\_\_ifloordiv\_\_<br>\_\_imod\_\_<br>\_\_ipow\_\_|+=<br>-=<br>*=<br>/=<br>//=<br>%=<br>**=|
|비트 연산자|\_\_invert\_\_<br>\_\_lshift\_\_<br>\_\_rshite\_\_<br>\_\_and\_\_<br>\_\_or\_\_<br>\_\_xor\_\_|~<br><<<br>>><br>&<br>\|<br>^|
|역순 비트 연산자|\_\_rlshift\_\_<br>\_\_rrshift\_\_<br>\_\_rand\_\_<br>\_\_rxor\_\_<br>\_\_ror\_\_||
|복합 할당 비트 연산자|\_\_ilshift\_\_<br>\_\_irshift\_\_<br>\_\_iand\_\_<br>\_\_ixor\_\_<br>\_\_ior\_\_|<<=<br>>>=<br>&=<br>^=<br>\|=
