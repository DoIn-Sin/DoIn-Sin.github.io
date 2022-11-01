---
title: Django 겉할기 시리즈 - Django 앱 생성 및 모델 만들기
description: 이지스 퍼블리싱의 Do it! Django + 부트스트랩책을 보며 공부한 내용을 정리했습니다.
categories:
- Django
- BackEnd
tags: 
- Django
- Doit
- BackEnd
- Django App
---

# 1. Django 프로젝트에서 앱 생성하기
장고 프로젝트는 보토 1개 이상의 앱을 가진다. 여기서 앱은 `블로깅 기능`, `단일 페이지 보여주기 기능`과 같이 특정 기능을 수행하는 단위 모듈을 말한다. 현재 프로젝트에서는 2개의 앱을 만든다. 하나는 블로깅 기능을 위한 `블로그 앱`이고, 또 다른 하나는 대문 페이지와 자기소개 페이지를 보여주기 위한 `싱글 페이지 앱`이다.

## 1-1. blog 앱, single_pages 앱

프로젝트를 생성할 때처럼 앱도 명령어를 통해 생성해주면 된다.

```bash
# blog 앱 생성
python manage.py startapp blog
# single_pages 앱 생성
python manage.py startapp single_pages
```

위 과정을 마치면 2개의 폴더가 생성되는 것을 확인할 수 있다.

```bash
root
├── do_it_django_prj # 생성한 프로젝트
│   ├── _pycache_
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── blog # 생성한 blog 앱
│   ├── migrations
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── single_pages # 생성한 single_pages 앱
│   ├── migrations
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
│
├── db.sqlite3
└── manage.py
```

>앱 파일들의 역할
- migrations: Django 앱의 데이터 구조에 대한 변경 사항인 migration 파일이 저장되는 디렉토리
- admin.py: 각각의 앱을 Django의 관리자 기능과 연결하거나 관리자 기능에 대해 설정
- apps.py: 각각의 App마다 추가적인 기능 및 설정을 넣어 주기 위한 파일
- models.py: 앱에서 사용하는 데이터 구조를 정의하고 데이터베이스와의 소통을 담당하는 파일
- tests.py: 앱에 대한 테스트 코드를 작성하는 파일. 프로젝트를 모두 완성한 다음 테스트를 준비하는 것이 아니라 앱 별로 작은 단위의 자동화된 테스트를 미리 만들어서 프로젝트 전체에 대한 테스트가 효율적으로 이루어질 수 있도록 작성
- views.py: 앱에서 어떤 기능을 할지에 대한 메인 로직을 담당하는 파일

이제 앱을 만들었으니, 안의 기능을 구성해보자.

## 1-2. 모델
장고의 장점 중 하나는 모델을 이용해 장고 웹 프레임워크 안에서 데이터베이스를 관리할 수 있다는 것이다. 여기서 말하는 모델은 데이터를 저장하기 위한 하나의 단위라고 생각하면 된다. 이전에 blog 앱과 single_pages 앱을 만들었다. blog 앱에 게시글의 형태를 설정하는 post 모델을 만들어보자.

아까 모델을 데이터를 저장하기 위한 하나의 단위라고 설명했다. 그 말을 곱씹으며 post 모델을 만들 때 어떤 내용이 들어가야할지 생각해보자. 게시글에는 어떤 내용이 들어갈까? 기본적으로 제목, 내용, 작성일, 작성자 정보가 필요하다는 것을 떠올릴 수 있다. 따라서 우리는 post 모델을 아래와 같이 작성해준다.

```python
# blog/models.py
from django.db import models

class Post(models.Model):
    title = models.CharField(max_length=30)
    content = models.TextField()

    create_at = models.DateTimeField()
    # author = 추후 작성 예정
```
> 모델 설명
- Post 모델은 models 모듈의 Model 클래스를 확장해서 만든 파이썬 클래스이다.
    - Django에서 대부분의 모델은 이런 방식으로 만든다.
- title 필드: 문자를 담는 필드인 CharField로 생성, max_length=30으로 지정하여 최대 길이 30자로 제한
- content 필드: 문자열의 길이 제한이 없는 TextField로 생성
- create_at 필드: 월, 일, 시, 분, 초까지 기록할 수 있게 해주는 DateTimeFeild로 생성
- author 필드: 나중에 외래키 역할을 구현할 때 사용할 것이므로 우선 보류

