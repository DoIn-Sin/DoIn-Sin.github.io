---
title: 데코레이터
description: 파이썬에서 사용하는 데코레이터에 대해 알아보자.
categories:
- OOP
- decorator
tags: 
- 객체지향프로그래밍
---

# 데코레이터
지금까지 클래스를 공부하면서 메소드를 만들 때 @staticmethod, @classmethod 등을 붙였는데, 이렇게 @로 시작하는 것들이 데코레이터이다. 즉, 메소드를 장식한다고 해서 데코레이터라고 불린다.
뜬금없이 메소드를 장식한다니 잘 와닿지 않았다. 조금 더 자세하게 알아보니 데코레이터는 함수를 수정하지 않은 상태에서 추가 기능을 구현할 때 사용한다고 한다! 예를 들어 함수의 시작과 끝을 출력하고 싶다면
함수의 시작, 끝 부분에 print문을 넣어서 출력할 것이다. 메소드의 수가 많아질수록 이 작업은 매우 비효율적으로 이루어질텐데 이런 경우 데코레이터를 사용하면 편리하다. 어떻게? 함수의 시작과 끝을 알리는 데코레이터를 만들어서 불러오기!!

## 데코레이터 더 알아보기

```python
# 기존의 비효율적인 코드
def hello():
    print('hello 함수 시작')
    print('hello')
    print('hello 함수 끝')
 
def world():
    print('world 함수 시작')
    print('world')
    print('world 함수 끝')
 
 ...

 # 함수의 시작과 끝을 알리는 데코레이터 만들기
def trace(func):                             # 호출할 함수를 매개변수로 받음
    def wrapper():                           # 호출할 함수를 감싸는 함수
        print(func.__name__, '함수 시작')    # __name__으로 함수 이름 출력
        func()                               # 매개변수로 받은 함수를 호출
        print(func.__name__, '함수 끝')
    return wrapper                           # wrapper 함수 반환
 
def hello():
    print('hello')
 
def world():
    print('world')

trace_hello = trace(hello)    # 데코레이터에 호출할 함수를 넣음
trace_hello()                 # 반환된 함수를 호출
trace_world = trace(world)    # 데코레이터에 호출할 함수를 넣음
trace_world()                 # 반환된 함수를 호출
```

데코레이터는 @를 붙여서 사용한다고 하지 않았나..? 맞다. 근데 사용하지 않고 클로저를 이용해서 사용할 수도 있다. 위의 예시가 바로 그렇다. 
데코레이터를 사용할 때는 trace에 호출할 함수를 인수로 넣는다. 그 다음에 데코레이터에서 반한된 함수를 호출하게 되고, 결과적으로 시작과 끝을 출력하게 되는 것이다!

```python
# @를 이용한 방법
def trace(func):                             # 호출할 함수를 매개변수로 받음
    def wrapper():                           # 호출할 함수를 감싸는 함수
        print(func.__name__, '함수 시작')    # __name__으로 함수 이름 출력
        func()                               # 매개변수로 받은 함수를 호출
        print(func.__name__, '함수 끝')
    return wrapper                           # wrapper 함수 반환
 
@trace    # @데코레이터
def hello():
    print('hello')
 
@trace    # @데코레이터
def world():
    print('world')
 
hello()    # 함수를 그대로 호출
world()    # 함수를 그대로 호출
"""
함수 시작
hello
함수 끝
함수 시작
world
함수 끝
```

우리가 늘 써왔던 대로 메소드 위에 @를 붙인 데코레이터를 적어주고 메소드를 호출해주면 끝이다! 참고로 데코레이터는 여러 개도 지정이 가능하다. 단순히 데코레이터를 여러 줄로 지정해주면 된다.
이때, 실행되는 순서는 위에서 아래 순으로 실행된다. 나중에 뒤에서 알아보기로 하고, 일단 인수를 가진 메소드를 데코레이팅하는 경우에 대해 알아보자.

```python
def decorator_function(original_function):
    def wrapper_function(*args, **kwargs):  #1
        print('{} 함수가 호출되기전 입니다.'.format(original_function.__name__))
        return original_function(*args, **kwargs)  #2
    return wrapper_function


@decorator_function
def display():
    print('display 함수가 실행됐습니다.')


@decorator_function
def display_info(name, age):
    print('display_info({}, {}) 함수가 실행됐습니다.'.format(name, age))


display()
print()
display_info('John', 25)

"""
display 함수가 호출되기전 입니다.
display 함수가 실행됐습니다.

display_info 함수가 호출되기전 입니다.
display_info(John, 25) 함수가 실행됐습니다.
"""
```

데코레이터 #1, #2에 인수를 추가해주면 인수가 있는 메소드도 데코레이팅이 가능하다. 그리고 이런 함수 형식의 데코레이터 말고 클래스 형식을 사용해서 만들 수도 있다.

