---
layout: article
title: 사용자인증(1)
key: django-authentication-01
aside:
  toc: true
sidebar:
  nav: docs-ko
pageview: true
tags: 장고 Django auth 인증 가입 CreateView 모델폼 ModelForm
metadata:
  og_title: 장고(Django) 사용자인증 제 1 편
  og_type: article
  og_locale: ko_KR
  og_description: 장고(Django) 웹프레임워크에서 기본 제공하는 auth 프레임워크를 이용하여 사용자인증을 구현하는 방법을 설명합니다. 또한 모델폼을 이용하여 쉽게 템플릿을 구현하는 방법을 알아봅니다.
  og_site_name: 장고(Django) 핥짝 맛보기
---

## 1. 새로운 사용자모델 정의

기본적인 게시판 기능이 완성되었는데, 누구라도 작성이 가능하고 누구라도 자신이 작성하지 않은 게시글까지 수정이 가능한 상황입니다. 또한 어뷰징을 하는 사용자가 있다면 그런놈들은 영구적으로 사용을 하지 못하도록 하고 싶습니다. 이럴 때 접속자들에게 꼬리표를 붙여주면 됩니다. 만일 꼬리표를 마음대로 떼버린다면 기능에 제한해버리면 됩니다. 이것을 `사용자인증` 이라고 합니다. 이제부터는 장고의 다양한 기본 기능들을 폭 넓게 활용해도록 하겠습니다.

### 장고 auth 프레임워크 소개
장고 admin 사이트에 접속할 때 생성했던 슈퍼유저가 기억나실 겁니다. 이것이 장고에서 기본적으로 제공하는 인증기능입니다. `id`와 `비밀번호`를 포함한 모든 사용자정보는 데이터베이스에 기록이 되고 로그인을 할 때 입력한 `id`와 `비밀번호`가 동일한 경우 해당 `id`의 사용자가 맞다고 판단하게 됩니다. 
장고 auth 프레임워크는 크게 `가입`, `로그인`, `로그아웃` 세가지의 기능을 제공합니다. 앞으로 각 기능이 어떻게 구현되어 있는지 메커니즘을 이해하며 공부를 하면 좀 더 안전한 웹서비스를 구축할 수 있습니다.

사용자 인증이라는 기능은 모두들 아시는 기능이겠지만 엔지니어링적으로 해석해보면 **사용자정보를 데이터베이스에 저장**하고, 저장된 데이터를 구분할 수 있는 **유일한(누구와도 중복되지 않는) 키(key)를 지정해서 사용자를 식별(구별)**하는 기능입니다. 그렇기 때문에 가입의 핵심기능은 사용자를 구분할 수 있는 키를 사용자로부터 얻어오고 이것이 중복되지 않도록 하는 것입니다. 추가로 우리는 로그인 기능을 제공할 것이기 때문에 비밀번호도 사용자에게 입력받아야 합니다. 비밀번호는 ~~사회공학적으로~~ 여러 사이트에서 동일하게 사용하는 경우도 있기 때문에 현재의 웹사이트가 어떠한 문제로 또는 내부자의 악의적인 행위에 의해 데이터베이스가 노출될 경우 사용자에게 큰 피해를 줄 수 있습니다. 때문에 보통 비밀번호는 암호화를 해서 해당 웹서비스에서만 알아볼 수 있게 하거나, 해싱함수를 통해서 원래의 비밀번호를 알아볼 수 없도록 만들어 저장합니다. 

암호화를 하는 방법은 성능적으로도 크게 차이나지 않지만 좋지않은 편이고, 암호화키를 알고있거나 소스코드를 볼 수 있다면 원래의 비밀번호를 알 수 있기 때문에 그리 좋은 방법은 아닙니다. 다만 어쩔 수 없이 원래의 비밀번호를 사용자에게 제공해야 할 경우에 한 해서만 사용할 수 있습니다.

**대부분의 웹사이트는 비밀번호를 해싱함수를 통해 원래의 비밀번호를 알아낼 수 없도록 저장**합니다. 혹시라도 데이터베이스가 해킹을 당한다 하더라도 원래의 비밀번호는 ~~해시 알고리즘의 보안강도와 키길이에 따라~~ 알아낼 수 없습니다. 

해시된 비밀번호를 사용하는 방법은 로그인 기능을 구현할 때 알아보도록 하고 지금은 장고에서 비밀번호가 **해시함수로 원래의 비밀번호를 알아볼 수 없게 저장한다는 사실을 기억**해두면 됩니다.

> 암호화와 해시
> 1. 공통점 - 원래의 텍스트(평문)을 알아볼 수 없는 텍스트(암호문)으로 변경시켜줍니다.
> 2. 차이점 - **암호화는 암호문에서 평문으로 되돌릴 수 있지만 해시는 되돌릴 수 없습니다**.
> 3. 암호 알고리즘의 종류 - `des`, `aes`, `seed`, `rsa` 등
> 4. 해시 알고리즘의 종류 - `md5`, `sha1`, `sha256`, `sha256` 등

