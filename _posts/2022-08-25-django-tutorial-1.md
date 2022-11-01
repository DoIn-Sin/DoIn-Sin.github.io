---
title: 장고 앱 작성하기, part 1
description: django를 공부하게 된 계기와 튜토리얼 part1을 따라해봤습니다.
categories:
- Django
- Web Framework
tags: 
- Django

published : false
---

# 주저리 주저리..
개인프로젝트를 하다보면 웹을 사용해야 하는 경우가 빈번했다. 그럴때마다 나는 파이썬의 웹 프레임워크 중 하나인 Flask를 사용해왔다. 가볍고 코드를 일일이 다 작성해서 구성해야한다는 점에서 공부도 되고 빼놓을 수 없는 편리성 때문이다!
그러던 중 카트라이더 관련 개인프로젝트를 하다가 문득 직접 서비스를 배포해보고 싶었다! 카트라이더를 즐기는 한 유저로써 "실제로 카트라이더 유저들이 사용하면 조금 더 게임을 재밌게 즐길 수 있지 않을까?"하는 생각이 들었기 때문이다.
그래서 기존에 만들었던 Flask에서 서비스를 배포하려고 하니 확장성, 유지보수, 보안 등등 신경쓸게 한 둘이 아니었다ㅠ.. 그래서 조금 더 나은 대안이 없는지 열심히 구글링을 하였다!
<br>
내가 쉽게 찾을 수 있는 답은 Django 였다! 이유라 함은 Flask와 달리 다 구축되어 있기 때문에 내가 설정을 빼먹어서 문제가 생길 일도 없고, 성능이 좋다는 말들이 많았다. 근데 써보질 않았고 사실 내가 만든 웹이 성능이 막 좋아야 한다는
그런 생각도 없었기에 더 와닿지 않았다. 그래서 조금 더 근본적으로 들어가서 왜 Django를 써왔는지, 어떤 경우에서는 Django를 쓰고 안 쓰는지를 알아봤다!
<br>
한 블로그에서 아주 좋은 글을 발견했는데 간략하게 정리하자면, Django는 거의 시간을 소모하지 않고 튼실한 관리자 페이지가 구성되어 있고 앞서 언급한 보안 문제를 해결할 방법들이 이미 구현되어 있다는 것, 확장성이 용이해서 서버의 대수를 늘려 수용력을 늘릴 수 있다는 것, 마지막으로 편리한 개발 및 디버깅 환경이 제공된다는 것이다! 하지만, 속도가 느리고 동적으로 처리하기 때문에 실시간 처리가 어렵고 비동기 방식으로 처리하기 때문에 딜레이가 있다는 점, 페이지를 동적으로 로딩하기 때문에 페이지 내 일부분만 바뀌더라도 화면 전체가 바뀐다는 단점도 존재했다.
<br>
하지만 나는 실시간 처리를 할 것도 없었고 설령 있다고 하더라도 따로 구성해서 처리할 수도 있다고 하니 서비스를 처음 배포할 초짜가 안전하고 관리가 편리한 프레임워크를 마다할 이유가 없었다. 그래서 나는 장고를 공부해보기로 했다! (참 말도 많네..ㅎ)

# 개발 환경
- OS : Ubuntu 20.04 LTS
- Python : 3.8.10
- Django : 4.1.1

# 프로젝트 만들기
우선, 장고가 잘 설치가 되어있는지 확인을 먼저 해보자.
```bash
python -m django --version

# output: 4.1.1
```

4.1.1 버전으로 설치 확인을 마치고 프로젝트를 만들어보자! 따라하기 전에 튜토리얼의 과정들을 읽어봤는데 본 튜토리얼에서는 간단한 설문조사 웹을 만든다고 한다.

프로젝트를 생성하는 명령어는 다음과 같다.
```bash
# django-admin startproject <프로젝트 이름>
django-admin startproject mysite
````

이렇게 입력하면 현재 위치한 디렉토리에 mysite라는 이름의 장고 프로젝트 폴더가 생성된다! 생성된 폴더 디렉토리 구조는 다음과 같다.
```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
```
>파일들의 역할
- file:mysite/ 디렉토리 밖은 프로젝트를 담는 공간입니다. 그 이름은 Django 와 아무 상관이 없으니, 원하는 이름으로 변경해도 됩니다.
- manage.py: Django 프로젝트와 다양한 방법으로 상호작용 하는 커맨드라인의 유틸리티 입니다. manage.py 에 대한 자세한 정보는 django-admin and manage.py 에서 확인할 수 있습니다.
- mysite/ 디렉토리 내부에는 프로젝트를 위한 실제 Python 패키지들이 저장됩니다. 이 디렉토리 내의 이름을 이용하여, (mysite.urls 와 같은 식으로) 프로젝트의 어디서나 Python 패키지들을 임포트할 수 있습니다.
- mysite/__init__.py: Python으로 하여금 이 디렉토리를 패키지처럼 다루라고 알려주는 용도의 단순한 빈 파일입니다. Python 초심자라면, Python 공식 홈페이지의 패키지를 읽어보세요.
- mysite/settings.py: 현재 Django 프로젝트의 환경 및 구성을 저장합니다. Django settings에서 환경 설정이 어떻게 동작하는지 확인할 수 있습니다.
- mysite/urls.py: 현재 Django project 의 URL 선언을 저장합니다. Django 로 작성된 사이트의 “목차” 라고 할 수 있습니다. URL dispatcher 에서 URL 에 대한 자세한 내용을 읽어보세요.
- mysite/asgi.py: 현재 프로젝트를 서비스하기 위한 ASGI-호환 웹 서버의 진입점입니다. 자세한 내용은 ASGI를 사용하여 배포하는 방법 를 참조하십시오.
- mysite/wsgi.py: 현재 프로젝트를 서비스하기 위한 WSGI 호환 웹 서버의 진입점입니다. WSGI를 사용하여 배포하는 방법를 읽어보세요.

# 개발 서버 확인하기
자 이제 장고 프로젝트가 잘 생성됐는지 서버를 실행시켜서 확인해보자.
```bash
python manage.py runserver


