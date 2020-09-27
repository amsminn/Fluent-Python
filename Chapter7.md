# chapter 7 정리 :  함수 데커레이터와 클로저
***
### Concept 
* 함수 데커레이터
    함수 데커레이터는 소스 코드에 있는 함수를 '표시'해서 함수의 작동을 개선할 수 있게 해준다  
    함수 데커레이터에는 클로저가 필요한데, 클로저는 데커레이터 외에도 콜백을 이용한   
    효율적인 비동기 프로그래밍과 필요에 따라 함수형 스타일로 코딩하는 데에도 필수적이다
      
    이 장에서는 단순한 등록 데케레이터에서부터 복잡한 매개변수화된 데커레이터에 이르기까지 함수 데커레이터가      
    정확히 어떻게 작동되는지 설명하는 것이다

---    
    
### Contents  

**1. 데커레이터 기본 지식**
    
**2. 파이썬이 데커레이터를 실행하는 시점**  
      
**3 클로저**  

**4 nonlocal 선언**  
 
**5 간단한 데커레이터 구현하기**  
     
**6 functools.lru_cache()를 이용한 메모이제이션**
       
**7 요약**    
    
    
---

### Usage        


    
    
  
**1. 데커레이터 기본 지식**    

데커레이터는 다른 함수를 인수로 받는 콜러블(데커레이트된 함수)이다
데커레이트된 함수에 어떤 처리를 수행하고 함수를 반환하거나 함수를 
다른 함수나 콜러블 객체로 대체한다

데커레이터는 다름 함수를 인수로 전달해서 호출하는 일반적인 콜러블과 동일하다
그렇지만 런타임에 프로그램 행위를 변경하는 "메타프로그래밍"을 할 때 데커레이터가 상당히 편리하다  
  
---  
  
**2. 파이썬이 데커레이터를 실행하는 시점**  
  
데커레이터의 핵심 특징은 데커레이트된 함수가 정의된 직후에 실행된다는 것이다  
이는 일반적으로 파이썬이 로딩하는 시점, 즉 **임포트 타임**에 실행된다  
  
아래 코드를 스크립트로 실행하지 않고 임포트하면 **임포트 타임**과 **런타임**을 확실히 알 수 있다  


```python
def trace(func):                           
    def wrapper():
        print(func.__name__, '함수 시작')
       # func()                              
        print(func.__name__, '함수 끝')
    return wrapper                        
 
@trace    # @데코레이터
def asd():
    print('hello')
 
@trace    # @데코레이터
def ds():
    print('world')
 
asd() 
ds()    
```

    asd 함수 시작
    asd 함수 끝
    ds 함수 시작
    ds 함수 끝
    


```python
registry=[]

def register(func):
    print('running register(%s)' %func)
    registry.append(func)
    return func

@register
def f1():
    print('running f1()')
    
@register
def f2():
    print('running f2()')

def f3():
    print('running f3()')
    
def main():
    print('running main()')
    print('registry ->', registry)
    f1()
    f2()
    f3()

if __name__=='__main__':
    main()
```

    running register(<function f1 at 0x000001EECC08D1F0>)
    running register(<function f2 at 0x000001EECC08DAF0>)
    running main()
    registry -> [<function f1 at 0x000001EECC08D1F0>, <function f2 at 0x000001EECC08DAF0>]
    running f1()
    running f2()
    running f3()
    

---  

**3. 클로저**  
  
블로그 글을 보다 보면 클로저를 익명 함수와 혼동하는 경우가 종종 있다  
아마도 익명 함수를 이용하면서 함수 안에 함수를 정의하는 방식이 보편화되었기 때문일거다
그리고 클로저는 내포된 함수 안에서만 의미가 있다

실제로 클로저는 함수 본체에서 정의하지 않고   
참조하는 비전역 변수를 포함한 확장 범위를 가진 함수다  
함수가 익명 함수인지 여부는중요하지 않다  
함수 본체 외부에 정의된 비전역 변수에 접근할 수 있다는 것이 중요하다

avg()는 이전 값을 기억해서 계산한다  
어떻게 구현했을지 알아보자


```python
class Average(): # class로 구현한 avg()
    def __init__(self):
        self.series = []
    def __call__(self, new_value):
        self.series.append(new_value)
        total = sum(self.series)
        return total/len(self.series)

avg = Average()
print(avg(10))
print(avg(11))
print(avg(12))
```

    10.0
    10.5
    11.0
    


```python
def make_averager(): # 고위 함수로 구현한 avg()
    series = []
    def averager(new_value):
        series.append(new_value)
        total = sum(series)
        return total/len(series)
    return averager

avg = make_averager()
print(avg(10))
print(avg(11))
print(avg(12))
```

    10.0
    10.5
    11.0
    

---  

**4. nonlocal 선언**  
  
앞에서 구현한 make_avrager()는 그리 효율적이지 않다
평균을 구할 때는 이전의 모든 값이 필요한게 아닌 이전 값들의 합만 알면 
되는 것이기 때문이다
count와 total 변수만 가지고 있다면 훨씬 효율적으로 평균을 구할 수 있다

하지만 리스트는 가변형이고 count total과 같은 변수는 불변형이기 때문에 
값을 갱신하려 하면 지역 변수가 된다
이를 해결해주는 것이 nonlocal이다


```python
def make_avergage():
    count = 0
    total = 0
    def averager(new_value):
        nonlocal count, total
        count+=1
        total+=new_value
        return total/count
    return averager

avg = make_averager()
print(avg(10))
print(avg(11))
print(avg(12))
```

    10.0
    10.5
    11.0
    

---  
  