장고의 기본 제공 모델들도 실제 데이터베이스에 마이그레이션이 되어야 동작이 되는데, 이 모델들이 언제 마이그레이션이 되었을 지 궁금하지 않으시더라도 알려드립니다. [모델 만들기]({{ site.url }}/build-model.html#모델-수정)를 보시면 `migrate` 명령어를 실행할 때 `Applying auth.000******` 이런식의 출력을 보셨을 겁니다. 이것이 장고의 `auth` 프레임워크의 모델들이 마이그레이션되었다는 로그입니다. 그럼 실제 db.sqlite3 파일을 열어서 테이블이 생성되었는지 ~~귀찮더라도~~ 확인을 해봅니다. `db.sqlite3` 파일은 `sqlite3` 이라는 유틸리티(또는 그외 `sqlite3` 지원 유틸리티)를 통해서 확인하셔야 합니다. 일반 텍스트에디터로는 내용을 확인할 수 없습니다. **`sqlite` 파일을 살펴보는 부분은 건너뛰셔도 상관**없습니다.

`sqlite3` 로 파일을 열면 프롬프트가 출력이 됩니다.
```bash
(test-venv-36) $ sqlite3 db.sqlite3
SQLite version 3.22.0 2018-01-22 18:45:57
Enter ".help" for usage hints.
sqlite> 
```

프롬프트에 `.tables` 라는 명령어를 입력하고 엔터를 입력하면 테이블 목록이 출력됩니다.
```bash
sqlite> .tables
auth_group                  bbs_article               
auth_group_permissions      django_admin_log          
auth_permission             django_content_type       
auth_user                   django_migrations         
auth_user_groups            django_session            
auth_user_user_permissions
```

테이블 목록에 여러 테이블 이름이 출력됩니다. 모든 테이블 이름이 `applabel_modelname`으로 구성되어 있습니다. 즉 `auth` 앱에는 6개의 테이블, `bbs` 앱에는 1개의 테이블, `django` 앱에는 3개의 테이블이 생성되어 있습니다. `auth` 앱의 `user`모델 즉 `auth_user` 테이블이 어떤 필드들이 있는지 확인해보면 10개의 필드로 구성되어 있다는 것을 알 수 있습니다.

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

**여기까지는 굳이 확인하지 않아도 되는 부분**이지만 아래 부터는 꼭 건너뛰지 마시고 잘 살펴보시길 바랍니다.

가입기능을 구현하기에 앞서 장고 `auth` 프레임워크의 `User` 모델을 그대로 사용할 지 결정해야 합니다. 우선 어떻게 정의되었는 지 확인해봅니다. 장고와 그 외의 라이브러리는 가상환경의 `lib/python3.6/site-packages` 디렉토리에 설치가 됩니다.
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

복잡하게 생겼지만 조금씩 뜯어보면 복잡한 내용은 없습니다. `AbstractUser`, `User` 두개의 클래스가 정의되어 있는데 **`User` 클래스는 `AbstractUser클래스를` 상속받아 정의**하고 `Meta` 클래스를 제외한 다른 부분은 오버라이딩한 것이 없네요. `AbstractUser는` inner 클래스인 `Meta` 클래스를 보면 `abstract = True`로 옵션이 설정되어 있는 것이 확인됩니다. **`abstract` 옵션이 `True` 로 설정된 클래스는 `makemigrations` 커맨드 실행시에 무시**합니다. `abstract` 클래스를 보통 비슷한 여러 개의 클래스를 정의할 때 사용합니다. `abstract` 모델 클래스의 서브클래스(상속받은 클래스)는 상속받은 필드와 메소드는 정의할 필요없고 추가되는 필드와 메소드만 정의하면 됩니다. `auth` 프레임워크는 여러개의 서브클래스를 제공하는 것은 아니지만 아마도 장고를 사용하는 여러분을 위해서 `abstract`로 제공하는 것 같습니다. 만일 여러 종류의 **사용자 모델이 필요하다면 이 `AbstractUser` 클래스를 상속**받아 사용하시면 편리합니다.

> `Meta` 클래스는 `outer` 클래스의 옵션을 설정합니다. 가장 빈번하게 사용하는 옵션은 `ordering`, `indexes`, `unique_together`, `index_together` 가 있습니다. 
> 1. [ordering](https://docs.djangoproject.com/ko/2.1/ref/models/options/#ordering) - 검색(SELECT) 시 기본 정렬 기준입니다.
> 2. [indexes](https://docs.djangoproject.com/ko/2.1/ref/models/options/#indexes) - 테이블의 인덱스를 정의하는 옵션입니다. 인덱스는 database에서 검색이 빠르게 하는 기능이라고 생각하시면 됩니다. 메모리를 많이 소모하니 빈번하게 검색되는 경우에만 사용합니다. **index_together는 deprecate 상태여서 추후 업데이트시 사라질 수 있으니 indexes를 사용**하면 된다고 생각하시면 됩니다.
> 3. [unique_together](https://docs.djangoproject.com/ko/2.1/ref/models/options/#unique-together) - 데이터의 중복을 막기 위해 사용하는 옵션으로 두개 이상의 필드를 조합할 수 있습니다. database 단에서 unique index를 생성합니다.
> 4. [index_togeter](https://docs.djangoproject.com/ko/2.1/ref/models/options/#index-together) - 데이터베이스에서 index를 생성합니다. 두개 이상의 필드로 정의할 수 있습니다.
>
> 사용법은 [참고문서](https://docs.djangoproject.com/en/2.1/ref/models/options/)를 보시면 됩니다.
> 
> 이 외도 클래스 옵션이 많이 있으니 [참고문서](https://docs.djangoproject.com/en/2.1/ref/models/options/)의 내용을 다 외우지는 마시고 이런 기능들이 있구나 정도만 기억하시면 됩니다.

### 커스텀 사용자 모델(User)
`AbstractUser` 모델에 불필요한 필드도 있고, 변경하고 싶은 필드가 있습니다. 예를들면 우리는 사용자 식별자로 `username` 이 아니라 `email` 을 사용하고 싶습니다. 그러면 중복될 일도 없고, 사용자별로 알림을 보내야 할 때 `email` 필드를 사용할 수 있겠죠. 또한 한국사람은 `first_name` 과 `last_name` 을 따로 구분할 필요가 없는데 불필요하게 구분되어 삭제하면 좋겠습니다. 그 외의 필드들은 필요하거나 있으면 나쁘지 않은 것 같습니다. 그럼 ~~굳이~~ `AbstractUser`를 사용하지 않고 새로 사용자 정보 모델을 정의하도록 하겠습니다. `bbs` 앱에 사용자 모델을 추가할 수도 있으나 `bbs` 앱을 제 3의 프로젝트에서도 사용할 수 있게 하려면 `auth` 프레임워크처럼 별도의 앱으로 분리하는 것이 좋을 것 같습니다. 이미 **제 3의 프로젝트에서 기존의 사용자 테이블을 사용하고 있다면 bbs 앱과 사용자정보가 호환되지 않아 문제가 될 수도 있습니다. 또 새로운 프로젝트에 개발할 때 이번에 정의한 사용자 모델을 가져다 쓸 수도 있겠죠**. ~~이렇게 세심하게 앱을 분리하는 모습이 기특합니다.~~

먼저 사용자 앱을 만들겠습니다.
```bash
(test-venv-36) $ ./manage.py startapp user
```

생성시키면 `bbs` 앱과 같이 `user` 디렉토리와 파일들이 생성될 것입니다. 먼저 `AbstractUser` 모델을 참고해서 새로운 `User` 모델을 정의합니다.

```python
# bbs/models.py

from django.contrib.auth.models import (
    AbstractBaseUser, PermissionsMixin, UserManager
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
    REQUIRED_FIELDS = ['name']                   # 필수입력값
```
모델을 추가한 뒤 `user` 앱을 등록합니다. 한가지 더 세팅해줘야 할 것이 있는데 사용자 모델은 여러 앱들에서 참조하고 있는데 장고에서는 **커스터마이징 될 것을 대비해서 `AUTH_USER_MODEL` 이라는 설정으로 현재 사용자 모델이 무엇인지 설정**할 수 있도록 했습니다. 물론 우리가 만들 앱에서도 이 설정을 참조해서 사용자 모델을 사용할 것입니다.
```python
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

AUTH_USER_MODEL = 'user.User'        # '앱label.모델명'
```

이제 `makemigrations` 커맨드로 마이그레이션 파일을 생성합니다.
```bash
(test-venv-36) $ ./manage.py makemigrations
Traceback (most recent call last):
  File "./manage.py", line 15, in <module>
    execute_from_command_line(sys.argv)
  
  # 생략

    del kwargs['editable']
KeyError: 'editable'
```
오류가 발생하는데 `Article` 모델의 **`created_at` 필드의 속성을 강제로 `editable`로 변경해서 생긴 문제**입니다. **`auto_now_add` 대신 디폴트값으로 현재시간을 저장하도록 수정**하면 자동으로 `created_at` 값이 생성될 뿐만 아니라 `editable` 속성도 `True` 로 설정되기 때문에 일석이조입니다. ~~진작에 이렇게 하지~~ 여러모로 장고에 다양한 기능이 있음을 실감합니다.

```python
# bbs/models.py

from django.db import models
from django.utils import timezone


class Article(models.Model):
    title      = models.CharField('제목', max_length=126, null=False)
    content    = models.TextField('내용', null=False)
    author     = models.CharField('작성자', max_length=16, null=False)
    created_at = models.DateTimeField('작성일', default=timezone.now)

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
Traceback (most recent call last):
  File "./manage.py", line 15, in <module>
    execute_from_command_line(sys.argv)

# 생략

    connection.alias,
django.db.migrations.exceptions.InconsistentMigrationHistory: Migration admin.0001_initial is applied before its dependency user.0001_initial on database 'default'.
```
이번에는 `admin` 의 마이그레이션 파일이 `user` 앱의 `0001_initial` 마이그레이션 파일에 의존적이다는 메시지인데...우리가 사용하는 `admin` 사이트에도 모델이 있는데 이것이 `AUTH_USER_MODEL에` 의존적이어서 문제가 생가는 겁니다. `bbs` 앱을 `migrate` 할 때는 빈 데이터베이스에 `admin` 앱과 `bbs` 앱이 같이 마이그레이션 되어서 문제가 없었는데, 이미 **`admin` 앱이 마이그레이션 된 상태에서 커스텀 유저 모델을 마이그레이션 하려니 문제**가 되는 상황입니다. 특별한 경우이기 때문에 이해가 되지 않아도 상관없습니다. 넘어가세요. 아래 해결 방법만 잘 따라 하시면 됩니다.

해결방법은 **`admin` 앱을 비활성화** 시키면 됩니다. **커스텀 사용자 모델을 마이그레이션할 동안만 비활성화** 합니다.
```python
# minitutorial/settings.py

# 생략

INSTALLED_APPS = [
    'bbs',
    'user',
    # 'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

# 생략
```
admin 앱이 장고에서 사용되지 않도록 변경했으니 **urls.py에 어드민 핸들러로 라우팅하는 부분도 잠깐만 비활성화** 시킵니다.
```python
# minitutorial/urls.py

from django.contrib import admin
from django.urls import path

from bbs.views import hello, ArticleListView, ArticleDetailView, ArticleCreateUpdateView

urlpatterns = [
    path('hello/<to>', hello), 

    path('article/', ArticleListView.as_view()),
    path('article/create/', ArticleCreateUpdateView.as_view()),
    path('article/<article_id>/', ArticleDetailView.as_view()),
    path('article/<article_id>/update/', ArticleCreateUpdateView.as_view()),

    # path('admin/', admin.site.urls),
]
```
이제 다시 `migrate` 커맨드를 실행해보시면 정상적으로 마이그레이션이 실행됩니다.

새로운 사용자 모델이 생성되었으니 `admin` 사이트를 접속할 수 있는 슈퍼유저 계정을 다시 생성해줘야 합니다. 기존 테이블(`auth_user`)는 이제 사용하지 않기 때문에 새로운 테이블(`user_user`)에 다시 만들어 줍니다.
```bash
(test-venv-36) $ ./manage.py migrate
Operations to perform:
  Apply all migrations: user
Running migrations:
  Applying user.0001_initial... OK
```
정상적으로 돌아왔다면 **비활성화 했던 admin 설정을 되돌려**주면 마이그레이션 성공입니다.
```bash
(test-venv-36) $ ./manage.py createsuperuser
Email address: swarf00@gmail.com
Password: 
Password (again): 
This password is too short. It must contain at least 8 characters.
This password is too common.
This password is entirely numeric.
Bypass password validation and create user anyway? [y/N]: y
Traceback (most recent call last):
  File "./manage.py", line 15, in <module>
    execute_from_command_line(sys.argv)

# 생략

    self.UserModel._default_manager.db_manager(database).create_superuser(**user_data)
TypeError: create_superuser() missing 1 required positional argument: 'username'
```
새로운 사용자모델에서 슈퍼유저를 생성하는 메소드에 `username` 이라는 필드가 필수로 설정되어 있다고 하는군요. auth 프레임워크에서 사용하던 매니저 코드를 살펴보니 살짝만 수정해주면 될 것 같습니다. 모든 사용자 생성 메소드에 `username` 필드가 필수로 정의되어 있는데 `username` 파라미터는 사용하지 않으니 삭제하면 됩니다.
```python
# user/models.py

from django.contrib.auth.base_user import AbstractBaseUser, BaseUserManager
from django.contrib.auth.models import PermissionsMixin
from django.core.mail import send_mail
from django.db import models
from django.utils import timezone
from django.utils.translation import ugettext_lazy as _

class UserManager(BaseUserManager):
    use_in_migrations = True

    def _create_user(self, email, password, **extra_fields):
        if not email:
            raise ValueError('The given email must be set')
        email = self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_user(self, email=None, password=None, **extra_fields):
        extra_fields.setdefault('is_staff', False)
        extra_fields.setdefault('is_superuser', False)
        return self._create_user(email, password, **extra_fields)

    def create_superuser(self, email, password, **extra_fields):
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)

        if extra_fields.get('is_staff') is not True:
            raise ValueError('Superuser must have is_staff=True.')
        if extra_fields.get('is_superuser') is not True:
            raise ValueError('Superuser must have is_superuser=True.')

        return self._create_user(email, password, **extra_fields)

# 생략

```
이제 다시 `createsuperuser` 커맨드를 실행하시면 정상적으로 슈퍼유저가 생성되고, 생성된 이메일로 로그인하면 정상적으로 `admin` 사이트에 접속이 가능합니다. 하지만 원래 있던 `Users` 모델이 사라지고, 새로운 사용자 모델이 보이지 않아서 **admin 사이트에 새로운 사용자모델을 추가**해줍니다.
```python
# user/admin.py

from django.contrib import admin

from .models import User


@admin.register(User)
class UserAdmin(admin.ModelAdmin):
    list_display = ('id', 'email', 'name', 'joined_at', 'last_login_at', 'is_superuser', 'is_active')
    list_display_links = ('id', 'email')
    exclude = ('password',)                           # 사용자 상세 정보에서 비밀번호 필드를 노출하지 않음

    def joined_at(self, obj):
        return obj.date_joined.strftime("%Y-%m-%d")

    def last_login_at(self, obj):
        return obj.last_login.strftime("%Y-%m-%d %H:%M")

    joined_at.admin_order_field = '-date_joined'      # 가장 최근에 가입한 사람부터 리스팅
    joined_at.short_description = '가입일'
    last_login_at.admin_order_field = 'last_login_at'
    last_login_at.short_description = '최근로그인'
```
이렇게 하면 기본 사용자 정보 모델의 정의는 완료됐습니다. 사용자 모델을 만들었으니 이제부터 `가입`, `로그인`, `로그아웃`을 하나씩 만들어 나가면 되겠습니다.

## 2. 회원가입
### 회원가입 뷰 생성
이미 회원가입을 위한 모델은 만들어져 있으니 먼저 뷰를 만들어 봅니다. 이번에는 `TemplateView` 대신 `CreateView` 를 이용할 예정입니다. `CreateView` 은 `TemplateView` 보다 많은 믹스인들이 추가되어 훨씬 다양한 기능들을 제공합니다. 이 기능들을 모두 사용하려면 **규칙에 따라 설정**해야 할 것들이 몇가지 있습니다.
```python
# user/views.py

from django.views.generic import CreateView

from user.models import User


class UserRegistrationView(CreateView):
    model = User                            # 자동생성 폼에서 사용할 모델
    fields = ('email', 'name', 'password')  # 자동생성 폼에서 사용할 필드
    
```
`TemplateView` 를 상속받아 정의하던 때랑 달라진 부분이 크게 **`model`, `fields` 클래스변수가 추가되고, `template_name` 이라는 클래스변수가 사라진 점**입니다. `CreateView` 의 `model` 클래스변수에 참조할 모델클래스를 정의하면 앞으로 이 뷰에서는 데이터 관련된 부분은 이 모델을 이용할 것이라는 의미입니다.(폼을 만들 때 `model` 변수에 정의한 클래스를 참조하고, 폼의 필드들은 모델의 필드의 속성을 참조합니다.)

`model` 이 정의되면 내부적으로 Form 객체를 자동 생성하는데 이 때 모델의 모든 필드를 이용해서 폼을 만드는 것이 아니라 `fields` 라는 클래스변수를 참조해서 정의되어 있는 필드만 이용합니다. Form 클래스도 커스터마이징이 가능합니다. `django.forms.forms.ModelFrom` 클래스를 상속받아 정의한 Form 클래스는 `form_class` 라는 클래스변수에 정의해서 `CreateView` 가 새로 만들지 않고 우리가 만든 Form 클래스를 사용하도록 설정할 수 있습니다. `UserRegistrationView` 에서는 기본 생성 폼을 이용할 예정입니다.

`template_name` 이라는 클래스변수를 정의하지 않으면 `CreateView` 변수에서는 자동으로 **해당 앱의 `templates` 디렉토리에서 앱이름의 디렉토리 하위의 `모델명_form.html` 파일을 템플릿으로 사용**합니다. 우리의 예제에서는 `user/template/user/user_model.html` 파일을 검색하게 되는 겁니다.

> 이번 `UserRegistrationView` 에서는 사용하지 않는 `template_suffix` 클래스변수를 정의하면 template 파일명의 `_form` 대신에 다른 문자열로 대치도 가능합니다. 예를들어 `template_suffix` 를 `'_registration'` 으로 변경하면 `user/template/user/user_registration.html` 파일을 찾게 되는 것이죠.

get, post 요청을 처리할 핸들러 메소드도 정의하지 않았지만 이것 또한 `CreateView` 에서 기본적인 것들은 처리해줍니다.

그러면 url `UserRegistrationView` 뷰를 라이팅할 수 있도록 `urlpatterns` 에 추가를 하고 `user_model.html` 이라는 템플릿도 뼈대만 생성해서 정상적으로 뷰가 동작하는 지 확인해봅니다.
```python
# minitutorial/urls.py

from django.contrib import admin
from django.urls import path

from bbs.views import hello, ArticleListView, ArticleDetailView, ArticleCreateUpdateView
from user.views import UserRegistrationView

urlpatterns = [
    path('hello/<to>', hello), 

    path('article/', ArticleListView.as_view()),
    path('article/create/', ArticleCreateUpdateView.as_view()),
    path('article/<article_id>/', ArticleDetailView.as_view()),
    path('article/<article_id>/update/', ArticleCreateUpdateView.as_view()),

    path('user/create/', UserRegistrationView.as_view()),

    path('admin/', admin.site.urls),
]
```
url 패턴들이 조금 많아져서 보기 좋게 빈줄로 앱들을 구분지었습니다. 이것도 맘에 들지 않지만 이 정도는 일단 참습니다. 로그아웃을 구현할 때 저 url 패턴들을 관리하기 편하게 분리하면 되니 일단 내버려둡니다.

```html
<!-- user/template/user/user_model.html -->

{% raw %}
{% extends 'base.html' %}

{% block title %}<title>회원 가입</title>{% endblock %}

{% block css %}
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
{% endblock css %}

{% block content %}

{% endblock content %}
{% endraw %}
```
부트스트랩의 css도 매번 나오는게 거슬리지만 우선은 두고 로그아웃 섹션에서 base.html로 옮기겠습니다. runserver 커맨드로 실행하고 브라우저에서 접속하면 정상적으로 아주 깨끗한 화면이 나옵니다. content 블록에 아무것도 넣지 않았으니 이것이 정상입니다.

### 회원가입 템플릿 생성
이제 템플릿에 form 태그와 input 태그들을 입력하면 될 듯 합니다. 이전 [Template 만들기]({{ site.url }}/build-template.html)에서 이런 방식은 해봤으니 이번은 조금 다르게 장고의 기본 form을 이용해서 template 을 만들어 보겠습니다.
```html
<!-- user/template/user/user_model.html -->

{% raw %}
{% extends 'base.html' %}

{% block title %}<title>회원 가입</title>{% endblock %}

{% block css %}
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
{% endblock css %}

{% block content %}
<div class="panel panel-default registration">
    <div class="panel-heading">
        가입하기
    </div>
    <div class="panel-body">
        <form action="." method="post">
            {% csrf_token %}
            {{ form.as_p }}
            <div class="form-actions">
                <button class="btn btn-primary btn-large" type="submit">가입하기</button>
            </div>
        </form>
    </div>
</div>
{% endblock content %}
{% endraw %}
```
장고의 기본 생성 Form 으로 템플릿을 만들어 봤습니다. 버튼과 부트스트랩 wrapper 컴포넌트를 제외하면 코드는 몇 줄 안되지만 화면이 생성되었습니다. 

![CreateView 기본폼 적용후]({{ site.url }}/snapshots/createview_form_01.png)

안타깝게도 기본 폼에서 미적 감각을 채워주진 않습니다. 아직 삐뚤빼뚤 볼품없는 화면이지만 css를 조금 추가하면 좀 더 나아질 것 같습니다. 먼저 렌더링된 html을 살펴보고 다음에 style을 맞추도록 하겠습니다.

> 렌더링(rendering)은 사전적으로 번역이라는 의미도 가지고 있는데, 이렇게 생각해보시면 이해가 빠르실 겁니다. **form(python) 데이터를 html 데이터로 번역**한다. 즉 장고가 이해하는 데이터를 브라우저가 이해할 수 있는 데이터 형태로 변환하는 작업을 장고템플릿에서 렌더링이라고 합니다. 실제로는 폼이 렌더링 역할을 하지 않고 각 필드들에게 렌더링을 위임하고 각 필드들을 관리합니다. 자세한 내용은 나중에 소개하겠습니다.

```html
<!-- /user/create/ -->

<!-- 생략 -->

<div class="panel panel-default registration">
  <div class="panel-heading">
    가입하기
  </div>
  <div class="panel-body">
    <form action="." method="post">
      <input type="hidden" name="csrfmiddlewaretoken" value="SDFHZlmawXy3KhWbQB6rSwqKV3u3houNZDlHP4zMLcNgp2EaKNH3N9K2iXXyOl1P">
      <p>
        <label for="id_email">Email address:</label> <input type="email" name="email" maxlength="254" required="" id="id_email">
      </p>
      <p>
        <label for="id_name">Name:</label> <input type="text" name="name" maxlength="30" id="id_name">
      </p>
      <p>
        <label for="id_password">Password:</label> <input type="text" name="password" maxlength="128" required="" id="id_password">
      </p>
      <div class="form-actions">
        <button class="btn btn-primary btn-large" type="submit">가입하기</button>
      </div>
    </form>
  </div>
</div>

<!-- 생략 -->
```
각 필드마다 `p` 태그로 둘러싸였고 각 `input` 들은 `label` 태그와 짝을 이루었습니다. 각 태그들은 관련된 필드의 이름으로 `name` 속성과 `id` 속성이 설정되어 있습니다. 좀 더 정확하게 `id` 는 `'id_' + field.name` 으로 되어있습니다. 

좀 보기 좋게 하려면 모든 `label` 의 너비를 동일하게 하고, 모든 `input` 들의 너비를 동일하게 맞추면 좀 더 깔끔해보일 듯 합니다.
```html
<!-- user/template/user/user_model.html -->

{% raw %}
{% extends 'base.html' %}

{% block title %}<title>회원 가입</title>{% endblock %}

{% block css %}
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
<style>
    .registration {
        width: 360px;
        margin: 0 auto;
    }
    p {
        text-align: center;
    }
    label {
        width: 50%;
        text-align: left;
    }
    .form-action {
        text-align: center;
    }
</style>
{% endblock css %}

{% block content %}
<div class="panel panel-default registration">
    <div class="panel-heading">
        가입하기
    </div>
    <div class="panel-body">
        <form action="." method="post">
            {% csrf_token %}
            {{ form.as_p }}
            <div class="form-actions">
                <button class="btn btn-primary btn-large" type="submit">가입하기</button>
            </div>
        </form>
    </div>
</div>
{% endblock content %}
{% endraw %}
```
flex로 간단하게 해결했다가 혹시나 ~~구시대의 폐물~~ ie 구버전을 사용하시는 분들이 계실까봐 비교적 새로운 css는 적용하지 않았습니다. 디자인에 관심이 없는 관계로 크롬에 최적화된 디자인으로 예시를 설명합니다. css의 적용과정은 ~~가르쳐드릴 정도의 실력이 안되므로~~ 빠른 진행을 위해 생략합니다.

![CreateView 기본폼 style 정리 후]({{ site.url }}/snapshots/createview_form_02.png)

하앍~ 깔꼼하당.^^. 이제 실제 이메일과 이름, 비밀번호를 입력하고 가입하기 버튼을 누르려고 했는데, 비밀번호가 적나라하게 보이네요ㅠㅠ. 이 부분은 일단 넘어가고 기능이 정상적으로 동작하는 지 버튼을 눌러 확인해 봅니다.

![CreateView 가입 오류]({{ site.url }}/snapshots/createview_form_03.png)

살짝 깜빡임과 함께 화면이 상단에 메시지가 나타났습니다.저것은 email 주소가 중복된다는 오류메시지인데 제 기억으로는 슈퍼유저계정만 만들었고, `swarfkim@gmail.com` 이라는 계정으로 사용자를 생성한 적이 없는데 뭔가 이상합니다. 실제로 db 파일을 확인해보도록 합니다.

> 장고 기본폼은 모델폼을 상속받아 생성하는데, 모델폼은 각 필드들이 정상적인 값들인지 검증을 하기도 하고 오류가 있을 경우 `messages` 프레임워크를 통해 각 필드에 메시지를 전달합니다. 또한 폼객체를 처리할 때 오류가 발생할 경우에는 오류가 발생한 폼의 필드에 오류메시지를 전달합니다. 

```sql
sqlite> SELECT * FROM user_user;
1|pbkdf2_sha256$120000$Fpn8scH5jxYJ$BXnUQBZLQXw6ZWf4oVqlS9U5UMSCYbaYZqleWSNTWgU=|2018-12-05 16:49:24.068214|1|swarf00@gmail.com||1|1|2018-12-05 16:45:10.608484
2|7h1515myp455w0d||0|swarfkim@gmail.com|swarf.kim|0|1|2018-12-06 09:30:37.181559
```
확인해보니 방금 입력한 값이 정상적으로 저장이 되었네요. 오류표시가 잘못된 것 같습니다. 한가지 심각한 문제점이 더 보이는데, 비밀번호가 제가 입력한 그대로(7h1515myp455w0d) 저장이 되어 있습니다. 바로 전에 발견한 문제점인 비밀번호를 화면에 그대로 노출시키는 문제와 입력한 그대로 데이터베이스에 저장하는 두가지의 문제를 종합해 볼 때 장고의 **기본폼은 비밀번호 필드를 일반 CharField와 구분할 수 없다**는 사실을 알 수 있습니다. 어쩔 수 없이 가입폼을 새롭게 정의해야 합니다. ~~OK 계획대로 되고 있어.~~

일단 비밀번호가 그대로 저장되어 있는 민망한 레코드는 삭제하도록 합니다.
```sql
sqlite> DELETE FROM  user_user WHERE  id = 2;
sqlite> SELECT * FROM user_user;
1|pbkdf2_sha256$120000$Fpn8scH5jxYJ$BXnUQBZLQXw6ZWf4oVqlS9U5UMSCYbaYZqleWSNTWgU=|2018-12-05 16:49:24.068214|1|swarf00@gmail.com||1|1|2018-12-05 16:45:10.608484
```

### 회원가입 폼 생성
가입폼을 새롭게 밑바닥부터 만들려면 알아야 할 것들도 많고, 해야 할 것들이 너무나 많습니다. ~~그럴 줄 알고~~ 장고 auth 프레임워크에서 기본적으로 제공하는 `UserCreationForm` 을 가져다 사용합니다. 추가로 우리도 `User` 모델을 직접 임포트해서 model 변수에 정의했는데 좀 더 유연한 방법으로 가져오도록 수정합니다.
```python
# user/views.py

from django.contrib.auth.forms import UserCreationForm
from django.views.generic import CreateView

from user.models import User


class UserRegistrationView(CreateView):
    model = User
    form_class = UserCreationForm
```
삭제된 `fields` 클래스변수는 장고에서 기본으로 생성되는 모델폼을 사용할 경우만 필요합니다. 모델폼 객체가 자동으로 생성될 때 참조하는 모델의 모든 필드를 폼의 필드로서 생성하지 않고 필요한 필드들만 생성시켜야 하는데 이 때 **`fields` 라는 클래스변수가 꼭 폼에서 생성시켜야 하는 모델의 필드명** 입니다. 자동생성되는 폼이다보니 뷰에서 전달해주지 않으면 폼에서 생성시켜야 할 필드들이 무엇인지 알 수 없었던 것 입니다. 다시 브라우저에서 `/user/create/` 주소로 접속합니다.

![CreateView UserCreationForm 적용 후]({{ site.url }}/snapshots/createview_form_04.png)

지저분합니다ㅠㅠ. 각 필드마다 밑에 무언가를 설명하는 여러가지 문구들이 있는 듯 한데 ~~영어는 까막눈이라~~ 보기가 싫습니다. 이유는 **`UserCreationForm` 내부에서 폼 생성의 기준이 되는 모델을 `auth` 의 모델로 하드코드되어 있기 때문**입니다. `settings.py` 에 설정한 `AUTH_USER_MODEL` 로 되어 있다면 이런 일이 없겠지만 이 기회에 `UserCreationForm` 을 상속받아 새로운 폼을 만들어 볼 기회를 ~~삽질(+1)을 획득하셨습니다.~~ 가져볼 수 있습니다.

우선 폼클래스는 보통 `forms.py` 파일에 구현합니다. 새로운 `forms.py` 파일을 생성 후 `UserCreationForm` 클래스를 상속받아 새로운 폼을 정의합니다.
```python
# user/forms.py

from django.contrib.auth import get_user_model
from django.contrib.auth.forms import UserCreationForm


class UserRegistrationForm(UserCreationForm):

    class Meta:
        model = get_user_model()
        fields = ('email', 'name')
```

model 변수를 `get_user_model` 이라는 함수를 호출해서 정의했는데, **settings 파일에서 `AUTH_USER_MODEL` 이 가리키는 모델을 자동으로 찾아주는 유용한 함수**입니다. `Meta` 클래스의 `fields` 라는 변수는 정의된 모델에서 폼에 보여줄 필드들을 정의하는 변수입니다. `password` 는 자동으로 `UserCreationForm` 에서 생성하기 때문에 `email` 과 `name` 필드만 추가하면 됩니다.

이제 뷰에서도 새로 생성된 폼을 사용하도록 변경합니다. 뷰에서도 마찬가지로 `get_user_model` 함수를 이용해서 model 변수를 지정하도록 합니다.
```python
# user/views.py

from django.contrib.auth import get_user_model
from django.views.generic import CreateView

from user.forms import UserRegistrationForm


class UserRegistrationView(CreateView):
    model = get_user_model()
    form_class = UserRegistrationForm
```

이제 다시 접속해서 어떻게 달라졌는 지 확인해 봅니다.

![CreateView 폼 참조필드 수정]({{ site.url }}/snapshots/createview_form_05.png)

여전히 비밀번호에는 지저분하게 텍스트들이 보이는데 `UserCreateForm` 의 필드들에 정의되어 있는 help_text 속성이 출력되기 때문입니다. `UserCreationFrom` 이 상속받는 **`ModelForm` 은 `as_p`(또는 `as_table`, `as_ul`) 함수 호출 시 폼객체 자신 또는 폼객체가 참조하는 모델의 필드에 정의된 `help_text` 속성을 설명문구로서 친철하게 출력**해줍니다. 이것들을 보이지 않게 하기 위해 템플릿을 수정하도록 하겠습니다. 폼에서 `as_p` 함수를 호출하지 않고 각 필드들을 직접 렌더링 하도록 하는 것이 좋겠습니다. 왜냐하면 어떻게 보여야 하느냐는 템플릿에서 담당하는 역할이기 때문입니다. 또한 이렇게 하면 화면을 좀 더 세밀하게 디자인할 수도 있게 됩니다.
```html
<!-- user/template/user/user_model.html -->

{% raw %}
{% extends 'base.html' %}

{% block title %}<title>회원 가입</title>{% endblock %}

{% block css %}
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
<style>
    .registration {
        width: 360px;
        margin: 0 auto;
    }
    .registration .form-actions > button {
        width: 100%;
    }
</style>
{% endblock css %}

{% block content %}
<div class="panel panel-default registration">
    <div class="panel-heading">
        가입하기
    </div>
    <div class="panel-body">
        <form action="." method="post">
            {% csrf_token %}
            {% for field in form %}
                <div class="form-group">
                    <label for="id_{{ field.html_name}}">{{ field.label }}</label>
                    {{ field }}
                </div>
            {% endfor %}
            <div class="form-actions">
                <button class="btn btn-primary btn-large" type="submit">가입하기</button>
            </div>
        </form>
    </div>
</div>
{% endblock content %}
{% endraw %}
```
**form 객체를 iterate 시키면 각 필드들이 출력**이 됩니다. field 들을 출력하면 각 필드에 맞는 태그로 렌더링 됩니다. 필드의 레이블도 `field.label_tag` 값으로 렌더링 할 수 있으나 이렇게 하면 필드 레이블 뒤에 콜론(:)이 자동으로 붙게되서 어쩔 수 없이 원초적으로 태그를 직접 하드코딩 해줬습니다. 각 필드들의 name 속성은 `field.html_name` 으로 접근이 가능하고 레이블은 `field.label` 으로 접근할 수 있습니다.

> field는 BoundField의 인스턴스입니다. BoundField의 각 속성들을 알면 템플릿에서 좀 더 편리하게 렌더링하실 수 있습니다.
> 1. **field.id_for_label** - field의 tag에서 사용될 id 값으로 보통 'id_ + field.name'
> 2. field.initial - 모델에서의 default 속성의 값
> 3. field.is_hidden - hidden 속성이 있다면 True 그렇지 않으면 False
> 4. **field.errors** - field의 유효성 검증할 때 발견된 오류들
> 5. field.html_name - 렌더링될 tag의 name 속성의 값. 즉, 'form.prefix + field.name'로 폼클래스에 prefix 변수가 선언되어 있지 않으면 field.name 과 동일
> 6. field.help_text - 도움말의 역할을 하는 텍스트로 form 필드에 해당 속성이 없으면 model 필드에서 참조
> 7. **field.label** - 모델의 verbose_name과 동일한 데이터로 해당 필드를 사람이 이해하기 쉬게 부르는 호칭
> 8. field.label_tag - field.label 을 렌더링한 태그
> 8. field.name - field 의 이름. 폼에 선언된 field의 변수명과 동일
> 9. **field.value** - field에 저장된 값

![CreateView 템플릿 커스터마이징]({{ site.url }}/snapshots/createview_form_06.png)

이렇게 해도 거의 완벽한데 label 값이 영어라서 참 많이 애석합니다. `email` 과 `name` 부분은 모델에서 수정하면 되는데, `password1`과 `password2`는 `UserCreationForm` 에 정의되어 있는 부분이어서 override 해줘야 합니다. 그런데 유심히 모델과 폼의 label에 해당하는 값들을 보시면 `_('msgid')` 형식으로 선언되어 있을 겁니다. 예를 들어 모델에서 `email` 필드를 보시면 `_('email address')`로 선언되어 있습니다. 이 `_`라는 함수는 `ugettext_lazy` 함수의 별칭(별명) 입니다. 이 함수는 **언어설정에 따라 출력되는 문자열을 변환해**주는 함수입니다. 설정파일의 `LANGUAGE_CODE` 만 변경시켜주면 장고에서 미리 번역해둔 문자열들로 치환되어 출력됩니다.

> 다국어 설정은 아무런 문자나 자동으로 변환(번역)이 되는 것이 아니라 번역파일에 미리 정의 해놓은 문자들만 변환이 됩니다. msgid에 대응하는 문자들을 언어별로 작성해야 합니다. ./manage.py 유틸리티의 `makemessages` 커맨드는 프로젝트내의 `ugettext_lazy` 함수의 인자들을 검색 후 언어별로 번역파일을 생성합니다. 번역파일에 해당 msgid 에 대응하는 msgstr 을 정의해주면 번역파일이 생성이 되고, 번역단어가 많고 여러 언어로 설정할 경우 검색하는 시간이 오래 걸려 전체적으로 서비스의 속도가 굉장히 저하됩니다. 그렇기 때문에 장고에서 읽기 편하게 미리 컴파일을 해둬야 하는데 이것이 ./manage.py 유틸리티의 `compilemessages` 커맨드입니다. 다국어 설정은 ~~인싸가 되기 위해~~ 중요하지만 여기서 길게 설명하지 않으니 더 자세한 설명은 [공식문서](https://docs.djangoproject.com/en/2.1/topics/i18n/translation/)를 참고하시기 바랍니다.
```python
# minitutorial/settings.py

# 생략

LANGUAGE_CODE = 'ko-KR'

TIME_ZONE = 'Asia/Seoul'

# 생략
```

### 한글화(국제화 a.k.a. i18n) 설정
기본언어 설정 후 다시 접속해보면 한글로 정상적으로 출력이 됩니다.

![CreateView 템플릿 한글화 후]({{ site.url }}/snapshots/createview_form_07.png)

이제 화면은 우리가 원하는대로 출력이 되는 것 같습니다. 이제 실제 가입하기가 정상적으로 작동하는 지 다시 확인해봅니다. 간단하게 인증정보를 입력하여 회원가입을 해봅니다.
어떤 케이스에는 화면만 깜박이고 비밀번호가 지워진 상태로 다시 화면이 나타나고, 또 다른 어떤 케이스는 오류가 발생합니다.

![CreateView 가입실패]({{ site.url }}/snapshots/createview_form_08.png)

화면만 깜박이는 케이스는 db파일을 확인해보니 아무것도 저장이 되지 않았고, `ImproperlyConfirue` 오류가 발생한 경우에는 데이터가 정상적으로 저장된 것으로 확인됩니다.

```bash
sqlite> SELECT * FROM user_user;
1|pbkdf2_sha256$120000$Fpn8scH5jxYJ$BXnUQBZLQXw6ZWf4oVqlS9U5UMSCYbaYZqleWSNTWgU=|2018-12-05 16:49:24.068214|1|swarf00@gmail.com||1|1|2018-12-05 16:45:10.608484
3|pbkdf2_sha256$120000$ZicY7sWoTpG5$u2TOhjtfiNZrP2XgN9iDgH3D45+O/0oBBZKsNQLfNvU=||0|swarkim@gmail.com|swarf00|0|1|2018-12-06 19:56:33.057128
```

우선 깜박이는 경우부터 원인을 짚어보겠습니다. 데이터베이스에 저장이 되지 않았다는 것은 **폼에서 입력된 값의 유효성 검사를 통과하지 못했다는 의미**인데, 어떤 문제가 있었는지 화면에 출력하지 않아서 알 수가 없는 상황입니다. 각 필드마다 `errors` 속성이 있는데 이 속성을 출력하면 해당 필드에서 발생한 문제들이 줄줄이 출력이 됩니다. 템플릿에 `errors` 를 출력하도록 수정하겠습니다.
```html
<!-- user/template/user/user_model.html -->

<!-- 생략 -->
{% raw %}

          <form action="." method="post">
            {% csrf_token %}
            {% for field in form %}
                {{ field.errors }}         <! -- 필드 상단에 오류 메시지 출력 -->
                <div class="form-group">
                    <label for="{{ field.id_for_label }}">{{ field.label }}</label>
                    {{ field }}
                </div>
            {% endfor %}
            <div class="form-actions">
                <button class="btn btn-primary btn-large" type="submit">가입하기</button>
            </div>
        </form>
{% endraw %}

<!-- 생략 -->

```

다시 가입을 해봤더니 오류메시지가 잘 나타나는 군요. 오류 메시지가 별로 눈에 띄지 않는데 css변경은 모든 오류를 수정한 후에 하기로 하고 우선 가입하기를 성공시키도록 하겠습니다.

`UserCreationForm` 에서 password2의 필드에 비밀번호 검증하는 루틴이 추가되어 있는데 내부를 살펴보면 결국 설정파일의 `AUTH_PASSWORD_VALIDATORS` 리스트에 정의된 validator들을 모두 통과시키도록 되어 있습니다. 기본적으로 4개의 validator 들이 정의되어 있는데 4가지 모두 통과시켜 **오류가 발생하면 모두 필드객체의 `errors` 변수에 오류내용이 추가되고 폼은 데이터를 저장하지 않습니다**. validator가 귀찮다면 `AUTH_PASSWORD_VALIDATORS` 에서 해당 항목을 삭제하시고, 더 많은 검증이 필요하다면 더 추가하셔도 상관없습니다. 다른 필드들은 정상적으로 입력하고 비밀번호를 '1'이라고만 입력하고 가입하기를 시도해보면 모든 검증 패턴의 오류내용을 보실 수 있습니다.

{% raw %}
> 4가지 중 오류 메시지가 발생된 3가지는 쉽게 이해하실 수 있으리라 생각됩니다. 필터에 걸리지 않은 `UserAttributeSimiarityValidator` 의 기능에 대해 궁금하지 않으신 분은 이 부분을 건너 뛰셔도 됩니다.
>
> `UserAttributeSimiarityValidator` 는 사용자 모델의 속성(즉, 필드)와 비교해서 유사한 경우 오류를 발생시키는 유틸리티입니다. 기본으로 비교하는 필드는 정해져 있지만 수정이 가능합니다. `username`, `first_name`, `last_name`, `email` 이 네 가지를 비교하는데 `email`을 제외하고 나머지 3개의 필드들은 새로운 사용자 모델에서 삭제했었죠. 그러니 `email`과 `name` 두 가지의 필드를 `user_attributes` 이라는 이름의 옵션으로 전달해주면 됩니다. 또한 유사도를 나타내는 `max_similarity` 옵션도 있지만 기본값으로 0.7이라는 값이 설정되어 있는데 굳이 변경할 필요가 없어 보입니다. validator에 옵션을 전달하는 방법은 설정파일의 `AUTH_PASSWORD_VALIDATORS` 변수에 옵션을 전달해주면 됩니다.
> ```python
> {
>
>     'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
>
>     'OPTION': {'user_attributes': ('email', 'name')},
> },
> ```
{% endraw %}

그럼 두번째로 `ImproperlyConfirue` 오류가 발생하는 원인을 살펴봐야 하는데 오류 메시지를 보니 **redirect 될 URL이 정의되어 있지 않다**고 합니다. 데이터가 정상적으로 저장되는 것으로 보아 회원가입까지는 성공이 되었는데 어디로 이동해야 할 지 몰라서 발생하는 오류로 보여집니다. `CreateView` 에서 내부적으로 폼을 처리한 이후 `get_success_url()` 함수를 호출해서 이동할 페이지의 주소를 결정하는데 뷰의 클래스변수인 `success_url` 을 참조합니다. 그렇다면 **UserRegistrationView에 success_url 클래스변수를 정의**하면 해결될 것 입니다. ~~참 간단하쥬?~~ 장고는 왜 이렇게 복잡하게 설정할 것이 많은 지 모르겠습니다.

어쨌든 회원가입이 성공한 뒤를 생각해보지 않았지만 일단 게시글 목록보기 화면으로 이동하도록 하고 `success_url` 에 게시물 목록보기의 주소 `'/article/'` 을 설정합니다.
```python
# user/views.py

from django.contrib.auth import get_user_model
from django.views.generic import CreateView

from user.forms import UserRegistrationForm


class UserRegistrationView(CreateView):
    model = get_user_model()
    form_class = UserRegistrationForm
    success_url = '/article/'
```
다시 한번 정상적인 입력값으로 회원가입을 시도해봅니다. 이번에는 정상적으로 게시글 목록 화면으로 이동했습니다.

> 보통 다른 블로그나 책을 보면 `success_url` 은 `reverse` 또는 `reverse_lazy` 함수를 이용해서 세팅하곤 합니다. 이번 예제는 꼭 그럴 필요없다는 것을 보여드리고 싶어서 url을 하드코딩하는 예제를 보여드립니다. `reverse` 또는 `reverse_lazy` 함수는 **여러 개의 앱과 다양한 3rd 파티 앱들을 사용할 때 유용**합니다. 모든 앱들의 url들을 다 외울 수 없으니 앱이름과 라우팅 이름만 가지고 편리하게 사용할 수 있는 기능입니다. 라우팅 이름은 보통 핸들러 함수(또는 뷰클래스) 이름과 유사하게 정의하기 때문에 쉽게 기억할 수 있습니다. 반대로 앱이 복잡하지 않고 url이 명확하다면 굳이 라우팅 이름을 정의할 필요도 없고 굳이 함수를 거칠 필요 없습니다. 장고의 기능은 너무나 방대해서 한가지의 기능을 구현하기 위해 다양한 방법들을 사용할 수 있습니다. `why`를 많이 알면 알수록 장고를 좀 더 `효율적`이고 `효과적`으로 사용할 수 있을 것 입니다.

마지막으로 아까 약속했던 각 필드들의 errors출력에 style을 입히는 작업을 하겠습니다. 뾰롱뾰롱~뾰로롱~뿅~
```html
<!-- user/template/user/user_model.html -->

{% raw %}
{% extends 'base.html' %}
{% load i18n %}

{% block title %}<title>회원 가입</title>{% endblock %}

{% block css %}
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
<style>
    .registration {
        width: 360px;
        margin: 0 auto;
    }
    p {
        text-align: center;
    }
    label {
        width: 50%;
        text-align: left;
    }
    .control-label {
        width: 100%;
    }
    .registration .form-actions > button {
        width: 100%;
    }
</style>
{% endblock css %}

{% block content %}
<div class="panel panel-default registration">
    <div class="panel-heading">
        가입하기
    </div>
    <div class="panel-body">
        <form action="." method="post">
            {% csrf_token %}
            {% for field in form %}
                <div class="form-group {% if field.errors|length > 0 %}has-error{%endif %}">
                    <label for="{{ field.id_for_label }}">{{ field.label }}</label>
                    <input name="{{ field.html_name }}" id="{{ field.id_for_lable }}" class="form-control" value="{{ field.value }}">
                    {% for error in field.errors %}
                        <label class="control-label" for="{{ field.id_for_label }}">{{ error }}</label>
                    {% endfor %}
                </div>
            {% endfor %}
            <div class="form-actions">
                <button class="btn btn-primary btn-large" type="submit">가입하기</button>
            </div>
        </form>
    </div>
</div>
{% endblock content %}
{% endraw %}
```

![CreateView 에러 CSS 적용 후]({{ site.url }}/snapshots/createview_form_09.png)

참 쉽죠? 이제 에러가 발생하면 에러가 발생한 필드 아래에 메시지가 빨간 표시되고, 해당 필드도 빨간색으로 표시되도록 변경했습니다. label이 필드의 상단으로 이동되도록 변경했습니다.

300라인 정도면 가입하기 내용을 다 설명할 수 있을 것이라 생각했는데 예상보다 2배가 분량이 되었습니다. 더 자세히 설명하다간 가입하기 본질을 놓치고 장고 프레임워크만 ~~코딱지 파듯이~~ 파다가 코피 퐝 쏟을 것 같습니다. `Responsive Web Design`도 적용하려 했으나 중요해 보이지 않는 것들은 분량이 적을 것 같은 로그아웃 편으로 다 몰아버릴 예정입니다. 다음 편인 로그인을 구현할 때도 좀 새로운 기능들과 속성들을 사용할 건데 얼마나 오래 걸릴 지 모르겠으나 잠시만 안녕 ~

> 잔소리가 적은 개발자가 좋은 개발자다.
>
> swarf00, 그래 나 설명충이다...