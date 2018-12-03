---
layout: article
title: 인증과 권한
key: django-authentication
aside:
  toc: true
sidebar:
  nav: docs-ko
pageview: true
tags: 장고 Django auth 인증 가입 로그인 로그아웃
metadata:
  og_title: 장고(Django) 사용자인증 활용하기
  'og_type': article
  'og_locale': 'ko_KR'
  'og_description': 장고(Django) 웹프레임워크에서 기본 제공하는 auth 프레임워크를 이용하여 사용자인증을 구현하는 방법을 설명합니다.
  'og_site_name': 장고(Django) 핥짝 맛보기
---

# 1. 사용자인증

기본적인 게시판 기능이 완성되었는데, 누구라도 작성이 가능하고 누구라도 자신이 작성하지 않은 게시글까지 수정이 가능한 상황입니다. 또한 어뷰징을 하는 사용자가 있다면 그런놈들은 영구적으로 사용을 하지 못하도록 하고 싶습니다. 이럴 때 접속자들에게 꼬리표를 붙여주면 됩니다. 만일 꼬리표를 마음대로 떼버린다면 기능에 제한해버리면 됩니다. 이것을 `사용자인증` 이라고 합니다.

## 장고 auth 프레임워크
장고 admin 사이트에 접속할 때 생성했던 슈퍼유저가 기억나실 겁니다. 이것이 장고에서 기본적으로 제공하는 인증기능입니다. `id`와 `비밀번호`를 포함한 모든 사용자정보는 데이터베이스에 기록이 되고 로그인을 할 때 입력한 `id`와 `비밀번호`가 동일한 경우 해당 `id`의 사용자가 맞다고 판단하게 됩니다. 
장고 auth 프레임워크는 크게 `가입`, `로그인`, `로그아웃` 세가지의 기능을 제공합니다. 앞으로 각 기능이 어떻게 구현되어 있는지 메커니즘을 이해하며 공부를 하면 좀 더 안전한 웹서비스를 구축할 수 있습니다.

### 가입
가입이라는 기능은 모두들 아시는 기능이겠지만 엔지니어링적으로 해석해보면 **사용자정보를 데이터베이스에 저장**하고, 저장된 데이터를 구분할 수 있는 **유일한(누구와도 중복되지 않는) 키(key)를 지정해서 사용자를 식별(구별)**하는 기능입니다. 그렇기 때문에 가입의 핵심기능은 사용자를 구분할 수 있는 키를 사용자로부터 얻어오고 이것이 중복되지 않도록 하는 것입니다. 추가로 우리는 로그인 기능을 제공할 것이기 때문에 비밀번호도 사용자에게 입력받아야 합니다. 비밀번호는 ~~사회공학적으로~~ 여러 사이트에서 동일하게 사용하는 경우도 있기 때문에 현재의 웹사이트가 어떠한 문제로 또는 내부자의 악의적인 행위에 의해 데이터베이스가 노출될 경우 사용자에게 큰 피해를 줄 수 있습니다. 때문에 보통 비밀번호는 암호화를 해서 해당 웹서비스에서만 알아볼 수 있게 하거나, 해싱함수를 통해서 원래의 비밀번호를 알아볼 수 없도록 만들어 저장합니다. 

암호화를 하는 방법은 성능적으로도 크게 차이나지 않지만 좋지않은 편이고, 암호화키를 알고있거나 소스코드를 볼 수 있다면 원래의 비밀번호를 알 수 있기 때문에 그리 좋은 방법은 아닙니다. 다만 어쩔 수 없이 원래의 비밀번호를 사용자에게 제공해야 할 경우에 한 해서만 사용할 수 있습니다.

대부분의 웹사이트는 비밀번호를 해싱함수를 통해 원래의 비밀번호를 알아낼 수 없도록 저장합니다. 혹시라도 데이터베이스가 해킹을 당한다 하더라도 원래의 비밀번호는 ~~해시 알고리즘의 보안강도와 키길이에 따라~~ 알아낼 수 없습니다. 

해시된 비밀번호를 사용하는 방법은 로그인 기능을 구현할 때 알아보도록 하고 지금은 장고에서 비밀번호가 **해시함수로 원래의 비밀번호를 알아볼 수 없게 저장한다는 사실을 기억**해두면 됩니다.

> 암호화와 해시
> 1. 공통점 - 원래의 텍스트(평문)을 알아볼 수 없는 텍스트(암호문)으로 변경시켜줍니다.
> 2. 차이점 - 암호화는 암호문에서 평문으로 되돌릴 수 있지만 해시는 되돌릴 수 없습니다.
> 3. 암호 알고리즘의 종류 - des, aes, seed, rsa 등
> 4. 해시 알고리즘의 종류 - md5, sha1, sha256, sha256 등

