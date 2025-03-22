
## python 의 언패킹

python 의 언패킹 (unpacking) 에 대해서 좀 더 알아보겠습니다. 먼저 간단한 예시로부터 시작합니다.

```
a, b = [ 1, 2 ]
```

대입문 우변(rhs, right-hand side)은 두 요소로 이루어진 리스트이며, 좌변(lhs, left-hand side)은 대입이 가능한 변수 입니다.

동작은, 리스트의 각 요소가 좌변의 개개 변수에 대입됩니다. 아래 두 대입문과 동일합니다.

```
a = 1
b = 2
```

공식 문서에서는 값이 저장이 되는 lhs 쪽을 target (또는 target-list) 라고 부릅니다.


##  공식 문서에서의 정의

python 의 공식 레퍼런스 문서에서는 대입문 (assignment statement) 섹션에서 이를 설명하고 있습니다.

한글: <https://docs.python.org/ko/3.13/reference/simple_stmts.html#assignment-statements>

대입문의 형태는 매우 복잡해 보입니다. 수 많은 다양한 대입 동작을 모두 다 설명하기 위해서입니다. 지금은 언패킹 에만 관심이 있으니, 이 경우로만 국한해서 좀 간단하게 표현해 보겠습니다.

```
<target> = <표현식>  # 단일 대입
<target> = <target> = <표현식> # 복수 대입
```


```
<target> = <expression>
```

>
> 타겟 목록에 개체를 할당하는 것은 다음과 같이 재귀적으로 정의됩니다. ..
>
> If (선택적으로 둥근 괄호 안에 있는) 타겟 목록이 후행 쉼표가 없는 단일 대상인 경우, ..
> Else:
>     If 타겟 목록에 별표가 앞에 붙은 타겟이 하나 있는 경우, ..
>     Else:
>         객체는 타겟 목록에 있는 타겟 수와 동일한 수의 항목이 있는 이터러블이어야 하며, 항목들은 왼쪽에서 오른쪽으로 해당 타겟들에 할당됩니다.
>

언패킹에 대해 설명하는 부분은 위 발췌 부분의 맨 마지막 케이스에 규정되어 있습니다.
- 객체는 타겟 목록에 있는 타겟 수와 동일한 수의 항목이 있는 이터러블이어야 하며, 항목들은 왼쪽에서 오른쪽으로 해당 타겟들에 할당됩니다.




## Unpacking 대상이 될 수 있는 유형 (Iterable, Iterator)
Python의 공식 문서에서 iterable을 다음과 같이 정의합니다:

> * Iterable: __iter__() 또는 __getitem__()을 구현한 객체
> * Iterator: __next__() 메서드를 가지고 있으며, iterable 자체도 된다.
>

unpacking에 대한 규격은 Python Reference 문서의 Assignment 문법에서 정의됩니다:
>
> Assignment statements - Python 3.12 Reference
>
특히, 아래 부분이 중요합니다:

> If the target is a sequence (as defined in the Python glossary), the corresponding value must be an iterable.
(타겟이 시퀀스일 경우, 대응하는 값은 반드시 iterable이어야 한다.)

즉, a, b = [1, 2] 같은 unpacking이 가능하려면 오른쪽 값이 iterable이어야 함을 명확하게 규정하고 있습니다.



## Iterable 정의
Iterable 의 정의는 Python Glossary 에서 찾을 수 있습니다:

> Iterable: An object capable of returning its members one at a time.
It must implement either __iter__() or __getitem__().

즉, iterable 한 객체는 for 루프에서 순회할 수 있고, iter(obj) 를 호출하면 iterator 가 생성되는 객체입니다. 리스트, 튜플, 세트, 문자열, 사전 등이 여기에 속합니다.



### Dictionary 언패킹 (**dict) 은 어떻게 가능한가?
일반적인 unpacking 에서 `a, b = {"x": 1, "y": 2}` 를 수행하면 keys 가 unpacking 됩니다.

```
a, b = {"x": 1, "y": 2"}  # a = "x", b = "y"
```

