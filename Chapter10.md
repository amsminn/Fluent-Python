# chapter 10 정리 :  시퀀스 해킹, 해시, 슬라이스
***
### Concept 
* 이 장에서는 다차원 벡터를 나타내는 클래스를 생성해서 
  앞 장에서 구현한 2차원 Vector2d 클래스를 상당히 개선할 것이다.
  벡터는 표준 파이썬의 불변 균일 시퀀스와 비슷하게 작동할 것이며
  실수를 요소로 가지게 되며 다음과 같은 기능을 지원할 것이다.  
  
> 기본 시퀀스 프로토콜 : __len__()과 __getitem__() 메서드  
> 여러 항목을 가진 객체를 안전하게 표현  
> 슬라이싱을 지원해서 새로운 객체 생성  
> 포함된 요소 값을 모두 고려한 집합 해싱  
> 커스터마이즈된 포맷 언어 확장

---    
    
### Contents  

**1 Vector : 사용자 정의 시퀀스형**
      
**2 Vector 버전 \#1 : Vector2d 호환**  

**3 프로토콜과 덕 타이핑**  
 
**4 Vector 버전 \#2 : 슬라이싱 가능한 시퀀스**  
     
**5 Vector 버전 \#3 : 동적 속성 접근**
       
**6 Vector 버전 \#4 : 해싱 및 더 빠른 ==**   
    
---

### Usage        
  
**1.Vector : 사용자 정의 시퀀스형**  
  
우리의 전략은 상속이 아니라 구성을 이용해서 벡터를 구현하는 것이다.
요소들은 실수형 배열에 저장하고, 벡터가 불변 균일 시퀀스처럼
작동하게 만들기 위해 필요한 메서드들을 구현한다.

그러나 시퀀스 메서드를 구현하기 전에 앞에서 구현한 Vector2d 클래스와 호환성이 높은
기본 Vector 클래스를 먼저 만들어보겠다  
  
---  
  
**2. Vector 버전 \#2 : Vector2d 호환**  
  
Vector 생성자는 Vector2d 생성자와 호환되지 않도록 설계되어 있다.  
\_\_init\_\_() 메서드에서 임의의 인수 \*args를 받아서 Vector(3,4)나 Vector(3,4,5) 형태로 작동하게  
만들 수도 있지만, 시퀀스 생성자는 내장 시퀀스처럼 반복형을 인수로 받게 하는것이 좋다


```python

from array import array
import reprlib
import math

class Vector:
    typecode = 'd'
    
    def __init__(self, components):
        self._components = array(self.typecode, components)
        
    def __iter__(self):
        return iter(self._components) 
    
    def __repr__(self):
        components = reprlib.repr(self._components) 
        components = components[components.find('['):-1]
        return 'Vector({})'.format(components)
    
    def __str__(self):
        return str(tuple(self))
    
    def __bytes__(self):
        return (bytes([ord(self.typecode)]) + bytes(self._components)) 
    
    def __eq__(self, other):
        return tuple(self) == tuple(other)
    
    def __abs__(self):
        return math.sqrt(sum(x*x for x in self)) 
    
    def __bool__(self):
        return bool(abs(self))
    
    @classmethod
    def frombytes(cls, octects):
        typecode = chr(octects[0])
        memv = memoryview(octects[1:]).cast(typecode)
        return cls(memv)
```

---  
  
**3. 프로토콜과 덕타이핑**  
  
덕타이핑은 간단하다

> 오리처럼 보이면 오리다

이 말은 조건만 충족하면 해당 자료형으로 볼 수 있다는 것이다

즉, 시퀀스 프로토콜을 충족하면 시퀀스다

---  
  
**4. Vector 버전 \#2 : 슬라이싱 가능한 시퀀스**  
  
한참 앞 내용에서 나온 FrenchDeck 예제를 재활용하겠다

여기서 \_\_len\_\_()과 \_\_getitem\_\_() 메서드를 다음과 같이 구현할 수 있다


