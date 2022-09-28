---
title: 장고 앱 작성하기, part 2
description: django 튜토리얼 part2를 통해 데이터베이스를 설치하고 관리자 페이지를 확인했습니다.
categories:
- Django
- Web Framework
tags: 
- Django
---

# 데이터베이스 설치하기
지난번 runserver 당시 경고 문구가 자꾸 출력되는 것을 확인했을 것이다. 
```
You have 18 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
```
변경/추가된 사항이 18개가 있는데 마이그레이션이 되지 않았다는 말인데, 왜 이런 문구가 나오는걸까? 장고 공식 문서에 따르면 마이그레이션을 통해 모델의 변경 내역은 DB 스키마에 적용시켜 DB 스키마를 git 처럼 버전으로 나눠서 관리할 수 있게 해주는 시스템이라고 한다! 더 쉽게 생각하면 게임에서 save 파일을 만드는 것과 비슷하다. 중간 저장이 안 돼있다고 경고를 해주는 것이니 안심하고 이제 DB를 설치하고 마이그레이션 해주자!

근데 사실 설치할 피요가 없을지도 모른다ㅎ 왜냐하면 나는 SQLite를 사용할거니까!! SQLite는 장고에서 기본적으로 사용하게끔 구성되어 있다. 어차피 지금은 연습용으로 하고 있으니 SQLite를 적극 활용할 것이다.
>장고 공식 문서에서 실제 프로젝트를 시작할 때는 나중에 데이터베이스를 교체하느라 골치 아파질 일을 피하기 위해서라도 PostgreSQL과 같은 좀 더 확장성있는 데이터베이스 사용을 권장하고 있다.

바로 마이그레이션을 해도 좋지만, mysite의 settings.py에서 수정할 사항이 있다.

```python
# settings.py

LANGUAGE_CODE = 'en-us'

TIME_ZONE = 'Asia/Seoul'  # 한국 시간 적용 

USE_I18N = True

USE_L10N = True

USE_TZ = False  # False 로 설정해야 DB에 변경 된 TIME_ZONE 이 반영 됨 
```

필자가 바꾸고 싶었던 것은 시간대이다. 장고 공식 문서에서 원하는 시간대에 맞춰 TIME_ZONE을 설정하라고 안내되어 있다. 그래서 "TIME_ZONE"과 "USE_TZ"를 다음과 같이 수정했다!

그럼 이제 manage.py가 있는 디렉토리로 가서 마이그레이션을 통해 중간저장을 해보자!!

```bash
python manage.py migrate

# output
"""
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying auth.0012_alter_user_first_name_max_length... OK
  Applying sessions.0001_initial... OK
"""
```

내가 만들고 수정했던 파일들이 저장되는 모습을 확인할 수 있다.
>이렇게 저장된 파일의 모습을 보고싶다면, DB 클라이언트로 접속해서 SELECT <TABLE_NAME> FROM <USER_TABLES>;를 통해 확인할 수 있다.

