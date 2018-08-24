---
title: '동적 기본 인수를 지정하기 위한 None과 docstring'
date: 2018-03-16 22:07:45
description: 의도치 않은 함수 작동을 방지하기 위한 동적 기본 인수 지정에 대해 알아봅니다.
categories:
- python
tags:
- python
---

## `None`과 `docstring`을 통한 동적 기본 인수 지정

> 출처: 파이썬 코딩의 기술 (브렛 슬라킨, 길벗)

이벤트 발생 시각을 포함해 로깅 메시지를 출력하려고 다음 함수를 작성하고 실행했다.

```python
# print time and message 1

from datetime import datetime
import time

def log(message, when=datetime.now()):
    print("{}: {}".format(when, message))
    
log("Hi there!")
time.sleep(0.1)
log("Hi again!")
```

```python
2018-03-04 23:33:53.884102: Hi there!
2018-03-04 23:33:53.884102: Hi again!
```

이벤트 발생 시각을 포함해 로깅 메시지를 출력하려고 시도했지만 `datetime.now`는 함수를 정의할 때 딱 한 번만 실행되므로 타임스탬프가 동일하게 출력됨. 모듈이 로드된 후에는 기본 인수인 `datetime.now`는 다시 평가되지 않는다. 이를 목적대로 실행하기 위해서는 기본값을 `None`으로 설정하고 `docstring`으로 실제 동작을 문서화하는게 관례이다.

다시 함수를 작성하고 실행하면 의도했던대로 잘 동작한다.

```python
# print time and message 2

def log(message, when=None):
    """Log message with a timestamp
    
    Args:
        message: Message to print.
        when: datetime of when the message occurred.
            Defaults to the present time.
    """
    when = datetime.now() if when is None else when
    print("{}: {}".format(when, message))
    
log("Hi there!")
time.sleep(0.1)
log("Hi again!")
```

```python
2018-03-04 23:39:00.842097: Hi there!
2018-03-04 23:39:00.945287: Hi again!
```

**핵심 정리**

- 기본 인수는 모듈 로드 시점에 함수 정의 과정에서 딱 한 번만 평가됨. 그래서 동적 값에는 이상하게 동작하는 원인이 되기도 함
- 값이 동적인 키워드 인수에는 기본값으로 `None`을 사용하고, docstring을 통해 실제 동작을 문서화



비슷한 예로, JSON 포맷의 데이터를 불러와 딕셔너리 형태로 생성하는 코드를 작성하면 다음과 같다.

```python
# JSON 데이터로 인코드된 값을 로드
# 데이터 디코딩이 실패하면 기본으로 빈 딕셔너리를 리턴

def decode(data, default={}):
    try:
        return json.loads(data)
    except ValueError:
        return default
```

```python
# 하나를 수정하면 다른 하나도 수정됨

foo = decode('bad data')
foo['stuff'] = 5
bar = decode('also bad')
bar['meep'] = 1
print("Foo: ", foo)
print("Bar: ", bar)
```

```python
Foo:  {'stuff': 5, 'meep': 1}
Bar:  {'stuff': 5, 'meep': 1}
```

이 함수 역시 의도했던대로 동작하게 하려면 `None`을 이용한다.

```python
def decode(data, default=None):
    """Load JSON data from a string.
    
    Args:
        data: JSON data to decode.
        default: Value to return if decoding fails.
            Defaults to an empty dictionary.
    """
    if default is None:
        default = {}
    
    try:
        return json.loads(data)
    except ValueError:
        return default
```

```python
foo = decode('bad data')
foo['stuff'] = 5
bar = decode('also bad')
bar['meep'] = 1
print("Foo: ", foo)
print("Bar: ", bar)
```

```python
Foo:  {'stuff': 5}
Bar:  {'meep': 1}
```

`None`을 이용하면 의도했던대로 잘 동작하는 것을 확인할 수 있다.