```python
class example:
    def __getitem__(self, idx):
        return idx
    def __len__(self):
        return len(self)
```

---  
  
**5. Vector 버전 \#3 : 동적 속성 접근**  
  
Vector2d에서 Vector로 진화하면서 v.x, v.y처럼 벡터 요소 이름으로 접근하는 능력을
상실했다. 이제는 아주 많은 요소를 가진 벡터를 이루고 있다. 그렇지만 앞에 있는 요소 몇개는
v\[0\], v\[1\], v\[2\] 대신 x, y, z로 접근할 수 있으면 편리할 것이다.

벡터의 처음 네 요소를 읽는 다른 방법도 있다 
아래 코드를 보자


```python
from array import array
import reprlib
import math
import numbers

class Vector:
    typecode = 'd'  #it is class characteristics to change between Vector2d and bytes
    
    def __init__(self, components):
        self._components = array(self.typecode, components) #vector components are array
        
        
    def __iter__(self):
        return iter(self._components) #iternable
    
    def __repr__(self):
        components = reprlib.repr(self._components) #self._components is expressed restricted length (uses ...)
        components = components[components.find('['):-1] # remove 'array('d',' and the end of the bracket to be able to give strings vector constructor
        return 'Vector({})'.format(components)
    
        
    def __str__(self):
        return str(tuple(self))
    
    def __bytes__(self):
        return (bytes([ord(self.typecode)]) +
                bytes(self._components)) # self._components makes byte object directly
    
    def __eq__(self, other):
        return tuple(self) == tuple(other) 
    
    def __abs__(self):
        return math.sqrt(sum(x*x for x in self)) 
    
    def __bool__(self):
        return bool(abs(self))    
    
    def __len__(self):
        return len(self._components)
    
    def __getitem__(self, index):
        cls = type(self) 
        if isinstance(index, slice): 
            return cls(self._components[index]) 
        elif isinstance(index, numbers.Integral):  
            return self._components[index] 
        else:
            msg = '{cls.__name__} indices must be integers'
            raise TypeError(msg.format(cls=cls))
            
    shortcut_names = 'xyzt'
    
    def __getattr__(self, name):
        cls = type(self)
        if len(name) ==1: 
            pos = cls.shortcut_names.find(name)
            if 0<= pos < len(self._components):
                return self._components[pos]
        
        msg = '{.__name__!r} object has no attribute {!r}'
        raise AttributeError(msg.format(cls, name))
        
    def __setattr__(self, name, value): 
        cls = type(self)
        if len(name) ==1: 
            if name in cls.shorcut_names:
                error = 'readonly attribute {attr_name!r}'
            elif name.islower():
                error = "Can't set attributes 'a' to 'z' in {cls_name!r}"
            else:
                error = ''
            if error:
                msg = error.format(cls_name=cls.__name__, attr_name=name)
                raise AttributeError(msg)
        super().__setattr__(name, value) 
        
        
    
@classmethod
def frombytes(cls, octets): 
    typecode = chr(octets[0])
    memv = memoryview(octets[1:]).cast(typecode)
    return cls(memv) 
```

---  
  
**6. Vector 버전 \#4 : 해싱 및 더 빠른 ==**    
  
\_\_hash\_\_() 메서드를 구현하겠다. 기존 \_\_eq\_\_() 메서드와 함께 구현하면 
Vector 객체를 해시할 수 있게 된다.