이렇게 모델을 만들었으니 뭘 해야할까? 바로 마이그레이션이다! 마이그레이션을 해줘야 내가 추가한 모델을 적용할 수 있다. 그럼 바로 마이그레이션을 해볼까?! 정답은 틀렸다.

지금 상황에서 마이그레이션을 한다면 에러가 발생할 것이다. 그 이유는 프로젝트 파일에 내가 만든 앱을 등록시켜주지 않았기 때문이다. 따라서 먼저 앱을 프로젝트에 등록시켜주고, 블로그 앱의 모델 변경 사항을 마이그레이션 해주자!

### 1-2-1. 프로젝트에 앱 등록
프로젝트에 만든 앱을 등록시켜줘야 한다고 설명했다. 어떻게 등록한다는 말인가? 바로 프로젝트의 환경과 구성을 저장하고 있는 `settings.py`에 등록시켜주면 된다! settings.py를 열어보면 굉장히 많은 내용이 담겨있는데, 그 중 `INSTALLED_APPS`에 내가 만든 앱의 이름을 적어주면 된다.

```python
# do_it_django_prj/settings.py
(...생략...)
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog',
    'single_pages',
]
(...생략...)
```

내가 만든 앱의 이름은 `blog`, `single_pages`이기 때문에 모두 작성하고 저장했다. 이 과정이 끝났다면 이제 변경사항을 마이그레이션할 차례다. 

### 1-2-2. 모델의 변경사항 마이그레이션
마이그레이션을 위해 앱의 변경사항을 알려줄 파일을 만들고, 마이그레이션을 진행하면 된다. 우선 마이그레이션할 내용을 알려줄 파일을 만들자.

```bash
python manage.py makemigrations
```

위의 명령어를 입력했다면, blog 앱의 migrations 디렉토리에 "0001_initial.py" 라는 파일이 생성됐을 것이다. 그럼 이제 진짜 마이그레이션 준비가 끝났다! 바로 마이그레이션을 해보자.

```bash
python manage.py migrate
```

성공했다는 문구가 출력되면 끝이다! 지금까지 앱을 만들고 앱 등록, 모델 생성, 마이그레이션의 과정을 수행했다. 그럼 이제 관리자 페이지에서 확인해볼까??

## 1-3. 관리자 페이지
관리자 페이지에 접속하면 우리가 만든 앱을 확인할 수 있다. 그리고 앱의 모델에 작성한 것이 적용되어 있는 것도 확인할 수 있을 것이다. 그럼 마냥 서버를 실행시켜서 확인해보면 될까? 당연히 안 된다.

앱은 만들어졌지만, 앱의 내용을 담고 있는 models.py의 수정된 내용을 관리자 정보에 등록시켜줘야 한다. 등록해주지 않으면 비어있는 앱으로 간주되어 관리자 페이지에서는 확인할 수 없을 것이다. 따라서 관리자 정보를 저장하는 admin.py에 내가 만든 모델의 정보를 등록시켜보자.

### 1-3-1. 관리자 정보에 앱 등록하기
```python
# blog/admin.py
from django.contrib import admin
from .models import Post 

admin.site.register(Post)
```

>models가 아닌 .models인 이유?
- from에서 .을 붙이면 현재 패키지를 뜻하는데, 현재 패키지의 models.py를 가져올 것이기 때문에 .models로 쓴다.

이렇게 관리자 정보 파일에 등록시켰다면 이제 서버를 가동시켜 확인해보자.

