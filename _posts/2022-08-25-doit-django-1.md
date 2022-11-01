---
title: Django 겉할기 시리즈 - Django 구조 이해하고 프로젝트 생성하기
description: 이지스 퍼블리싱의 Do it! Django + 부트스트랩책을 보며 공부한 내용을 정리했습니다.
categories:
- Django
- BackEnd
tags: 
- Django
- Doit
- BackEnd
- Django Architecture
---

# Django란?
- 파이썬 언어로 만들어진 웹 프레임 워크
- 웹 프레임워크: CRUD(Create, Read, Update, Delete)를 수행하고, 관리자 페이지를 지원하여 웹 사이트 개발을 편리하게 해주는 도구

# Django의 작동 구조
- 사용자가 컴퓨터나 웹 브라우저를 사용해 주소 창에 doitjango.com를 입력하여 접속할 때 일어나는 과정을 살펴보자.

![image](https://user-images.githubusercontent.com/77676907/199134576-c44cf917-52eb-41a0-957c-0e28a2cc0c34.png)

0. 접속하려는 웹 사이트가 Django로 만들어졌다고 가정한다.
1. 사용자가 `doitjango.com`을 입력하면 클라이언트(웹 브라우저)는 사용자가 입력한 url의 서버를 찾아간다.
2. `urls.py를 요청`해 어떤 `동작`이 일어나는지 `확인`한다. 보통 urls.py에는 특정 url에 접속했을 때 어떤 함수를 실행시켜라와 같은 내용이 기술되어 있다.
3. urls.py에 명시된 `함수 또는 클래스를 실행`하기 위해 `views.py`를 확인한다. 참고로 views.py에는 urls.py에서 사용할 함수, 클래스의 상세 내용이 적혀있다.
4. `웹 사이트 정보의 종류`는 `models.py`에 담겨있다. 예를들어, 게시글을 확인하려 한다면 models.py에서는 '게시글이 담아할 정보는 제목, 글 내용, 작성자, 작성일자이다.' 와 같이 DB에서 원하는 정보를 추출할 수 있도록 작성되어있다.
5. `models.py에 명시된대로` `DB`에서 필요한 `자료`를 가져온다.
6. 마지막으로 `Db에서 가져온 자료`를 `템블릿(html 파일)`의 지정된 곳에 출력시킬 수 있도록 `사용자의 웹 브라우저에 전송`하게 된다.

## Django의 디자인 패턴
- 앞의 예시로 장고로 만들 웹 사이트는 모델로 자료의 형태를 정의하고, 뷰로 어떤 자료를 어떤 동작으로 보여줄지 정의하고, 템플릿으로 웹 페이지에서 출력할 모습을 정의함
- 이를 MTV(Model Template View) 패턴이라 한다.

# 1. Django 시작하기
장고는 크게 프로젝트와 앱으로 구성된다. 프로젝트는 하나의 웹 사이트 뼈대이고, 앱은 웹 사이트의 내용 기능들로 이해하면 쉽다. 참고로 웹 사이트에는 다양한 내용과 기능이 있기 때문에 하나의 프로젝트에 여러 개의 앱이 존재할 수 있다. 이 구조를 이해하고 장고를 시작해보자!

## 1-1. Django 프로젝트 생성

장고를 시작하기 위해 프로젝트를 생성해야한다. 자신이 작업하는 디렉토리에서 프로젝트를 생성해보자.

```bash
django-admin startproject do_it_django_prj .
```

생성 후 디렉토리를 확인해보면, 아래와 같은 구조일 것이다.
```bash
root
├── do_it_django_prj # 생성한 프로젝트
│   ├── _pycache_
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
│
└── manage.py
```
>파일들의 역할
- root: 현재 작업하는 디렉토리의 최상단 디렉토리
- manage.py: Django django프로젝트 관리를 위하 명령어 지원. 앱(App)생성, 데이터베이스 관련 명령, 개발서버 실행 등 수행
- do_it_django_prj: 현재 사용하는 Django 프로젝트 디렉토리, 디렉토리 내부에는 프로젝트를 위한 실제 Python 패키지들이 저장된다. 이 디렉토리 내의 이름을 이용하여, (do_it_django_prj.urls 와 같은 식으로) 프로젝트의 어디서나 Python 패키지들을 임포트할 수 있다.
- do_it_django_prj/__init__.py: Python으로 하여금 이 디렉토리를 패키지처럼 다루라고 알려주는 용도의 단순한 빈 파일
- do_it_django_prj/settings.py: 현재 Django 프로젝트의 환경 및 구성을 저장
- do_it_django_prj/urls.py: 현재 Django project 의 URL 선언을 저장한다. Django 로 작성된 사이트의 “목차” 라고 할 수 있다.
- do_it_django_prj/asgi.py: ASGI 호환 웹서버와 파이썬 어플리케이션인 django가 소통하는데 필요한 프로토콜
- do_it_django_prj/wsgi.py: WSGI 호환 웹서버와 파이썬 어플리케이션인 django가 소통하는데 필요한 프로토콜

그럼 이제 프로젝트가 잘 생성되었는지 확인하기 위해 아래의 명령어로 실행시켜보자.

```bash
python manage.py runserver
```

![image](https://user-images.githubusercontent.com/77676907/199136705-e0d7d58e-e867-46c6-a4f5-1677043f4357.png)

실행해서 나오는 url에 접속하면, 위와 같은 그림이 나타날 것이다.

그럼 프로젝트 생성을 완료했으니, DB에 이를 반영시켜줘야 한다. 이를 마이그레이션(migration)이라 부른다.

## 1-2. 마이그레이션
생성되어 있는 DB에 마이그레이션을 해보자. 아래의 명령어로 마이그레이션을 수행할 수 있다.

>DB를 생성한 적이 없는데 왜 생성된건가요?

- Django는 새 프로젝트를 생성할 때 변경 사항을 저장할 DB를 같이 생성한다. 
- 그리고 기본적으로 필요한 테이블도 미리 생성하기 때문에 마이그레이션 항목들이 나타난 것이다.

```bash
python manage.py migrate
```

마이그레이션을 성공적으로 마쳤다면 `db.sqlite3` 파일이 생성되었음을 확인할 수 있다!

## 1-3. 관리자 계정
Django의 장점 중 하나는 관리자 사이트가 굉장히 편리하게 돼있어서 유지보수가 편리하다는 것이다! 관리자 사이트를 사용하기 위해 관리자 계정을 만들어보자.

```bash
python manage.py createsuperuser
```

위의 명령어를 입력하면, 생성할 관리자 이름, 이메일 주소, 비밀번호를 입력하여 관리자 계정을 생성한다. 관리자 계정 생성을 마쳤다면, admin 페이지로 이동해서 관리자 페이지를 확인할 수 있다.


```bash
python manage.py runserver

# url
# http://127.0.0.1:8000/admin/ 로 접속!
```

![image](https://user-images.githubusercontent.com/77676907/199143164-11a3f800-ead2-48e4-b07c-da950a225fdf.png)

그럼 이렇게 로그인 화면이 나타난다. 아까 만들었던 관리자 이름과 비밀번호를 입력하여 접속해보자.

![image](https://user-images.githubusercontent.com/77676907/199143270-b587f853-b327-436f-944a-57a96d85cec9.png)

![image](https://user-images.githubusercontent.com/77676907/199143345-be5fd998-5182-4973-87df-27b0199f3f79.png)


접속하면 위의 기본 화면이 나타나고, Users에 들어가보면 현재 지정된 관리자 계정의 목록이 나온다.