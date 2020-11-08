# 콘텍스트 관리자와 else 블록

---

**이 장에서는 다음과 같은 제어 기능을 살펴본다**
> with문과 콘텍스트 관리자
> for, while, try 문에서의 else 블록

---

**1. if문 이외에서의 else 블록**   

else 절은 if문 뿐만 아니라 for, while, try 문에서도 사용할 수 있다.

규칙은 다음과 같다.
>for, while  
 루프가 완전히 실행된 후에(break문으로 중단하는 경우 제외) else 블록이 실행된다.  
> try  
 try 블록에서 예외가 발생하지 않을 때만 else 블록이 실행된다.  
 또한 else 블록에서 발생한 예외는 else 블록 앞에 나오는 except 블록에서  
 처리되지 않는다.


```python
def dangerous_call():
    print(1)
def after_call():
    print(1)
try:
    dangerous_call()
    after_call()
except OSError:
    log('OSError')
```

    1
    1
    


```python
def dangerous_call():
    print(1)
def after_call():
    print(1)
try:
    dangerous_call()
except OSError:
    log('OSError')
else:
    after_call()
```

    1
    1
    

위와 같이 try에서 else를 사용하면 가독성에 도움이 된다.

---  
  
**2. contextlib 유틸리티**

>closing()  
 엔터, 엑시트 프로토콜을 구현하지 않는 객체로부터 콘텍스트 관리자를 생성하는 함수    
 
>suppress    
 지정한 예외를 임시로 무시하는 콘텍스트 관리자    
 
>@contextmanager  
 클래스를 생성하고 프로토콜을 구현하는 대신  
 간단한 제너레이터 함수로부터 콘텍스트 관리자를   
 생성하 수 있게 해주는 함수


```python
import contextlib

@contextlib.contextmanager
def looking_glass():
    import sys
    original_write = sys.stdout.write
    
    def reverse_write(text):
        original_write(text[::-1])
        
    sys.stdout.write = reverse_write
    yield 'JABBERWOCKY'
    sys.stdout.write = original_write
    
with looking_glass() as what:
    print('Alice, Kitty and Snowdrop')
    print(what)
```

    pordwonS dna yttiK ,ecilA
    YKCOWREBBAJ
    
