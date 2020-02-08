**Introducing Python**
# Ch1. Py


```python
import os
os.getcwd()
```




    'C:\\Users\\user'




```python
quotes = {
"Moe": "A wise guy, huh?",
"Larry": "Ow!",
"Curly": "Nyuk nyuk!",
}
stooge = "Curly"
print(stooge, "says:", quotes[stooge])
```

    Curly says: Nyuk nyuk!
    


```python
import json # 표준 라이브러리에서 가져옴 
from urllib.request import urlopen, Request # 표준 urlib 라이브러리에서 urlopen함수만을 가져옴 
url = "https://gdata.youtube.com/feeds/api/standardfeeds/top_rated?alt=json" # 이 url대신 다른 것 이용
response = urlopen(url) # 웹 서버에 연결해서 특정 웹 서비스를 요청 
contents = response.read() # 응답데이터를 얻고 변수에 콘텐츠를 할당 
text = contents.decode('utf8') #  콘텐츠를 JSON 텍스트 문자열로 디코드한다.
data = json.loads(text) # 텍스트를 데이터(비디오에 관한 파이썬 데이터 구조)로 변환시킨다. 
for video in data['feed']['entry'][:6]: # 하나의 비디오에 관한 정보를 얻는다, two-level 파이썬 딕셔너리 사용 
    print(video['title'['$t']]) # 비디오의 제목을 출력
```


    ---------------------------------------------------------------------------

    HTTPError                                 Traceback (most recent call last)

    <ipython-input-6-e333a0fa8466> in <module>
          2 from urllib.request import urlopen, Request
          3 url = "https://gdata.youtube.com/feeds/api/standardfeeds/top_rated?alt=json"
    ----> 4 response = urlopen(url)
          5 contents = response.read()
          6 text = contents.decode('utf8')
    

    ~\Anaconda3\lib\urllib\request.py in urlopen(url, data, timeout, cafile, capath, cadefault, context)
        220     else:
        221         opener = _opener
    --> 222     return opener.open(url, data, timeout)
        223 
        224 def install_opener(opener):
    

    ~\Anaconda3\lib\urllib\request.py in open(self, fullurl, data, timeout)
        529         for processor in self.process_response.get(protocol, []):
        530             meth = getattr(processor, meth_name)
    --> 531             response = meth(req, response)
        532 
        533         return response
    

    ~\Anaconda3\lib\urllib\request.py in http_response(self, request, response)
        639         if not (200 <= code < 300):
        640             response = self.parent.error(
    --> 641                 'http', request, response, code, msg, hdrs)
        642 
        643         return response
    

    ~\Anaconda3\lib\urllib\request.py in error(self, proto, *args)
        567         if http_err:
        568             args = (dict, 'default', 'http_error_default') + orig_args
    --> 569             return self._call_chain(*args)
        570 
        571 # XXX probably also want an abstract factory that knows when it makes
    

    ~\Anaconda3\lib\urllib\request.py in _call_chain(self, chain, kind, meth_name, *args)
        501         for handler in handlers:
        502             func = getattr(handler, meth_name)
    --> 503             result = func(*args)
        504             if result is not None:
        505                 return result
    

    ~\Anaconda3\lib\urllib\request.py in http_error_default(self, req, fp, code, msg, hdrs)
        647 class HTTPDefaultErrorHandler(BaseHandler):
        648     def http_error_default(self, req, fp, code, msg, hdrs):
    --> 649         raise HTTPError(req.full_url, code, msg, hdrs, fp)
        650 
        651 class HTTPRedirectHandler(BaseHandler):
    

    HTTPError: HTTP Error 404: Not Found


decode: 부호로 변환된 데이터를 원래 형태로 변환하는 작업


```python
import requests 
url = "https://gdata.youtube.com/feeds/api/standardfeeds/top_rated?alt=json" # 이 url대신 다른 것 이용
response = requests.get(url)
data = response.json()
for video in data['feed']['entry'][:6]: 
    print(video['title'['$t']]) 
```

# Ch2. Py Ingredients: Numvers, Strings, and Variables

## bool

문자열, 리스트, 튜플, 딕셔너리 등의 값이 비어 있으면(" ", [ ], ( ), { }) 거짓이 된다. 당연히 비어있지 않으면 참이 된다. 숫자에서는 그 값이 0일 때 거짓이 된다. 

![image.png](attachment:image.png)


```python
a = [1, 2, 3, 4]
while a:
    print(a.pop())
```

    4
    3
    2
    1
    


```python
class Tree(object):
    def __init__(self, children):
        self.children

    def has_children(self):
        return bool(self.children)
```

## Numbers


```python
a = 7
b = a
print(
    type(a),
    type(b),
    type(58),
    type(99.9),
    type('abc'),
)
```

    <class 'int'> <class 'int'> <class 'int'> <class 'float'> <class 'str'>
    


```python
print(+123, -123)
```

    123 -123
    


```python
5 / 0
```


    ---------------------------------------------------------------------------

    ZeroDivisionError                         Traceback (most recent call last)

    <ipython-input-11-adafc2937013> in <module>
    ----> 1 5 / 0
    

    ZeroDivisionError: division by zero



```python
a = 95
a -= 3
print(a)
a += 8
print(a)
a /= 3
print(a)
a //= 11
print(a)
a *= 10
print(a)
print(a % 4)

divmod(9, 5) # 몫과 나머지를 함께 얻는다 divide, modulus연산. quotient 와 remainder를 얻는다.
             # 반환값의 타입이 튜플이다
```

    92
    100
    33.333333333333336
    3.0
    30.0
    2.0
    




    (1, 4)




```python
# float와 str도 가능
print(int(True), int(False))
print(int('99'), int('-23'), int('+12')) # 우와
print(True + 2)
```

    1 0
    99 -23 12
    3
    


```python
googol = 10**100
googol*googol # integer overflow를 발생시키지 않는다!
```




    100000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000



## Strings


```python
# double quotes & single quotes
"There's the man that shot my paw!' cied the limping bound"
```




    "There's the man that shot my paw!' cied the limping bound"




```python
# Triple quotes: multiline strings
poem = '''There was a Young Lady of Norway,
... Who casually sat in a doorway;
... When the door squeezed her flat,
... She exclaimed, "What of that?"
... This courageous Young Lady of Norway.'''
print(poem)
```

    There was a Young Lady of Norway,
    ... Who casually sat in a doorway;
    ... When the door squeezed her flat,
    ... She exclaimed, "What of that?"
    ... This courageous Young Lady of Norway.
    


```python
poem2 = '''I do not like thee, Doctor Fell.
... The reason why, I cannot tell.
... But this I know, and know full well:
... I do not like thee, Doctor Fell.
... '''
print(poem2)
poem2 # 둘의 차이점을 보라
```

    I do not like thee, Doctor Fell.
    The reason why, I cannot tell.
    But this I know, and know full well:
    I do not like thee, Doctor Fell.
    
    




    'I do not like thee, Doctor Fell.\nThe reason why, I cannot tell.\nBut this I know, and know full well:\nI do not like thee, Doctor Fell.\n'




```python
# empyty string
print('', "", '''''', """""") # empty string 출력(아무것도 보이지않지만 출력이 된 것)
'' # print했을 때와 다른 결과 
```

       
    




    ''



다른 문자열로부터 문자열을 만들고 싶을 때 blank slate와 함께 시작할 필요가 있다.


```python
bottles = 99
base = '' # 이거 없어도 되는데.. 
base = 'current inventory: '
base += str(bottles)
base
```




    'current inventory: 99'




```python
testimony = "\"I did nothing!\" he said. \"Not that either! Or the otherthing.\""
print(testimony)
speech = 'Today we honor our friend, the backslash:\\.'
print(speech)
```

    "I did nothing!" he said. "Not that either! Or the otherthing."
    Today we honor our friend, the backslash:\.
    


```python
# 차이점 주목
print('Relases the kraken!' + 'At once!')
print('Relases the kraken!', 'At once!')
'Relases the kraken!' + 'At once!'
```

    Relases the kraken!At once!
    Relases the kraken! At once!
    




    'Relases the kraken!At once!'




```python
"My word! " "A gentleman caller!" # literal string을 합칠 수 있다
```




    'My word! A gentleman caller!'




```python
start = 'Na' * 4 + '\n'
middle = 'Hey' * 3 + '\n'
end = 'Goodbye'
print(start + start + middle + end)
```

    NaNaNaNa
    NaNaNaNa
    HeyHeyHey
    Goodbye
    

python의 인덱스는 position이 아니라 offset


```python
letters = 'abcdefghijklmnopqrstuvwxyz'
print(letters[0], letters[1], letters[-1])
```

    a b z
    


```python
# 문자열은 immutable
name = 'Henny'
name[0] = 'P'
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-60-6e2d3e91fcdb> in <module>
          1 # 문자열은 immutable
          2 name = 'Henny'
    ----> 3 name[0] = 'P'
    

    TypeError: 'str' object does not support item assignment



```python
name = 'Henny'
name.replace('H', 'P')
print(name) # replace를 사용하든가 
print('P' + name[1:]) # slice를 사용한든가 
```

    Henny
    Penny
    

### Slice with [start : end : step]

- [:] 처음부터 끝까지 전체 시퀀스를 추출한다.
- [start:] start오프셋부터 끝까지 시퀀스를 추출한다.
- [:end] 처음부터 (end - 1) 오프셋까지 시퀀스를 추출한다.
- [start:end] start 오프셋부터 (end - 1) 오프셋까지 시퀀스를 추출한다.
- [start:end:step] step만큼 문자를 건너뛰면서, start 오프셋부터 (end - 1)오프셋까지 시퀀스를 추출한다.


```python
letters[:]
```




    'abcdefghijklmnopqrstuvwxyz'




```python
letters[20:]
```




    'uvwxyz'




```python
letters[12:15] # 마지막 오프셋은 포함하지 않는다.
```




    'mno'




```python
letters[-3:]
```




    'xyz'




```python
letters[18:-3]
```




    'stuvw'




```python
letters[4:20:3]
```




    'ehknqt'




```python
letters[19::4]
```




    'tx'




```python
letters[:21:5]
```




    'afkpu'




```python
letters[::-1]
```




    'zyxwvutsrqponmlkjihgfedcba'



첫 번째 문자열 이전의 슬라이스 오프셋은 0으로 간주하고, 마지막의 다음 오프셋은 -1로 간주한다.


```python
letters[-50:]
```


```python
letters[-51:-50]
```


```python
letters[:70]
```




    'abcdefghijklmnopqrstuvwxyz'




```python
letters[70:71]
```




    ''




```python
len("")
```




    0




```python
todos = 'get gloves,get mask,give cat vitamins,call ambulance'
todos.split(',')
```




    ['get gloves', 'get mask', 'give cat vitamins', 'call ambulance']




```python
# 구분자를 지정하지 않으면 split()는 문자열에 등장하는 공백 문자(줄바꿈, 스페이스, 탭)를 사용한다.
todos.split()
```




    ['get', 'gloves,get', 'mask,give', 'cat', 'vitamins,call', 'ambulance']



### Combine with join()

먼저 결합할 문자열을 지정한 다음에 문자열 리스트를 결합한다. 마치 split()함수를 역행하는 것 처럼 보인다. join()함수는 문자열 리스트를 `string.join(list)`형태로 결합한다. 그러므로 lines리스트를 각각 줄바꿈하여 하나의 문자열로 결합하기 위해서는 `\n'.join(lines)`를 입력한다. 다음 예제는 리스트를 콤마와 스페이스로 구분하여 하나의 문자열로 결합한다.


```python
crypto_list = ['Yeti', 'Bigfoot', 'Loch Ness Monster']
crypto_string = ', '.join(crypto_list)
print(crypto_string)
```

    Yeti, Bigfoot, Loch Ness Monster
    


```python
poem = '''All that doth flow we cannot liquid name
Or else would fire and water be the same;
But that is liquid which is moist and wet
Fire that property can never get.
Then 'tis not cold that doth the fire put out
But 'tis the wet that makes it die, no doubt.'''
```


```python
# 스페이스와 줄바꿈을 포함해서
len(poem)
```




    250




```python
#이 시는 all로 시작하는가?
poem.startswith('All')
```




    True




```python
#이 시는 That's all, folks!로 끝나는가?
poem.endswith('That\'s all, folks!')
```




    False




```python
# 이 시에서 첫 번째로 the가 나오는 오프셋은?
word = 'the'
poem.find(word)
```




    73




```python
# 뒤에서 부터 the가 나오는 오프셋은?
poem.rfind(word)
```




    214




```python
poem.count(word)
```




    3




```python
# 글자와 숫자로만 이루어져 있는가?
poem.isalnum() 
```

### Case and Alighnment(대소문자와 배치)


```python
setup = 'a duck goes into a bar...'
setup.strip('.')
```




    'a duck goes into a bar'



--- 
**NOTE**

문자열은 불변하기 때문에 다음의 어느 예제에서도 setup 문자열을 바꿀 수 없다. 각 예제는 단지 값을 설정하고, 함수를 수행한 뒤, 새로운 문자열로 결과를 반환한다.

---


```python
# 첫 번째 단어를 대문자로 
print(setup.capitalize())

# 모든 단어의 첫 글자를 대문자로 
print(setup.title())

# 모두 대문자로
print(setup.upper())

# 모두 소문자로
print(setup.lower())

# 대문자는 소문자로, 소문자는 대문자로
print(setup.swapcase())
```

    A duck goes into a bar...
    A Duck Goes Into A Bar...
    A DUCK GOES INTO A BAR...
    a duck goes into a bar...
    A DUCK GOES INTO A BAR...
    


```python
print(setup.center(30))
print(setup.ljust(30))
print(setup.rjust(30))
```

      a duck goes into a bar...   
    a duck goes into a bar...     
         a duck goes into a bar...
    

### Substitue with replace()


```python
print(setup.replace('duck', 'marmoset'))

# 100회까지 바꾼다.
print(setup.replace('a ', 'a famous ', 100))
```

    a marmoset goes into a bar...
    a famous duck goes into a famous bar...
    

# Ch3. Py Filing: Lists, Tuples, Dictionaries, and Sets

## LIsts and Tuples

파이썬의 문자열은 문자의 시퀀스이다. 리스트는 모든 것의 시퀀스라는 것을 알게 될 것이다. 파이썬에는 두 가지 다른 시퀀스 구조가 있다. tuple과 list이다. 이 구조에는 0 혹은 그 이상의 항목이 포함되어 있다. 문자열과는 달리, 이들 항목은 다른 타입이 될 수 있다. 다시 말해 각 요소는 어떤 객체도 될 수 있다. 튜플은 immutable, 리스트는 mutable하다. 

## Lists

리스트는 데이터를 순차적으로 파악하는 데 유용하다. 특히 내용의 순서가 바뀔 수 있다는 점에서 유용하다. 문자열과 달리 리스트는 변경 가능하다. 리스트의 현재 위치에서 새로운 요소를 추가하거나 삭제 혹은 기존 요소를 덮어쓸 수 있다. 그리고 리스트에는 동일한 값이 여러 번 나타날 수 있다.


```python
empty_list = []
another_empty_list = list()
```


```python
weekdays = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday']
big_birds = ['emu', 'ostrich', 'cassowary']
first_names = ['Graham', 'John', 'Terry', 'Terry', 'Michael']
```

weekdays 리스트는 리스트의 순서를 가장 잘 활요할 수 있는 예제 중 하나다. 그리고 first_name 리스트는 값이 유일할 필요가 없다는 것을 보여준다.

--- 

**NOTE**

요소들이 순서는 상관없이 유일한 값으로만 유지될 필요가 있다면, 리스트보다 set을 사용하는 것이 더 좋다. 이전 예제에서 big_birds를 셋으로 쓸 수 있다. 

---

### Convert Other Data Types to Lists with list()

list()함수는 다른 데이터 타입을 리스트로 변환한다.


