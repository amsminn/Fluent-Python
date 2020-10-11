# chapter 12 정리 :  내장 자료형 상속과 다중 상속
***
### Concept 
* 이 장에서는 상속에 대해 설명하며, 특히 두 가지 특징을 집중적으로 알아본다.
>내장 자료형 상속의 위험성
>다중 상속과 메서드 결정 순서

다중 상속은 장점보다 단점이 많다고 생각하는 사람이 많다.
자바에 다중 상속이 없다고 해서 문제가 되진 않는다. 
오히려 C++에서 다중 상속을 과도하게 하다가
고통받은 많은 사람들이 이런 추세를 만들었다고 생각된다.

---    
    
### Contents  

**1 **
    
**2.파이썬은 시퀀스를 찾아낸다**  
      
**3 런타임에 프로토콜을 구현하는 멍키 패칭**   

**7 요약**    
    
    
---

### Usage        
  
**1. 내장 자료형의 상속은 까다롭다.**    

파이썬 2.2 이전까지는 list나 dict 등 내장 자료형을 상속할 수 없었다.  
C로 작성된 내장 클래스의 코드는 사용자가 오버라이드한 코드를 호출하지 않으므로  
상당한 주의가 필요하다.

PyPy 문서의 'PyPy와 CPython의 차이점' 중 '내장 자료형의 서브 클래스'절에서는
다음과 같이 문제를 간략히 설명하고 있다.

>공식적으로 CPython은 내장 자료형의 서브클래스에서 오버라이드한 메서드가 언제 호출되는지,    
>혹은 언제 호출되지 않는지에 대해 명확한 규칙을 정의하지 않는다.   
>일반적으로 서브클래스에서 오버라이드한 메서드는 같은 객체의 다른 내장 메서드에 의해 결코  
>호출되지 않는다. 예를 들어 dict의 서브클래스에서 오버라이드한 getitem 특별 메서드는   
>내장된 get()과 같은 메서드에 의해 호출되지 않는다.



```python
class DoppelDict(dict):
    def __setitem__(self, key, value):
        super().__setitem__(key, [value] * 2)
        
dd = DoppelDict(one=1)
dd
dd['two'] = 2
dd
```




    {'one': 1, 'two': [2, 2]}




```python
class AnswerDict(dict):
    def __getitem__(self, key):
        return 42
    
ad = AnswerDict(a='foo')
ad['a']
```




    42



---  
  
**2. 다중 상속과 메서드 결정 순서**

다중 상속을 지원하는 언어에서는 별개의 상위클래스가 
동일한 이름으로 메서드를 구현할 때 발생하는 이름 충돌 문제를 
해결해야 한다. 이런 이름 충돌 문제를 '다이아몬드 문제'라고 한다.


```python
class A:
    def ping(self):
        print('ping:', self)
    
class B(A):
    def pong(self):
        print('pong:', self)

class C(A):
    def pong(self):
        print('PONG:', self)
        
class D(B, C):
    def ping(self):
        super().ping()
        print('post-ping:', self)
        
    def pingpong(self):
        self.ping()
        super().ping()
        self.pong()
        super().pong()
        C.pong(self)
```

[](https://ibb.co/Wk9V82L)
위는 다이아몬드 문제를 클래스 다이어그램으로 나타낸 것이다.


---

**3. 다중 상속 다루기**

>1 다중 상속 다루기  
>2 인터페이스 상속과 구현 상속을 구분한다.  
>3 ABC를 이용해서 인터페이스를 명확히 한다.  
>4 코드를 재사용하기 위해 믹스인을 사용한다.  
>5 이름을 통해 믹스인임을 명확히 한다.  
>6 ABC가 믹스인이 될 수는 있지만, 믹스인이라고 해서 ABC인 것은 아니다.  
>7 두 개 이상의 구상 클래스에서 상속받지 않는다.  
>8 사용자에게 집합 클래스를 제공한다.  
>9 클래스 상속보다 객체 구성을 사용한다.
