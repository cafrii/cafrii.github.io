---
layout: post
title: "Python 의 iterable 과 iterator"
category: python
tags: iterator iterable
---


## Iterable, Iterator

일단 iterate 라는 용어를 어떻게 번역할 것인지 부터 정리하고 시작합니다. 직역하면 "반복하다" 입니다. repeat 라는 말도 반복하다 로 번역되지만, python 의 영역에서라면 혼동의 여지가 거의 없으니 "반복"이라는 단어는 나쁘지 않은 선택입니다. 그러면, iterable 은 "반복가능한" 으로 번역 가능하고, iterator 는 "반복자" 로 번역을 해 볼 수 있습니다.

그런데, '반복자' 라는 단어는 제게는 좀 어색한 표현으로 느껴집니다. 이미 많은 번역서에서 반복자 라는 단어를 사용하고 있으므로, 개발자 집단에서 이를 특히 문제시 하진 않는 것 같습니다만.. 일단 이 문서에서는 원문을 쓰거나 아니면 소리나는 그대로 '이터러블', '이터레이터' 라고 적겠습니다.

### Iterable
Iterable 은 (어미가 ~able 이니까) '~할 수 있는' 능력에 대한 일종의 '자격' 인데, "반복적으로 무엇인가를 수행할 수 있다"는 것이 자격 조건입니다.
여기서 그 "수행 할 무언가" 는 보통은 집합 (컨테이너) 에서 요소를 하나씩 순회(traverse)하며 리턴하는 경우가 대부분이지만, 꼭 그래야 할 필요는 없습니다. 누구를 대상으로 수행하는지, 어떤 순서로 수행하는지, 수행의 결과는 어떠해야 하는지 등은 자격의 조건에는 없습니다

"iterable" 의 자격을 좀 기술적으로 표현해 본다면, "`__iter__()` 라는 메소드를 지원한다" 라는 말과 같습니다. 어떠한 객체가 `__iter__()` 메소드를 구현하고 있으면 그 객체는 iterable 입니다. `__iter__()` 메소드는 iterator 객체를 리턴해야 햡니다.

메소드가 아닌 외부에서 호출할 때는 `iter()` 함수를 사용해야 합니다. 즉, `iter(이터러블)` 과 같이 호출하면 이터레이터가 리턴됩니다.


### Iterator
반복하는 동작을 수행하는 객체를 iterator 라고 합니다. 파이썬 설계자는, 이터러블한 객체가 바로 반복 작업을 수행 하도록 설계하지 않고, 이터러블 객체와 이터레이터 객체를 별도의 개념으로 구분해 놓았습니다. 하지만 이터러블 한 객체 자신이 바로 이터레이터 일 수도 있는 것을 굳이 배제하지 않습니다.

객체 A가 iterable 하다는 말은, A.iter() 를 호출할 수 있다는 말이고, A.iter() 의 호출 결과로 리턴되는 iterator 가 바로 A 자신과 동일한 객체일 수도 있습니다.

반복 동작은 next()라는 함수를 이용합니다. next() 함수의 인자로 이터레이터를 지정하면 지정한 이터레이터가 정한 규칙에 따라서 어떤 객체들이 순회 됩니다. 내부적으로는 해당 객체의 `__next__()` 메소드가 호출됩니다.

다시 말하자면, `__next__()` 메소드를 구현하는 객체는 이터레이터 라고 말할 수 있습니다.

### 커스텀 이터레이터
이터러블 한 객체를 직접 만들 수 있습니다. 이터러블이 '자격' 이라고 했으니 자격의 조건을 갖추게끔 하면 됩니다.

```python
# 이터레이터 객체 클래스 예시
class Iterator:
    def __init__(self, begin, end):
        # 지정한 범위의 숫자를 하나씩 리턴해 주는 이터레이터
        self.current = begin
        self.end = end

    def __iter__(self):
        return self

    def __next__(self):
        if self.current <= self.end:
            num = self.current
            self.current += 1
            return num
        else:
            raise StopIteration


# 이터레이터 사용 예
it = Iterator(1, 3)
print(it.__next__())
print(it.__next__())
print(next(it)) # 위 문장과 같은 효과

# 한번 더 호출하면 StopIteration 예외 발생.
try:
    print(next(it))
except Exception as e:
    print('exception:', e.__class__)
```

위 사용 에에서도 나와 있는 것 처럼, 클래스 외부에서는 `__next__()` 메소드를 사용하는 대신 `next()` 전역 함수를 사용합니다.