```python
list('cat')
```




    ['c', 'a', 't']




```python
a_tuple = ('ready', 'fire', 'aim')
list(a_tuple)
```




    ['ready', 'fire', 'aim']




```python
birthday = '1/21/1995'
birthday.split('/')
```




    ['1', '21', '1995']




```python
# 문자열에 구분자가 여러 개 있다면 어떻게 처리할까? 다음과 같이 리스트에 빈 문자열이 들어갈 것이다.
splitime = 'a/b//c/d///e'
splitime.split('/')
```




    ['a', 'b', '', 'c', 'd', '', '', 'e']




```python
splitime.split('//')
```




    ['a/b', 'c/d', '/e']



### Get an Item by Using [offset]

문자열과 마찬가지로 리스트는 오프셋으로 하나의 특정 값을 추출할 수 있다.

--- 

**NOTE**

리스트의 오프셋은 값을 할당한 위치에 맞게 입력되어야 한다. 오프셋의 위치가 리스트의 범위를 벗어날 경우 에러가 발생한다. 다섯 형제의 marxes(유효한 오프셋의 범위:0 ~ 4)에서 6번째 오프셋을 입력하거나, 끝에서 5번째 오프셋을 입력하면 무슨 일이 일어나는지 살펴보자.

![image.png](attachment:image.png)

---

### Lists of Lists

리스트는 다음과 같이 리스트뿐만 아니라 다른 타입의 요소도 포함할 수 있다. 


```python
small_birds = ['hummingbird', 'finch']
extinct_birds = ['dodo', 'passenger pigeon', 'Norwegian Blue']
carol_birds = [3, 'French hens', 2, 'turtledoves']
all_birds = [small_birds, extinct_birds, 'macaw', carol_birds]
all_birds
```




    [['hummingbird', 'finch'],
     ['dodo', 'passenger pigeon', 'Norwegian Blue'],
     'macaw',
     [3, 'French hens', 2, 'turtledoves']]




```python
all_birds[0]
```




    ['hummingbird', 'finch']




```python
all_birds[1][0]
```




    'dodo'



### Change an Item by [offset]


```python
marxes = ['Groucho', 'Chico', 'Harpo']
marxes[2] = 'Wanda'
marxes[::-2]
marxes.append("Zeppo")
```

### Combine Lists by Using extend() or +=

`extend()`를 사용하여 다른 리스트를 병합할 수 있다. 리스트 marxes에 새로운 리스트 others를 병합해보자.


```python
others = ['Gummo', 'Karl']
marxes.extend(others)
marxes
```




    ['Groucho', 'Chico', 'Wanda', 'Zeppo', 'Gummo', 'Karl']




```python
marxes = ['Groucho', 'Chico', 'Harpo', 'Zeppo']
others = ['Gummo', 'Karl']
marxes += others
marxes
```




    ['Groucho', 'Chico', 'Harpo', 'Zeppo', 'Gummo', 'Karl']




```python
marxes = ['Groucho', 'Chico', 'Harpo', 'Zeppo']
others = ['Gummo', 'Karl']
marxes.append(others) # append()를 사용하면 항목을 병합하지 않고, others가 하나의 리스트로 추가된다.
marxes
```

이것은 리스트가 다른 타입의 요소를 포함할 수 있다는 것을 보여준다. marxes에는 4개의 문자열과 두 문자열의 리스트가 존재한다.

### Add an Item by Offset with insert()

`append()`함수는 단지 리스트의 끝에 항목을 추가한다. 그러나 `insert()`함수는 원하는 위치에 항목을 추가할 수 있다. 오프셋 0은 시작 지점에 항목을 삽입한다. 리스트의 끝을 넘는 오프셋은 append()처럼 끝에 항목을 추가한다.


```python
marxes.insert(3, 'Gummo')
marxes
```




    ['Groucho', 'Chico', 'Harpo', 'Gummo', 'Zeppo', ['Gummo', 'Karl']]




```python
marxes.insert(10, 'Karl')
marxes
```




    ['Groucho',
     'Chico',
     'Harpo',
     'Gummo',
     'Zeppo',
     ['Gummo', 'Karl'],
     'Karl',
     'Karl']



### Delete an Item 


```python
del marxes[-1]
marxes
```




    ['Groucho', 'Chico', 'Harpo', 'Gummo', 'Zeppo', ['Gummo', 'Karl'], 'Karl']



---

**NOTE**

del은 리스트 함수가 아니라 python statement이다. 즉, `marxes[-2].del()`를 수행할 수 없다. del은 할당(=)의 반대다. 이것은 객체로부터 이름을 분리하고, (이 이름이 객체의 마지막 참조일 경우) 객체의 메모리를 비워준다.

---


```python
marxes = ['Groucho', 'Chico', 'Harpo', 'Gummo', 'Zeppo']
marxes.remove('Gummo')
```


```python
marxes
```




    ['Groucho', 'Chico', 'Harpo', 'Zeppo']



`pop()`은 리스트에서 항목을 가져오는 동시에 그 항목을 삭제한다. 오프셋과 함께 pop()을 호출했다면 그 오프셋의 항목이 반환된다. 인자가 없다면 -1을 사용한다. pop(0)은 리스트의 head를 반환한다. 그리고 pop() 혹은 pop(-1)은 리스트의 tail을 반환한다.


```python
marxes = ['Groucho', 'Chico', 'Harpo', 'Zeppo']
marxes.pop()
print(marxes)
marxes.pop(1)
print(marxes)
```

    ['Groucho', 'Chico', 'Harpo']
    ['Groucho', 'Harpo']
    

---

**NOTE**

컴퓨터 용어를 이야기해보자. 만약 append()로 새로운 항목을 끝에 추가한 뒤 `pop()`으로 다시 마지막 항목을 제거했다면, 여러분은 **LIFO**(Last In First Out)자료구조인 stack을 구현한 것이다. 그리고 `pop(0)`을 사용했다면 **FIFO**(First In First Out)자료구조인 queue를 구현한 것이다. 이들은 수집한 데이터에서 가장 오래된 것을 먼저 사용하거나(FIFO), 최근 것을 먼저 사용할 때(LIFO)유용하다.

---


```python
marxes.index('Harpo')
```




    1



### Test for a Value with in


```python
words = ['a', 'deer', 'a', 'female', 'deer']
'deer' in words
```




    True




```python
['female', 'a'] in words
```




    False




```python
words.count('a')
```




    2



### Convert to a String with join()


```python
marxes = ['Groucho', 'Chico', 'Harpo']
', '.join(marxes)
```




    'Groucho, Chico, Harpo'



이 예제는 앞에서 본 것과 다를 게 없는 것 같다.`join()`은 문자열 메서드지, 리스트 메서드가 아니다. `marxes.join(', ')`이 조금 더 직관적으로 보일지라도, 이렇게 사용할 수 없다. `join()`의 인자는 문자열 혹은 iterable 문자열의 시퀀스(리스트 포함)다. 그리고 결과로 반환되는 값은 문자열이다. `join()`이 리스트 메서드였다면, 튜플과 문자열 같은 다른 반복 가능한 객체와 함께 사용하지 못할 것이다. 어떤 반복 가능한 타입을 처리하기 위해 각 타입을 실제로 병합할 수 있도록 특별한 코드가 필요했다. 다음 예제와 같이 **join()을 split()의 반대라고 생각**하면 기억하기 쉬울 것이다.


```python
friends = ['Harry', 'Hermione', 'Ron']
joined = '*'.join(friends)
print(joined)
separated = joined.split('*')
print(separated)
separated == friends
```

    Harry*Hermione*Ron
    ['Harry', 'Hermione', 'Ron']
    




    True



### Reorder Items with sort()

오프셋을 이용하여 리스트를 정렬할 때도 있지만, 값을 이용하여 리스트를 정렬할 때도 있다. 파이썬은 두 가지 함수를 제공한다.

- `sort()`는 리스트 자체를 내부적으로 정렬한다.
- `sorted()`는 리스트의 정렬된 복사본을 반환한다.


```python
# 리스트의 항목이 숫자인 경우, 기본적으로 오름차순으로 정렬한다. 문자열인 경우, 알파벳순으로 정렬한다.
numbers = [2, 1, 4.0, 3]
numbers.sort(reverse=True)
numbers
print(len(numbers))
```

    4
    

### Assign with = , Copy with copy()

다음 예제와 같이 한 리스트를 변수 두 곳에 할당했을 경우, 한 리스트를 변경하면 다른 리스트도 따라서 변경된다.


```python
a = [1, 2, 3]
b = a
a[0] = 'surprise'
print(a, b) # 함께 변경되었다!
```

    ['surprise', 2, 3] ['surprise', 2, 3]
    

b는 단지 같은 리스트 객체의 a를 참조한다. 그러므로 a 혹은 b 리스트의 내용을 변경하면 두 변수 모두에 반영된다.

다음과 같은 방법을 이용하여 한 리스트를 새로운 리스트로 **복사**할 수 있다.

- `copy()`함수
- `list()`변환 함수
- 슬라이스`[:]`

원본 리스트 a가 있다. `copy()`로 리스트 b를 만든다. `list()`변환 함수로 리스트 c를 만든다. 그리고 a를 슬라이스해서 리스트 d를 만든다.


```python
a = [1, 2, 3]
b = a.copy()
c = list(a)
d = a[:]
```

b, c, d는 a의 **복사본**이다. 이들은 자신만의 값을 가진 새로운 객체다. 그리고 원본 리스트 객체 `[1,2,3]`을 참조하는 아무론 참조가 없다. 복사본 b,c,d를 바꾸더라도 원본 a에는 아무런 영향을 주지 않는다.


```python
a[0] = 'integer lists are boring'
print(a, b, c, d)
```

    ['integer lists are boring', 2, 3] [1, 2, 3] [1, 2, 3] [1, 2, 3]
    

## Tuples


```python
empty_tuple = ()
empty_tuple
```




    ()




```python
marx_tuple = ('Groucho', 'Chico', 'Harpo')
a, b, c = marx_tuple
print(a, b, c)
```

    Groucho Chico Harpo
    

이것을 **Tuple unpacking**이라고 한다.

한 문장에서 값을 교환하기 위해 임시 변수를 사용하지 않고 튜플을 사용할 수 있다.


```python
password = 'swordfish'
icecream = 'tuttifrutti'
password, icecream = icecream, password
print(password, icecream)
```

    tuttifrutti swordfish
    


```python
marx_list = ['Groucho', 'Chico', 'Harpo']
tuple(marx_list)
```

### Tuples versus Lists
튜플은 `append()`, `insert()`등과 같은 함수가 없고, 함수의 수가 매우 적다. 튜플을 사용하는 이유는 다음과 같다.

- 튜플은 더 적은 공간을 사용한다.
- 실수로 튜플의 항목이 손상될 염려가 없다.
- 튜플을 딕셔너리 키로 사용할 수 있다.
- **named tuple**은 객체의 단순한 대안이 될 수 있다.
- 함수의 인자들은 튜플로 전달된다.

## Dictionaries

딕셔너리는 리스트와 비슷하다. 다른 점은 항목의 순서를 따지지 않으며, 0또는 1과 같은 오프셋으로 항목을 선택할 수 없다. 대신 value에 대응하는 고유한 key를 지정한다. 이 키는 대부분 문자열이지만, 불변하는 파이썬의 어떤 타입이 될 수도 있다. 딕셔너리는 변경 가능하므로 키-값 요소를 추가, 삭제, 수정할 수 있다.

---

**NOTE**

다른 언어에서는 딕셔너리를 associate array, hash, hashmap이라고 부른다. 또한 파이썬에서는 딕셔너리를 dict라고도 부른다.

---


```python
bierce = {
"day": "A period of twenty-four hours, mostly misspent",
"positive": "Mistaken at the top of one's voice", 
    "misfortune": "The kind of fortune that never misses"}
bierce
```




    {'day': 'A period of twenty-four hours, mostly misspent',
     'positive': "Mistaken at the top of one's voice",
     'misfortune': 'The kind of fortune that never misses'}




```python
lol = [ ['a', 'b'], ['c', 'd'], ['e', 'f'] ]
dict(lol)
```




    {'a': 'b', 'c': 'd', 'e': 'f'}




```python
lot = [ ('a', 'b'), ('c', 'd'), ('e', 'f') ]
dict(lot)
```




    {'a': 'b', 'c': 'd', 'e': 'f'}




```python
# 문자열로 된 리스트
los = [ 'ab', 'cd', 'ef' ]
dict(los)
```




    {'a': 'b', 'c': 'd', 'e': 'f'}




```python
# 문자열로 된 튜플
tos = ( 'ab', 'cd', 'ef' )
dict(tos)
```




    {'a': 'b', 'c': 'd', 'e': 'f'}



### Add or Change an Item by [key]

키가 딕셔너리에 이미 존재하는 경우, 그 값은 새값으로 존재하지 않는 경우, 새 값이 키와 사전에 추가된다. 리스트와 달리 딕셔너리를 할당할 때는 인덱스의 범위 지정이 벗어났다는 예외에 대해 걱정할 필요가 없다


```python
pythons = {
    'Chapman': 'Graham',
    'Cleese': 'John',
    'Idle': 'Eric',
    'Jones': 'Terry',
    'Palin': 'Michael',
}
pythons['Gilliam'] = 'Gerry'
print(pythons)

pythons['Gilliam'] = 'Terry'
print(pythons)
```

    {'Chapman': 'Graham', 'Cleese': 'John', 'Idle': 'Eric', 'Jones': 'Terry', 'Palin': 'Michael', 'Gilliam': 'Gerry'}
    {'Chapman': 'Graham', 'Cleese': 'John', 'Idle': 'Eric', 'Jones': 'Terry', 'Palin': 'Michael', 'Gilliam': 'Terry'}
    

딕셔너리의 키들은 반드시 유일해야 한다. 그래서 성 대신에 이름을 키로 사용했다. 몬티 파이튼의 두 멤버의 성이 'Terry'이다. 만약 같은 키를 두 번 이상 사용하면 마지막 값이 승리한다.


```python
some_pythons = {
    'Graham': 'Chapman',
    'John': 'Cleese',
    'Eric': 'Idle',
    'Terry': 'Gilliam',
    'Michael': 'Palin',
    'Terry': 'Jones',
}
some_pythons
```




    {'Graham': 'Chapman',
     'John': 'Cleese',
     'Eric': 'Idle',
     'Terry': 'Jones',
     'Michael': 'Palin'}



### Combine Dictionaries with update()

`update()`함수는 한 딕셔너리의 키와 값들을 복사해서 다른 딕셔너리에 붙여준다.


```python
pythons = {
    'Chapman': 'Graham',
    'Cleese': 'John',
    'Gilliam': 'Terry',
    'Idle': 'Eric',
    'Jones': 'Terry',
    'Palin': 'Michael',
}
others = { 'Marx': 'Groucho', 'Howard': 'Moe' }
pythons.update(others)
pythons
```




    {'Chapman': 'Graham',
     'Cleese': 'John',
     'Gilliam': 'Terry',
     'Idle': 'Eric',
     'Jones': 'Terry',
     'Palin': 'Michael',
     'Marx': 'Groucho',
     'Howard': 'Moe'}




```python
# 같은 key라면 두 번째 딕셔너리에 있는 값이 승리
first = {'a': 1, 'b': 2}
second = {'b': 'platypus'}
first.update(second)
first
```




    {'a': 1, 'b': 'platypus'}




```python
del pythons['Marx']
# 모든 항목 삭제하기 
pythons.clear() 
pythons

# 아예 빈 딕셔너리를 할당한다
pythons = {}
pythons
```


    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    <ipython-input-108-0794f5a1ee30> in <module>()
    ----> 1 del pythons['Marx']
          2 # 모든 항목 삭제하기
          3 pythons.clear()
          4 pythons
          5 
    

    KeyError: 'Marx'