# output
"""
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 18 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
September 23, 2022 - 22:49:07
Django version 4.1.1, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
"""
```
이런 출력이 나오고 http://localhost:8000으로 접속했을 때, 아래와 같은 사진이 뜨면 성공!!

![image](https://user-images.githubusercontent.com/77676907/192066563-957b3939-4453-4482-9593-3a3ff25802d8.png)

>위에 나오는 경고들은 변경사항들을 장고 데이터베이스에 적용하라는 말인데, 나중에 뒤에서 다룬다고 하니 지금은 무시해도 괜찮다.

그럼 이제 아까 말했던 설문조사 웹을 위해 앱을 만들어야 한다.

# 설문조사 앱 만들기
>프로젝트와 앱
- 프로젝트는 특성 웹 사이트에 대한 구성 및 앱의 모음
- 앱은 웹 애플리케이션(블로그, 의견조사 웹 등등)<br>
따라서 한 프로젝트에 여러 앱이 포함될 수 있고, 앱도 여러 프로젝트에 존재할 수 있다!

프로젝트와 앱에 대한 차이점을 알았으니 설문조사 앱을 만들어보자!
```bash
# python manage.py startapp <앱 이름>
python manage.py startapp polls
```

그럼 polls 라는 이름의 디렉토리가 생상되고 구조는 다음과 같다.
```
polls/
	__init__.py
	admin.py
	apps.py
	migrations/
		__init__.py
	models.py
	tests.py
	views.py
```

그럼 설문조사 웹을 위한 앱을 만들었으니 화면에 보여질 **뷰**를 작성해보자!

# 첫 번째 뷰 작성하기
뷰를 작성하기 위해 polls 폴더에 있는 views.py를 열어서 아래의 코드를 입력하자.

```python
from django.http import HttpResponse

def index(request):
	return HttpResponse("Hello, world! You're at the polls index.")
```

이렇게 만든 뷰를 호출하려면 연결된 URL이 있어야 하는데, 이를 위해 URLconf가 사용된다.

polls 디렉토리에서 URLconf를 생성하려면 urls.py라는 파일을 생성해야 하고 아래의 코드를 입력하자.


```python
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

다음은 최상위 디렉토리인 mysite의 URLconf에서 polls 디렉토리에 있는 urls.py를 바라보게 설정해야 한다.

mysite의 urls.py를 열어서 아래와 같이 코드를 입력하자.

```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

그럼 이제 runserver를 하면 아까 index 뷰로 정의한 문구가 출력될 것이다ㅎㅎ

참고로 여기서 사용한 include()함수는 다른 URLconf를 참조할 수 있도록 도와주는 함수이다!!

```python
python manage.py runserver
```

![image](https://user-images.githubusercontent.com/77676907/192124036-886ae364-4cb6-4a44-a660-68f8e4448b41.png)

이렇게 안 떴다면 아까 mysite의 URLconf를 설정했을 때를 잘 떠올려보라! 자신이 path를 어떻게 적고 polls의 urls.py를 참조시켰는지!!(필자는 polls/라고 설정했기에 https://127.0.0.1:8000/polls/ 로 접속했다!)

그럼 여기서 드는 의문! path에 내가 원하는 경로의 이름으로 만들고 접속하면 되나??

## path 사용법
path() 함수에는 2개의 필수 인수인 route와 view가 들어가고 2개의 선택 인수로 kwargs와 name이 있다. 선택 인수는 사용할 때 자세히 보기로 하고 지금은 필수 인수만 보도록 하자!

1. route 인수
route 인수는 URL 패턴을 가지는 문자열로 위에서는 polls가 되겠다. 이것은 요청이 처리될 때, urlpatterns의 첫 번째 패턴부터 시작하여, 일치하는 패턴을 찾을 때 까지 요청된 URL을 각 패턴과 리스트의 순서대로 비교한다!

2. view 인수
route 인수를 통해 Django가 URL 패턴을 찾으면, HttpRequest를 통해 이와 연결된 view 함수를 호출해서 우리에게 보여지게 된다!


튜토리얼 1을 통해 뷰를 작성하는 법과 연결시켜서 접속하는 방법에 대해 배웠다! 다음은 runserver를 하면 나오는 경고 문구를 지울 수 있도록 데이터베이스에 연결할 것이다!!

# Reference
[1] [참조한 블로그](https://blog.lxf.kr/2018-11-19---why-or-not-django/)

[2] [Django tutorial](https://docs.djangoproject.com/ko/4.1/intro/tutorial01/)