```python
# def decorator_function(original_function):
#     def wrapper_function(*args, **kwargs):
#         print '{} 함수가 호출되기전 입니다.'.format(original_function.__name__)
#         return original_function(*args, **kwargs)
#     return wrapper_function


class DecoratorClass:  # 1
    def __init__(self, original_function):
        self.original_function = original_function

    def __call__(self, *args, **kwargs):
        print('{} 함수가 호출되기전 입니다.'.format(self.original_function.__name__))
        return self.original_function(*args, **kwargs)


@DecoratorClass  # 2
def display():
    print('display 함수가 실행됐습니다.')


@DecoratorClass  # 3
def display_info(name, age):
    print('display_info({}, {}) 함수가 실행됐습니다.'.format(name, age))


display()
print()
display_info('John', 25)
"""
display 함수가 호출되기전 입니다.
display 함수가 실행됐습니다.

display_info 함수가 호출되기전 입니다.
display_info(John, 25) 함수가 실행됐습니다.
"""
```

이제 감이 조금 오는가? 그럼 조금 더 복잡한 경우에 대해 살펴보자. 

## 데코레이터 더 자세하게 알아보기
데코레이터를 통해 아래와 같은 로깅 기능을 만들어보자!
```python
"""
$ date; time sleep 1; date

Thu Sep  8 00:13:28 JST 2016

real    0m1.007s
user    0m0.001s
sys     0m0.001s
Thu Sep  8 00:13:29 JST 2016
"""
```

이와 같은 기능을 구현하기 위해서는 데코레이터를 2개 사용해야 할 것같다. 로그를 출력하는 데코레이터 하나, 실행시간을 측정하는 데코레이터 하나!
우선 로그를 출력하는 데코레이터 먼저 만들어보자.

```python
import datetime
import time


def my_logger(original_function):
    import logging
    filename = '{}.log'.format(original_function.__name__)
    logging.basicConfig(handlers=[logging.FileHandler(filename, 'a', 'utf-8')],
                        level=logging.INFO)

    def wrapper(*args, **kwargs):
        timestamp = datetime.datetime.now().strftime('%Y-%m-%d %H:%M')
        logging.info('[{}] 실행결과 args - {}, kwargs - {}'.format(timestamp, args, kwargs))
        return original_function(*args, **kwargs)

    return wrapper


@my_logger
def display_info(name, age):
    time.sleep(1)
    print('display_info({}, {}) 함수가 실행됐습니다.'.format(name, age))

"""
display_info(John, 25) 함수가 실행됐습니다.

- 로그 파일 안
INFO:root:[2022-08-07 20:13] 실행결과 args - ('John', 25), kwargs - {}
"""
```

로그 파일을 보니 로그가 잘 찍힌다! 그럼 실행시간도 해볼까?

```python
import datetime
import time


def my_logger(original_function):
    import logging
    filename = '{}.log'.format(original_function.__name__)
    logging.basicConfig(handlers=[logging.FileHandler(filename, 'a', 'utf-8')],
                        level=logging.INFO)

    def wrapper(*args, **kwargs):
        timestamp = datetime.datetime.now().strftime('%Y-%m-%d %H:%M')
        logging.info(
            '[{}] 실행결과 args - {}, kwargs - {}'.format(timestamp, args, kwargs))
        return original_function(*args, **kwargs)

    return wrapper


def my_timer(original_function):  # 1
    import time

    def wrapper(*args, **kwargs):
        t1 = time.time()
        result = original_function(*args, **kwargs)
        t2 = time.time() - t1
        print('{} 함수가 실행된 총 시간: {} 초'.format(original_function.__name__, t2))
        return result

    return wrapper


@my_timer  # 2
def display_info(name, age):
    time.sleep(1)
    print('display_info({}, {}) 함수가 실행됐습니다.'.format(name, age))


display_info('John', 25)

"""
display_info(John, 25) 함수가 실행됐습니다.
display_info 함수가 실행된 총 시간: 1.00157594681 초
"""
```

약 1초 정도 걸린다. 그럼 이제 두 데코레이터를 동시에 사용해보자!

```python
import datetime
import time


def my_logger(original_function):
    import logging
    filename = '{}.log'.format(original_function.__name__)
    logging.basicConfig(handlers=[logging.FileHandler(filename, 'a', 'utf-8')],
                        level=logging.INFO)

    def wrapper(*args, **kwargs):
        timestamp = datetime.datetime.now().strftime('%Y-%m-%d %H:%M')
        logging.info(
            '[{}] 실행결과 args - {}, kwargs - {}'.format(timestamp, args, kwargs))
        return original_function(*args, **kwargs)

    return wrapper


def my_timer(original_function):
    import time

    def wrapper(*args, **kwargs):
        t1 = time.time()
        result = original_function(*args, **kwargs)
        t2 = time.time() - t1
        print('{} 함수가 실행된 총 시간: {} 초'.format(original_function.__name__, t2))
        return result

    return wrapper


@my_logger  # 1
@my_timer  # 2
def display_info(name, age):
    time.sleep(1)
    print('display_info({}, {}) 함수가 실행됐습니다.'.format(name, age))


display_info('John', 25)
"""
display_info(John, 25) 함수가 실행됐습니다.
display_info 함수가 실행된 총 시간: 1.00419592857 초

- 로그 파일 안
INFO:root:[2022-08-07 20:13] 실행결과 args - ('John', 25), kwargs - {}
"""
```