```python
pythons = {'Chapman': 'Graham', 'Cleese': 'John',
'Jones': 'Terry', 'Palin': 'Michael'}
'Chapman' in pythons
```




    True



### Get an Item


```python
pythons['Cleese']
```




    'John'




```python
print(pythons.get('Cleese'), pythons.get('Hong'), pythons.get('Hong', '없다')) # 키가 존재하면 값을 얻고 없으면 Nonr
# None은 print가 아니고 그냥 출력하면 아무것도 나타나지 않음
```

    John None 없다
    


```python
signals = {'green': 'go', 'yellow': 'go faster', 'red':
'smile for the camera'}
signals.keys() # iterable keys
```




    dict_keys(['green', 'yellow', 'red'])




```python
list(signals.keys())
```




    ['green', 'yellow', 'red']




```python
signals.values()
```




    dict_values(['go', 'go faster', 'smile for the camera'])




```python
signals.items()
```




    dict_items([('green', 'go'), ('yellow', 'go faster'), ('red', 'smile for the camera')])



### Assign with =, Copy with copy()


```python
signals = {'green': 'go', 'yellow': 'go faster', 'red':
'smile for the camera'}
save_signals = signals
signals['blue'] = 'confuse everyone'
save_signals
```




    {'green': 'go',
     'yellow': 'go faster',
     'red': 'smile for the camera',
     'blue': 'confuse everyone'}




```python
signals = {'green': 'go', 'yellow': 'go faster', 'red':
'smile for the camera'}
original_signals = signals.copy()
signals['blue'] = 'confuse everyone'
print(signals, original_signals)
```

    {'green': 'go', 'yellow': 'go faster', 'red': 'smile for the camera', 'blue': 'confuse everyone'} {'green': 'go', 'yellow': 'go faster', 'red': 'smile for the camera'}
    

## Sets

A set is a collection of hashables. Even though the statement `1 is True` is `False`, the statement `1 == True` is `True`. Because of that, they have the same hash value and cannot exist separately in a set, and you cannot keep them both in a set
set은 값은 버리고 키만 남은 딕셔너리와 같다. 딕셔너리와 마찬가지로 각 키는 유일해야 한다. 어떤 것이 존재하는지 여부만 판단하기 위해서는 셋을 사용한다. 그리고 키에 어떤 정보를 첨부해서 그 결과를 얻고 싶으면 딕셔너리를 사용한다. 


```python
{True, 1, 2, "악", 0, False}
```




    {0, 2, True, '악'}




```python
empty_set = set()
even_numbers = {0, 2, 4, 6, 8}
odd_numbers = {1, 3, 5, 7, 9}
empty_set
```




    set()



딕셔너리의 키와 마찬가지로 셋은 순서가 없다.

--- 

**NOTE**

`[]`가  빈 리스트를 생성하기 때문에 `{}`도 빈 셋을 생성할 것이라고 추측했을 것이다. 그러나 `{}`는 빈 딕셔너리를 생성한다. 이것이 인터프리터가 빈 셋을 `{}`대신 `set()`으로 출력하는 이유이기도 하다. 왜 그럴까? 파이썬에서 딕셔너리가 먼저 등장해서 중괄호를 이미 차지하고 있었기 때문이다.

---

### Convert from Other Data Types with set()

리스트, 문자열, 튜플, 딕셔너리로부터 중복된 값을 버린 셋을 생성할 수 있다.
먼저 'letters'의 알파벳 중 하나 이상 나온 문자를 살펴보자.


```python
set('letters')
```




    {'e', 'l', 'r', 's', 't'}




```python
set( ['Dasher', 'Dancer', 'Prancer', 'Mason-Dixon'] )
```




    {'Dancer', 'Dasher', 'Mason-Dixon', 'Prancer'}




```python
set( ('Ummagumma', 'Echoes', 'Atom Heart Mother') )
```




    {'Atom Heart Mother', 'Echoes', 'Ummagumma'}




```python
set( {'apple': 'red', 'orange': 'orange', 'cherry': 'red'} )
```




    {'apple', 'cherry', 'orange'}



### Test for Value by Using in

이것은 일반적으로 사용되는 셋의 용도다. drinks라는 딕셔너리를 만들어보자. 각 키는 혼합음료의 이름이고, 값은 음료에 필요한 재료들의 셋이다.


```python
drinks = {
    'martini': {'vodka', 'vermouth'},
    'black russian': {'vodka', 'kahlua'},
    'white russian': {'cream', 'kahlua', 'vodka'},
    'manhattan': {'rye', 'vermouth', 'bitters'},
    'screwdriver': {'orange juice', 'vodka'}
}
for name, contents in drinks.items():
    if 'vodka' in contents:
        print(name)
print()
        
for name, contents in drinks.items():
    if 'vodka' in contents and not ('vermouth' in contents or 'cream' in contents):
        print(name)
```

    martini
    black russian
    white russian
    screwdriver
    
    black russian
    screwdriver
    

### Combinations and Operators

셋값의 combination을 어떻게 확인할까? 오렌주 주스 혹은 vermouth가 들어 있는 음료를 마시고 싶다고 가정해보자. set intersection operator인 &를 사용해서 우리가 원하는 음료를 찾아보자.


```python
for name, contents in drinks.items():
    if contents & {'vermouth', 'orange juice'}:
        print(name)
print()

for name, contents in drinks.items():
    if 'vodka' in contents and not contents & {'vermouth', 'cream'}:
        print(name)
```

    martini
    manhattan
    screwdriver
    
    black russian
    screwdriver
    


```python
bruss = drinks['black russian']
wruss = drinks['white russian']
```


```python
a = {1, 2}
b = {2, 3}
print(a & b, a.intersection(b))
```

    {2} {2}
    


```python
bruss & wruss
```




    {'kahlua', 'vodka'}




```python
print(a | b, a.union(b))
print(bruss | wruss)
```

    {1, 2, 3} {1, 2, 3}
    {'vodka', 'kahlua', 'cream'}
    


```python
print(a - b, a.difference(b))
```

    {1} {1}
    


```python
print(a ^ b, a.symmetric_difference(b)) # exclusive대칭 차집합
```

    {1, 3} {1, 3}
    


```python
print(a <= b, a.issubset(b)) # 부분집합
```

    False False
    


```python
bruss <= wruss
```




    True




```python
# proper subset(진부분집합)
print(a < b, a < a, bruss < wruss)
```

    False False True
    


```python
# superset은 subset의 반대
print(a >= b, a.issuperset(b), wruss >= bruss)
print(a > b, wruss > bruss, a > a) # 모든 셋은 자신의 proper subset이 될 수 없다.
```

    False False True
    

## Compare Data Structures


```python
marx_list = ['Groucho', 'Chico', 'Harpo']
marx_tuple = 'Groucho', 'Chico', 'Harpo'
marx_dict = {'Groucho': 'banjo', 'Chico': 'piano', 'Harpo': 'harp'}

print(marx_list[2], marx_tuple[2], marx_dict['Harpo'])
```

    Harpo Harpo harp
    

## Make Bigger Data Structures


```python
marxes = ['Groucho', 'Chico', 'Harpo']
pythons = ['Chapman', 'Cleese', 'Gilliam', 'Jones', 'Palin']
stooges = ['Moe', 'Curly', 'Larry']
tuple_of_lists = marxes, pythons, stooges
list_of_lists = [marxes, pythons, stooges]

dict_of_lists = {'Marxes': marxes, 'Pythons': pythons,
'Stooges': stooges}
dict_of_lists
```




    {'Marxes': ['Groucho', 'Chico', 'Harpo'],
     'Pythons': ['Chapman', 'Cleese', 'Gilliam', 'Jones', 'Palin'],
     'Stooges': ['Moe', 'Curly', 'Larry']}



제한사항이 있다면 데이터 타입 그 자체다. 예를 들어 딕셔너리의 키는 불변하기 때문에 리스트, 딕셔너리, 셋은 다른 딕셔너리의 키가 될 수 없다. 그러나 튜플은 딕셔너리의 키가 될 수 있다. 예를 들어 관심 있는 장소를 GPS좌표(latitude, longitude, altitude)로 인덱싱할 수 있다.


```python
houses = {
    (44.79, -93.14, 285): 'My House',
    (38.89, -77.03, 13): 'The White House'
}
houses
```




    {(44.79, -93.14, 285): 'My House', (38.89, -77.03, 13): 'The White House'}



## Exercises

3.9


```python
surprise = ["Groucho", "Chico", "Harpo"]
surprise[-1] = surprise[-1].lower()[::-1].capitalize()
surprise
```




    ['Groucho', 'Chico', 'Oprah']



3.10 ~ 3.14


```python
e2f = {
    'dog': 'chien',
    'cat': 'chat',
    'walrus': 'morse'
}
e2f
```




    {'dog': 'chien', 'cat': 'chat', 'walrus': 'morse'}




```python
e2f['walrus']
```




    'morse'




```python
f2e = {}
for eng, fra in e2f.items():
    f2e[fra] = eng
print(f2e)
```

    {'chien': 'dog', 'chat': 'cat', 'morse': 'walrus'}
    


```python
print(set(e2f.keys()))
```

    {'dog', 'cat', 'walrus'}
    

3.15 ~ 3.18


```python
life = {
    'animals': {'cats':['Henri', 'Grumpy', 'Lucy'],
                'octopi':{}, # 빈 딕셔너리가 아니라 빈 리스트나 튜플이여도 됨  
                'emus':{}
               },
    'plants':{},
    'other':{}
}
```


```python
life.keys()
```




    dict_keys(['animals', 'plants', 'other'])




```python
life['animals'].keys()
```




    dict_keys(['cats', 'octopi', 'emus'])




```python
life['animals']['cats']
```




    ['Henri', 'Grumpy', 'Lucy']



# Ch4. Py Crust: Code Structures

## Continue Lines with \

한 라인에서 권장하는 최대 문자수는 80자다. 이 길이 안에 넣고 싶은 코드를 모두 입력할 수 없다면 \문자를 입력한 후 다음 라인에 계속 입력한다. 라인의 끝에 \를 입력하면, 파이썬은 다음 라인을 여전히 같은 라인으로 인식한다.


```python
alphabet = ''
alphabet += 'abcdefg'
alphabet += 'hijklmnop'
alphabet += 'qrstuv'
alphabet += 'wxyz'

alphabet = 'abcdefg' + \
        'hijklmnop'+ \
        'qrstuv'+ \
        'wxyz'
print(alphabet)
```

    abcdefghijklmnopqrstuvwxyz
    

## Compare with if, elif, and else


```python
while True:
    value = input("Integer, please [q to quit]: ")
    if value == 'q':
        break
    number = int(value)
    if number % 2 == 0:
        continue
    print(number, "squared is", number*number)
```

### Check break Use with else

break는 어떤 것을 체크하여 그것을 발견했을 경우 종료하는 while문을 작성할 때 사용한다. while문이 모두 실행되었지만 발견하지 못했을 경우에만 else가 실행된다.


```python
numbers = [1, 3, 5]
position = 0
while position < len(numbers):
    number = numbers[position]
    if number % 2 == 0:
        print('Found even number', number)
        break
    position += 1
else:
    print('No even number found')
```

## Iterate with for

파이썬에서 iterator는 자주 유용하게 쓰인다. 자료구조가 얼마나 큰지, 어떻게 구현되었는지에 관계없이 자료구조를 순회할 수 있도록 해준다. 심지어 바로 생성되는 데이터도 순회할 수 있다. 데이터가 메모리에 맞지 않더라도 데이터 스트림을 처리할 수 있도록 허용해준다. 

딕셔너리의 iterating은 키를 반환한다. 값을 순회하려면 `values()`를 키와 값을 모두 반환하려면 `items()`를 사용한다.


```python
accusation = {'room': 'ballroom', 'weapon': 'lead pipe',
'person': 'Col. Mustard'}
for card in accusation:
    print(card)
```

    room
    weapon
    person
    

한 번에 튜플 하나씩 할당할 수 있다는 것을 명심하라.


```python
for card, contents in accusation.items():
    print('Card', card, 'has the contents', contents)
```

    Card room has the contents ballroom
    Card weapon has the contents lead pipe
    Card person has the contents Col. Mustard
    

### Check break Use with else

while문과 같이, for문에서도 모든 항목을 순회했는지 확인하는 부가적인 옵션의 else문이 있다. for 문에서 break문이 호출되지 않으면 else문이 실행된다. 즉, else문은 break문에 의해 반복문이 중단되었는지 확인한다.

### Iterate Multiple Sequences with zip()

zip()을 사용해서 여러 시퀀스를 병렬로 순회하는 것이다.


```python
days = ['Monday', 'Tuesday', 'Wednesday']
fruits = ['banana', 'orange', 'peach']
drinks = ['coffee', 'tea', 'beer']
desserts = ['tiramisu', 'ice cream', 'pie', 'pudding']
for day, fruit, drink, dessert in zip(days, fruits, drinks, desserts):
    print(day, ": drink", drink, "- eat", fruit, "- enjoy", dessert)
```

    Monday : drink coffee - eat banana - enjoy tiramisu
    Tuesday : drink tea - eat orange - enjoy ice cream
    Wednesday : drink beer - eat peach - enjoy pie
    

여러 시퀀스 중 가장 짧은 시퀀스가 완료되면 zip()은 멈춘다. 위 예제에서는 리스트 중 하나(desserts)가 다른 리스트보다 길다. 그래서 다른 리스트를 모두 확장하지 않는 한 pudding을 얻을 수 없다.


```python
english = 'Monday', 'Tuesday', 'Wednesday'
french = 'Lundi', 'Mardi', 'Mercredi'
print(list(zip(english, french)))
dict(zip(english, french))
```

### Generate Number Sequences with range()

range()는 리스트나 튜플 같은 자료구조를 생성하여 저장하지 않더라도 특정 범위 내에서 숫자 스트림을 반환한다. 이것은 컴퓨터의 메모리를 전부 사용하지 않고, 프로그램의 충돌없이 아주 큰 범위를 생성할 수 있게 해준다.

zip(), range()와 같은 함수는 순회 가능한 객체를 반환한다. 그러므로 for .. in 형태로 값을 순회할 수 있다. 또한 객체를 리스트와 같은 시퀀스로 변환할 수 있다.

### Other Iterators

8장에서는 파일에 대한 iteration를 배우고, 6장에서는 직접 정의한 객체를 순회가 가능하도록 설정하는 방법을 살펴본다.

## Comprehensions

comprehension은 하나 이상의 이터레이터로부터 파이썬의 자구조를 만드는 콤팩트한 방법이다. 컴프리헨션은 비교적 간편한 구문으로 반복문과 조건 테스트를 결합할 수 있도록 해준다. 때때로 컴프리헨션을 사용하는 것은 초급 이상의 단계에서 파이썬을 어느 정도 알고 있다는 것을 의미한다. 즉, 더 파이썬스럽게 사용한다는 것을 의미한다.

### List Comprehensions


```python
number_list = []
for number in range(1, 6):
    number_list.append(numver)

number_list = list(range(1, 6))
```


```python
number_list = [number for number in range(1, 6)]
```


```python
number_list = [number - 1 for number in range(1, 6)]
```

`[expression for item in iterable if condition]` 


```python
a_list = [number for number in range(1, 6) if number % 2 == 1]
a_list = []
for number in range(1, 6):
    if number % 2 == 1:
        a_list.append(number)
```




    [1, 3, 5]




```python
rows = range(1, 4)
cols = range(1, 3)
for row in rows:
    for col in cols:
        print(row, col)

cells = [(row, col) for row in rows for col in cols]
for cell in cells:
    print(cell)
```

    1 1
    1 2
    2 1
    2 2
    3 1
    3 2
    (1, 1)
    (1, 2)
    (2, 1)
    (2, 2)
    (3, 1)
    (3, 2)
    

