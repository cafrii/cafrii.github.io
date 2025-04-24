---
layout: post
title: "Python 의 프로퍼티와 디스크립터"
category: python
tags: property descriptor
---

# property 와 descriptor

날짜: 2025/04/25
태그: 파이썬

#### ✅ Note
- 본 포스트는 아래 notion 으로 작성된 것을 export 한 것입니다.
  - [Python 의 프로퍼티와 디스크립터](https://eight-cicada-d69.notion.site/property-descriptor-1dfa821bbd72808ca154d391653bf921)
  - export 날짜: 2025.4.25
- export 과정에서 일부 레이아웃이 틀어졌습니다. 원본을 보시길 권장합니다.

<br>

# 배경

최근에 pytorch 를 이용한 ML/DL 공부를 하는 중 입니다. 이 블로그에 작성된 포스트들은 모두 이 ML 스터디 중에 좀 더 이해할 수 있게 된 파이썬의 숨은(?) 기능들 중 메모 할 만한 내용들을 정리한 것입니다.

pandas 의 Series 를 보던 중, 다음과 같이 내부의 .array 속성을 통해 직접 ndarray 객체에 접근하는 코드를 보게 되었습니다. 그런데 이 array 의 타입을 확인해 보니, 처음 듣는 생소한 타입입니다.

```python
df = pd.DataFrame({'fruit':['apple', 'orange'], 'count':[20,30]})
df.fruit.array
# <NumpyExtensionArray>
# ['apple', 'orange'] ...
```

게다가 df.fruit (Series 타입의 객체) 내부의 `__dict__` 에도 관찰되지 않았습니다.

```python
print(df.fruit.__dict__.keys())
# dict_keys(['_is_copy', '_mgr', '_item_cache', '_attrs', '_flags', '_name', '_cacher'])
```

결론을 먼저 얘기하자면, array 라는 속성은, 인스턴스의 속성(attribute)이 아니라, 클래스의 프로퍼티(property)로 구현되어 있기 때문입니다.

이번 포스트에서는, 파이썬의 프로퍼티에 대해 검토 해 본 내용을 위주로 몇 가지 내용을 정리해 보겠습니다.

## 용어

클래스 내부에 속한 데이터는 부르는 이름이 여러가지가 있어서, 이걸 먼저 정리합니다.

| 용어 | 주로 사용되는 언어 | 특징 |
| --- | --- | --- |
| **속성 (Attribute)** | Python, 일반적 용어 | 가장 포괄적인 데이터 표현 |
| **프로퍼티 (Property)** | Python, C# | getter/setter로 제어되는 특수 속성 |
| **멤버 변수 (Member Variable)** | C++, Java | 클래스에 소속된 변수 |
| **인스턴스 변수 (Instance Variable)** | Python, Ruby | 객체별로 독립적인 데이터 |
| **필드 (Field)** | Java, C# | 클래스의 데이터 멤버 |

위 용어 정리표는 간단한 질문에 대해 DeepSeek가 정리해준 것입니다. 절대적인게 아니라 대체로 공감되는 내용일 것이고, 당연히 사용하는 사람들, 문서 마다 다를 수 있습니다.

본 포스트에서는 property 를 많이 설명할텐데, ‘속성’ 이라는 번역이 property가 아닌 일반 attribute와 혼동이 되므로, 그냥 “프로퍼티” 라고 칭하겠습니다. attribute 또한 한글 표현이 필요한 부분에서는 ‘어트리뷰트’ 라고도 사용합니다.

## Series의 array는 어디에 정의되어 있는가?

다시 처음 궁금증으로 돌아가서, 왜 `df.fruit.__dict__` 에 array가 보이지 않은가 에 대해서는, 주요 AI가 내리는 설명 요약으로 대체합니다.

이번에는 GPT 보다 Grok 의 답변이 더 깔끔했습니다.

### Grok 의 답변

Pandas의 Series 객체에서 .array를 통해 내부 배열에 접근할 수 있지만, `__dict__`에 array 라는 키가 보이지 않는 이유는 .array가 프로퍼티(property)으로 구현되어 있기 때문입니다. 이를 이해하기 위해 몇 가지 핵심 사항을 살펴보겠습니다.

**1. .array는 프로퍼티로 구현됨**

Pandas의 Series 클래스에서 .array는 데이터 멤버(즉, 인스턴스 변수)로 저장된 것이 아니라, `@property` 데코레이터를 사용한 프로퍼티로 정의되어 있습니다. 프로퍼티는 메서드처럼 동작하지만, 속성처럼 접근할 수 있게 해줍니다. 따라서 df.fruit.array는 실제로는 Series.array 프로퍼티의 getter 메서드를 호출하는 것입니다.

`__dict__`는 객체의 인스턴스 변수(데이터 멤버)만 포함하며, 프로퍼티나 메서드는 포함하지 않습니다. 그래서 `__dict__`에 array가 보이지 않는 것입니다.

**2. .array의 역할**

Series.array는 Series 객체의 내부 데이터를 Pandas의 ExtensionArray 또는 NumPy 배열 형태로 반환합니다. 이는 Series의 데이터 저장소인 `_mgr` (내부 데이터 관리 객체)에서 데이터를 추출하여 반환하도록 설계되었습니다.

Pandas의 Series 객체는 데이터를 직접적으로 인스턴스 변수에 저장하지 않고, `_mgr` 라는 내부 관리 객체를 통해 데이터를 관리합니다. .array 프로퍼티는 이 `_mgr`에서 데이터를 꺼내어 적절한 형태로 반환합니다.

**3. 내부 구현 살펴보기**

Series 클래스의 .array 프로퍼티의 대략적인 구현 모습입니다.

```python
class Series:
    @property
    def array(self):
        return self._mgr.array
```

- _mgr는 Series의 데이터 관리 객체로, 실제 데이터가 저장된 곳을 가리킵니다.
- _mgr.array는 데이터를 ExtensionArray 또는 NumPy 배열로 반환합니다.

이 프로퍼티는 동적으로 데이터를 계산하거나 변환하여 반환하므로, `__dict__`에 저장되지 않습니다.

…

**5. 추가 확인 방법**

.array가 어떻게 동작하는지 더 확인하려면, Python의 property 객체를 직접 확인하거나 inspect 모듈을 사용할 수 있습니다:

```python
import inspect
print(inspect.getsource(pd.Series.array.fget))
```

이 코드는 .array 프로퍼티의 getter 메서드 소스 코드를 출력합니다. 이를 통해 Pandas가 .array를 어떻게 구현했는지 더 자세히 알 수 있습니다.

    화살표를 눌러서 펼쳐 읽으세요

<aside>
<img src="https://www.notion.so/icons/archive_blue.svg" alt="https://www.notion.so/icons/archive_blue.svg" width="40px" />

**Grok 답변 요약**

- `df.fruit.array` 는 Series 클래스의 `@property`로 정의된 프로퍼티입니다.
- `__dict__`에 array 가 보이지 않는 이유는 array 가 인스턴스 변수가 아니라 동적으로 계산되는 프로퍼티이기 때문입니다.
- .array는 내부적으로 `_mgr`에서 데이터를 추출하여 ExtensionArray 또는 NumPy 배열로 반환합니다.
</aside>

답변의 내용은 잘 정리되어 있어서 궁금증 해소엔 부족함이 없습니다. 답변에는 실제 Series 내부의 코드 구현의 얼개, 그리고 파이썬 명령행 등에서 쉽게 소스 코드를 확인해 볼 수 있는 팁 까지 소개해 주고 있습니다.

그런데, 위에 발췌하지 않은 내용인 GPT의 답변에는 다음과 같은 설명이 있었습니다.

> pandas.Series.array 속성은 일반적인 인스턴스 변수처럼 `__dict__` 안에 저장된 것이 아니라, 속성 메서드(property) 나 디스크립터(descriptor) 를 통해 동적으로 제공되는 것입니다.
>

이 설명에서 짚어야 할 부분은 다음과 같습니다.

1. “속성 메서드”라는 어색한 표현은, 아마도 프로퍼티의 특징, 즉 동적으로 “계산”되는 속성을 강조한 것 때문으로 보입니다. 데이터가 아닌 계산을 하는 코드(메서드)를 강조한 표현으로서, 일반적으로 통용되는 표현은 아닌 것 같습니다.
2. GPT는 프로퍼티를 설명하는 중에 “디스크립터”에 대한 얘기를 합니다. 하지만 디스크립터에 대한 자세한 설명은 해 주지 않습니다.

디스크립터에 대해서 좀 더 검토를 해 보겠습니다.

## 디스크립터

파이썬의 공식 문서에 howto 문서로 “디스크립터 가이드”가 제공되고 있습니다. 일부분이긴 하지만, 한글 번역까지 되어 있습니다.

[Descriptor Guide](https://docs.python.org/ko/3.13/howto/descriptor.html)

<aside>
<img src="https://www.notion.so/icons/archive_blue.svg" alt="https://www.notion.so/icons/archive_blue.svg" width="40px" />

일부 기계 번역에서 디스크립터를 “설명자” 또는 “기술자”로 번역하는 경우가 있지만, 이들 용어는 너무 어색하니, 항상 “디스크립터” 로만 사용합니다.

</aside>

어째선지 실제 핵심 내용보다 이 HowTo 문서의 설명이 더 복잡해 보입니다.

Howto 문서의 핵심 기본 사항만 간추려 보면 다음과 같습니다. (문서의 Primer 의 요약)

1. 디스크립터는 `__get__()`, `__set__()` 또는 `__delete__()` 를 정의하는 모든 객체를 말합니다.
2. 선택적으로, 디스크립터는 `__set_name__()` 메서드를 가질 수 있습니다. 이것은 디스크립터가 생성된 클래스 이름 (포함하는 바깥 클래스를 말함) 또는 할당된 클래스 변수의 이름을 알아야 하는 경우에만 사용됩니다. (이 메서드가 존재하면, 클래스가 디스크립터가 아닌 경우에도 호출됩니다.)
3. 디스크립터는 속성 조회 중에 점 연산자에 의해 호출됩니다. 디스크립터가 "vars(some_class)[descriptor_name]" 를 사용하여 간접적으로 액세스되는 경우, 디스크립터 인스턴스는 호출하지 않고 반환됩니다.
4. 디스크립터는 클래스 변수로 사용될 때만 작동합니다. 인스턴스에 넣으면, 효과가 없습니다.
5. 디스크립터의 주요 동기는 클래스 변수에 저장된 객체가 어트리뷰트 조회 중에 발생하는 일을 제어 할 수 있도록 하는 훅을 제공하는 것입니다.
6. 전통적으로, 호출하는 클래스가 조회 중에 어떤 일이 일어날지 제어합니다. 디스크립터는 그 관계를 역전시키고 조회 중인 데이터가 발언권을 갖도록 합니다.
7. 디스크립터는 언어 전체에서 사용됩니다. 함수가 바인딩된 메서드로 바뀌는 방식입니다. classmethod(), staticmethod(), property() 및 functools.cached_property() 와 같은 일반적인 도구는 모두 디스크립터로 구현됩니다.

위 설명 하나하나가 다 중요하지만, 본 포스트의 주제는 디스크립터가 아닌 프로퍼티 이니까, 디스크립터에 대해서는 일부 만 살펴보겠습니다.

### 디스크립터의 정의

<aside>
<img src="https://www.notion.so/icons/archive_blue.svg" alt="https://www.notion.so/icons/archive_blue.svg" width="40px" />

1. 디스크립터는 `__get__()`, `__set__()` 또는 `__delete__()` 를 정의하는 모든 객체를 말합니다.

</aside>

디스크립터는 클래스이어야 하므로, 위의 세 함수는 “메소드”로 구현해야 합니다. 프로토타입까지 완벽하게 기술해 보면 다음과 같습니다.

```python
class Descriptor:
    def __get__(self, instance, owner): ...
    def __set__(self, instance, value): ...
    def __delete__(self, instance): ...
```

그리고, 위 셋을 “모두” 구현할 필요는 없고, 필요한 것 만 구현하면 됩니다. 만약 `__get__()` 만 구현하면, 그 디스크립터는 읽기 기능만을 인스턴스의 속성에 제공하게 됩니다.

### 디스크립터 인스턴스

일단 `__get__()` 에 대해서만 살펴봅시다.

다음은 `__get__()` 만을 구현한, 그다지 실용적이지는 않은 예시입니다.

```python
class AgeDescriptor:
    def __get__(self, inst, owner): return 10

class Person:
    age = AgeDescriptor()
    def __init__(self, name):
        self.name = name
	  def set_age(self, age):
	      self.age = age # class 수준의 age 를 가림!

dave = Person("Dave")
print(dave.age) # 10
```

Person 클래스에는 age 라는 클래스 변수가 있습니다. 이 클래스 변수는 AgeDescriptor 디스크립터의 인스턴스 입니다. 중요한 부분은 `age = AgeDescriptor()` 에서 self.age 가 아니라 그냥 age 라는 점이 중요합니다.

<aside>
<img src="https://www.notion.so/icons/archive_blue.svg" alt="https://www.notion.so/icons/archive_blue.svg" width="40px" />

1. 디스크립터는 클래스 변수로 사용될 때만 작동합니다. 인스턴스에 넣으면, 효과가 없습니다.
</aside>

만약 self.age 와 같이 인스턴스 스코프의 네임스페이스에 동일한 이름의 변수가 존재하게 되면, 디스크립터 인스턴스를 “가리기”때문에 접근이 어렵게 됩니다. 더 자세한 내용은 본 포스트의 범위를 벗어나므로, 일단은 “디스크립터는 클래스 변수에만 적용한다” 라고 외우는 것이 좋습니다.

### 디스크립터의 호출

Person 객체 인스턴스 “dave” 의 “age” 속성을 읽을 때에는 `dave.age` 와 같이 사용합니다.

이때, 해당되는 디스크립터 클래스, 즉 AgeDescriptor 의 `__get__()` 메소드가 호출됩니다. 이때 중요한 점은, 반드시 도트 연산자를 이용하여 읽을 때에만 디스크립터가 의도한 대로 동작됩니다. 이는 핵심 요약 사항 3번에서도 설명하고 있습니다.

<aside>
<img src="https://www.notion.so/icons/archive_blue.svg" alt="https://www.notion.so/icons/archive_blue.svg" width="40px" />

3. 디스크립터는 속성 조회 중에 점 연산자에 의해 호출됩니다. …

</aside>

객체의 속성에 접근하는 방법은, 점 연산자 외에도 `__dict__` 또는 `var()` 로 얻을 수 있는 이름 사전을 이용하는 방법이 있습니다.

이 방법으로는 접근하는 경우에는, 디스크립터의 원래 의도대로는 동작하지 않음을 말합니다.

디스크립터가 "vars(some_class)[descriptor_name]" 를 사용하여 간접적으로 액세스되는 경우, 디스크립터 인스턴스가 호출되는게 아니고, 디스크립터 인스턴스 자체를 반환합니다.

따라서, 리턴되는 디스크립터 객체 참조에 대해 `__get__()` 을 한번 더 호출할 수는 있습니다. 하지만 이는 의도된 사용은 아닙니다.

### 디스크립터의 목적

<aside>
<img src="https://www.notion.so/icons/archive_blue.svg" alt="https://www.notion.so/icons/archive_blue.svg" width="40px" />

5. 디스크립터의 주요 동기는 클래스 변수에 저장된 객체가 어트리뷰트 조회 중에 발생하는 일을 제어 할 수 있도록 하는 훅을 제공하는 것입니다.

6. 전통적으로, 호출하는 클래스가 조회 중에 어떤 일이 일어날지 제어합니다. 디스크립터는 그 관계를 역전시키고 조회 중인 데이터가 발언권을 갖도록 합니다.

</aside>

어트리뷰터는 메소드와 달리 단순히 데이터이므로, “읽히는 과정”에서 특별히 어떠한 부가적인 동작을 추가할 수 없습니다.

하지만, 디스크립터를 이용하면 위와 같은 단순한 “읽기” 과정에 “개입”할 수 있습니다. 보통 이러한 행위를 “hook”이라고 합니다. 즉, 디스크립터는 속성에 여러가지 형태의 훅을 제공할 수 있습니다.

앞서 프로퍼티를 설명할 때, GPT는 “계산된 속성”이라고 설명했는데, 디스크립터가 바로 그렇습니다. 디스크립터는 계산된 속성 기능을 제공해 줍니다.

속성을 읽을 때, 객체에 저장되어 있는 데이터를 그냥 리턴해 주는 게 아니라, 읽는 시점에 (즉 `__get__()` 이 호출될 때) 계산을 해서 되돌려줄 수 있습니다.

이제 그러면 프로퍼티와 디스크립터의 관계가 궁금해집니다.

<aside>
<img src="https://www.notion.so/icons/archive_blue.svg" alt="https://www.notion.so/icons/archive_blue.svg" width="40px" />

7. 디스크립터는 언어 전체에서 사용됩니다. 함수가 바인딩된 메서드로 바뀌는 방식입니다. classmethod(), staticmethod(), property() 및 functools.cached_property() 와 같은 일반적인 도구는 모두 디스크립터로 구현됩니다.

</aside>

핵심 사항에서도 언급하고 있지만, property() 라는 도구 또한 디스크립터로 구현됩니다.

## 디스크립터 로서의 프로퍼티

다시 프로퍼티 주제로 되돌아와서, 아래의 간단한 프로퍼티 예제를 보겠습니다.

```python
class MyClass:
    @property
    def x(self):
        print("called x")
        return 42

obj = MyClass()
obj.x    # --> 내부적으로: type(obj).__dict__['x'].__get__(obj, type(obj))
```

MyClass 의 x 는 속성이 아닌 메소드의 형태를 갖추고 있습니다. 하지만 “@property” 데코레이션에 의해 일반 속성 처럼 동작할 것으로 기대합니다. 이는 프로퍼티의 가장 간단한 사용 예로서, 메소드 x 는 프로퍼티의 getter 함수로 동작하게 됩니다.

앞에서 잠깐 설명했지만, 프로퍼티는 곧 디스크립터 입니다. 디스크립터의 `__get__()` 호출은, 프로퍼티의 getter 요구 사항을 완벽하게 지원할 수 있습니다.

`obj.x` 는 객체 obj 의 x 라는 속성을 읽습니다. x 는 프로퍼티 객체이고, 사실상 디스크립터 입니다.

애초에 디스크립터 이므로, 당연히 다음과 같이 직접 Descriptor 클래스로 정의할 수 있습니다.

```python
class MyClass1:
    class Descriptor:
        def __get__(self, inst, owner):
            print("called x")
            return 42
    x = Descriptor()

obj1 = MyClass1
obj1.x
```

형태를 보면, 별도의 클래스를 하나 정의 해야 하는 디스크립터 방식 보다는, 프로퍼티의 방식이 좀 더 깔끔합니다. 이것이 프로퍼티를 사용하는 이유입니다. 아 물론 데코레이터의 덕분에 좀 더 간결해 진 부분도 있습니다.

## 데코레이터의 활용

그러면 궁금증이 생깁니다. 프로퍼티는 어떻게 데코레이터를 활용하여 메소드에 디스크립터 기능을 제공할 수 있을까요?

위 MyClass 예제를 보면, `@property` 키워드를 일반 전역 함수가 아닌 메소드에 장식을 한 것을 볼 수 있습니다. 즉, 장식이 되는 대상인 x 는 MyClass.x 이며, 클래스의 메소드 입니다.

```python
class MyClass:
    @property
    def x(self): ...
```

데코레이터의 정의를 위 코드에 그대로 적용해 볼 수 있습니다. 그러면 대략 다음과 같이 동작하는 것으로 이해할 수 있습니다.

`MyClass.x = property(MyClass.x)`

즉, 코드 상의 x 메소드는, 데코레이터에 의해 장식이 된 후 치환이 됩니다. “@” 뒤의 이름으로 데코레이션 호출이 이루어집니다. “property”라는 이름은, 함수 이거나 클래스 이름이어야 합니다. 함수라면 “호출”이 될 것이고, 클래스라면 “생성자 호출”이 될 겁니다.

치환된 새 MyClass.x 가 디스크립터 역할을 한다는 것은, MyClass.x 가 `__get__()` 과 같은 메소드를 제공함을 의미하는데요, 이는 곧 “property 는 클래스 이다” 라고 말할 수 있습니다.

property 클래스는 대략 다음과 같이 구현되어 있습니다. (복잡한 내용 다 빼고 아주 단순한 `__get__` 만 표시)

```python
class property:
    def __init__(self, fget=None, ...):
        self.fget = fget # property 객체는 getter

    def __get__(self, instance, owner):
        return self.fget(instance)
```

데코레이션 과정에서는 다음과 같은 일이 벌어집니다.

1. @property는 함수 def x(self)를 property(x)로 래핑.
    1. property 객체가 생성이 되고, 인자로 x 가 전달되는 것임.
2. 그 결과 MyClass.x 는 이제 property 객체가 됨.
    1. 이름은 MyClass.x 로 그대로 이지만, 타입이 바뀐 것임.
3. 그 property 객체는 디스크립터이기 때문에, obj.x 접근 시 자동으로 `__get__()`을 호출함.

앞서, 디스크립터 방식은 별도의 Descriptor 클래스를 구현해야 하는 번거로움이 있다고 말했는데요, 바로 이 property 클래스가 Descriptor 클래스의 역할을 대신하는 셈 입니다.

적어도 getter 동작에 대해서만 본다면, 설명은 이것이 전부입니다.

## 요약

다음과 같이 요약할 수 있습니다.

- 디스크립터를 이용하면 파이썬의 클래스에 동적 어트리뷰트, 계산된 속성과 같은 향상된 기능을 구현할 수 있습니다.
- 프로퍼티는 디스크립터로 구현됩니다. 디스크립터 보다는 좀 더 간단하게 사용할 수 있습니다.
- 데코레이터 까지 이용하면 더욱 깔끔하게 프로퍼티를 사용할 수 있습니다.

## 향후 계획

프로퍼티를 이용하면 getter 이외의 setter 기능도 제공할 수 있고, 디스크립터로는 `__set__()` 이나 `__delete__()` , set_name 등 더 많은 기능들이 있습니다.

데코레이터에 대한 내용, 그리고 네임스페이스에 대한 내용도 시간을 내서 검토해 볼 만한 주제들입니다.

더 상세한 설명은 본 포스트의 범위를 벗어나므로 일단 생략합니다. 언제가 될 지 모르겠지만 나중에 기회가 될 때 정리를 해 보도록 하겠습니다.