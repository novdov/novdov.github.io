---
title: '[Python 3.9] 새로 추가된 Dictionary Merge & Update Operators 살펴보기'
date: 2020-03-08 02:00:00
description: 파이썬 3.9 alpha4에서 새로 추가된 기능인 Dictionary merge & update 연산자에 대해 알아봅니다.
categories:
- python
tags:
- python
---

파이썬 3.9.0a4가 3월 7일 릴리즈 되었습니다. 현재 `pyenv` 에서 `3.9-dev` 로 설치할 수 있습니다. 얼마전 까지만 하더라도 새로 추가된 기능이 없었는데, 이번 릴리즈에서는 새로운 기능이 도입되었습니다. 바로 사전 병합 (`|`), 업데이트 (`|=`) 연산자의 추가입니다. 이와 관련된 [PEP-584](https://www.python.org/dev/peps/pep-0584/)와 [이슈 (bpo-36144)](https://bugs.python.org/issue36144)에서 찾아볼 수 있습니다.


## 왜 도입되었을까?

**PEP 584 -- Add Union Operators To dict** 의 Motivation에서는 현재 두 사전을 병합하는 방법에는 몇 가지 단점이 있다면서 해당 연산자의 도입 동기를 설명하고 있습니다.

### 기존 방법들의 단점

- `dict.update`
  - `d1.update(d2)` 는 `d1`을 변경하는 in-place 연산으로, `e = d1.copy(); e.update(d2)` 는 표현식이 아니며 임시 변수를 필요로 합니다.

- `{**d1, **d2}`
  - 사전 언패킹은 못생겼으며 (ugly), 알아보기 쉽지 않습니다. 해당 표현이 무엇을 뜻하는지 알아보는 사람이 적고, 해당 표현이 두 사전을 병합한다는 의미를 알아보기 쉽지 않습니다. 이에 대한 귀도의 생각도 함께 언급했습니다. 즉, 해당 표현이 알아보기 쉽지 않다는 겁니다.

    > *I'm sorry for* [PEP 448](https://www.python.org/dev/peps/pep-0448)*, but even if you know about* `**d` *in simpler contexts, if you were to ask a typical Python user how to combine two dicts into a new one, I doubt many people would think of* `{**d1, **d2}`*. I know I myself had forgotten about it when this thread started!*

  - 또한, 해당 표현의 단점은 mapping의 타입을 무시하고 언제나 `dict(d1)({**d1, **d2})` 를 리턴한다는 점입니다. 즉, `defaultdict` 같은 mapping의 타입이 무시된채 언젠나 기본 `dict` 타입으로만 리턴됩니다.

- `collections.ChainMap`
  - `ChainMap` 은 매우 알려지지 않았으며, 명백하지 않습니다. (또한, 중복 key를 처리하는 방법도 일반적인 방법과 다릅니다. 즉, 먼저 등장한 사전의 키를 사용합니다. 일반적인 동작은 “last seens wins” 입니다.) 언패킹과 마찬가지로, `dict` 서브클래스의 타입을 모두 보존하지는 않습니다. 또한, `ChainMap` 은 사전을 감싸, `ChainMap` 을 변경하면 원래 사전도 함께 변경됩니다.

- `dict(d1, **d2)`
  - 이 또한 널리 알려지지 않았으며, `d2` 의 key가 모두 문자열일 때만 동작합니다.


### 도입 근거

도입 근거를 한마디로 정의하자면 “명백함” 입니다. 사전을 병합하는 간단하고 명백한 기본 동작을 도입하기 위함입니다.


## 연산자 동작

### Dict union (`|`)

병합 연산자 (`|`) 두 사전을 병합한 새로운 사전을 리턴하며, 각각은 모두 `dict` 타입 (혹은 서브클래스)이어야 합니다. 중복되는 key가 있을 경우 가장 오른쪽 사전의 key를 사용합니다 (last sees wins).

```python
d = {"spam": 1, "eggs": 2, "cheese": 3}
e = {"cheese": "cheddar", "aardvark": "Ethel"}
d | e
>>>
# e의 cheese (cheddar)를 사용합니다.
{'spam': 1, 'eggs': 2, 'cheese': 'cheddar', 'aardvark': 'Ethel'}
e | d
>>>
# d의 cheese (3)를 사용합니다.
{'aardvark': 'Ethel', 'spam': 1, 'eggs': 2, 'cheese': 3}
```


### Dict update (`|=`)

업데이트 연산자 (`|=`)는 in-place 연산을 수행합니다. 

```python
d |= e
d
>>>
{'spam': 1, 'eggs': 2, 'cheese': 'cheddar', 'aardvark': 'Ethel'}
```


해당 연산자는 단일 positional argument를 입력한 `update` 메서드와 동일하게 동작합니다. 즉, Mapping 프로토콜 (`keys`, `__getitem__`)를 구현하는 객체나 key-value 쌍의 iterable에도 동작합니다. (리스트 뿐만 아니라 어떠한 iterable에도 동작하는 `list +=`, `list.extend`와 동일합니다.)

```python
d | [("spam", 999)]
>>>
Traceback (most recent call last):
  ...
TypeError: unsupported operand type(s) for |: 'dict' and 'list'
    
d |= [("spam", 999)]
d
>>>
{'eggs': 2, 'cheese': 'cheddar', 'aardvark': 'Ethel', 'spam': 999}
```


### 구현

대략적인 순수 파이썬 구현은 다음과 같습니다. (실제는 C로 구현되었으며, 자세한 관련 PR은 bpo-36144 에서 찾아볼 수 있습니다.)

```python
def __or__(self, other):
    if not isinstance(other, dict):
        return NotImplemented
    new = dict(self)
    new.update(other)
    return new

def __ror__(self, other):
    if not isinstance(other, dict):
        return NotImplemented
    new = dict(other)
    new.update(self)
    return new

def __ior__(self, other):
    dict.update(self, other)
    return self
```


## 마치며

파이썬 3.9.0a4에서 새롭게 추가된 Dict union & update 연산자를 간단하게 살펴봤습니다. 개인적으로는 기존 방법들에 비해 이해하고, 사용하기 편하다고 생각합니다. 릴리즈 일정에 따르면 beta1 (2020-05-18)까지 새로운 기능이 추가될 예정인데, 그 전까지 또 어떤 기능들이 추가될지 기대 됩니다.