데코레이터 2개를 한 번에 사용했더니 터미널에는 문제없이 출력되는 것을 확인했는데, 기존의 로그 파일에는 아무것도 기록되지 않았다. 대신 wrapper.log라는 로그 파일이 생성되고, 방금 실행한 로그가 찍혀있는 것을 확인했다.
순서가 잘못된 것인가 싶어서 순서를 바꿔서 사용해봤다.

```python
from functools import wraps
import datetime
import time


def my_logger(original_function):
    import logging
    filename = '{}.log'.format(original_function.__name__)
    logging.basicConfig(handlers=[logging.FileHandler(filename, 'a', 'utf-8')],
                        level=logging.INFO)

    @wraps(original_function)  # 1
    def wrapper(*args, **kwargs):
        timestamp = datetime.datetime.now().strftime('%Y-%m-%d %H:%M')
        logging.info(
            '[{}] 실행결과 args - {}, kwargs - {}'.format(timestamp, args, kwargs))
        return original_function(*args, **kwargs)

    return wrapper


def my_timer(original_function):
    import time

    @wraps(original_function)  # 2
    def wrapper(*args, **kwargs):
        t1 = time.time()
        result = original_function(*args, **kwargs)
        t2 = time.time() - t1
        print('{} 함수가 실행된 총 시간: {} 초'.format(original_function.__name__, t2))
        return result

    return wrapper


@my_timer
@my_logger
def display_info(name, age):
    time.sleep(1)
    print('display_info({}, {}) 함수가 실행됐습니다.'.format(name, age))


display_info('John', 25)  # 3
"""
display_info(John, 25) 함수가 실행됐습니다.
wrapper 함수가 실행된 총 시간: 1.0019299984 초

- 로그 파일 안
INFO:root:[2022-08-07 20:13] 실행결과 args - ('John', 25), kwargs - {}
INFO:root:[2022-08-07 20:20] 실행결과 args - ('John', 25), kwargs - {}
"""
```

이번엔 로깅은 정삭적으로 됐는데 터미널 출력이 이상하다. 원래라면 display_info의 실행시간이 출력돼야 하는데 wrapper가 출력이 됐다. 왜 이럴까? 사실 이유는 간단하다.
아까 앞에서 잠깐 말하고 넘어갔는데 데코레이터는 여러 개를 사용하게 되면 위에서 부터 아래로 실행이 된다. 위의 코드의 경우 #1의 my_logger가 실행되고 #2의 my_timer에게 #3에서 wrapper 함수를
인자로 리턴하게 되면서 생기는 현상이다. 위와 같은 현상을 방지하기 위해서 만들어진 모듈이 바로 functools의 wraps 데코레이터이다!

```python
from functools import wraps
import datetime
import time


def my_logger(original_function):
    import logging
    filename = '{}.log'.format(original_function.__name__)
    logging.basicConfig(handlers=[logging.FileHandler(filename, 'a', 'utf-8')],
                        level=logging.INFO)

    @wraps(original_function)  # 1
    def wrapper(*args, **kwargs):
        timestamp = datetime.datetime.now().strftime('%Y-%m-%d %H:%M')
        logging.info(
            '[{}] 실행결과 args - {}, kwargs - {}'.format(timestamp, args, kwargs))
        return original_function(*args, **kwargs)

    return wrapper


def my_timer(original_function):
    import time

    @wraps(original_function)  # 2
    def wrapper(*args, **kwargs):
        t1 = time.time()
        result = original_function(*args, **kwargs)
        t2 = time.time() - t1
        print('{} 함수가 실행된 총 시간: {} 초'.format(original_function.__name__, t2))
        return result

    return wrapper


@my_timer
@my_logger
def display_info(name, age):
    time.sleep(1)
    print('display_info({}, {}) 함수가 실행됐습니다.'.format(name, age))


display_info('John', 25)  # 3

"""
display_info(John, 25) 함수가 실행됐습니다.
display_info 함수가 실행된 총 시간: 1.013833999633789 초

- 로그 파일 안
INFO:root:[2022-08-07 20:13] 실행결과 args - ('John', 25), kwargs - {}
INFO:root:[2022-08-07 20:20] 실행결과 args - ('John', 25), kwargs - {}
INFO:root:[2022-08-07 20:27] 실행결과 args - ('John', 25), kwargs - {}
"""
```

원하는 결과대로 출력되고 확인됐다!! 이제 데코레이터의 쓰임새와 사용 방법을 어느정도 숙지한 것아서 좋다ㅎㅎ 굿~~

# Reference
[1] [스쿨오브웹](https://schoolofweb.net/blog/posts/%ed%8c%8c%ec%9d%b4%ec%8d%ac-%eb%8d%b0%ec%bd%94%eb%a0%88%ec%9d%b4%ed%84%b0-decorator/)

[2] CodeStates AI 부트캠프 학습 자료