**5. 간단한 데커레이터 구현하기**  


```python
import time

def clock(func):
    def clocked(*args):
        t0 = time.perf_counter()
        result = func(*args)
        elapsed = time.perf_counter() - t0
        name = func.__name__
        arg_str = ', '.join(repr(arg) for arg in args)
        print('[%0.8fs] %s(%s) -> %r' % (elapsed, name, arg_str, result))
        return result
    return clocked

@clock
def factorial(n):
    return 1 if n < 2 else n * factorial(n - 1)

if __name__ == '__main__':
    print('*' * 40, 'Calling factorial(6)')
    print('f6! = %d'%factorial(6))
```

    **************************************** Calling factorial(6)
    [0.00000030s] factorial(1) -> 1
    [0.00002630s] factorial(2) -> 2
    [0.00004030s] factorial(3) -> 6
    [0.00005240s] factorial(4) -> 24
    [0.00006430s] factorial(5) -> 120
    [0.00007720s] factorial(6) -> 720
    f6! = 720
    

---  
  
**6. functools.lru_cache()를 이용한 메모이제이션**  
  
functools.lru_cahce() 데커레이터는 실제로 쓸모가 많은 데커레이터로써, 메모이제이션을 구현한다  
중복된 데이터에 대한 호출을 처리하기 위해 메모이제이션이 필요한 것은 모두 알 것이다  
이러한 메모이제이션을 알아서 해주는 functools.lru_cahce()을 알아보자
 


```python
def clock(func):
    def clocked(*args):
        t0 = time.perf_counter()
        result = func(*args)
        elapsed = time.perf_counter() - t0
        name = func.__name__
        arg_str = ', '.join(repr(arg) for arg in args)
        print('[%0.8fs] %s(%s) -> %r' % (elapsed, name, arg_str, result))
        return result
    return clocked

@clock
def fibonacci(n):
    if n<2:
        return n
    return fibonacci(n-2) + fibonacci(n-1)

if __name__ =='__main__':
    print(fibonacci(6))
```

    [0.00000040s] fibonacci(0) -> 0
    [0.00000030s] fibonacci(1) -> 1
    [0.00009270s] fibonacci(2) -> 1
    [0.00000030s] fibonacci(1) -> 1
    [0.00000030s] fibonacci(0) -> 0
    [0.00000020s] fibonacci(1) -> 1
    [0.00002150s] fibonacci(2) -> 1
    [0.00004410s] fibonacci(3) -> 2
    [0.00016130s] fibonacci(4) -> 3
    [0.00000020s] fibonacci(1) -> 1
    [0.00000020s] fibonacci(0) -> 0
    [0.00000020s] fibonacci(1) -> 1
    [0.00001930s] fibonacci(2) -> 1
    [0.00003980s] fibonacci(3) -> 2
    [0.00000020s] fibonacci(0) -> 0
    [0.00000030s] fibonacci(1) -> 1
    [0.00002160s] fibonacci(2) -> 1
    [0.00000020s] fibonacci(1) -> 1
    [0.00000030s] fibonacci(0) -> 0
    [0.00000020s] fibonacci(1) -> 1
    [0.00002060s] fibonacci(2) -> 1
    [0.00004040s] fibonacci(3) -> 2
    [0.00008230s] fibonacci(4) -> 3
    [0.00014150s] fibonacci(5) -> 5
    [0.00032410s] fibonacci(6) -> 8
    8
    


```python
import functools

def clock(func):
    def clocked(*args):
        t0 = time.perf_counter()
        result = func(*args)
        elapsed = time.perf_counter() - t0
        name = func.__name__
        arg_str = ', '.join(repr(arg) for arg in args)
        print('[%0.8fs] %s(%s) -> %r' % (elapsed, name, arg_str, result))
        return result
    return clocked

@functools.lru_cache()
@clock
def fibonacci(n):
    if n<2:
        return n
    return fibonacci(n-2) + fibonacci(n-1)

if __name__ =='__main__':
    print(fibonacci(6))
```

    [0.00000040s] fibonacci(0) -> 0
    [0.00000060s] fibonacci(1) -> 1
    [0.00007810s] fibonacci(2) -> 1
    [0.00000090s] fibonacci(3) -> 2
    [0.00010760s] fibonacci(4) -> 3
    [0.00000060s] fibonacci(5) -> 5
    [0.00013430s] fibonacci(6) -> 8
    8
    

좀 더 극단적인 케이스를 말하자면 
메모이제이션을 하지 않은 코드는 fibonacci(30)을 계산하기 위해
2,692,537번의 호출이 이뤄졌고 메모이제이션을 적용한 코드는 
정확히 31번만에 호출이 종료되었다

---
  
**7. 요약**    
  
등록 데커레이터는 본질적으로 간단한 메커니즘이지만 고급 파이썬 프레임워크에서 실제 사용되고 있다
매개변수화된 데커레이터는 대부분 최소 두 단계의 내포된 함수를 가지고 있으며
더 고급 기법을 지원하는 데커레이터를 구현하기 위해 @functools.wraps를 사용하는 경우 
세 단계 이상 내포되기도 한다
  
데커레이터가 실제 작동하는 방식을 이해하려면 **임포트 타임**과 **런타임**의 차이를 알아야 하며
변수 범위, 클로저, nonlocal 선언에 대해서도 자세히 알아야 한다
클로저와 nonlocal을 완전히 이해하면 데커레이터를 만드는 것 뿐만 아니라
GUI 방식의 이벤트 주도 프로그램이나 콜백을 이용한 비동기 입출력을 구현할 때도 
큰 도움이 된다고 한다