그리고 cells리스트를 순회한 것처럼, 각 튜플로부터 row와 col의 값만 출력하기 위해 tuple unpacking을 할 수 있다.


```python
for row, col in cells:
    print(row, col)
```

    1 1
    1 2
    2 1
    2 2
    3 1
    3 2
    

리스트 컴프리헨션의 for row...와 for col...코드에서 자신의 if 테스트를 만들 수 있다.

### Dictionary comprehensions

`{key_expression: value_expression for expression in iterable}`


```python
word = 'letters'
letter_counts = {letter: word.count(letter) for letter in set(word)}
letter_counts
```




    {'l': 1, 'e': 2, 's': 1, 't': 2, 'r': 1}



### Set Comprehensions

`{expression for expression in iterable}`


```python
a_set = {number for number in range(1, 6) if number % 3 == 1}
a_set
```




    {1, 4}



### Generator Comprehensions

튜플은 컴프리헨션이 없다! 아마 리스트 컴프리헨션의 `[]`를 `()`로 바꿔서 사용하면 튜플 컴프리헨션이 생성될 것이라고 생각할 것이다. 그리고 다음과 같이 입력했을 때 예외가 나타나지 않기 때문에 뭔가 잘 동작하는 것처럼 보인다.


```python
number_thing = (number for number in range(1, 6))
type(number_thing)
```




    generator



괄호 안의 내용은 **generator comprehension**이다. 그리고 이것은 **제너레이터 객체**를 반환한다. 제너레이터는 이터레이터에 데이터를 제공하는 하나의 방법이다.


```python
for number in number_thing:
    print(number)
number_list = list(number_thing)
number_list # 제너레이터를 다시 순회하려면 아무것도 볼 수 없다.
```




    []



---

**NOTE**

제너레이터는 한 번만 실행될 수 있다. 리스트, 셋, 문자열, 딕셔너리는 메모리에 존재하지만, 제너레이터는 즉석에서 그 값을 생성하고, 이터레이터를 통해서 한 번에 값을 하나씩 처리한다. 제너레이터는 이 값을 기억하지 않으므로 다시 시작하거나 제너레이터를 백업할 수 없다.

---

제너레이터 컴프리헨션이나 제너레이터 함수를 사용해서 생성할 수 있다.

## Functions

함수로 전달한 값을 **argument**라고 부른다. 인자와 함수를 호출하면 인자의 값은 함수 내에서 해당하는 **parameter**에 복사된다. 

함수의 인자는 개수에 상관없이 모든 타입의 인자를 취할 수 있다. 반환값도 마찬가지로 개수에 상관없이 모든 타입을 반환할 수 있다. 만약 함수가 명시적으로 return을 호출하지 않으면, 호출자는 반환값으로 None을 얻는다.


```python
def do_nothing():
    pass # 이 함수가 아무것도 하지 않는다는 것을 보여줌
print(do_nothing())
```

    None
    

---

**NOTE**

None은 아무것도 없다는 것을 뜻하는 파이썬의 특별한 값이다. None이 bool로 평가될 때는 False처럼 보이지만 bool값의 False와는 다르다. 다음 예제를 보자.
```
thing = None
if thing:
    print("It's something")
else:
    print("It's nothing")

It's no thing
```
bool값 False와 None을 구분하기 위해 is 연산자를 사용하자.

```
if thing is None:
    print("It's nothing")
else:
    print("It's something")

It's nothing
```

이것은 아주 미묘한 것처럼 보이지만 중요하다. 빠뜨린 빈 값을 구분하기 위해 None을 사용했다. 정수(0) 혹은 부동소수점수(0.0), 빈 문자열(' '), 빈 리스트([]), 빈 튜플((,)), 빈 딕셔너리({}), 빈 셋(set())은 모두 False지만, None과 같지 않다는 것을 기억하라.

인자가 None인지 출력하는 함수를 작성해보자.
```
def is_none(thing):
    if thing is None:
        print("It's None")
    elif thing:
        print("It's True")
    else:
        print("It's False")
```
이제 몇 가지 테스트를 해보자.
```
>>> is_none(None)
It's None
>>> is_none(True)
It's True
>>> is_none(False)
It's False
>>> is_none(0)
It's False
>>> is_none(0.0)
It's False
>>> is_none(())
It's False
>>> is_none([])
It's False
>>> is_none({})
It's False
>>> is_none(set())
It's False
```

---

### Positional Arguments

파이썬은 다른 언어에 비해 함수의 인자를 유연하고 독특하게 처리한다. 인자의 가장 익숙한 타입은 값을 순서대로 상응하는 매개변수에 복사하는 **positional arguments**다.


```python
def menu(wine, entree, dessert):
    return {'wine': wine, 'entree': entree, 'dessert': dessert}
```

### Keyword Arguments

위치 인자의 혼동을 피하기 위해 매개변수에 상응하는 이름을 인자에 지정할 수 있다.


```python
menu('frontenac', dessert='flan', entree='fish')
```




    {'wine': 'frontenac', 'entree': 'fish', 'dessert': 'flan'}



### Specify Default Parameter Values

매개변수에 기본값을 지정할 수 있다. 호출자가 대응하는 인자를 제공하지 않으면 기본값을 사용한다. 이 단조로운 기능은 꽤 유용하다. 이전 예제를 사용해보자.


```python
def menu(wine, entree, dessert='pudding'):
    return {'wine': wine, 'entree': entree, 'dessert': dessert}
```


```python
menu('chardonnay', 'chicken')
```




    {'wine': 'chardonnay', 'entree': 'chicken', 'dessert': 'pudding'}



---

**NOTE**

기본 인자값은 함수가 실행될 때 계산되지 않고, 함수를 정의할 때 계산된다. 파이썬을 막 시작한 프로그래머는 리스트 혹은 딕셔너리와 같은 변경 가능한 데이터 타입을 기본 인자로 사용할 때 실수를 하게 된다.

---


```python
def buggy(arg, result = []):
    result.append(arg)
    print(result)

buggy('a')
buggy('b')
```


```python
def nonbuggy(arg, result = None):
    if result is None:
        result = []
    result.append(arg)
    print(result)
nonbuggy('a')
nonbuggy('b')
```

    ['a']
    ['b']
    

### Gather Positional Arguments with *

함수의 매개변수에 asterisk를 사용할 때, asterisk는 매개변수에서 위치 인자 변수들을 튜플로 묶는다. print_args()함수에 인자를 전달하여, args 매개변수의 튜플 결과를 살펴보자.


```python
def print_args(*args):
    print('Positional argument tuple:', args)
    
print_args()
print_args(3, 2, 1, 'wait!', 'uh...')
```

    Positional argument tuple: ()
    Positional argument tuple: (3, 2, 1, 'wait!', 'uh...')
    


```python
def print_more(required1, required2, *args):
    print('Need this one:', required1)
    print('Need this one too:', required2)
    print('All the rest:', args)

print_more('cap', 'gloves', 'scarf', 'monocle', 'mustache wax')
```

    Need this one: cap
    Need this one too: gloves
    All the rest: ('scarf', 'monocle', 'mustache wax')
    

\*를 사용할 때 가변 인자의 이름으로 args를 사용할 필요가 없지만 관용적으로 args를 사용한다.

### Gather Keyword Arguments with **

키워드 인자를 딕셔너리로 묶기 위해 두 개의 asterisk(\*\*)를 사용할 수 있다. 인자의 이름은 key고, value는 이 key에 대응하는 딕셔너리 value이다. 다음 예제는 print_kwargs()를 정의하여 키워드 인자를 출력한다.


```python
def print_kwargs(**kwargs):
    print('keyword arguments:', kwargs)
print_kwargs(wine='merlot', entree='mutton', dessert='macaroon')
```

    keyword arguments: {'wine': 'merlot', 'entree': 'mutton', 'dessert': 'macaroon'}
    

위치 매개변수와 \*args, \*\*kwargs를 섞어서 사용하려면 이들을 순서대로 배치해야 한다. 그리고 args와 마찬가지로 키워드 매개변수의 이름을 kwargs로 할 필요는 없지만 관용적으로 kwargs를 사용한다.

### Dostrings

redability counts라는 구절이 1장에서 소개한 Zen of Python에 들어있다. 함수 몸체 시작 부분에 문자열을 포함시켜 함수 정의에 문서를 붙일 수 있다. 이것이 바로 함수의 **docstring**이다.


```python
def echo(anything):
    'echo returns its input argument'
    return anything
```


```python
def print_if_true(thing, check):
    '''
    Prints the first arguemnt if a second argument is true.
    The operation is:
        1. Check whether the *second* argument is true.
        2. If it is, print the *first* argument.
    '''
    if check:
        print(thing)
        
help(echo)
print(echo.__doc__) # 서식 없는 docstring을 그대로 보고 싶다면
```

    Help on function echo in module __main__:
    
    echo(anything)
        echo returns its input argument
    
    echo returns its input argument
    

### Functions Are First-Class Citizens

모든 것이 객체다라는 파이썬의 철학. 객체는 숫자, 문자열, 튜플, 리스트, 딕셔너리, 그리고 함수를 포함한다. 파이썬에서 함수는 first-class citizen이다. 이 뜻은 함수를 변수에 할당할 수 있고, 다른 함수에서 이를 인자로 쓸 수 있으며, 함수에서 이를 반환할 수 있다는 것이다. 


```python
def answer():
    print(42)
    
def run_something(func):
    func()

run_something(answer)
```

    42
    


```python
def run_something_with_args(func, arg1, arg2):
    func(arg1, arg2)

run_something_with_args(add_args, 5, 9)  
```


```python
def sum_args(*args):
    return sum(args) # sum()은 iterable number 인자의 값을 모두 더하는 함수

def run_with_positional_args(func, *args):
    return func(*args)

run_with_positional_args(sum_args, 1, 2, 3, 4)
```




    10



### Inner Functions

내부함수는 루프나 코드 중복을 피하기 위해 또 다른 함수 내에 어떤 복잡한 작업을 한 번 이상 수행할 때 유용하게 사용된다. 다음 문자열 예제에서 내부 함수는 인자에 텍스트를 붙여준다.


```python
def knights(saying):
    def inner(quote):
        return "We are the knights who say: '%s'" % quote
    return inner(saying)

knights('Ni!')
```




    "We are the knights who say: 'Ni!'"



### Closures

내부 함수는 **closure**처럼 행동할 수 있다. 클로져는 다른 함수에 의해 동적으로 생성된다. 그리고 바깥 함수로부터 생성된 변수값을 변경하고, 저장할 수 있는 함수다.

다음 함수는 이전과는 달리 `inner2()`라는 클로져를 사용한다.

- inner2()는 인자를 취하지 않고, 외부 함수의 변수를 직접 사용한다.
- knights2()는 inner2함수 이름을 호출하지 않고, 이를 반환한다.


```python
def knights2(saying):
    def inner2():
        return "We are the knights who say: '%s'" % saying
    return inner2
```

inner2()함수는 knights2()함수가 전달받은 saying변수를 알고 있다. 코드에서 return inner2라인은 inner2함수의 특별한 복사본을 반환한다. 이것이 외부 함수에 의해 동적으로 생성되고, 그 함수의 변수값을 알고 있는 함수인 클로져다.


```python
a = knights2('Duck')
b = knights2('Hasenpfeffer')

print(type(a), type(b))
print(a, b)
```

    <class 'function'> <class 'function'>
    <function knights2.<locals>.inner2 at 0x0000024D31888C80> <function knights2.<locals>.inner2 at 0x0000024D318886A8>
    

이들을 호출하면, knights2()에 전달되어 사용된 saying을 기억한다.


```python
a()
b()
```




    "We are the knights who say: 'Hasenpfeffer'"



### Anonumous Functions: the lambda() Function

파이썬의 lambda function은 단일문으로 표현되는 익명 함수다. edit_story()를 정의해보자. 인자는 다음과 같다.
- words: words리스트
- func: words의 각 word 문자열에 적용되는 함수


```python
def edit_story(words, func):
    for word in words:
        print(func(word))
```

이제 words 리스트와 각 word에 적용할 함수가 필요하다. 


```python
stairs = ['thud', 'meow', 'thud', 'hiss']
```

그리고 각 word의 첫 글자를 대문자로 만들고 느낌표를 붙여준다. 고양이를 위한 타블로이드 신문의 헤드라인을 만들어주는 함수를 정의해보자.


```python
def enliven(word):
    return word.capitalize() + '!'

edit_story(stairs, enliven)
```

    Thud!
    Meow!
    Thud!
    Hiss!
    


```python
# 람다를 사용
edit_story(stairs, lambda word: word.capitalize() + '!')
```

    Thud!
    Meow!
    Thud!
    Hiss!
    


```python
pow(5, 2)
```




    25



람다에서 하나의 word 인자를 취했다. 람다의 콜론(:)과 닫는 괄호 사이에 있는 것이 함수의 정의 부분이다. 

대부분의 경우 `enlien()`과 같은 실제 함수를 사용하는 것이 람다를 사용하는 것보다 훨씬 더 명확하다. 람다는 많은 작은 함수를 정의하고, 이들을 호출해서 얻은 모든 결과값을 저장해야하는 경우에 유용하다. 특히 *callback functions*를 정의하는 GUI에서 람다를 사용할 수 있다.

## Generators

제너레이터는 파이썬의 시퀀스를 생성하는 객체다. 제너레이터로, 전체 시퀀스를 한 번에 메모리에 생성하고 정렬할 필요 없이, 잠재적으로 아주 큰 시퀀스를 순회할 수 있다. 제너레이터는 이터레이터에 대한 데이터의 소스로 자주 사용된다. 이전 예제를 기억한다면, 우리는 이미 제너레이터 중 하나인 range()함수를 사용했다. range()는 일련의 정수를 생성한다. 파이썬 2의 range()는 메모리에 제한적인 리스트를 반환한다. 또한 파이썬 2의 xrange()가 있는데, 이는 파이썬 3의 일반적인 range()가 되었다. 

`>>> sum(range(1, 101))
5050`

제너레이터를 순회할 때마다 마지막으로 호출된 항목을 기억하고 다음 값을 반환한다. 제너레이터는 일반 함수와 다르다. 일반 함수는 이전 호출에 대한 메모리가 없고, 항상 똑같은 상태로  첫 번째 라인부터 수행한다.

잠재적으로 큰 시퀀스를 생성하고, 제너레이터 컴프리헨션에 대한 코드가 아주 긴 경우에는 제너레이터 함수를 사용하면 된다. 이것은 일반 함수지만 return문으로 값을 반환하지 않고, yield문으로 값을 반환한다. 우리만의 range()함수를 작성해보자.


```python
def my_range(first = 0, last = 10, step = 1):
    number = first
    while number < last:
        yield number
        number += step

my_range
```




    <function __main__.my_range(first=0, last=10, step=1)>




```python
ranger = my_range(1, 5)
ranger
```




    <generator object my_range at 0x00000294AF206F48>




```python
for x in ranger:
    print(x)
```

    1
    2
    3
    4
    

## Decorators

가끔식 소스 코드를 바꾸지 않고, 사용하고 있는 함수를 수정하고 싶을 때가 있다. 일반적인 예는 함수에 전달된 인자를 보기 위해 디버깅 문을 추가하는 것이다. 

decorator는 하나의 함수를 취해서 또 다른 함수를 반환하는 함수다.

- \*args와 \*\*kwargs
- 내부함수
- 함수 인자 

`document_it()`함수는 다음과 같이 데커레이터를 정의한다.

- 함수 이름과 인자값을 출력한다.
- 인자로 함수를 실행한다.
- 결과를 출력한다.
- 수정된 함수를 사용할 수 있도록 반환한다.


```python
def document_it(func):
    def new_function(*args, **kwargs):
        print('Running function:', func.__name__)
        print('Positional arguments:', args)
        print('Keyword arguments:', kwargs)
        result = func(*args, **kwargs)
        print('Result:', result)
        return result
    return new_function
```

