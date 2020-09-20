# chapter 4 정리 : 텍스트와 바이트
***
### Concept 
* 문자열 인코딩
    파이썬3부터는 인간이 사용하는 텍스트 문자열과 원시 바이트 시퀀스를   
    엄격히 구분하기 시작했다 
    이 장에서는 유니코드 문자열, 이진 시퀀스 그리고   
    이 둘간의 변환에 사용되는 인코딩에 대해 설명한다
    
---    
    
### Contents
**1. 문자, 코드 포인트, 바이트 표현**  

    코드 포인트와 인코딩의 개념
    바이트에 대한 기본 지식
      
      
**2. 이진 시퀀스의 고유한 특징**    

    구조체와 메모리 뷰      
    
**3. 인코딩 에러를 피하고 다루기**  

    UnicodeEncodeError 처리하기
    UnicodeDecodeError 처리하기
    예상과 달리 인코딩된 모듈을 로딩할 때 발생하는 SyntaxError

**4. 제대로 비교하기 위해 유니코드 정규화하기**  
  
    케이스 홀딩
    유틸리티 함수
    
**5. 유니코드 텍스트 정렬하기**  
  
    유니코드 대조 알고리즘을 이용한 정렬 (PyUca)
    
    
---

### Usage    
  
**1. 문자, 코드 포인트, 바이트 표현** 
  
**코드 포인트와 인코딩의 개념**  

     코드 포인트   
     문자의 단위 원소, 10진수 0에서 1,114,111까지의 숫자며 유니코드 표준에서는   
     'U+'접두사를 붙여 4자리에서 6자리 사이의 16진수로 표현한다
                                                  ex) 'A'의 코드 포인트는 U+0041  
                                                    
     인코딩   
     코드 포인트를 바이트 시퀀스로 변환하는 알고리즘
                                                  ex) UTF-8기준 문자 'A'(U+0041)은
                                                      1바이트 /x41이다
                                                      
---

  
**2. 이진 시퀀스의 고유한 특징**
      
    struct 모듈은 패킹된 바이트를 형의 필드로 구성된 튜플로 분석하고  
    이와 반대로 튜플을 패킹된 바이트로 제공하는 함수를 제공한다
    struct는 bytes, bytesarray, memoryview 객체와 사용된다
    
    아래 코드는 memoryview와 struct를 이용해 GIF의 종류, 버전, 너비, 높이를 추출한다
    저 코드만을 실행하면 GIF 파일이 없기때문에 오류가 날 것이다
    파일이 있다면 (b'GIF', b'89a', 555,230)과 같이 출력된다


```python
import struct
fmt = '<3s3sHH'
with open('filter.gif', 'rb') as fp:
    img = memoryview(fp.read())
header = img[:10]
bytes(header)
st = struct.unpack(fmt, header)
print(st)
```


    ---------------------------------------------------------------------------

    FileNotFoundError                         Traceback (most recent call last)

    <ipython-input-2-8016271fad07> in <module>
          1 import struct
          2 fmt = '<3s3sHH'
    ----> 3 with open('filter.gif', 'rb') as fp:
          4     img = memoryview(fp.read())
          5 header = img[:10]
    

    FileNotFoundError: [Errno 2] No such file or directory: 'filter.gif'


---

**3. 인코딩 에러를 피하고 다루기**  

    UnicodeEncodeError 처리하기  
        대부분의 비UTF 코덱은 유니코드 문자의 일부만 처리할 수 있다
        텍스트를 바이트로 변환할 때 문자가 인코딩에 정의되어 있지 않으면
        인코딩 메서드나 함수의 errors 인수에 별도의 처리기를 지정하지 않는 한
        UnicodeEncodeError가 발생한다 
        
        아래 코드는 에러 처리기를 사용하는 예제이다