이터레이터를 구현할 때, `__next__()` 메소드만 있으면 될 것 같은데, 왜 `__iter__()` 까지 구현해야만 하는지는 뒤에서 설명합니다.

다음은 커스텀 이터러블을 구현한 예시 입니다.

```python
# 이터러블한 객체를 만드는 클래스 예시
class Iterable:
    def __init__(self, begin, end):
        self.begin = begin
        self.end = end

    def __iter__(self):
        return Iterator(self.begin, self.end)

# 이터러블 객체를 생성하고
itbl = Iterable(4, 6)

# 이터레이터를 얻음
it = itbl.__iter__()

while True:
    try:
        print(it.__next__())
    except StopIteration:
        break

```






## 모든 이터레이터는 이터러블

파이썬 규격에서는, "모든 iterator 는 iterable 이다" 라고 규정되어 있습니다.
이는 파이썬의 디자인 원칙 중 하나인데, `iter(iterator)` 를 호출하면 그 자체를 반환하도록 설계되어 있기 때문입니다.

언어 버전 3.8 규격에서는 Data Model 섹션에서 __iter__ 메소드에 대한 내용에서 확인 가능합니다.

<https://docs.python.org/3.8/reference/datamodel.html#object.__iter__>

> 3 Data model <br>
> 3.3. Special method names <br>
> 3.3.7. Emulating container types <br>
>
> object.__iter__(self) <br>
> ... <br>
>   Iterator objects also need to implement this method (__iter__); they are required to return themselves.
>

최신 규격 (3.12) 에서는 (무슨 이유에서인지는 잘 모르겠지만) 위 문장이 삭제된 대신, 라이브러리 레퍼런스의 표준 타입 부분에서 설명하고 있습니다.

<https://docs.python.org/ko/3.12/library/stdtypes.html#iterator-types>

```
이터레이터 기능을 제공하려면 아래의 메서드가 구현되어야 합니다.

- container.__iter__()
  - 이터레이터 객체를 리턴합니다. ...

위의 __iter__()에 의해 리턴되는 이터레이터 객체는 그 자신이 아래의 두 메서드를 지원해야 합니다. ..

- iterator.__iter__()
  - 이터레이터 객체 자신을 리턴합니다. for 나 in 구문에서 사용되기 위해서 입니다.

- iterator.next()
  - 컨테이너 에서 다음 아이템을 리턴합니다. ..
```

앞에서 `__iter__()` 를 구현하고 있는 객체를 '이터러블' 이라고 정의하였는데, 여기서 다시 이터레이터는 `__iter__()` 를 자기 자신을 리턴하게끔 구현해야 한다고 규정하고 있으니, 결론적으로는 "이터레이터는 이터러블 하다"고 볼 수 있게 됩니다.

이를 코드로 확인해 보겠습니다.

``` python
it1 = iter([3, 4, 5])  # it1 은 iterator
print(iter(it1) is it1)  # True. iterator는 자기 자신을 반환함.
```

예제에서 확인할 수 있듯이, iter(it1) 은 it1 그 자신을 리턴합니다.

참고로, 이터레이터로부터 iter() 호출을 통해 이터레이터를 다시 얻는다고 하더라도, 다시 처음부터 이터레이션 할 수 없습니다.
리턴되는 이터레이션 는 그 자신이기 때문에, 이터레이션 중인 이터레이터는 __next__() 는 현재 요소의 다음 요소를 리턴할 것입니다.
새 이터레이터는 반드시 원래의 이터러블 로부터 다시 __iter__() 를 통해 얻어야 합니다.

<br>


## 모든 이터러블이 다 이터레이터는 아님

반면, 모든 iterable 이 다 iterator 는 아닙니다.

리스트 (list), 튜플 (tuple), 사전 (dict) 등의 컨테이너 타입은 iterable 이지만, 그 자체로 iterator 는 아닙니다.
이러한 객체들은 iter() 를 호출해야만 iterator 가 '생성' 되어 반환 됩니다.

아래 코드는 이러한 내용을 잘 보여줍니다.

``` python
lst = [1, 2, 3]
print(iter(lst) is lst)  # False. 리스트는 이터레이터가 아님
```


### 팁

다음 키워드를 이용하여 구글 이미지 검색을 해 보면 관계를 잘 설명해 놓은 그림들을 쉽게 찾아볼 수 있습니다.
- Iterables vs. Iterators vs. Generators
- 예시
  - https://nvie.com/posts/iterators-vs-generators/
  - ..


끝