![image](https://user-images.githubusercontent.com/77676907/192125067-cd6c24b5-429e-433f-961b-283dbfddee01.png)

SQLiteBrowser를 통해 테이블명을 확인하고 내용을 살펴보니 아직 큰 내용은 없었다! 더 만들고 나서 확인해보는게 좋을 것 같았다ㅎㅎ

그럼 이제 모델을 만들어보자! 

# 모델 만들기
모델을 만들기에 앞서 모델이 뭐길래 만들어야 하는걸까? Django에서 모델은 앱을 구성하는 데이터 단위이다. 지금 내가 만드려는 설문조사 앱으로 이해해보자.

설문조사를 위해서는 질문이 있어야 할 것이고, 질문에 대한 답을 할 수 있도록 해야한다!

- 질문
	- 질문 내용
	- 발행일
- 답변
	- 답변 내용
	- 답변 집계 결과

따라서 우리는 질문과 답변이라는 모델을 만들어야 하고 각 모델의 세부 내용은 위와 같이 설정하면 된다! 조금 이해가 가는가?? 그럼 모델을 만들어보자!

```python
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

- 만들어진 각 모델은 :class:`django.db.models.Model`의 하위 클래스로 표현된다. 모델마다 여러 클래스 변수가 있으며, 각 클래스 변수는 모델에서 데이터베이스 필드를 나타낸다.
- 데이터베이스의 각 필드는 Field 클래스의 인스턴스로서 표현된다. 
  - CharField 는 문자(character) 필드를 표현 
  - DateTimeField 는 날짜와 시간(datetime) 필드를 표현
- 몇몇 Field 클래스들은 필수 인수가 필요하다. 
  - 예를 들어, CharField 의 경우 max_length 를 입력해 주어야 함. (이것은 데이터베이스 스키마에서만 필요한것이 아닌 값을 검증할때도 사용된다.)
- 또한, Field 는 다양한 선택적 인수들을 가질 수 있습니다. 
  - 예를 들어, default 로 하여금 votes 의 기본값을 0 으로 설정
- 마지막으로, ForeignKey 를 사용하여 Choice 가 하나의 Question 에 관계되므로 일대일 관계이다. 
  - Django 는 다-대-일(many-to-one), 다-대-다(many-to-many), 일-대-일(one-to-one) 과 같은 모든 일반 데이터베이스의 관계들를 지원함

## 모델 활성화
- 방금 만들 모델을 통해 Django에서는 다음과 같은 작업을 수행할 수 있다.
  - 앱을 위한 데이터베이스 스키마 생성(CREATE TABLE)
  - Question과 Choice 객체에 접근하기 위한 Python 데이터베이스 접근 API 생성
앱을 만들고 그 앱의 동작을 담당하는 모델을 만들었다! 그럼 만든 것을 등록해서 올바른 동작을 하도록 설정해보자! mysite에 있는 settings.py에 앱을 등록시켜보자.

```python
INSTALLED_APPS = [
    'polls.apps.PollsConfig',      # PollsConfig 클래스가 polls/apps.py에 위치하기 때문에 polls.apps.PollsConfig로 경로를 지정
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

이렇게 만든 앱을 settings.py의 INSTALLED_APPS예 등록시켜줬다! 그럼 이제 만든 모델을 스키마에 저장해보자!!

```bash
python manage.py makemigrations polls

# output
"""
Migrations for 'polls':
  polls/migrations/0001_initial.py
    - Create model Question
    - Create model Choice
"""
```

makemigrations 을 실행시킴으로서, 당신이 모델을 변경시킨 사실과(지금은 새로운 모델을 만들었다!) 이 변경사항을 migration으로 저장시키기 위해 migration할 정보를 만드는 것이다.

아까 데이터베이스를 만드는 이유에서 알아봤듯 migration은 Django가 모델(즉, 데이터베이스 스키마)의 변경사항을 디스크에 것이다. 원하는 경우 polls/migrations/0001_initial.py 파일로 저장된 새 모델에 대한 migration을 읽어볼 수 있으니 Django의 변경점을 수동으로 수정할 수 있다.

그럼 이제 migrate를 실행시켜 모델을 반영시키자!!

```bash
python manage.py migrate


# output
"""
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Applying polls.0001_initial... OK
"""
```

이런 문구가 나왔다면 성공이다!! 앞으로도 모델을 변경하게 makemigration 해주고 migrate를 해주자!
- models.py에서 모델을 변경 > python manage.py makemigrations을 통해 마이그레이션 정보 만들기 > python manage.py migrate를 통해 데이터베이스에 저장

# API로 Django 다루기
Django의 데이터베이스를 API로 다뤄보자! DRF를 다루기 위해서는 쉘에 접속해야한다!

```bash
python manage.py shell
```

접속했다면 아래의 명령어들을 따라치면서 출력이 올바르게 나오는지 확인해보자!
```python
from polls.models import Choice, Question # 앞에서 만들었던 모델 불러오기

Question.objects.all() # 질문들의 모든 항목들을 불러오기

#output
# <QuerySet []>  # 아직 만든 질문이 없어서 비어있음을 확인

#################################################################################################
# 새로운 질문을 만들어보자
# Django는 기본 설정 파일에 표준 시간대를 사용할 수 있도록 되어있다.
# Question 모델의 pub_date는 time zone 정보를 담은 datetime을 넘겨줘야 하기 때문에 timezone.now()를 사용하자.
from django.utils import timezone

q = Question(question_text="What's new?", pub_date=timezone.now())
q.save() # q 객체를 데이터베이스에 저장한다. (모델이 데이터스키마에 작성되어 있기 때문!)
#################################################################################################

#################################################################################################
q.id # q 객체의 id 값을 출력해보자! 저장했기 때문에 id가 생겼을 것이다.
# output
# 1 # 방금 생성한 q 객체의 id 값이다.

q.question_text # q 객체는 Question 모델의 필드 값에 접근할 수 있다.
# output
# "What's new?"

q.pun_date # 이것도 마찬가지다!
#output
# datetime.datetime(2022, 9, 26, 13, 0, 0, 775217, tzinfo=<UTC>)
#################################################################################################

#################################################################################################
q.question_text = "What's up?"  # Question 모델의 필드 값에 접근하여 값을 변경시킬 수 있다.
q.save() # 변경한 값을 반영하기 위해 save()로 다시 저장

Question.objects.all() # objects.all(): Question의 스키마에 있는 모든 값을 출력
# output
# <QuerySet [<Question: Question object (1)>]>
#################################################################################################
```

마지막 출력을 보면 내가 저장한 질문이 나오면 좋았을 텐데 저장된 질문의 id가 나왔다. 따라서 models.py를 수정하여 id가 아닌 값들이 나오도록 해보자.

```python
from django.db import models
 
 
class Question(models.Model):
  question_text = models.CharField(max_length=200)
  pub_date = models.DateTimeField('date published')

  # 추가한 부분
  def __str__(self):
    return self.question_text
 
class Choice(models.Model):
  question = models.ForeignKey(Question, on_delete=models.CASCADE)     
  choice_text = models.CharField(max_length=200)
  votes = models.IntegerField(default=0)
  
  # 추가한 부분
  def __str__(self):             
    return self.choice_text
```

__str__ 메서드를 통해 문자열인 question_text와 choice_text를 반환하도록 하였다! 잘 됐는지 확인해보자.

```python
from polls.models import Choice, Question

Question.objects.all()
# output
# <QuerySet [<Question: What's up?>]> # 정상적으로 출력된다!

# 이외에도 filter, get, create, count, delete가 있다! 자세한 내용은 공식문서를 참조하자ㅎㅎ
# 또한, 내장 메소드(__str__)말고도 직접 새로운 기능을 정의해서 메소드를 만들 수도 있다!!
```

# 관리자 사이트 접속하기
Django의 장점 중 하나는 관리자 사이트가 굉장히 편리하게 돼있어서 유지보수가 편리하다는 것이다! 관리자 사이트를 사용하기 위해 관리자 계정을 만들어보자.

```bash
python manage.py createsuperuser

# output
"""
Username (leave blank to use 'user_name'): <name>
Email address:  <abc123@abcd.com>
Password: **********
Password (again): ***********
Superuser created successfully.
"""
```

위와 같이 입력하고 successfully 문구가 나왔다면 성공이다!! 그럼 이제 개발 서버를 가동시켜서 /admin/으로 접속해보자!

```bash
python manage.py runserver
```

접속하면 아래와 같은 로그인 창이 나타난다!

![image](https://user-images.githubusercontent.com/77676907/192797519-af0ab3ae-e971-44bc-a1dd-4c154b586fc9.png)

여기에 아까 지정한 Username과 Password를 입력해서 로그인을 해보자.

![image](https://user-images.githubusercontent.com/77676907/192797876-6d92b0e8-155c-4831-92a6-a937147b8b40.png)

그럼 이렇게 관리 페이지가 보인다. 관리 페이지의 주내용은 이 프로젝트를 관리할 수 있는 그룹, 사용자의 정보이다. 그럼 여기서 드는 의문점..

내가 만든 앱은 어떻게 관리하지..? 바로 admin.py를 수정해줘야 나온다!! 그럼 polls 디렉토리로 가서 admin.py를 아래와 같이 수정하자.

```python
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```

수정을 하고 다시 접속해보니 내가 만든 polls 앱이 보이고 질문을 수정할 수 있도록 화면이 나타나는 것을 확인할 수 있다ㅎㅎㅎ

![image](https://user-images.githubusercontent.com/77676907/192800037-bc46604a-2e27-4b94-90f4-6722b07dfeaf.png)

![image](https://user-images.githubusercontent.com/77676907/192800906-106a12b7-52d9-4be1-bcae-e6a16e669163.png)

그럼 여기까지, 데이터베이스를 만들어서 모델의 변경사항을 저장하고, 관리자 페이지에 접속해서 앱을 관리하는 기능을 구현해봤다! 다음에는 투표 앱에 뷰를 추가해보는 예제를 따라해보도록 하겠다!!
# Reference
[1] [참조한 블로그: 마이그레이션](https://tibetsandfox.tistory.com/24)

[2] [위키독스: 점프 투 장고(모델)](https://wikidocs.net/70650)

[3] [Django tutorial](https://docs.djangoproject.com/ko/4.1/intro/tutorial02/)