```python
city = 'São Paulo'
print(city.encode('utf-8'))
print(city.encode('utf-16'))
print(city.encode('iso8859_1'))
# print(city.encode('cp437')) 에러 발생
print(city.encode('cp437', errors='ignore')) # 에러 처리기 사용법 errors = ""
print(city.encode('cp437', errors='replace'))
print(city.encode('cp437', errors='xmlcharrefreplace'))
```

    b'S\xc3\xa3o Paulo'
    b'\xff\xfeS\x00\xe3\x00o\x00 \x00P\x00a\x00u\x00l\x00o\x00'
    b'S\xe3o Paulo'
    b'So Paulo'
    b'S?o Paulo'
    b'S&#227;o Paulo'
    

    UnicodeDecodeError 처리하기
        이진 시퀀스를 텍스트로 변환할 때 정당한 문자로 변환할 수 없다면 
        UnicodeDecodeError가 발생한다
        
        그렇지만 cp1252, iso8859_1, koi8_r 등 많은 레거시 8비트 코덱은 무작위 비트 배열에 대해서도 
        에러를 발생시키지 않고 바이트 스트림으로 디코딩할 수 있다  
          
          
    예상과 달리 인코딩된 모듈을 로딩할 때 발생하는 SyntaxError
        파이썬2.5부터는 아스키를, 파이썬3부터는 utf-8을 소스 코드 기본 인코딩 방식으로 사용했다
        파이썬3에서 인코딩 선언 없이 비utf-8로 인코딩된 .py 모듈을 로딩하면 에러가 발생한다  
        
        해결법은 파일 최상단에 # coding: cp1252 라고 주석을 적는 것이다


```python
#coding: cp1252
print('olá')
```

    São Paulo
    

---

**4. 제대로 비교하기 위해 유니코드 정규화하기**  
  
      케이스 폴딩
          본질적으로 케이스 폴딩은 모든 텍스트를 소문자로 변환하는 연산이며
          약간의 변환을 동반한다 케이스 폴딩은 파이썬 3.3에 추가된
          str.casefold() 메서드를 이용해서 수행한다
            
          ex) 마이크로 기호 'μ'는 그리스 소문자 뮤로 변환한다 (비슷하게 생겼다)


```python
micro = 'μ'
print(micro)
micro_fold = micro.casefold()
print(micro_fold)
```

    μ
    μ
    

    유틸리티 함수
        NFC와 NFD는 안전하면 유니코드 문자열을 적절히 비교할 수 있게 해준다
        NFC는 대부분의 애플리케이션에서 사용할 수 있는 최고의 정규화된 형태며
        str.casefold()는 대소문자 구분 없이 문자를 비교할 때 갖아 좋은 방법이다

**아래에서 구현하는 nfc_equal()과 fold_equal() 메서드는 굉장히 유용하다**


```python
from unicodedata import normalize

def nfc_equal(str1, str2):
    return normalize('NFC', str1) == normalize('NFC', str2)

def fold_equal(str1, str2):
    return (normalize('NFC',str1).casefold() == normalize('NFC', str2).casefold())

s1 = 'StraSSe'
s2 = 'strasse'

print(s1 == s2)
print(nfc_equal(s1,s2))
print(fold_equal(s1,s2))
```

    False
    False
    True
    

---

**5. 유니코드 텍스트 정렬하기**  
  
    유니코드 대조 알고리즘을 이용한 정렬 (PyUca)
        장고에 다양한 기여를 하고 있는 제임스 토버는 유니코드 대조 알고리즘(UCA)을
        순수 파이썬으로 구현한(PyUCA)를 만들었다
        PyUca는 사용법이 아주 간단하다
        
        아래는 예제 코드이다


```python
!pip install pyuca
import pyuca #쥬피터 노트북 환경에는 

coll = pyuca.Collator()
ca = ["cafe", "caff", "café"]
sorted_ca = sorted(ca, key = coll.sort_key)
print(sorted_ca)
```

    Collecting pyuca
      Downloading pyuca-1.2-py2.py3-none-any.whl (1.5 MB)
    Installing collected packages: pyuca
    Successfully installed pyuca-1.2
    ['cafe', 'café', 'caff']
    