```python
from array import array
import reprlib
import math
import numbers
import functools
import operator
import itertools


class Vector:
    typecode = 'd'  #it is class characteristics to change between Vector2d and bytes
    
    def __init__(self, components):
        self._components = array(self.typecode, components) #vector components are array
        
        
    def __iter__(self):
        return iter(self._components) #iternable
    
    def __repr__(self):
        components = reprlib.repr(self._components) #self._components is expressed restricted length (uses ...)
        components = components[components.find('['):-1] # remove 'array('d',' and the end of the bracket to be able to give strings vector constructor
        return 'Vector({})'.format(components)
    
        
    def __str__(self):
        return str(tuple(self))
    
    def __bytes__(self):
        return (bytes([ord(self.typecode)]) +
                bytes(self._components)) # self._components makes byte object directly
    
    def __eq__(self, other): # Always __eq__() and __hash__() works together. Please make the mothods close each other.
        if len(self) != len(other): # To use very big object
            return False
        for a, b in zip(self, other): # zip function makes tuple generator consist of iternable arguments.
            if a!=b:
                return False
        return True
    
    '''def __eq__(self, other):
            return len(self) == len(other) and all(a==b for a, b in zip(self, other))'''
    
    #zip function stops the shortest argument. Comparing length is always first.
    
    def __hash__(self):
        hashes = (hash(x) for x in self._components) #mapping -> calculating hash values. map stage. refer below line.
        # hashes = map(hash, self._components)
        return functools.reduce(operator.xor, hashes, 0) #0 is initial value. The value must be used Identity (+, | , ^ = 0, x, & = 1)
    
    
    def __abs__(self):
        return math.sqrt(sum(x*x for x in self)) #hypot function is not available(two arguments needed)
    
    def __bool__(self):
        return bool(abs(self))    
    
    def __len__(self):
        return len(self._components)
    
    def __getitem__(self, index):
        cls = type(self) # taking object class(Vector) to use later
        if isinstance(index, slice): #if index argument is slice,
            return cls(self._components[index]) # make Vector object by _components slice
        elif isinstance(index, numbers.Integral): #if index argument is integer, 
            return self._components[index] # return components[index]
        else:
            msg = '{cls.__name__} indices must be integers'
            raise TypeError(msg.format(cls=cls))
            
    shortcut_names = 'xyzt'
    
    def __getattr__(self, name):
        cls = type(self) # taking object class(Vector) to use later
        if len(name) ==1: # if name is one character, it is possible that it is one of the shortcut_names. 
            pos = cls.shortcut_names.find(name)
            if 0<= pos < len(self._components):
                return self._components[pos]
        
        msg = '{.__name__!r} object has no attribute {!r}'
        raise AttributeError(msg.format(cls, name))
        
    def __setattr__(self, name, value):  #To avoid object working inconsistency, __getattr__() and __setattr__() methods are needed together
        cls = type(self)
        if len(name) ==1: # specific access to one character property
            if name in cls.shorcut_names:
                error = 'readonly attribute {attr_name!r}'
            elif name.islower():
                error = "Can't set attributes 'a' to 'z' in {cls_name!r}"
            else:
                error = ''
            if error:
                msg = error.format(cls_name=cls.__name__, attr_name=name)
                raise AttributeError(msg)
        super().__setattr__(name, value) # Call __setattr__ in superclass when no error is occurred.
    
    def angle(self, n): #refer wikipedia for hyperspherical envionments
        r = math.sqrt(sum(x*x for x in self[n:]))
        a = math.atan2(r, self[n-1])
        if (n==len(self)-1) and (self[-1] <0):
            return math.pi *2 -a
        else:
            return a
        
    def angles(self): # All radian is calculated
        return (self.angle(n) for n in range(1, len(self)))
    
            
    def __format__(self, fmt_spec = ''):
        if fmt_spec.endswith('h'): #hyperspherical coordinate
            fmt_spec = fmt_spec[:-1]
            coords = itertools.chain([abs(self)], #To use chain function, itertools module is imported
                                      self.angles()) # length and radians iternate sequencely
            outer_fmt = '<{}>'
        else:
            coords = self
            outer_fmt = '({})'
        components = (format(c, fmt_spec) for c in coords)
        return outer_fmt.format(', '.join(components))  
        
    
@classmethod
def frombytes(cls, octets): 
    typecode = chr(octets[0])
    memv = memoryview(octets[1:]).cast(typecode)
    return cls(memv) 
```