`document_it()`함수에 어떤 func함수 이름을 전달하든지 간에 `document_it()`함수에 추가 선언문이 포함된 새 함수를 얻는다. 데커레이터는 실제로 func함수로부터 코드를 실행하지 않는다. 하지만 document_it()함수로부터 func를 호출하여 결과뿐만 아니라 새로운 함수를 얻는다. 

그러면 데커레이터를 어떻게 사용할까? 수동으로 데커레이터를 적용해보자.


```python
def add_ints(a, b):
    return a + b

cooler_add_ints = document_it(add_ints)
cooler_add_ints(3, 5)
```

    Running function: add_ints
    Positional arguments: (3, 5)
    Keyword arguments: {}
    Result: 8
    




    8



위와 같이 수동으로 데커레이터를 할당하는 대신, 다음과 같이 데코레이터를 사용하고 싶은 함수에 *@decorator_name*을 추가한다.


```python
@document_it
def add_ints(a,b):
    return a + b

add_ints(3, 5)
```

    Running function: add_ints
    Positional arguments: (3, 5)
    Keyword arguments: {}
    Result: 8
    




    8



함수는 여러 개의 데커레이터를 가질 수 있다. 결과를 제곱하는 square_it()데커레이터를 작성해보자.


```python
def square_it(func):
    def new_function(*args, **kwargs):
        result = func(*args, **kwargs)
        return result * result
    return new_function
```

함수에서 가장 가까운(def위) 데커레이터를 먼저 실행한 후, 그 위의 데커레이터가 실행된다. 이 예제에서 순서를 바꿔도 똑같은 결과를 얻지만, 중간 과정이 바뀐다.


```python
@document_it
@square_it
def add_ints(a, b):
    return a + b

add_ints(3, 5)
```

    Running function: new_function
    Positional arguments: (3, 5)
    Keyword arguments: {}
    Result: 64
    




    64




```python
@square_it
@document_it
def add_ints(a, b):
    return a + b

add_ints(3, 5)
```

    Running function: add_ints
    Positional arguments: (3, 5)
    Keyword arguments: {}
    Result: 8
    




    64



## Namespaces and Scope

이름은 사용되는 위치에 따라 다른 것을 참조할 수 있다. 파이썬 프로그램에는 다양한 네임스페이스가 있다. 네임스페이스는 특정 이름이 유일하고, 다른 네임스페이스에서의 같은 이름과 관계가 없는 것을 말한다.

각 함수는 자신의 네임스페이스를 정의한다. 메인 프로그램에서 x라는 변수를 정의하고, 함수에서 x라는 변수를 정의했을 때, 이들은 서로 다른 것을 참조한다. 하지만 이 경계를 넘을 수 있다. 다양한 방법으로 다른 네임스페이스의 이름을 접근할 수 있다.

메인 프로그램은 전역 네임스페이스를 정의한다. 이와 같이 이 네임스페이스의 변수들은 전역변수다.

함수에서 전역 변수의 값을 얻어서 바꾸려 하면 에러가 발생한다.

전역 변수를 바꾸려 한다면, 또 다른 이름의 animal 변수를 변경하려 한다. 다음은 함수 내에 있는 지역 변수다.

함수 내의 지역 변수가 아닌 전역 변수를 접근하기 위해 global 키워드를 사용해서 전역 변수의 접근을 명시해야 한다.


```python
animal = "fruitbat"
def change_and_print_global():
    global animal
    animal = 'wombat'
    print('inside change_and_print_global:', animal)

animal
```




    'fruitbat'




```python
change_and_print_global()
```

    inside change_and_print_global: wombat
    


```python
animal
```




    'wombat'



파이썬은 네임스페이스의 내용을 접근하기 위해 두 가지 함수를 제공한다.

- locals()함수는 로컬 네임스페이스의 내용이 담긴 딕셔너리를 반환한다.
- globals()함수는 글로벌 네임스페이스의 내용이 담긴 딕셔너리를 반환한다.


```python
animal = 'fruitbat'
def change_local():
    animal = 'wombat' 
    print('locals:', locals())

animal
```




    'fruitbat'




```python
change_local()
```

    locals: {'animal': 'wombat'}
    


```python
print('globals:', globals())
```

    globals: {'__name__': '__main__', '__doc__': 'Automatically created module for IPython interactive environment', '__package__': None, '__loader__': None, '__spec__': None, '__builtin__': <module 'builtins' (built-in)>, '__builtins__': <module 'builtins' (built-in)>, '_ih': ['', 'def my_range(first = 0, last = 10, step = 1):\n    number = first\n    while number < last:\n        yiled number\n        number += step\n\nmy_range', 'def my_range(first = 0, last = 10, step = 1):\n    number = first\n    while number < last:\n        yield number\n        number += step\n\nmy_range', 'ranger = my_range(1, 5)\nranger', 'for x in ranger:\n    print(X)', 'for x in ranger:\n    print(x)', 'def my_range(first = 0, last = 10, step = 1):\n    number = first\n    while number < last:\n        yield number\n        number += step\n\nmy_range', 'ranger = my_range(1, 5)\nranger', 'for x in ranger:\n    print(x)', "def document_it(func):\n    def new_function(*args, **kwargs):\n        print('Running function:', func.__name__)\n        print('Positional arguments:', args)\n        print('Keyword arguments:', kwargs)\n        result = func(*args, **kwargs)\n        print('Result:', result)\n        return result\n    return new_function", 'def add_ints(a, b):\n    return a + b\n\ncooler_add_ints = document_it(add_ints)\ncooler_add_ints(3, 5)', '@document_it\ndef add_ints(a,b):\n    return a + b\n\nadd_ints(3, 5)', 'def square_it(func):\n    def new_function(*args, **kwargs):\n        result = func(*args, **kargs)\n        return result * result\n    return new_function', '@document_it\n@squre_it\ndef add_ints(a, b):\n    return a + b\n\nadd_ints(3, 5)', '@document_it\n@square_it\ndef add_ints(a, b):\n    return a + b\n\nadd_ints(3, 5)', 'def square_it(func):\n    def new_function(*args, **kwargs):\n        result = func(*args, **kwargs)\n        return result * result\n    return new_function', '@document_it\n@square_it\ndef add_ints(a, b):\n    return a + b\n\nadd_ints(3, 5)', '@square_it\n@document_it\ndef add_ints(a, b):\n    return a + b\n\nadd_ints(3, 5)', 'animal = "fruitbat"\ndef change_and_print_global():\n    global animal\n    animal = \'wombat\'\n    print(\'inside change_and_print_global:\', animal)\n\nanimal', 'change_and_print_global()', 'animal', "animal = 'fruitbat'\ndef change_local():\n    animal = 'wombat' \n    print('locals:', locals())\n\nanimal", 'change_local()', "print('globals:', globals())", "print('globals:', globals(), 3)"], '_oh': {2: <function my_range at 0x00000294AEDC61E0>, 3: <generator object my_range at 0x00000294AECE04F8>, 6: <function my_range at 0x00000294AF205D90>, 7: <generator object my_range at 0x00000294AF206F48>, 10: 8, 11: 8, 16: 64, 17: 64, 18: 'fruitbat', 20: 'wombat', 21: 'fruitbat'}, '_dh': ['D:\\python'], 'In': ['', 'def my_range(first = 0, last = 10, step = 1):\n    number = first\n    while number < last:\n        yiled number\n        number += step\n\nmy_range', 'def my_range(first = 0, last = 10, step = 1):\n    number = first\n    while number < last:\n        yield number\n        number += step\n\nmy_range', 'ranger = my_range(1, 5)\nranger', 'for x in ranger:\n    print(X)', 'for x in ranger:\n    print(x)', 'def my_range(first = 0, last = 10, step = 1):\n    number = first\n    while number < last:\n        yield number\n        number += step\n\nmy_range', 'ranger = my_range(1, 5)\nranger', 'for x in ranger:\n    print(x)', "def document_it(func):\n    def new_function(*args, **kwargs):\n        print('Running function:', func.__name__)\n        print('Positional arguments:', args)\n        print('Keyword arguments:', kwargs)\n        result = func(*args, **kwargs)\n        print('Result:', result)\n        return result\n    return new_function", 'def add_ints(a, b):\n    return a + b\n\ncooler_add_ints = document_it(add_ints)\ncooler_add_ints(3, 5)', '@document_it\ndef add_ints(a,b):\n    return a + b\n\nadd_ints(3, 5)', 'def square_it(func):\n    def new_function(*args, **kwargs):\n        result = func(*args, **kargs)\n        return result * result\n    return new_function', '@document_it\n@squre_it\ndef add_ints(a, b):\n    return a + b\n\nadd_ints(3, 5)', '@document_it\n@square_it\ndef add_ints(a, b):\n    return a + b\n\nadd_ints(3, 5)', 'def square_it(func):\n    def new_function(*args, **kwargs):\n        result = func(*args, **kwargs)\n        return result * result\n    return new_function', '@document_it\n@square_it\ndef add_ints(a, b):\n    return a + b\n\nadd_ints(3, 5)', '@square_it\n@document_it\ndef add_ints(a, b):\n    return a + b\n\nadd_ints(3, 5)', 'animal = "fruitbat"\ndef change_and_print_global():\n    global animal\n    animal = \'wombat\'\n    print(\'inside change_and_print_global:\', animal)\n\nanimal', 'change_and_print_global()', 'animal', "animal = 'fruitbat'\ndef change_local():\n    animal = 'wombat' \n    print('locals:', locals())\n\nanimal", 'change_local()', "print('globals:', globals())", "print('globals:', globals(), 3)"], 'Out': {2: <function my_range at 0x00000294AEDC61E0>, 3: <generator object my_range at 0x00000294AECE04F8>, 6: <function my_range at 0x00000294AF205D90>, 7: <generator object my_range at 0x00000294AF206F48>, 10: 8, 11: 8, 16: 64, 17: 64, 18: 'fruitbat', 20: 'wombat', 21: 'fruitbat'}, 'get_ipython': <function get_ipython at 0x00000294ABEC4730>, 'exit': <IPython.core.autocall.ZMQExitAutocall object at 0x00000294AEBFEC50>, 'quit': <IPython.core.autocall.ZMQExitAutocall object at 0x00000294AEBFEC50>, '_': 'fruitbat', '__': 'wombat', '___': 'fruitbat', 'json': <module 'json' from 'C:\\Users\\quasa\\Anaconda3\\lib\\json\\__init__.py'>, 'autopep8': <module 'autopep8' from 'C:\\Users\\quasa\\Anaconda3\\lib\\site-packages\\autopep8.py'>, 'getsizeof': <built-in function getsizeof>, 'NamespaceMagics': <class 'IPython.core.magics.namespace.NamespaceMagics'>, '_nms': <IPython.core.magics.namespace.NamespaceMagics object at 0x00000294AF183F28>, '_Jupyter': <ipykernel.zmqshell.ZMQInteractiveShell object at 0x00000294AEA4B780>, 'np': <module 'numpy' from 'C:\\Users\\quasa\\Anaconda3\\lib\\site-packages\\numpy\\__init__.py'>, '_getsizeof': <function _getsizeof at 0x00000294AEDC62F0>, '_getshapeof': <function _getshapeof at 0x00000294AECE8C80>, 'var_dic_list': <function var_dic_list at 0x00000294AF068A60>, '_i': "print('globals:', globals())", '_ii': 'change_local()', '_iii': "animal = 'fruitbat'\ndef change_local():\n    animal = 'wombat' \n    print('locals:', locals())\n\nanimal", '_i1': 'def my_range(first = 0, last = 10, step = 1):\n    number = first\n    while number < last:\n        yiled number\n        number += step\n\nmy_range', '_i2': 'def my_range(first = 0, last = 10, step = 1):\n    number = first\n    while number < last:\n        yield number\n        number += step\n\nmy_range', 'my_range': <function my_range at 0x00000294AF205D90>, '_2': <function my_range at 0x00000294AEDC61E0>, '_i3': 'ranger = my_range(1, 5)\nranger', 'ranger': <generator object my_range at 0x00000294AF206F48>, '_3': <generator object my_range at 0x00000294AECE04F8>, '_i4': 'for x in ranger:\n    print(X)', 'x': 4, '_i5': 'for x in ranger:\n    print(x)', '_i6': 'def my_range(first = 0, last = 10, step = 1):\n    number = first\n    while number < last:\n        yield number\n        number += step\n\nmy_range', '_6': <function my_range at 0x00000294AF205D90>, '_i7': 'ranger = my_range(1, 5)\nranger', '_7': <generator object my_range at 0x00000294AF206F48>, '_i8': 'for x in ranger:\n    print(x)', '_i9': "def document_it(func):\n    def new_function(*args, **kwargs):\n        print('Running function:', func.__name__)\n        print('Positional arguments:', args)\n        print('Keyword arguments:', kwargs)\n        result = func(*args, **kwargs)\n        print('Result:', result)\n        return result\n    return new_function", 'document_it': <function document_it at 0x00000294AF205E18>, '_i10': 'def add_ints(a, b):\n    return a + b\n\ncooler_add_ints = document_it(add_ints)\ncooler_add_ints(3, 5)', 'add_ints': <function square_it.<locals>.new_function at 0x00000294AF220C80>, 'cooler_add_ints': <function document_it.<locals>.new_function at 0x00000294AF15FD08>, '_10': 8, '_i11': '@document_it\ndef add_ints(a,b):\n    return a + b\n\nadd_ints(3, 5)', '_11': 8, '_i12': 'def square_it(func):\n    def new_function(*args, **kwargs):\n        result = func(*args, **kargs)\n        return result * result\n    return new_function', 'square_it': <function square_it at 0x00000294AF205EA0>, '_i13': '@document_it\n@squre_it\ndef add_ints(a, b):\n    return a + b\n\nadd_ints(3, 5)', '_i14': '@document_it\n@square_it\ndef add_ints(a, b):\n    return a + b\n\nadd_ints(3, 5)', '_i15': 'def square_it(func):\n    def new_function(*args, **kwargs):\n        result = func(*args, **kwargs)\n        return result * result\n    return new_function', '_i16': '@document_it\n@square_it\ndef add_ints(a, b):\n    return a + b\n\nadd_ints(3, 5)', '_16': 64, '_i17': '@square_it\n@document_it\ndef add_ints(a, b):\n    return a + b\n\nadd_ints(3, 5)', '_17': 64, '_i18': 'animal = "fruitbat"\ndef change_and_print_global():\n    global animal\n    animal = \'wombat\'\n    print(\'inside change_and_print_global:\', animal)\n\nanimal', 'animal': 'fruitbat', 'change_and_print_global': <function change_and_print_global at 0x00000294AF15FE18>, '_18': 'fruitbat', '_i19': 'change_and_print_global()', '_i20': 'animal', '_20': 'wombat', '_i21': "animal = 'fruitbat'\ndef change_local():\n    animal = 'wombat' \n    print('locals:', locals())\n\nanimal", 'change_local': <function change_local at 0x00000294B04A20D0>, '_21': 'fruitbat', '_i22': 'change_local()', '_i23': "print('globals:', globals())", '_i24': "print('globals:', globals(), 3)"} 3
    

### Use _ and __ in Names

두 언더스코어( __ )로 시작하고 끝나는 이름은 파이썬 내의 사용을 위해 예약되어 있다. 그러므로 변수를 선언할 때 두 언더스코어를 사용하면 안 된다. 어플 개발자들이 이와 같은 변수 이름을 선택할 가능성이 낮아서, 이러한 네이밍 패턴을 선택한 것이다.

예를 들어 함수의 이름은 시스템 변수 function.\_\_name\_\_에 있다. 그리고 함수의 docstring은 function.\_\_doc\_\_에 있다.


