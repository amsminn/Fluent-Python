# chapter 9 정리 :  파이썬스러운 객체
***
### Concept 
* 이 장에서는 파이썬 객체에서 흔히 볼 수 있는 
  여러 가지 특별 메서드를 
  구현하는 방법을 보여준다.

---    
    
### Contents  

**1 객체 표현**
    
**2.대안 생성자**  
      
**3 @classmethod와 @staticmethod**  

**4 해시 가능한 Vector2d**
     
**5 __slots__ 클래스 속성으로 공간 절약하기**
       
**6 요약**    
    
    
---

### Usage        
  
**1. 객체 표현**    

> repr()
> 객체를 개발자가 보고자 하는 형태로 표현한 문자열로 반환한다.  
> str()
> 객체를 사용자가 보고자 하는 형태로 펴햔한 문자열로 반환한다.  
  
chapter1에서 설명한 것처럼 repr()과 str() 메서드를 지원하려면  
\_\_repr\_\_()과 \_\_str\_\_() 특별 메서드를 구현해야 한다.  
  
객체를 표현하는 다른 방법을 지원하는 \_\_bytes\_\_()와 \_\_format\_\_() 두개의 특별 메서드도 있다. 

---  
  
**2. 대안 생성자**  
  
Vector2d라는 객체를 bytes로 변환하는 메서드가 있다면
당연히 bytes를 Vector2d로 변환하는 메서드도 있어야 할 것이다.


```python
from array import array
import math

class Vector2d:
    typecode = 'd'
    
    def __init__(self, x, y):
        self.x = float(x)
        self.y = float(y)
        
    def __iter__(self):
        return (i for i in (self.x, self.y))
    
    def __repr__(self):
        class_name = type(self).__name__
        return '{}({!r}, {!r})'.format(class_name, *self)
    
    def __str__(self):
        return str(tuple(self))
    
    def __bytes__(self):
        return (bytes([ord(self.typecode)]) +
                bytes(array(self.typecode, self)))
    
    def __eq__(self, other):
        return tuple(self) == tuple(other)
    
    def __abs__(self):
        return math.hypot(self.x, self.y)
    
    def __bool__(self):
        return bool(abs(self))    

    def __format__(self, fmt_spec=''):
        components = (format(c, fmt_spec) for c in self)
        return '({}, {})'.format(*components)
```


```python
@classmethod
def frombytes(cls, octets):
    typecode = chr(octets[0])
    memv = memoryview(octets[1:].cast(typecode))
    return cls(*memv)
```

---  
  
**3. @classmethod와 @staticmethod**  

파이썬 튜토리얼에서는 @classmethod와 @staticmethod 데커레이터에 대해 설명하지 않는다.  
staticmethod의 경우 부모 클래스의 클래스 속성 값을 가져오지만   
classmethod의 경우 cls인자를 활용하여 현재 클래스의 클래스 속성을 가져온다.


```python
class Demo:
    @classmethod
    def klassmeth(*args):
        return args
    @staticmethod
    def statmethod(*args):
        return args
print(Demo.klassmeth())
print(Demo.klassmeth('spam'))
print(Demo.statmethod())
print(Demo.statmethod('spam'))
```

    (<class '__main__.Demo'>,)
    (<class '__main__.Demo'>, 'spam')
    ()
    ('spam',)
    

---  
  
**4. 해시 가능한 Vector2d**  
  
  


```python
from array import array
import math

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
    
    def __eq__(self, other):
        return tuple(self) == tuple(other)
    
    def __hash__(self):
        return hash(self.x) ^ hash(self.y)
```

---  
  
**5. slots 클래스 속성으로 공간 절약하기**  
  
기본적으로 파이썬은 객체 속성을 각 객체 안의 \_\_dict\_\_라는 딕셔너리형 속성에 저장한다.
딕셔너리는 3장에서 설명했듯이 메모리를 많이 먹는다.
만약 속성이 몇개 없는 수백만의 객체를 다루다면, \_\_slots\_\_ 클래스 속성을 이용해
메모리를 많이 절약할 수 있다.
\_\_slots\_\_ 속성은 파이썬 인터프리터가 객체 속성을 딕셔너리 대신 튜플에 저장하게 만든다  
  
**__slots__를 정의하려면**
> 1. 이 이름의 클래스 속성을 생성한다
> 2. 객체 속성 식별자들을 담은 문자열의 반복형을 할당한다  
> -끝-  
  
  


```python
class Vector2d:
    __slots__ = ('__x', '__y')
    # ...
```

---  
  
**6. 요약**  
  
이 장에서는 특별 메서드 및 파이썬스러운 클래스를 생성하는 관례에 대해 설명했다
9장을 읽으면서 1장으로 돌아간듯한 느낌을 받았다... 뭐 연결되는 내용이긴하니...
