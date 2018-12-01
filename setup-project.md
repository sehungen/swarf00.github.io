---
layout: article
title: 프로젝트 만들기
aside:
  toc: true
sidebar:
  nav: docs-ko
metadata:
  og_title: 장고(Django) 튜토리얼 프로젝트 만들기
  og_type: article
  og_locale: 'ko_KR'
  og_description: 장고(Django) 웹프레임워크의 프로젝트와 앱 생성하는 방법을 설명합니다. bbs라는 앱을 만들어 간단한 장고의 기능을 활용하는 방법을 배웁니다.
  og_site_name: 장고(Django) 핥짝 맛보기
---

## 1. 프로젝트 생성

먼저 virtualenv 가상환경에서 django-admin 커맨드로 project를 만듭니다. 프로젝트 명은 **minitutorial**이라고 이름을 짓겠습니다.
```bash
(test-venv-36) $ django-admin startproject minitutorial
```
아래와 같이 디렉토리와 파일들이 생성이 됩니다.
```
minitutorial/        # 프로젝트의 루트 디렉토리입니다. 디렉토리이름은 변경하셔도 됩니다.
     manage.py       # CLI에서 장고 프로젝트의 다양한 기능들을 사용할 수 있게 해주는 유틸리티입니다.
     minitutorial/   # 실제 프로젝트 디렉토리입니다. 프로젝트의 설정을 할 수 있으며, 파이썬 패키지로 사용됩니다.
         __init__.py # 파이썬 패키지에 필수로 들어있는 초기화 파일입니다. 프로젝트를 패키지로 불러올 때 가장 먼저 실행되는 스크립트입니다.
         settings.py # 프로젝트 설정파일입니다.
         urls.py     # 웹 url들을 view와 매칭시켜주는 파일입니다.
         wsgi.py     # WSGI 호환 웹 서버로 서비스할 때 실행되는 시작점입니다.
```
일단 프로젝트 디렉토리와 파일들이 정상적으로 생성되었다면 장고 서버를 구동시킬 수 있습니다.
```bash
(test-venv-36) $ ./manage.py runserver
Performing system checks...

System check identified no issues (0 silenced).

You have 15 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.

November 21, 2018 - 07:06:42
Django version 2.1.3, using settings 'minitutorial.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```
`Starting development server at http://127.0.0.1:8000/` 와 같은 문구가 나타났다면 정상적으로 실행된 것입니다. 브라우저에서 http://127.0.0.1:8000/으로 접속하셔서 django의 기본적으로 제공하는 화면을 확인하실 수 있습니다.
manage.py 유틸리티의 `runserver` 커맨드는 장고를 서버로서 실행시키는 커맨드입니다. 하지만 동시 접속이 일어날 때 성능이 좋지 못 합니다. 왜냐하면 python 하나의 프로세스에서 동작하기 때문입니다. 앞으로 개발을 진행하는 동안은 이렇게 계속 실행할 테지만 실제 프로덕션 환경에서는 Apache 또는 nginx 등의 웹서버를 이용해야 안정적이고 빠른 서비스를 제공할 수 있습니다.

> 프로젝트는 여러 앱들로 구성되고 이러한 앱들을 적절하게 설정해주는 곳입니다. 프로젝트를 구성하는 앱은 하나의 어플리케이션입니다. 보통 앱은 현재의 프로젝트 뿐만 아니라 다른 프로젝트에서도 `재사용가능하도록 기능을 독립적`으로 개발합니다. 예를 들어 현재 개발하려는 게시판 앱은 현재의 프로젝트에서(minitutorial)도 사용할 수 있지만 또다른 프로젝트 B에서도 재사용할 수 있습니다. 재사용 가능하도록 앱 단위로 분리해 개발한다면 여러 프로젝트를 개발한다 하더라도 효율성을 높일 수 있습니다.

## 2. 게시판 앱 생성

현재 디렉토리(프로젝트 루트 디렉토리)에서 게시판 앱을 생성합니다. 

`manage.py` 를 이용하면 간단하게 앱의 뼈대(디렉토리와 기본 파일들)를 생성할 수 있습니다.
```bash
(test-venv-36) $ ./manage.py startapp bbs
```
`startapp` 이라는 커맨드로 bbs라는 앱을 생성합니다. bbs라는 디렉토리를 생성하고 그 안에 앱에서 필요한 파일들을 생성합니다.
```
bbs/
    __init__.py      # 앱 패키지 초기화 스크립트입니다.
    admin.py         # 장고 어드민 설정파일입니다. 어드민에 등록된 모델은 장고에서 자동 생성하는 어드민 페이지에서 관리할 수 있습니다.
    apps.py          # 앱 설정 파일입니다.
    migrations/      # 데이터베이스 마이그레이션 디렉토리. 장고 ORM은 모델 스키마의 변화가 생길 때마다 migration 파일을 생성하고 이것을 통해 스키마를 업데이트 합니다. migration 파일을 통해 협업자들과 함께 데이터베이스의 스키마를 동기화할 수 있습니다.
        __init__.py  # 마이그레이션 패키지 초기화 스크립트입니다.
    models.py        # 앱 모델 파일입니다. 게시판의 모든 데이터를 저장할 데이터베이스를 장고 ORM을 통해 모델화합니다.
    tests.py         # 앱 내의 기능들을 테스트하는 기능을 구현하는 파일입니다.
    views.py         # 앱의 화면(template)과 데이터(model) 사이에서 사용자의 요청을 처리하여 모델에 저장하고, 모델에 저장된 데이터를 화면에 전달하는 역할을 합니다.
```
> 장고는 MVC 웹 프레임워크입니다. Model, View, Controller라는 컴포넌트로 기능들을 분리해서 개발하는 개발방식입니다.
>
> 1. Model : Models.py에서 구현합니다. 주로 데이터베이스에 저장하거나 저장된 데이터를 불러오는 역할을 합니다.
> 2. View : template에서 구현합니다. template은 여러개의 template들을 조합하여 하나의 뷰를 생성합니다. 컨트롤러를 통해 전달받은 데이터를 화면에 출력하고, 사용자의 입력을 컨트롤러에게 전달해주는 역할을 합니다.
> 3. Controller : views.py에서 구현합니다. 뷰와 모델 사이에서 데이터를 전달하고 사용자의 요청에 대해 적절하게 처리하는 부분입니다.