```python
def amazing():
    '''This is the amazing function.
    Want to see it again?'''
    print('This function is named:', amazing.__name__)
    print('And its docstring is:', amazing.__doc__)
    
amazing()
```

    This function is named: amazing
    And its docstring is: This is the amazing function.
        Want to see it again?
    

## Handle Errors with try and except


```python
short_list = [1, 2, 3]
position = 5
try:
    short_list[position]
except:
    print('Need a position between 0 and', len(short_list) - 1, ' but fot', position)
```

    Need a position between 0 and 2  but fot 5
    

위와 같이 인자 없는 except문을 지정하는 것은 모든 예외 타입을 잡는다는 것을 말한다. 두 개 이상의 예외 타입이 발생하면 각각 별도의 예외 핸들러를 제공하는 것이 가장 좋은 방법이다.

예외 타입을 넘어 예외사항에 대한 세부정보를 얻고 싶다면 다음과 같이 변수 이름에서 예외 객체 전체를 얻을 수 있다.

`except exceptiontype as name`

다음 예제는 먼저 IndexError를 찾는다. 이것은 시퀀스에서 잘못된 위치를 입력할 대 발생하는 예외 타입이다. 그리고 err변수에 IndexError 예외를, other변수에 다른 기타 예외를 저장한다. 다음 예제는 사용자가 입력한 값에 따라 other 변수에 저장된 객체를 출력한다.

exceptiontype은 자동으로 지정되어 있다.


```python
short_lsit = [1, 2, 3]
while True:
    value = input("Position [q to quit]?")
    if value == 'q':
        break
    try:
        position = int(value)
        print(short_list[position])
    except IndexError as err: 
        print('Bad index:', position)
    except Exception as other:
        print('Something else broke:', other)
```

    Position [q to quit]?1
    2
    Position [q to quit]?0
    1
    Position [q to quit]?2
    3
    Position [q to quit]?3
    Bad index: 3
    Position [q to quit]?2
    3
    Position [q to quit]?two
    Something else broke: invalid literal for int() with base 10: 'two'
    Position [q to quit]?10
    Bad index: 10
    Position [q to quit]?q
    

## Make Your Own Exceptions

IndexError같은 예외는 파이썬 표준라이브러리에 미리 정의되어 있는 것이다. 우리는 필요한 예외 처리를 선택해서 사용할 수 있다. 또한 우리가 만든 프로그램에서 특별한 상황에 발생할 수 있는 예외를 처리하기 위해 예외 타입을 정의할 수 있다.

예외는 클래스고, Exception클래스의 자식이다. 다음 예제에서 words 문자열에 대문자가 있을 때 예외를 발생하는 UppercaseException 예외를 만들어보자.


```python
class UppercaseException(Exception):
    pass

words = ['eeenie', 'meenie', 'miny', 'MO']
for word in words:
    if word.isupper():
        raise UppercaseException(word)
```


    ---------------------------------------------------------------------------

    UppercaseException                        Traceback (most recent call last)

    <ipython-input-4-442c3da44bcd> in <module>
          5 for word in words:
          6     if word.isupper():
    ----> 7         raise UppercaseException(word)
    

    UppercaseException: MO


부모 클래스인 Exception은 예외가 발생했을 때 무엇을 출력하는지 알아내도록 처리하고 있다.

다음과 같이 예외 객체에 접근해서 내용을 출력할 수 있다.

![image.png](attachment:image.png)

## Exercises

4.5


```python
{i: i*i for i in range(10)}
```




    {0: 0, 1: 1, 2: 4, 3: 9, 4: 16, 5: 25, 6: 36, 7: 49, 8: 64, 9: 81}






```python
{i for i in range(10) if i % 2 == 1}
```




    {1, 3, 5, 7, 9}




```python
def get_odds():
    for number in range(1, 10, 2):
        yield number
        
for count, number in enumerate(get_odds(), 1):
    if count == 3:
        print("The thirs odd number is", number)
        break
```

    The thirs odd number is 5
    


```python
def document_it(func):
    def new_function(*args, **kwargs):
        print("Running function: ", func.__name__)
        print("Positional arguments: ", args)
        print("Keyword arguments: ", args)
        result = func(*args, **kwargs)
        print("Result: ", result)
        return result
    return new_function


@document_it
def add_int(a, b):
    return a + b

add_int(1, 2)
```

    Running function:  add_int
    Positional arguments:  (1, 2)
    Keyword arguments:  (1, 2)
    Result:  3
    




    3




```python
class OopsException(Exception):
    pass

try:
    raise OopsException
except OopsException:
    print('Caught an oops')
```

    Caught an oops
    


```python
titles = ['Creature of Habit', 'Crewel Fate'] 
plots = ['A nun turns into a monster', 'A haunted yarn shop']

movies = dict(zip(titles, plots))
movies
```




    {'Creature of Habit': 'A nun turns into a monster',
     'Crewel Fate': 'A haunted yarn shop'}



# Py Boxes: Moduels, Packages, and Programs

## Standlone Programs


```python
import sys
print('Program arguments:', sys.argv) # sys.argv[0]은 파일이름이다.
```

    Program arguments: ['C:\\Users\\quasa\\Anaconda3\\lib\\site-packages\\ipykernel_launcher.py', '-f', 'C:\\Users\\quasa\\AppData\\Roaming\\jupyter\\runtime\\kernel-956383d2-38d2-4ecc-8338-108893ee7508.json']
    

## Modules and the import Statement

### Import a Module

import된 코드가 여러 장소에서 사용되는 경우, import문을 함수 밖으로 빼내는 것도 고려해야 한다. 그리고 임포트된 코드의 사용이 내부에만 제한되는 경우, import문을 내부에 놓는다. 몇몇 사람은 코드의 모든 의존성을 명시하기 위해 모든 import문을 파일의 맨 앞에 두는 것을 선호한다.

- 필요한 모듈만 임포트하기: `from report import get_description as do_it`

### Module Search Path

파이썬은 임포트할 파일을 어디에서 찾을까? 디렉터리 이름의 리스트와 포준 sys 모듈에 저장되어 있는 ZIP아카이브 파일을 변수 path로 사용한다. 이 리스트를 접근하여 수정할 수 있다. 


```python
import sys
for place in sys.path:
    print(place)
```

    D:\python
    C:\Users\quasa\Anaconda3\python37.zip
    C:\Users\quasa\Anaconda3\DLLs
    C:\Users\quasa\Anaconda3\lib
    C:\Users\quasa\Anaconda3
    
    C:\Users\quasa\Anaconda3\lib\site-packages
    C:\Users\quasa\Anaconda3\lib\site-packages\win32
    C:\Users\quasa\Anaconda3\lib\site-packages\win32\lib
    C:\Users\quasa\Anaconda3\lib\site-packages\Pythonwin
    C:\Users\quasa\Anaconda3\lib\site-packages\IPython\extensions
    C:\Users\quasa\.ipython
    

## Packages

한 번은 다음 날, 또 한 번은 다음 주와 같이 다른 타입의 날씨 정보가 필요하다고 하자. 코드를 구성하는 한 가지 방법은 sources 디렉토리를 만든 뒤 daily.py와 weekly.py라는 두 개의 모듈을 생성하는 것이다. 각 모듈은 forecast함수를 갖는다. daily버전은 한 문자열을 반환하고, weekly버전은 일곱 개의 문자열이 담긴 리스트를 반환한다. 

sources 디렉터리에 한 가지 더 필요한 것이 있는데, \_\_init\_\_.py파일이다. 이 파일의 내용은 비워도 되지만, 파이썬은 이 파일을 포함하는 디렉토리를 패키지로 간주한다. 이 파일의 내용은 비워도 상관없다.

## The Python Standard Library

### Handle Missing Keys with setdefault() and defaultdict()

존재하지 않는 키로 딕셔너리에 접근하려 하면 예외가 발생한다.  기본값을 반환하는 딕셔너리의 `get()`함수를 사용하면 이 예외를 피할 수 있다. `setdefault()`함수는 `get()`함수와 같지만, 키가 누락된 경우 딕셔너리에 항목을 할당할 수 있다.


```python
periodic_table = {'Hydrogen': 1, 'Helium': 2}
print(periodic_table)
```

    {'Hydrogen': 1, 'Helium': 2}
    


```python
carbon = periodic_table.setdefault('Carbon', 12)
carbon
```




    12




```python
periodic_table
```




    {'Hydrogen': 1, 'Helium': 2, 'Carbon': 12}



존재하는 키에 다른 기본값을 할당하려 하면 키에 대한 원래 값이 반환되고 아무것도 바뀌지 않는다.


```python
helium = periodic_table.setdefault('Helium', 947)
print(helium)
print(periodic_table)
```

    2
    {'Hydrogen': 1, 'Helium': 2, 'Carbon': 12}
    

`defaultdict()`함수도 비슷하다. 다른 점은 딕셔너리를 생성할 때 모든 새 키에 대한 기본값을 먼저 지정한다는 것이다. 이 함수의 인자는 함수다. 다음 예제에서 int()함수를 호출하고 정수0을 반환하는 함수 int를 전달해보자


```python
from collections import defaultdict
periodic_table = defaultdict(int)
```

이제 모든 누락된 기본값은 0이다.


```python
periodic_table['Hydrogen'] = 1
periodic_table['Lead']
periodic_table
```




    defaultdict(int, {'Hydrogen': 1, 'Lead': 0})



`defaultdict()`의 인자는 값을 누락된 키에 할당하여 반환하는 함수다. 다음 예제에서 no_idea()함수는 필요할 때 값을 반환하기 위해 실행된다. 


```python
from collections import defaultdict

def no_idea():
    return 'Huh?'

bestiary = defaultdict(no_idea)
bestiary['A'] = 'Abominable Snowman'
bestiary['B'] = 'Basilisk'
print(bestiary['A'], bestiary['B'], bestiary['C'])
```

    Abominable Snowman Basilisk Huh?
    

빈 기본값을 반환하기 위해 int()함수는 정수0, list()함수는 빈 리스트, dict()함수는 빈 딕셔너리를 반환한다. 인자를 입력하지 않으면 새로운 키의 초깃값이 None으로 설정된다. 그리고 lambda를 사용하여 인자에 기본값을 만드는 함수를 정의할 수 있다.


```python
bestiary = defaultdict(lambda: 'Huh?')
bestiary['E']
```




    'Huh?'




```python
from collections import defaultdict
food_counter = defaultdict(int)
for food in ['spam', 'spam', 'eggs', 'spam']:
    food_counter[food] += 1
    
for food, count in food_counter.items():
    print(food, count)
```

    spam 3
    eggs 1
    

### Count Items with Counter()


```python
from collections import Counter
breakfast = ['spam', 'spam', 'eggs', 'spam']
Counter(breakfast)
```




    Counter({'spam': 3, 'eggs': 1})



`most_common()`은 모든 요소를 내림차순으로 반환한다. 혹은 숫자를 입력하는 경우, 그 숫자만큼 상위 요소를 반환한다.


```python
breakfast_counter = Counter(breakfast)
print(breakfast_counter.most_common(), breakfast_counter.most_common(1))
```

    [('spam', 3), ('eggs', 1)] [('spam', 3)]
    


```python
lunch = ['eggs', 'eggs', 'bacon']
lunch_counter = Counter(lunch)
# + 연산자를 사용해서 두 카운터를 결합할 수 있다.
breakfast_counter + lunch_counter
```




    Counter({'spam': 3, 'eggs': 3, 'bacon': 1})




```python
breakfast_counter - lunch_counter
```




    Counter({'spam': 3})




```python
lunch_counter - breakfast_counter
```




    Counter({'eggs': 1, 'bacon': 1})




```python
breakfast_counter & lunch_counter # 교집합
```




    Counter({'eggs': 1})




```python
breakfast_counter | lunch_counter # 합집합. eggs는 높은 숫자의 공토 항목을 선택한다.
```




    Counter({'spam': 3, 'eggs': 2, 'bacon': 1})



### Order by Key with OrderedDict()




```python
quotes = {
    'Mpe': 'A wise guy, huh?',
    'Larry': 'Ow!',
    'Curly': 'Nyuk nuuk!'
}
for stooge in quotes:
    print(stooge)
```

    Mpe
    Larry
    Curly
    

OrderedDict()함수는 키의 추가 순서를 기억하고, 이터레이터로부터 순서대로 키값을 반환한다. (키, 값) 튜플의 시퀀스로부터 OrderedDict를 생성해보자.


```python
from collections import OrderedDict
quotes = OrderedDict([
    ('Moe', 'A wise guy, huh?'),
    ('Larry', 'Ow!'),
    ('Curly', 'Nyuk nyuk!')
])

for stooge in quotes:
    print(stooge)
```

    Moe
    Larry
    Curly
    

### Stack + Queue == deque

deque는 스택과 큐의 기능을 모두 가진 출입구가 양 끝에 있는 큐다. 데크는 시퀀스의 양 끝으로부터 항목을 추가하거나 삭제할 때 유용하게 쓰인다. 여기에는 회문인지 확인하기 위해 양쪽 끝에서 중간까지 문자를 확인한다. popleft()함수는 데크로부터 왼쪽 끝의 항목을 제거한 후, 그 항목을 반환한다. pop()함수는 오른쪽 끝의 항목을 제거한 후, 그 항목을 반환한다. 양쪽 끝에서부터 이 두 함수가 중간 지점을 향해서 동작한다. 양쪽 문자가 서로 일치한다면 단어 중간에 도달할 때까지 데크를 팝한다.


```python
def palindrome(word):
    from collections import deque
    dq = deque(word)
    while len(dq) > 1:
        if dq.popleft() != dq.pop():
            return False
        return True
```


```python
def another_palindrome(word):
    return word == word[::-1]
```

### Iterate over Code Structures with itertools

itertools는 특수 목적의 이터레이터 함수를 포함하고 있다. for...in 로프에서 이터레이터 함수를 호출할 때 함수는 한 번에 한 항목을 반환하고, 호출 상태를 기억한다.

chain()함수는 순회 가능한 인자들을 하나씩 반환한다.


```python
import itertools
for item in itertools.chain([1, 2], ['a', 'b']):
    print(item)
```

    1
    2
    a
    b
    

cycle()은 인자를 순환하는 무한 이터레이터다.
```
import itertools
for item in itertools.cycle([1, 2]):
    print(tiem)
```

accumulate()는 축적된 값을 계산한다. 기본적으로 합계를 계산한다.


```python
import itertools
for item in itertools.accumulate([1, 2, 3, 4]):
    print(item)
```

    1
    3
    6
    10
    

accumulate()함수의 두 번째 인자로 함수를 전달하여, 합계를 구하는 대신 이 함수를 사용할 수 있다. 이 함수는 두 개의 인자를 취하여 하나의 결과를 반환한다. 이 예제는 축적된 곱을 계산한다.


```python
import itertools
def multiply(a, b):
    return a * b

for item in itertools.accumulate([1, 2, 3, 4], multiply):
    print(item)
```

    1
    2
    6
    24
    

itertools 모듈은 시간을 단축할 수 있는 순열과 조합에 대한 더 많음 함수를 제공한다.

### Print Nicely with pprint()


```python
from pprint import pprint
quotes = OrderedDict([
    ('Moe', 'A wise guy, huh?'),
    ('Larry', 'Ow!'),
    ('Curly', 'Nyuk nyuk!')
])
print(quotes)
pprint(quotes) # 가독성을 위해 요소들을 정렬하여 출력한다.
```

    OrderedDict([('Moe', 'A wise guy, huh?'), ('Larry', 'Ow!'), ('Curly', 'Nyuk nyuk!')])
    OrderedDict([('Moe', 'A wise guy, huh?'),
                 ('Larry', 'Ow!'),
                 ('Curly', 'Nyuk nyuk!')])
    

## Exercises


```python
plain = {'a': 1, 'b': 2, 'c': 3}
ordered_plain = OrderedDict(plain)
```




    OrderedDict([('a', 1), ('b', 2), ('c', 3)])