이것이 가능한 이유는 dictionary는 iterable 이기 때문입니다.
dict 객체는 __iter__() 를 구현하며, iter(dict) 를 호출하면 key 들이 반환되는 iterator 가 생성됩니다.



### Python 공식 문서에서 확인할 수 있는 부분

> Dictionary Views - Python 3.12 Documentation
> Data Model - Python 3.12
> Dictionaries implement the __iter__() method to iterate over keys by default.
>

즉, dictionary 는 기본적으로 keys 를 반환하는 iterable 이므로, a, b = some_dict 형태의 unpacking 이 가능합니다.

### 사전의 key-value 동시 언패킹

사전의 key-value 를 한꺼번에 unpacking 하려면 `**dict` 를 사용합니다.

참고: Python의 Function Call 문법
이 부분의 내용은 나중에 시간이 되면 다시 정리하겠습니다.



### 정리

- a, b = iterable 형태의 unpacking 이 가능하려면, 오른쪽 값이 iterable 이어야 합니다.
- dictionary는 기본적으로 iterable이며, iter(dict)를 호출하면 key iterator가 생성되므로, a, b = some_dict 시 key만 unpacking 됩니다.

<br>


## Unpacking 에 대한 추가 설명
Unpacking 은 파이썬에서 시퀀스(sequence) 자료형의 요소를 여러 개의 변수에 한 번에 할당하는 강력한 기능입니다. "A, B = <unpacking 되는 대상>" 과 같은 형태로 사용되며, 코드를 간결하고 가독성 좋게 만들어줍니다.

### Unpacking 가능한 대상
Unpacking은 다양한 시퀀스 자료형에 적용될 수 있습니다.

리스트(List):
```
my_list = [1, 2, 3]
a, b, c = my_list  # a = 1, b = 2, c = 3
```

튜플(Tuple):
```
my_tuple = (4, 5, 6)
d, e, f = my_tuple  # d = 4, e = 5, f = 6
```

문자열(String):
```
my_string = "hello"
h, e, l, l, o = my_string  # h = 'h', e = 'e', l = 'l', l = 'l', o = 'o'
```

집합(Set):
```
my_set = {7, 8, 9}
g, h, i = my_set  # g = 7, h = 8, i = 9 (순서는 보장되지 않음)
```
딕셔너리(Dictionary):<br>
딕셔너리를 그냥 unpacking 하면 키(key) 가 변수에 할당됩니다.
```
my_dict = {"apple": 1, "banana": 2}
k1, k2 = my_dict  # k1 = "apple", k2 = "banana"
```
값을 unpacking 하려면 my_dict.values() 를 사용합니다.

```
val1, val2 = my_dict.values()  # val1 = 1, val2 = 2
```

Iterator:<br>
map(), filter(), zip() 등의 함수는 iterator를 반환합니다. 이 iterator 역시 unpacking을 통해 각 요소를 변수에 할당할 수 있습니다.
```
a, b = map(int, input().split())
```
<br>

### Unpacking 시 주의사항

* 변수의 개수와 요소의 개수 일치: unpacking 시 변수의 개수는 unpacking 대상의 요소 개수와 정확히 일치해야 합니다. 그렇지 않으면 ValueError 가 발생합니다.
* 순서: 시퀀스 자료형 (리스트, 튜플, 문자열) 의 경우 unpacking 은 요소의 순서대로 변수에 할당됩니다.
* 집합(Set): 집합은 순서가 없으므로 unpacking 시 어떤 요소가 어떤 변수에 할당될지는 알 수 없습니다.

### Unpacking의 활용
* 여러 값 동시 할당: 여러 변수에 동시에 값을 할당할 수 있습니다.
* 함수 반환 값 처리: 함수가 여러 개의 값을 반환할 때 unpacking을 사용하여 각 값을 개별 변수에 할당할 수 있습니다.
* 반복문: 반복문에서 unpacking 을 사용하여 각 요소에 쉽게 접근할 수 있습니다.