장고의 모델도 실제 데이터베이스에 마이그레이션이 되어야 동작이 되는데 언제 마이그레이션이 되었을지 궁금하지 않아 하실 것 같아 알려드립니다. [모델 만들기]({{ site.url }}/build-model.html#모델-수정)를 보시면 migrate 명령어를 실행할 때 `Applying auth.000******` 이런식의 출력을 보셨을 겁니다. 이것이 장고의 auth 프레임워크의 모델들이 마이그레이션되었다는 로그입니다. 그럼 실제 db.sqlite3 파일을 열어서 테이블이 생성되었는지 ~~귀찮더라도~~ 확인을 해봅니다. db.sqlite3 파일은 sqlite3이라는 유틸리티(또는 그외 sqlite3 지원 유틸리티)를 통해서 확인하셔야 합니다. 일반 텍스트에디터로는 내용을 확인할 수 없습니다. sqlite 파일을 살펴보는 부분은 건너뛰셔도 상관없습니다.

sqlite3로 파일을 열면 프롬프트가 출력이 됩니다.
```bash
(test-venv-36) $ sqlite3 db.sqlite3
SQLite version 3.22.0 2018-01-22 18:45:57
Enter ".help" for usage hints.
sqlite> 
```

프롬프트에 .tables 라는 명령어를 입력하고 엔터를 입력하면 테이블 목록이 출력됩니다.
```bash
sqlite> .tables
auth_group                  bbs_article               
auth_group_permissions      django_admin_log          
auth_permission             django_content_type       
auth_user                   django_migrations         
auth_user_groups            django_session            
auth_user_user_permissions
```

테이블 목록에 여러 테이블 이름이 출력됩니다. 모든 테이블 이름이 applabel_modelname으로 구성되어 있습니다. 즉 auth 앱에는 6개의 테이블, bbs 앱에는 1개의 테이블, django 앱에는 3개의 테이블이 생성되어 있습니다. auth 앱의 user모델 즉 auth_user 테이블이 어떤 필드들이 있는지 확인해보면 10개의 필드로 구성되어 있다는 것을 알 수 있습니다.

```bash
sqlite> PRAGMA table_info(auth_user);
0|id|integer|1||1
1|password|varchar(128)|1||0
2|last_login|datetime|0||0
3|is_superuser|bool|1||0
4|username|varchar(150)|1||0
5|first_name|varchar(30)|1||0
6|email|varchar(254)|1||0
7|is_staff|bool|1||0
8|is_active|bool|1||0
9|date_joined|datetime|1||0
10|last_name|varchar(150)|1||0
```

특별히 sqlite3 유틸리티를 종료(빠져나오는) 방법을 알려드리겠습니다.
```bash
sqlite> .quit
```

> sqlite3 다운로드
>
> 맥은 기본설치되어 있으나 리눅스에서는 apt-get 또는 yum으로 설치하시면 됩니다. 윈도우의 경우에만 특별히 [다운로드](https://www.sqlite.org/download.html)를 하시면 됩니다.

여기까지는 굳이 확인하지 않아도 되는 부분이지만 아래 부터는 꼭 건너뛰지 마시고 잘 살펴보시길 바랍니다.

가입기능을 구현하기에 앞서 장고 auth 프레임워크의 user 모델을 그대로 사용할 지 결정해야 합니다. 우선 어떻게 정의되었는 지 확인해봅니다. 장고와 그 외의 라이브러리는 가상환경의 `lib/python3.6/site-packages` 디렉토리에 설치가 됩니다.
```python
# test-venv-36/lib/python3.6/site-packages/django/contrib/auth/models.py

class AbstractUser(AbstractBaseUser, PermissionsMixin):
    username = models.CharField(
        _('username'),
        max_length=30,
        unique=True,
        help_text=_('Required. 30 characters or fewer. Letters, digits and @/./+/-/_ only.'),
        validators=[
            validators.RegexValidator(
                r'^[\w.@+-]+$',
                _('Enter a valid username. This value may contain only '
                  'letters, numbers ' 'and @/./+/-/_ characters.')
            ),
        ],
        error_messages={
            'unique': _("A user with that username already exists."),
        },
    )
    first_name = models.CharField(_('first name'), max_length=30, blank=True)
    last_name = models.CharField(_('last name'), max_length=30, blank=True)
    email = models.EmailField(_('email address'), blank=True)
    is_staff = models.BooleanField(
        _('staff status'),
        default=False,
        help_text=_('Designates whether the user can log into this admin site.'),
    )
    is_active = models.BooleanField(
        _('active'),
        default=True,
        help_text=_(
            'Designates whether this user should be treated as active. '
            'Unselect this instead of deleting accounts.'
        ),
    )
    date_joined = models.DateTimeField(_('date joined'), default=timezone.now)

    class Meta:
        verbose_name = _('user')
        verbose_name_plural = _('users')
        abstract = True

class User(AbstractUser):
    class Meta(AbstractUser.Meta):
        swappable = 'AUTH_USER_MODEL'
```

복잡하게 생겼지만 조금씩 뜯어보면 복잡한 내용은 없습니다. AbstractUser, User 두개의 클래스가 정의되어 있는데 User 클래스는 AbstractUser클래스를 상속받아 정의하고 Meta클래스를 제외한 다른 부분은 오버라이딩한 것이 없네요. AbstractUser는 inner 클래스인 Meta 클래스를 보면 abstract = True로 옵션이 설정되어 있는 것이 확인됩니다. abstract 옵션이 True로 설정된 클래스는 makemigrations 커맨드 실행시에 무시합니다. abstract 클래스를 보통 비슷한 여러 개의 클래스를 정의할 때 사용합니다. abstract 모델 클래스의 서브클래스(상속받은 클래스)는 상속받은 필드와 메소드는 정의할 필요없고 추가되는 필드와 메소드만 정의하면 됩니다. auth 프레임워크는 여러개의 서브클래스를 제공하는 것은 아니지만 아마도 장고를 사용하는 여러분을 위해서 abstract로 제공하는 것 같습니다. 만일 여러 종류의 사용자 테이블이 필요하다면 이 AbstractUser 클래스를 상속받아 사용하시면 편리합니다.

> Meta 클래스는 outer 클래스의 옵션을 설정합니다. 가장 빈번하게 사용하는 옵션은 ordering, indexes, unique_together, index_together가 있습니다. 
> 1. [ordering](https://docs.djangoproject.com/ko/2.1/ref/models/options/#ordering) - 검색(SELECT) 시 기본 정렬 기준입니다.
> 2. [indexes](https://docs.djangoproject.com/ko/2.1/ref/models/options/#indexes) - 테이블의 인덱스를 정의하는 옵션입니다. 인덱스는 database에서 검색이 빠르게 하는 기능이라고 생각하시면 됩니다. 메모리를 많이 소모하니 빈번하게 검색되는 경우에만 사용합니다. **index_together는 deprecate 상태여서 추후 업데이트시 사라질 수 있으니 indexes를 사용**하면 된다고 생각하시면 됩니다.
> 3. [unique_together](https://docs.djangoproject.com/ko/2.1/ref/models/options/#unique-together) - 데이터의 중복을 막기 위해 사용하는 옵션으로 두개 이상의 필드를 조합할 수 있습니다. database 단에서 unique index를 생성합니다.
> 4. [index_togeter](https://docs.djangoproject.com/ko/2.1/ref/models/options/#index-together) - 데이터베이스에서 index를 생성합니다. 두개 이상의 필드로 정의할 수 있습니다.
>
> 사용법은 [참고문서](https://docs.djangoproject.com/en/2.1/ref/models/options/)를 보시면 됩니다.
> 
> 이 외도 클래스 옵션이 많이 있으니 [참고문서](https://docs.djangoproject.com/en/2.1/ref/models/options/)의 내용을 다 외우지는 마시고 이런 기능들이 있구나 정도만 기억하시면 됩니다.

AbstractUser 모델에 불필요하거나 변경하고 싶은 필드가 있습니다. 예를들면 우리는 id로 username이 아니라 email을 사용하고 싶습니다. 그러면 사용자별로 알림을 할 때 email필드를 사용할 수 있겠죠. 또한 한국사람은 first_name과 last_name을 따로 구분할 필요가 없는데 불필요하게 구분되어 삭제하면 좋겠습니다. 그 외의 필드들은 필요하거나 있으면 나쁘지 않은 것 같습니다. 그럼 ~~굳이~~ AbstractUser를 사용하지 않고 새로 사용자 정보 모델을 정의하도록 하겠습니다. bbs 앱에 사용자정보 모델을 추가할 수도 있으나 bbs앱을 제 3의 프로젝트에서도 사용할 수 있게 하려면 auth 프레임워크처럼 별도의 앱으로 분리하는 것이 좋을 것 같습니다. 다른 프로젝트에서는 장고의 User 모델을 그대로 사용할 수도 있으니까요.

먼저 사용자 앱을 만들겠습니다.
```bash
(test-venv-36) $ ./manage.py startapp user
```
생성시키면 bbs 앱과 같이 user 디렉토리와 파일들이 생성될 것입니다. 먼저 AbstractUser 모델을 참고해서 새로운 User 모델을 정의합니다.


```python
# bbs/models.py

from django.contrib.auth.models import (
    AbstractBaseUser, PermissionsMixin
)
from django.db import models
from django.utils import timezone


class User(AbstractBaseUser, PermissionsMixin):
    email = models.EmailField('email', unique=True)
    name = models.CharField('이름', max_length=30, blank=True)
    is_staff = models.BooleanField('스태프 권한', default=False)
    is_active = models.BooleanField('사용중', default=True)
    date_joined = models.DateTimeField('가입일', default=timezone.now)
    
    objects = UserManager()
    
    USERNAME_FIELD = 'email'                     # email을 사용자의 식별자로 설정
    REQUIRED_FIELD = 'name'                      # username
```
모델을 추가한 뒤 user 앱을 등록합니다.
```pyhon
# minitutorial/settings.py

# 생략

INSTALLED_APPS = [
    'bbs',
    'user',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

# 생략
```
이제 makemigrations 커맨드로 마이그레이션 파일을 생성합니다.
```bash
(test-venv-36) $ ./manage.py makemigrations
Traceback (most recent call last):
  File "./manage.py", line 15, in <module>
    execute_from_command_line(sys.argv)
  File "/Users/sehunkim/test-venv-36/lib/python3.6/site-packages/django/core/management/__init__.py", line 381, in execute_from_command_line
    utility.execute()
  File "/Users/sehunkim/test-venv-36/lib/python3.6/site-packages/django/core/management/__init__.py", line 375, in execute
    self.fetch_command(subcommand).run_from_argv(self.argv)
  File "/Users/sehunkim/test-venv-36/lib/python3.6/site-packages/django/core/management/base.py", line 316, in run_from_argv
    self.execute(*args, **cmd_options)
  File "/Users/sehunkim/test-venv-36/lib/python3.6/site-packages/django/core/management/base.py", line 353, in execute
    output = self.handle(*args, **options)
  File "/Users/sehunkim/test-venv-36/lib/python3.6/site-packages/django/core/management/base.py", line 83, in wrapped
    res = handle_func(*args, **kwargs)
  File "/Users/sehunkim/test-venv-36/lib/python3.6/site-packages/django/core/management/commands/makemigrations.py", line 144, in handle
    ProjectState.from_apps(apps),
  File "/Users/sehunkim/test-venv-36/lib/python3.6/site-packages/django/db/migrations/state.py", line 222, in from_apps
    model_state = ModelState.from_model(model)
  File "/Users/sehunkim/test-venv-36/lib/python3.6/site-packages/django/db/migrations/state.py", line 411, in from_model
    fields.append((name, field.clone()))
  File "/Users/sehunkim/test-venv-36/lib/python3.6/site-packages/django/db/models/fields/__init__.py", line 493, in clone
    name, path, args, kwargs = self.deconstruct()
  File "/Users/sehunkim/test-venv-36/lib/python3.6/site-packages/django/db/models/fields/__init__.py", line 1209, in deconstruct
    del kwargs['editable']
KeyError: 'editable'
```
오류가 발생하는데 Article 모델의 created_at 필드의 속성을 강제로 editable로 변경해서 생긴 문제입니다. admin 페이지에서 보기위해 추가했던 건데 이제 필요없으니 우선 주석표시를 해둡니다.

```python
# bbs/models.py

from django.db import models


class Article(models.Model):
    title      = models.CharField('제목', max_length=126, null=False)
    content    = models.TextField('내용', null=False)
    author     = models.CharField('작성자', max_length=16, null=False)
    created_at = models.DateTimeField('작성일', auto_now_add=True)
    # created_at.editable = True

    def __str__(self):
        return '[{}] {}'.format(self.id, self.title)
```
다시 makemigration 커맨드를 실행하면 정상적으로 마이그레이션 파일이 생성이 됩니다. migrate 커맨드까지 이어서 실행하면 마이그레이션이 완료됩니다.
```bash
(test-venv-36) $ ./manage.py makemigrations
Migrations for 'user':
  user/migrations/0001_initial.py
    - Create model User
(test-venv-36) $ ./manage.py migrate
Migrations for 'user':
  user/migrations/0001_initial.py
    - Create model User
(test-venv-36) SEHUNui-MacBook-Pro:minitutorial sehunkim$ ./manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions, user
Running migrations:
  Applying user.0001_initial... OK
```


### 로그인


### 로그아웃


## 회원가입

## 로그인

## 로그아웃

# 2. 권한부여

# 3. 