```python
dict_of_lists = defaultdict(list)
dict_of_lists['a'] = 'something for a'
dict_of_lists['a']
```




    'something for a'



# Oh Oh: Object and Classes

## What Are Objects?

객체는 data와 code 모두를 포함한다. 7과 8이 모두 속하는 integer클래스가 있다. 그리고 문자열 'cat'과 'duck' 또한 파이썬 객체이고 capitalize()와 replace()같은 문자열 메소드를 가진다. 

이전에 만들어지지 않은 새로운 객체를 만들 때 클래스를 만들 필요가 있다.

## Define a Class with class

문자열, 리스트, 딕셔너리 등을 만들기 위한 많은 built-in 클래스들이 존재한다. 파이썬의 커스텀 객체를 만들기 위해 먼저 *class* 키워드를 사용하여 클래스를 정의할 필요가 있다. 

사람에 과한 정보를 표현하기 위해 객체를 정의하기를 원한다고 가정하자. 각 객체는 하나의 사람을 나타낼 것이다. 


```python
class Person():
    pass # 클래스가 비었다는 것을 나타내기 위하여

someone = Person() 
```

Person클래스는 비었기 때문에 someone 객체는 아무것도 할 수 없다.

```
class Person():
    def __init__(self):
        pass
```
\_\_init\_\_()은 클래스 정의로부터 개별 객체를 초기화하는 메소드를 위한 특별한 파이썬 이름이다. self 인자는 개별 객체 그 자체를 명시한다.

클래스 정의에서 \_\_init\_\_()를 정의할 때, 첫 번째 매개변수는 self여야한다. 


```python
class Person():
    def __init__(self, name):
        self.name = name

hunter = Person('Elmer Fudd')
```

이런 새로운 객체는 list, tuple, dictionary, 또는 set의 성분으로 이용가능하다. 함수를 인자로서 전달하거나 결과로 반환할 수 있다.

name은 객체의 attribute로서 저장된다.

Person 클래스 정의 내에서 self.name으로 name에 접근할 수 있다. 실제 객체를 hunter로 만든다면 huter.name으로 접근이 가능하다.

모든 클래스 정의에서 \_\_init\_\_() 메소드를 가질 필요는 없다. 같은 클래스로부터 만들어지는 다른 객체들로부터 객체를 구분하기 위해 이용된다.

## Inheritance


```python
class Car():
    def exclaim(self):
        print("I'm a Car!")

class Yugo(Car):
    pass

give_me_a_car = Car()
give_me_a_yugo = Yugo()
give_me_a_car.exclaim()
give_me_a_yugo.exclaim()
```

    I'm a Car!
    I'm a Car!
    

## Override a Method


```python
class Person():
    def __init__(self, name):
        self.name = name

class MDPerson(Person):
    def __init__(self, name):
        self.name = "Doctor" + name
        
class JDPerson(Person):
    def __init__(self, name):
        self.name = name + ", Esquire"
```

## Get Help from Your Parent with super

부모클래스의 메소드를 호출하고 싶다면 어떻게 해야할까?


```python
class Person():
    def __init__(self, name):
        self.name = name

class EmailPersion(Person):
    def __init__(self, name, email):
        super().__init__(name)
        self.email = email
```

당신의 클래스의 \_\_init\_\_()메소드를 정의할 때, 부모 클래스의 \_\_init\_\_()메소드를 대체하고 그 후에는 더이상 자동으로 호출하지 못한다. 이것을 명시적으로 호출할 필요가 있다.

- super() 는 부모 클래스의 정의를 얻는다.
- \_\_init\_\_()메소드는 Person.\_\_init\_\_() 메소드를 호출한다. 이 메서드는 self 인자를 부모클래스에 전달한다. 그러므로 슈퍼 클래스에 어떤 선택적 인자를 제공하기만 하면 된다. 이 경우 Person()에서 받는 인자는 name이다.
- self.email = email라인은 Person과는 다른 EmailPerson새로운 코드다


```python
class EmailPerson(Person):
    def __init__(self, name, email):
        self.name = name
        self.email = email
```

물론 이와 같이 정의할 수는 있지만, 상속을 사용할 수 없게 된다.
만약 미래에 Person()의 정의를 변경한다면, super()는 Person의 속성과 메소드가 EmailPerson으로 상속되는 것을 보장한다.

자식클래스가 자신의 방식으로 사용되지만 여전히 부모로부터 뭔가를 할 필요가 있다면 super()를 사용해라.

## In self Defense

파이썬의 한 가지 비판은 self를 인스턴스 메소드의 첫 번째 인자로 포함시켜야 한다는 것이다. 파이썬은 올바른 객체의 속성과 메소드를 찾기 위해 self 인자를 사용한다. 예를 들어, 객체의 메소드를 어떻게 호출하고 파이썬이 실제로 어떻게 동작하는지 보여줄 것이다.


```python
car = Car()
car.exclaim()
```

    I'm a Car!
    


```python
Car.exclaim(car) 
```

    I'm a Car!
    

여전히 잘 동작하지만 굳이 긴 코드를 짤 이유는 없다.

## Get and Set Attribute Values with Properties

어떤 OOP에서는 외부로부터 바로 접근할 수 없는 private객체 속성을 지원한다. 프로그래머는 private속성의 값을 읽고 쓰기 위해 getter메서드와 setter 메서드를 사용한다.

파이썬에서는 getter나 setter메서드가 필요 없다. 왜냐하면 모든 속성과 메서드는 public이고, 우리가 예상한대로 쉽게 동작하기 때문이다. 만약 속성에 직접 접근하는 것이 부담스럽다면 getter와 setter메서드를 작성할 수 있다. 그러나 파이써닉하게 property를 사용하자.

이 예제에서는 hidden_name이라는 속성으로 Duck 클래스를 정의한다(다음 절에서 속성이 private를 유지하도록 이름을 짓는 더 좋은 방법을 배울 것이다). 이 속성을 외부에서 직접 접근하지 못하도록 getter(get_name())와 setter(set_name)) 메서드를 정의한다. 각 메서드가 언제 호출되는지 알기 위해 print()문을 추가한다. 마지막으로 이 메서드들을 name속성의 property로 정의한다.


```python
class Duck():
    def __init__(self, input_name):
        self.hidden_name = input_name
    def get_name(self):
        print('inside the getter')
        return self.hidden_name
    def set_name(self, input_name):
        print('inside the setter')
        self.hidden_name = input_name
    name = property(get_name, set_name)
```

새 메서드들은 마지막 라인 전까지 평범한 getter와 setter 메서드처럼 행동한다. 마지막 라인에서 두 메서드를 name이라는 속성의 프로퍼티로 정의한다. property()의 첫 번째 인자는 getter메서드, 두 번째 인자는 setter메서드다. 이제 Duck 객체의 name을 참조할 때 get_name()메서드를 호출해서 hidden_name 값을 반환한다.


```python
fowl = Duck('Howard')
fowl.name
```

    inside the getter
    




    'Howard'



여전히 보통의 getter메서드처럼 get_name() 메서드를 직접 호출할 수 있다.


```python
fowl.get_name()
```

    inside the getter
    




    'Howard'



name 속성에 값을 할당하면 set_name()메서드를 호출한다.


```python
fowl.name = 'Daffy'
fowl.name
```

    inside the setter
    inside the getter
    




    'Daffy'



set_name()메서드를 여전히 직접 호출할 수 있다.


```python
fowl.set_name('Daffy')
fowl.name
```

    inside the setter
    inside the getter
    




    'Daffy'



프로퍼티를 정의하는 또 다른 방법은 데커레이터를 사용하는 것이다. 다음 예제는 두 개의 다른 메서드를 정의한다. 각 메서드는 name()이지만, 서로 다른 데커레이터를 사용한다.

- getter메서드 앞에 @property 데커레이터를 쓴다.
- setter메서드 앞에 @name.setter 데커레이터를 쓴다.


```python
class Duck():
    def __init__(self, input_name):
        self.hidden_name = input_name
    @property
    def name(self):
        print('inside the getter')
        return self.hidden_name
    @name.setter
    def name(self, input_name):
        print('inside the setter')
        self.hidden_name = input_name
```


```python
fowl = Duck('Howard')
fowl.name
```

    inside the getter
    




    'Howard'




```python
fowl.name = 'Donald'
fowl.name
```


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    <ipython-input-60-c4e8e2fd515b> in <module>
    ----> 1 fowl.name = 'Donald'
          2 fowl.name
    

    AttributeError: can't set attribute


---

**NOTE**

어떤 사람이 hidden_name 속성을 알고 있다면, 그들은 fowl.hidden_name으로 이 속성을 바로 읽고 쓸 수 있다. 다음 절에서 private속성의 이름을 짓는 특수한 방법을 배울 것이다.

---


```python
fowl.hidden_name
```




    'Howard'



이전 예제에서 객체에 저장된 속성(hidden_name)을 참조하기 위해 name 프로퍼티를 사용했다. 또한 프로퍼티는 계산된 값을 참조할 수 있다. radius 속성과 계산된 diameter 프로퍼티를 가진 circle 클래스를 정의해보자.


```python
class Circle():
    def __init__(self, radius):
        self.radius = radius

    def diameter(self):
        return 2 * self.radius
```


```python
c = Circle(5)
c.radius
```




    5




```python
c.diameter
```




    <bound method Circle.diameter of <__main__.Circle object at 0x0000014E2A73D128>>




```python
c.diameter()
```




    10




```python
class Circle():
    def __init__(self, radius):
        self.radius = radius
    @property
    def diameter(self):
        return 2 * self.radius
```


```python
c = Circle(5)
c.radius
```




    5




```python
c.diameter
```




    10




```python
c.diameter()
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-39-55dafa7d2c23> in <module>
    ----> 1 c.diameter()
    

    TypeError: 'int' object is not callable



```python
c.radius = 7
c.diameter
```




    14



속성에 대한 setter프로퍼티를 명시하지 않는다면 외부로부터 이 속성을 설정할 수 없다. 이것은 read-only 속성이다.


```python
c.diameter = 20
```


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    <ipython-input-41-808ea3f73d1a> in <module>
    ----> 1 c.diameter = 20
    

    AttributeError: can't set attribute


직접 속성을 접근하는 것보다 프로퍼티를 통해서 접근하면 큰 이점이 있다. 예를 들어 만약 속성의 정의를 바꾸려면 모든 호출자를 수정할 필요 없이 클래스 정의에 있는 코드만 수정하면 된다.

## Name Mangling for Privacy

이전 절의 Duck 클래스 예제에서 숨겨진 hidden_name 속성을 호출했다. 파이썬은 클래스 정의 외부에서 볼 수 없도록 하는 속성에 대한 네이밍 컨벤션이있다. 속성 이름 앞에 두 언더스코어를 붙이면 된다.


```python
class Duck():
    def __init__(self, input_name):
        self.__name = input_name
    @property
    def name(self):
        print('insidte the getter')
        return self.__name
    @name.setter
    def name(self, input_name):
        print('inside the setter')
        self.__name = input_name
```


```python
fowl = Duck('Howard')
fowl.name
```

    insidte the getter
    




    'Howard'




```python
fowl.name = 'Donald'
```

    inside the setter
    


```python
fowl.__name
```


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    <ipython-input-71-39d081ea2ef8> in <module>
    ----> 1 fowl.__name
    

    AttributeError: 'Duck' object has no attribute '__name'



```python
fowl._Duck__name
```




    'Donald'



inside the getter를 출력하지 않았다. 비록 이것이 속성을 완벽하게 보호할 수는 없지만, 네임 맹글링은 속성의 의도적인 직접 접근을 어렵게 만들다.

## Method Types

클래스 정의에서 메서드의 첫 번째 인자가 self라면 이 메서드는 instance method다. 이것은 일반적인 클래스를 생성할 때의 메서드 타입이다. 인스턴스 메서드의 첫 번째 매개변수는 self고, 파이썬은 이 메서드를 호출할 때 객체를 전달한다.

이와 반대로 class method는 클래스 전체에 영향을 미친다. 클래스에 대한 어떤 변화는 모든 객체에 영향을 미친다. 클래스 정의에서 함수에 @classmethod 데커레이터가 있다면 이것은 클래스 메서드다. 또한 이 메서드의 첫 번째 매개변수는 클래스 자신이다. 파이썬에서는 보통 이 클래스의 매개변수를 cls로 쓴다. class는 예약어기 때문에 사용할 수 없다. A클래스에서 객체 인스턴스가 얼마나 만들어졌는지에 대한 클래스 메서드를 정의해보자.


```python
class A():
    count = 0
    def __init__(self):
        A.count += 1
    def exclaim(self):
        print("I'm an A!")
    @classmethod
    def kids(cls):
        print('A has', cls.count, 'little objects.')
```


```python
easy_a = A()
breezy_a = A()
wheezy_a = A()
A.kids()
```

    A has 3 little objects.
    

클래스 정의에서 메서드의 세 번째 타입은 클래스나 객체에 영향을 미치지 못한다. 이 메서드는 단지 편의를 위해 존재한다. 정적 메서드는 @staticmethod 데커레이터가 붙어 있고, 첫 번째 매개변수로 self나 cls가 없다. 


```python
class CoyoteWeapon():
    @staticmethod
    def commercial():
        print('This CoyoteWeapon has been brought to you by Acme')

CoyoteWeapon.commercial()
```

    This CoyoteWeapon has been brought to you by Acme
    

이 메서드를 접근하기 위해 CoyoueWeapon클래스에서 객체를 생성할 필요가 없다.

## Duck Typing

파이썬은 다형성을 느슨하게 구현했다. 이것은 클래스에 상관없이 같은 동작을 다른 객체에 적용할 수 있다는 것을 의미한다.

세 Quote 클래스에서 같은 \_\_init\_\_()이니셜라이저를 사용해보자. 클래스에 다음 두 메서드를 추가한다.

- who()메서드는 저장된 person 문자열의 값을 반환한다.
- says()메서드는 특정 구두점과 함께 저장된 words 문자열을 반환한다.


```python
class Quote():
    def __init__(self, person, words):
        self.person = person
        self.words = words
    def who(self):
        return self.person
    def says(self):
        return self.words + '.'
    
class QuestionQuote(Quote):
    def says(self):
        return self.words + '?'
    
class ExclamationQuote(Quote):
    def says(self):
        return self.words + '!'
```

QuestionQuote와 ExclamationQuote클래스에서 초기화 함수를 쓰지 않았다. 그러므로 부모의 \_\_init\_\_()메서드를 오버라이드하지 않는다. 파이썬은 자동으로 부모 클래스 Quote의 \_\_init\_\_()메서드를 호출해서 인스턴스 변수 person과 words를 저장한다. 그러므로 서브클래스 QuestionQuote와 ExclamationQuote에서 생성된 객체의 self.words에 접근할 수 있다.


```python
hunter = Quote('Elmer Fudd', "I'm hunting wabbits")
print(hunter.who(), 'says:', hunter.says())
```

    Elmer Fudd says: I'm hunting wabbits.
    


```python
hunted1 = QuestionQuote('Bugs Bunny', "What's up, doc")
print(hunted1.who(), 'says:', hunted1.says())
```

    Bugs Bunny says: What's up, doc?
    


```python
hunted2 = ExclamationQuote('Daffy Duck', "It's rabbit season")
print(hunted2.who(), 'says:', hunted2.says())
```

    Daffy Duck says: It's rabbit season!
    

세 개의 서로 다른 says()메서드는 세 클래스에 대해 서로 다른 동작을 제공한다. 이것은 객체 지향 언어에서 전통적인 다형성의 특징이다. 더 나아가 파이선은 who()과 says()메서드를 갖고 있는 모든 객체에서 이 메서드들을 실행할 수 있게 해준다. Quote클래스와 관계없는 BabblingBrook클래스를 정의해보자.


```python
class BabblingBrook():
    def who(self):
        return 'Brook'
    def says(self):
        return 'Babble'
    