## 3. 간단한 앱 테스트

실제 views.py파일을 열어 안에 있는 모든 내용을 지우고 사용자의 요청을 처리하도록 아래와 같이 파이썬 코드를 작성해봅니다.
```python
# bbs/views.py

from django.http import HttpResponse

def hello(request):                     # 핸들러 선언. 언제나 첫번째 인자는 request 객체입니다.
    return HttpResponse('Hello world.') # 핸들러의 반환값. HttpResponse 함수를 통해 문자열을 반환합니다.
```
아직 모델을 생성시키지 않았기 때문에 모델과 연결하는 부분은 없습니다.
views.py에 사용자의 요청을 처리해 줄 핸들러를 생성했지만, 아직 핸들러가 언제 호출되어야 하는지에 대한 정의가 없습니다.
언제 `hello_world`핸들러를 호출할 지 프로젝트 내의 urls.py에 추가합니다.
```python
# minitutorial/urls.py

from django.contrib import admin
from django.urls import path

from bbs.views import hello          # 작성한 핸들러를 사용할 수 있게 가져옵니다.

urlpatterns = [
    path('hello/', hello),           # 'hello/'라는 경로로 접근했을 때 hello 핸들러가 호출되도록 추가합니다.
    path('admin/', admin.site.urls),
]
```
urls.py는 url과 핸들러를 연결해주는 설정파일입니다.urlpatterns라는 리스트 객체에 path함수의 결과값을 넣어주면 됩니다. 

`hello/` 경로에 handler를 연결했기 때문에 장고 서버를 재시작하고(`./manage.py runserver`) 브라우저에서 `http://127.0.0.1:8000/hello/` 로 접속하면 `Hello world.`가 출력되는 것을 확인할 수 있습니다.

path 함수는 4개의 인자를 받을 수 있는데 route(url), view(handler)는 필수로 입력해야 합니다. route는 `hello/`처럼 평범한 url로 표현할 수 있지만 url에 변수가 포함이 된다면 `<variable>` 로 표현해서 핸들러에게 키워드인자 variable을 전달합니다. 
> 변수를 이용해서 구현해야 하는 이유를 이해해야 합니다. 예를들어 `hello/swarf00`라는 url로 접속할 경우 `hello swarf00`이라고 출력하고, `hello/dragonball`라는 url로 접속한다면 `hello dragonball`라고 출력한다고 합시다. 변수를 사용하지 않는다면 각각의 url을 처리해 줄 핸들러를 `hello_swarf00`, `hello_dragonball` 처럼 각각 함수로 만들어줘야 합니다. `swarf00` 또는 `dragonball` 등의 두번째 단어들만 변경해서 출력하면 되는데 각각 핸들러를 만들어 주는 것은 굉장히 번거롭고, 기능을 수정해야 할 때 핸들러의 수 만큼 수정작업을 해줘야 하는 불편함이 있습니다. 지금 당장은 복붙으로 해결가능하지만 앞날을 위해 반복될 만한 것들은 모두 변수로 처리해줘야 합니다.

예를 들어 `path('hello/<to>', hello),` 라고 수정하면 브라우저에서 `http://127.0.0.1:8000/hello/world`라고 접속할 수 있고(world 대신 어떠한 문자열도 가능) 핸들러에게 to 라는 키워드 인자가 전달되어 이것을 처리할 수 있습니다.
```python
# bbs/views.py

from django.http import HttpResponse

def hello(request, to):                 # 핸들러 선언. 언제나 첫번째 인자는 request 객체입니다. request 파라미터 이후에 url패턴을 통해 전달받을 파라미터들을 선언합니다.
                                        # 이 때 파라미터이름은 url패턴에서 사용된 변수의 이름과 동일해야 합니다.(키워드인자이기 때문에)
    return HttpResponse('Hello {}.'.format(to))
```

```python
# minitutorial/urls.py

from django.contrib import admin
from django.urls import path

from bbs.views import hello

urlpatterns = [
    path('hello/<to>', hello),
    path('admin/', admin.site.urls),
]
```

`http://127.0.0.1:8000/hello/world`에서 world를 대신해서 다른 어떠한 문자열이든 입력하여 접속해보세요. 입력에 따라 출력되는 문자열이 달라집니다.

이렇게 생성한 프로젝트와 앱 뼈대에서 모델, 뷰, 템플릿을 하나씩 추가해나가면 어느새 다른 웹 프레임워크에서 느끼지 못했던 안정적이고 빠른 웹개발을 성공할 수 있습니다.

> 나는 실패한 적이 없다. 그저 동작하지 않는 10,000가지의 쓰레기 같은 코드를 발견했을 뿐이다.
>
> -- swarf00, 프로젝트를 시작하며...