![image](https://user-images.githubusercontent.com/77676907/199164248-990ed90e-4c02-4ff0-85c6-1bc1b75e2d84.png)

![image](https://user-images.githubusercontent.com/77676907/199164303-22f0a55d-e822-48a6-ac73-263393959086.png)

![image](https://user-images.githubusercontent.com/77676907/199164363-b30f3c90-2f42-4eb9-a8cc-2b1de8a3c82e.png)

확인해보니 Blog 앱에 Posts 모델이 보이고, 들어가보니 아직 아무 내용이 없었다. 그리고 "ADD Post +"를 누르니 아까 만든 형식대로 입력할 수 있게 되어 있는 것을 확인할 수 있었다. 아무 내용이나 입력해서 포스팅을 해볼까?

![image](https://user-images.githubusercontent.com/77676907/199165751-c7ad5b6d-eb3b-4b4b-8e38-3915bc3ed739.png)

포스팅을 해보니 포스팅을 한 흔적이 보인다! 근데 조금 불편하게 보인다. 포스팅의 순번, 제목이 보이지 않고 단순 Post object로 나타난다. 나중에 게시물이 많이지면 알아보기 힘들어지기 때문에 이를 개선해보자.

### 1-3-2. 관리자 페이지 Post 목록 개선하기
게시물을 작성하면 관리자 페이지에 작성된 순서(순번)와 제목이 보이면 좋겠다. 따라서 Post 모델의 내용을 조금 수정해주자.

```python
# blog/models.py
from django.db import models

class Post(models.Model):
    title = models.CharField(max_length=30)
    content = models.TextField()

    create_at = models.DateTimeField()
    # author = 추후 작성 예정

    """
    추가한 내용
    """
    def __str__(self):
        return f'[{self.pk}] {self.title}'
```

Post 객체가 생성되면 즉, 글을 작성하면 __str__ 함수가 호출되고, 장고 모델을 만들 떄 자동으로 생성된 pk 필드(=순번)와 작성한 제목(=title)을 반환하게 되므로 Post 목록에 "[순번] 제목"의 형태로 보이게 되는 원리다!

그럼 새로 게시글을 하나 써서 순번이 제대로 매겨지는지, 내가 지정한 형태로 보이는지 확인해보자.

![image](https://user-images.githubusercontent.com/77676907/199168245-4b8e1d7f-9821-4d4a-9c6f-0e0f042b0e25.png)

순번도 잘 매겨지고 지정한 형태로 잘 나오는 것을 확인할 수 있다. 다음은 작성 시간을 수정해보자.

### 1-3-3. Post 시간 수정하기
현재 설정된 기준시는 그리니치 표준시이다. 이를 한국 서울을 기준으로 한 표준시로 변경해보자.

변경을 위해 프로젝트의 전반적인 설정 정보를 담고있는 settings.py에서 수정해주면 된다.

```python
# do_it_django_prj/settings.py
(...생략...)
LANGUAGE_CODE = 'en-us'

TIME_ZONE = 'Asia/Seoul' # UTC에서 Asia/Seoul로 변경!

USE_I18N = True

USE_L10N = True

USE_TZ = False # True에서 False로 변경!
(...생략...)
```

수정을 완료했다면 방금 작성한 포스팅들을 살펴보자. 아마 서울 표준시로 잘 변경되어 있을 것이다.

![image](https://user-images.githubusercontent.com/77676907/199168673-da09bbbc-be20-416a-89aa-9c9554b72297.png)

아까 작성했던 시간을 잘 반영하고 있는 것이 확인된다. 마지막으로, 포스팅 시 자동으로 현재 시간을 적용하여 포스팅해주는 기능을 추가해보자!

### 1-3-4. 자동 시간 반영 기능
현재는 시간을 직접 지정하거나, 현재 시간을 적용하도록 직접 눌러줘야 한다. 이 작업이 번거롭기 때문에 자동으로 현재 시간을 반영할 수 있도록 하고, 만약 시간을 수정한다면 수정한 내역도 저장할 수 있도록 만들어보자.

```python
# blog/models.py
from django.db import models

class Post(models.Model):
    title = models.CharField(max_length=30)
    content = models.TextField()

    """
    수정/추가한 내용
    """
    create_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    # author = 추후 작성 예정

    def __str__(self):
        return f'[{self.pk}] {self.title}'
```

create_at 필드에 auto_new_add=True로 설정해서 처음 레코드가 생성될 떄 현재 시간이 자동으로 저장되게 한다. 그 다음 updated_at 필드를 만들고 auto_new=True로 설정해서 다시 저장할 때마다 그 시각이 저장되도록 하는 원리이다.

새로 포스팅을 작성하러 들어가니 시간과 관련된 창이 사라진 것을 확인할 수 있고, 수정하니 수정한 시간이 저장되는 것을 확인했다.

![image](https://user-images.githubusercontent.com/77676907/199172886-6db88cc9-4cb1-4d4f-9523-e072e588ba36.png)

그럼 이제 모델 수정이 끝났으니 마이그레이션 파일을 만들고 마이그레이션을 해주면 끝이다!!