brook = BabblingBrook()
```

다양한 객체의 who()와 says()메서드를 실행해보자. brook객체는 다른 객체와 전혀 관계가 없다.


```python
def who_says(obj):
    print(obj.who(), 'says', obj.says())

who_says(hunter)
```

    Elmer Fudd says I'm hunting wabbits.
    


```python
who_says(hunted1)
```

    Bugs Bunny says What's up, doc?
    


```python
who_says(brook)
```

    Brook says Babble
    

## Special Methods


```python
class Word():
    def __init__(self, text):
        self.text = text
        
    def equals(self, word2):
        return self.text.lower() == word2.text.lower()
```


```python
first = Word('ha')
second = Word('HA')
third = Word('eh')
```


```python
first.equals(second)
```




    True




```python
class Word():
    def __init__(self, text):
        self.text = text
    def __eq__(self, word2):
        return self.text.lower() == word2.text.lower()
```


```python
first == second
```




    True




```python
first == third
```




    False



![image.png](attachment:image.png)

![image.png](attachment:image.png)

\_\_init\_\_()외에도 \_\_str\_\_()을 사용하여 객체를 문자열로 출력하는 우리만의 메서드를 만들 수 있다. 7장에서 배울 string formatter와 print(), str()을 사용하면 된다. 대화식 인터프리터는 변수의 결과를 출력하기 위해 \_\_repr\_\_()함수를 사용한다. \_\_str\_\_()또는 \_\_repr\_\_()을 정의하지 않으면 기본 문자열을 출력한다.


```python
first = Word('ha')
first
```




    <__main__.Word at 0x1c79d500c88>




```python
print(first)
```

    <__main__.Word object at 0x000001C79D500C88>
    

\_\_str\_\_()과 \_\_repr\_\_()메서드를 추가하여 Word클래스를 예쁘게 만들어보자.


```python
class Word():
    def __init__(self, text):
        self.text = text
    def __eq__(self, word2):
        return self.text.lower() == word2.text.lower()
    def __str__(self):
        return self.text
    def __repr__(self):
        return "Word('" + self.text + "')"
    
first = Word('ha')
```


```python
first
```




    Word('ha')




```python
print(first)
```

    ha
    

## Composition


```python
class Bill():
    def __init__(self, description):
        self.description = description
        
class Tail():
    def __init__(self, length):
        self.length = length
        
class Duck():
    def __init__(self, bill, tail):
        self.bill = bill
        self.tail = tail
    def about(self):
        print('This duck has a', self.bill.description, 'bill and a', self.tail.length, 'tail')
```


```python
tail = Tail('long')
bill = Bill('wide orange')
duck = Duck(bill, tail)
duck.about()
```

    This duck has a wide orange bill and a long tail
    


```python
duck
```




    Duck(bill='wide orange', tail='long')



## When to Use Classes and Objects versus Modules

코드에서 클래스와 모듈의 사용 기준은 다음과 같다. 

- 비슷한 행동(메서드)을 하지만 내부 상태(속성)가 다른 개별 인스턴스가 필요할 때, 객체는 매우 유용하다.
- 클래스는 상속을 지원하지만, 모듈은 상속을 지원하지 않는다.
- 어떤 한 가지 일만 수행한다면 모듈이 가장 좋은 선택일 것이다. 프로그램에서 파이썬 모듈이 참조된 횟수에 상관없이 단 하나의 복사본만 불러온다.
- 여러 함수에 인자로 전달될 수 있는 여러 값을 포함한 여러 변수가 있다면, 클래스를 정의하는 것이 더 좋다. 예를 들어 화상 이미지를 나타내기 위해 size나 color를 딕셔너리의 키로 사용한다고 가정해보자. 프로그램에서 각 이미지에 대한 딕셔너리를 생성하고, scale()과 transform()같은 함수에 인자를 전달할 수 있다. 키와 함수를 추가하면 코드가 지저분해 질 수도 있다. size나 color를 속성으로 하고 scale()과 transform()을 메서드로 하는 이미지 클래스를 정의하는 것이 더 일관성 있다. 색상 이미지에 대한 모든 데이터와 메서드를 한 곳에 정의할 수 있기 때문이다.
- 가장 간단한 문제 해결 방법을 사용한다. 딕셔너리, 리스트, 튜플은 모듈보다 더 작고, 간단하며, 빠르다. 그리고 일반적으로 모듈은 클래스보다 더 간단하다.

### Named Tuples

named tuple은 튜플의 서브클래스이다. 이름(.name)과 위치(\[offset\])로 값에 접근할 수 있다.

이전 절의 예제를 활용하자 Duck클래스르 네임드 튜플로, bill과 tail을 간단한 문자열 속성으로 변환된다. 그리고 두 인자를 취하는 namedtuple 함수를 호출한다.

- 이름
- 스페이스로 구분된 필드 이름이 문자열 

파이썬에서 네임드 튜플은 자동으로 지원되지 않는다. 그래서 네임드 튜플을 쓰기 전에 모듈을 불러와야 한다. 다음 예제의 첫 번째 줄에서 namedtuple을 불러오고 있다.


```python
from collections import namedtuple
Duck = namedtuple('Duck', 'bill tail')
duck = Duck('wide orange', 'long')
duck
```




    Duck(bill='wide orange', tail='long')




```python
duck.bill
```




    'wide orange'




```python
duck.tail
```




    'long'




```python
parts = {'bill': 'wide orange', 'tail': 'long'}
duck2 = Duck(**parts)
duck2
```




    Duck(bill='wide orange', tail='long')




```python
duck2 = Duck(bill = 'wide orange', tail = 'long') # 위와 같음
```

네임드 튜플은 불변한다. 하지만 필드를 바꿔서 또 다른 네임드 튜플을 반환할 수 있다.


```python
duck3 = duck2._replace(tail = 'magnificent', bill = 'curshing')
duck3
```




    Duck(bill='curshing', tail='magnificent')




```python
duck_dict = {'bill': 'wide orange', 'tail':'long'}
duck_dict['color'] = 'green'
duck_dict
```




    {'bill': 'wide orange', 'tail': 'long', 'color': 'green'}




```python
duck.color = 'green' # 딕셔너리는 네임드 튜플이 아니다.
```


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    <ipython-input-33-fc3f0fde081c> in <module>
    ----> 1 duck.color = 'green' # 딕셔너리는 네임드 튜플이 아니다.
    

    AttributeError: 'Duck' object has no attribute 'color'


정리하면, 네임드 튜플의 특징은 다음과 같다.

- 불변하는 객체처럼 행동한다.
- 객체보다 공간 효율성과 시간 효율성이 더 좋다.
- 딕셔너리 형식의 괄호대신, 점 표기법으로 속성을 접근할 수 있다.
- 네임드 튜플을 딕셔너리의 키처럼 쓸 수 있다.

# Mangle Data Like a Pro

## Format

### New style formatting with {} and format


```python
n = 42
f = 7.03
s = 'string cheese'
```


```python
'{} {} {}'.format(n, f, s)
```




    '42 7.03 string cheese'




```python
'{2} {0} {1}'.format(f, s, n)
```

인자는 딕셔너리 혹은 이름을 지정한 인자가 될 수 있다. 그리고 지정자는 그들의 이름을 포함할 수 있다.


```python
'{n} {f} {s}'.format(n = 42, f = 7.03, s = 'string cheese')
```


```python
d = {'n': 42, 'f': 7.03, 's': 'string cheese'}
'{0[n]} {0[f]} {0[s]} {1}'.format(d, 'other')
```

:다음에 타입 지정자를 입력한다.


```python
'{0:d} {1:f} {2:s}'.format(n, f, s)
```




    '42 7.030000 string cheese'




```python
'{n:d} {f:f} {s:s}'.format(n = 42, f = 7.03, s = 'string cheese')
```




    '42 7.030000 string cheese'




```python
'{0:10d} {1:10f} {2:10s}'.format(n, f, s) # 최소 필드 길이가 10이고, 오른쪽 정렬
```




    '        42   7.030000 string cheese'



\> 기호는 오른쪽 정렬에 대해 더 명확하게 해준다.


```python
'{0:>10d} {1:>10f} {2:>10s}'.format(n, f, s)
```




    '        42   7.030000 string cheese'




```python
'{0:<10d} {1:<10f} {2:<10s}'.format(n, f, s)
```




    '42         7.030000   string cheese'




```python
'{0:^10d} {1:^10f} {2:^10s}'.format(n, f, s) # 중앙 정렬
```




    '    42      7.030000  string cheese'




```python
'{0:>10d} {1:>10.4f} {2:>10.4s}'.format(n, f, s)
```




    '        42     7.0300       stri'



마지막으로 문자를 채워 넣어 보자. :이후에, 그리고 정렬 혹은 길이 지정자 이전에 채워 넣고 싶은 문자를 입력하면 된다.


```python
'{0:!^20s}'.format('BIG SALE')
```




    '!!!!!!BIG SALE!!!!!!'



## Match with Regular Expressions


```python
import re


result = re.match('You', 'Young Frankenstein')
result
```




    <re.Match object; span=(0, 3), match='You'>



'You'는 pattern이고, 'Young Frankenstein'은 확인하고자 하는 문자열 source다. match()는 소스와 패턴의 일치 여부를 확인한다. 좀 더 복잡한 방법으로, 나중에 패턴 확인을 빠르게 하기 위해 패턴을 먼저 컴파일할 수 있다.


```python
youpattern = re.compile('You')
```

그리고 나서 컴파일된 패턴으로 패턴의 일치 여부를 확인할 수 있다.


```python
result = youpattern.match('Young Frankenstein')
```

match()가 패턴과 소스를 비교하는 유일한 방법은 아니다. 다른 메서드를 살펴보자. 

- search()는 첫 번쨰 일치하는 객체를 반환한다.
- findall()은 중첩에 상관없이 모두 일치하는 문자열 리스트를 반환한다.
- split()은 패턴에 맞게 소스를 쪼갠 후 문자열 조각의 리스트를 반환한다.
- sub()는 대체 인자를 하나 더 받아서, 패턴과 일치하는 모든 소스 부분을 대체 인자로 변경한다.

### Exact match with match()


```python
import re
source = 'Young Frankenstein'
m = re.match('You', source) # match객체거나 None이거나
if m:
    print(m.group())
```

    You
    


```python
m = re.match('.*Frank', source)
if m:
    print(m.group())
```

    Young Frank
    

- .은 한 문자를 의미한다.
- \*는 이전 패턴이 여러 개 올 수 있다는 것을 의미한다. 그러므로 .\*는 0회 이상의 문자가 올 수 있다는 것을 의미한다.
- Frank는 포함되어야 할 문구를 의미한다.

### First match with search()


```python
m = re.search('Frank', source)
if m:
    print(m.group())
```

    Frank
    

### All matches with findall()


```python
m = re.findall('n', source)
m
```




    ['n', 'n', 'n', 'n']




```python
m = re.findall('n.', source)
m # 마지막 n이 포함되지않음
```




    ['ng', 'nk', 'ns']




```python
m = re.findall('n.?', source) # 0또는 1회
m
```




    ['ng', 'nk', 'ns', 'n']



### Split at matches with split()


```python
m = re.split('n', source)
m
```




    ['You', 'g Fra', 'ke', 'stei', '']



### Replace at matches with sub()

`replace()` 메서드와 비슷하지만, 리터럴 문자열이 아닌 패턴을 사용한다.


```python
m = re.sub('n', '?', source)
m
```




    'You?g Fra?ke?stei?'



### Patterns: special characters

- 리터럴은 모든 비특수 문자와 일치한다.
- \n을 제외한 하나의 문자:.
- 0회 이상: \*
- 0 또는 1회: ?

<img style="float: left;" src="figure1.PNG" width = 500>


```python
import string
printable = string.printable
printable
```




    '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~ \t\n\r\x0b\x0c'




```python
re.findall('\d', printable)
```




    ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9']




```python
re.findall('\w', printable)
```




    ['0',
     '1',
     '2',
     '3',
     '4',
     '5',
     '6',
     '7',
     '8',
     '9',
     'a',
     'b',
     'c',
     'd',
     'e',
     'f',
     'g',
     'h',
     'i',
     'j',
     'k',
     'l',
     'm',
     'n',
     'o',
     'p',
     'q',
     'r',
     's',
     't',
     'u',
     'v',
     'w',
     'x',
     'y',
     'z',
     'A',
     'B',
     'C',
     'D',
     'E',
     'F',
     'G',
     'H',
     'I',
     'J',
     'K',
     'L',
     'M',
     'N',
     'O',
     'P',
     'Q',
     'R',
     'S',
     'T',
     'U',
     'V',
     'W',
     'X',
     'Y',
     'Z',
     '_']




```python
re.findall('\s', printable)
```




    [' ', '\t', '\n', '\r', '\x0b', '\x0c']



### Patterns: using specifiers

정규표현식에 나타낸 패턴 지정자를 사용할 수 있다.

<img style="float: left;" src="figure2.PNG" width = 500>


```python
source = '''I wish I may, I wish I might
Have a dish of fish tonight.'''
print(re.findall('wish', source))
print(re.findall('wish|fish', source))
print(re.findall('^wish', source))
print(re.findall('^I wish', source))
print(re.findall('fish$', source))
print(re.findall('fish tonight.$', source))
```

    ['wish', 'wish']
    ['wish', 'wish', 'fish']
    []
    ['I wish']
    []
    ['fish tonight.']
    

문자 ^와 \$는 anchor라고 부른다. ^는 검색 문자열의 시작 위치에, \$는 검색 문자열의 마지막 위치에 고정한다. 그리고 .\$는 가장 마지막에 있는 한 문자와 .을 매칭한다. 더 정확하게 하려면 문자 그대로 매칭하기 위해 .에 이스케이프 문자를 붙여야 한다.


```python
print(re.findall('fish tonight\.$', source))
print(re.findall('[wf]ish', source))
print(re.findall('[wsh]+', source))
print(re.findall('ght/W', source))
print(re.findall('I (?=wish)', source))
print(re.findall('(?<=I) wish', source))
```

    ['fish tonight.']
    ['wish', 'wish', 'fish']
    ['w', 'sh', 'w', 'sh', 'h', 'sh', 'sh', 'h']
    []
    ['I ', 'I ']
    [' wish', ' wish']
    


```python
re.findall('\bfish', source)
```




    []



파이썬 문자열에서는 \b는 backspace를 의미하지만, 정규표현식에서는 단어의 시작 부분을 의미한다. 정규표현식의 패턴을 입력하기 전에 항상 문자 r(raw string)을 입력하라. 그러면 파이썬의 이스케이프 문자를 사용할 수 없게 되므로 실수로 이스케이프 문자를 사용하여 충돌이 일어나는 것을 피할 수 있게 된다. 


```python
re.findall(r'\bfish', source)
```




    ['fish']



### Patterns: specifying match output

`match()`또는 `search()`를 사용할 때 모든 매칭은 `m.group()`과 같이 객체 m으로부터 결과를 반환한다. 만약 패턴을 괄호로 둘러싸는 경우, 매칭은 그 괄호만의 그룹으로 저장된다. 그리고 다음과 같이 `m.groups()`를 사용하여 그룹의 튜플을 출력한다.


```python
m = re.search(r"(. dish\b).*(\bfish)", source)
print(m.group())
print(m.groups())
```

    a dish of fish
    ('a dish', 'fish')
    

만약 (?P\< name \> expr)패턴을 사용한다면, 표현식이 매칭되고, 그룹 이름의 매칭 내용이 저장된다.


```python
m = re.search(r'(?P<DISH>. dish\b).*(?P<FISH>\bfish)', source)
print(m.group())
print(m.groups())
print(m.group('DISH'))
print(m.group('FISH'))
```

    a dish of fish
    ('a dish', 'fish')
    a dish
    fish
    
