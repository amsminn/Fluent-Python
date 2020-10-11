# chapter 10 정리 :  인터페이스 : 프로토콜에서 ABC까지
***
### Concept 
* 11장의 주제는 인터페이스이다. 덕 타이핑의 상징인 동적 인터페이스에서부터   
인터페이스를 명시하고 구현이 인터페이스에 따르는지 검증하는   
추상 베이스 클래스(ABC)에 이르기까지 다양한 곳에서 인터페이스가 적용된다.      

---    
    
### Contents  

**1 파이썬 문화에서의 인터페이스와 프로토콜**
    
**2.파이썬은 시퀀스를 찾아낸다**  
      
**3 런타임에 프로토콜을 구현하는 멍키 패칭**   

**7 요약**    
    
    
---

### Usage        
  
**1. 파이썬 문화에서의 인터페이스와 프로토콜**    

파이썬은 ABC가 소개되기 전부터 이미 성공을 거두었고, 기존 코드 대부분은    
ABC를 전혀 사용하지 않는다. 

동적 자료형 언어에서는 인터페이스가 어떻게 작동하는지 알아보자.  
우선 파이썬에는 interface()라는 키워드가 없지만, ABC와 관계없이 
모든 클래스는 인터페이스를 가지고 있다.  
클래스가 상속하거나 구현한 공개 속성(메서드나 데이터 속성)들의 집합이
인터페이스다.
> 특별 메서드도 포함한다.

보호된 속성과 비공개 속성은 인터페이스로 사용하지 않는 것이 관례이고,  
이를 어기지 않는 것이 좋다. (항상 예외는 있으므로 절대적인 것은 아니다)

한편 공개 데이터 속성을 객체의 인터페이스로 사용하는 것은 나쁘지 않다.  
필요하면 언제나 데이터 속성을 호출 코드로 망가뜨리지 않고 <객체>.<속성> 구문을  
사용해서 게터/세터를 구현하는 프로퍼티로 변환할 수 있기 때문이다.

아래는 x와 y를 읽기 전용 프로퍼티로 변경한 코드다.


```python
class Vector2d:
    typecode = 'd'
    
    def __init__(self, x, y):
        self.__x = float(x)
        self.__y = float(y)
    
    @property
    def x(self):
        return self.__x
    
    @property
    def y(self):
        return self.__y
    
    # ...
```

---  
  
**2. 파이썬은 시퀀스를 찾아낸다**

파이썬 데이터 모델은 가능한 많이 핵심 프로토콜과 협업하겠다는  
철학을 가지고 있다. 시퀀스의 경우, 가장 단순한 객체를 사용하는 경우에도  
파이썬은 적극적이다.

아래 그림은 ABC로 정의된 공식적인 Seauence 인터페이스를 보여준다.

![](https://ibb.co/59kn23x)

아래 예제는 abc.Sequence를 상속하지 않으며,   
시퀀스 프로토콜 메서드 중 \_\_getitem\_\_() 하나만 구현한다.


```python
class Foo:
    def __getitem__(self, pos):
        return range(0, 30, 10)[pos]

f = Foo()
for i in f: 
    print(i)
```

    0
    10
    20
    

---  
  
**3. 런타임에 프로토콜을 구현하는 멍키 패칭(monkey patching)**  
  
멍키 패칭이란?
>코드를 수정하지 않고 런타임에 모듈이나 클래스를 변경하는 것


```python
import collections

Card = collections.namedtuple('Card', ['rank', 'suit'])

class FrenchDeck:
    ranks = [str(n) for n in range(2, 11)] + list('JQKA')
    suits = 'spades diamonds clubs hearts'.split()
    
    def __init__(self):
        self._cards = [Card(rank, suit) for suit in self.suits
                                        for rank in self.ranks]
        
    def __len__(self):
        return len(self._cards)
    
    def __getitem__(self, position):
        return self._cards[position]
```


```python
from random import shuffle

l = list(range(10))
shuffle(l)
l
```




    [4, 3, 5, 0, 9, 7, 6, 1, 8, 2]



위 코드에서 shuffle() 메서드를 구현하지 않았다.   
FrenchDeck 클래스는 시퀀스처럼 동작하기 때문이다.  
위 코드에서 shuffle()을 진행하면 할당을 지원하지 않는 
클래스이기 때문에 오류가 난다.  
아래는 그를 수정한 코드다.


```python
import collections

Card = collections.namedtuple('Card', ['rank', 'suit'])

class FrenchDeck:
    ranks = [str(n) for n in range(2, 11)] + list('JQKA')
    suits = 'spades diamonds clubs hearts'.split()
    
    def __init__(self):
        self._cards = [Card(rank, suit) for suit in self.suits
                                        for rank in self.ranks]
        
    def __len__(self):
        return len(self._cards)
    
    def __getitem__(self, position):
        return self._cards[position]
    
def set_card(deck, position, card):
    deck._cards[position] = card
    
deck = FrenchDeck()    
FrenchDeck.__setitem__ = set_card
shuffle(deck)
deck[:5]
```




    [Card(rank='K', suit='spades'),
     Card(rank='Q', suit='clubs'),
     Card(rank='5', suit='clubs'),
     Card(rank='10', suit='diamonds'),
     Card(rank='K', suit='diamonds')]


