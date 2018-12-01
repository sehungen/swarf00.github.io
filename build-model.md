---
layout: article
title: 모델 만들기
aside:
  toc: true
sidebar:
  nav: docs-ko
metadata:
  'og_title': 장고(Django) 모델 정의하기
  'og_type': article
  'og_locale': 'ko_KR'
  'og_description': 게시판이라는 앱의 기능을 미리 정의하고 모델 설계하는 방법을 설명합니다. 설계를 바탕으로 장고(Django) 웹프레임워크의 ORM을 통해 모델을 정의하고 활용하는 방법을 설명합니다.
  'og_site_name': 장고(Django) 핥짝 맛보기
---

## 1. 모델 설계
이제부터 진짜 게시판 앱을 만들 것입니다.

게시판 데이터를 저장하기 위해서 가장 먼저 모델부터 설계해야 합니다. 설계에 앞서 사용자들이 게시판을 어떻게 사용할 지 가정하여 나열합니다. 

> 1. 게시판(게시글 목록)에는 게시글들의 목록이 나열됩니다.
> 2. 게시글들은 제목과 작성자 표시됩니다. 
> 3. 게시글을 (클릭해서) 들어가면 게시글 상세화면으로 이동하고 제목, 내용, 작성일이 출력합니다.
> 4. 게시글 상세화면에서 수정하기 버튼을 누르면 수정하는 화면으로 이동합니다.
> 5. 게시글 수정화면에서 저장하기 버튼을 누르면 수정된 내용이 저장되고 게시판으로 이동합니다.
> 6. 게시글 수정화면에서 삭제하기 버튼을 누르면 게시글이 삭제되고 게시판으로 이동합니다.
> 7. 게시판에서 새글쓰기 버튼을 누르면 새로운 게시글을 입력할 수 있는 화면이 출력됩니다.
> 8. 게시글을 작성하고 저장하기 버튼을 누르면 수정된 내용이 저장되고 게시판으로 이동합니다.

게시글(Article)이라는 모델을 만들고 `제목(title)`, `내용(content)`, `작성자(author)`, `작성일(created_at)`  속성을 정의해주면 될 것 같습니다. 그러면 이 게시글들을 모두 불러와서 게시글 목록에서 보여주고, 게시글 상세화면과 수정화면에서는 해당하는 하나의 게시글만 불러와서 보여주면 될 것 같습니다.

## 2. 모델 생성
### 모델 정의
모델은 `Article`이라는 이름으로 models.py에 정의합니다.
```python
# bbs.models.py

from django.db import models
  
class Article(models.Model):
    title      = models.CharField('제목', max_length=126, null=False)
    content    = models.TextField('내용', null=False)
    auther     = models.CharField('작성자', max_length=16, null=False)
    created_at = models.DateTimeField('작성일', auto_now_add=True)
```
`models` 모듈은 장고의 모델과 관련된 모든 기능이 구현된 모듈입니다.

우리가 구현할 모델 클래스는 `models.Model` 클래스를 상속받습니다. 모델이 데이터베이스와 연결하는 모든 기능들이 이미 구현되어 있습니다.

> 모델의 속성은 장고ORM에서 제공하는 기본 필드로 구현합니다. 예제에서 다루는 필드 이외에 더 많은 종류의 필드와 옵션들이 있습니다.
>
> * CharField - sql에서의 `varchar` 자료형으로 변환됩니다. 글자수 제한있는 문자열 데이터를 저장합니다.
> * TextField - sql에서의 `text` 자료형으로 변환됩니다. 길이수 제한없는 문자열 데이터를 저장합니다.
> * DateTimeField - sql에서의 `datetime` 자료형으로 변환됩니다. 날짜와 시간이 utc 시간으로 저장됩니다.

### 데이터베이스 설정
정의된 모델이 실제 데이터베이스에 저장되도록 하기 위한 설정을 하기 위해 프로젝트의 settings.py 파일을 수정합니다. 우리는 `sqlite3`를 이용해 데이터를 저장할 것이므로 설정을 그대로 유지합니다. 
```python
# settings.py

# 생략
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',       # 데이터베이스는 sqlite3 사용
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'), # 루트 디렉토리에 db.sqlite3 파일로 데이터 저장
    }
}
# 생략
```
데이터베이스가 sqlite3로 설정된 상태이기 때문에 `Article` 모델을 sqlite3의 테이블로 생성시켜주기만 하면 됩니다.

manage.py 에서 제공하는 `makemigrations` 커맨드를 이용하면 migrations 디렉토리의 migration 파일들을 비교해서 어떻게 변경됐는 지 확인 후 새로운 **migration 파일을 생성하여 변경된 내용을 기록**합니다.

`Article` 모델은 새로 생성한 모델이기 때문에 비교없이 새로운 migration 파일을 생성합니다.
```bash
(test-venv-36) $ ./manage.py makemigrations
No changes detected
```
...... 어떻게 된 일인지 변경된 게 없다고 출력이 되며 bbs/migrations 디렉토리에 생성된 파일이 없습니다.

### 앱 등록하기
이유는 app을 생성하는 `startapp` 커맨드는 앱디렉토리와 관련 파일들을 생성할 뿐 앱관련 설정은 프로젝트에 추가하지 않기 때문입니다. settings.py 파일에 **bbs 앱을 등록**해줍니다.
```python
# settings.py

# 생략
INSTALLED_APPS = [
    'bbs',                         # 등록할 앱이름
    'django.contrib.admin',        # 장고 어드민 앱
    'django.contrib.auth',         # 장고 인증 앱
    'django.contrib.contenttypes', # 다양한 종류의 모델데이터를 관리할 수 있게 도와주는 앱.
    'django.contrib.sessions',     # 클라이언트 정보를 세션에서 관리하도록 하는 프레임워크
    'django.contrib.messages',     # 컨트롤러에서 발생한 정보를 뷰에서 쉽게 접근하도록 연결하는 프레임워크
    'django.contrib.staticfiles',  # html, css, js 파일등의 정적파일 들을 관리해주는 프레임워크
]
# 생략
LANGUAGE_CODE = 'ko-kr'  # 장고 기본언어 변경

TIME_ZONE = 'Asia/Seoul' # 시간대를 서울로 변경

USE_TZ = False           # 기본 시간대(UTC)를 사용하지 않겠다고 변경
# 생략
```
bbs를 제외한 다른 앱들은 아직까지 필요하지 않지만 장고에서 기본적으로 필요한 프레임워크들입니다. 모두

이제 다시 makemigrations 커맨드로 `migration` 파일을 생성합니다.
```python
(test-venv-36) $ ./manage.py makemigrations
Migrations for 'bbs':
  bbs/migrations/0001_initial.py
    - Create model Article
```
`bbs/migraions/0001_initial.py` 파일이 생성되었습니다. migration 파일의 간단한 내용이 표시되어 있습니다. 상세한 내용은 migration 파일에 기록되어 있습니다.
```python
# bbs/migrations/0001_initial.py

from django.db import migrations, models


class Migration(migrations.Migration):

    initial = True

    dependencies = [
    ]

    operations = [
        migrations.CreateModel(
            name='Article',
            fields=[
                ('id', models.AutoField(auto_created=True, primary_key=True, serialize=False, verbose_name='ID')),
                ('title', models.CharField(max_length=126, verbose_name='제목')),
                ('content', models.TextField(verbose_name='내용')),
                ('auther', models.CharField(max_length=16, verbose_name='작성자')),
                ('created_at', models.DateTimeField(auto_now_add=True, verbose_name='작성일')),
            ],
        ),
    ]
```
`operation` 리스트를 보면 `migrations.CreateModel` 이 추가되어 있는데 여기에 생성될 테이블의 내용이 기록되어 있습니다. 

`id`라는 필드가 자동으로 들어가 있습니다. `primary key`를 설정하지 않으면 자동으로 `id`라는 필드를 생성하고 `pk`로 설정을 합니다.
field를 자세히 보니 author가 아닌 auther로 입력되었습니다. 직접 타이핑을 하지 않고 복붙(copy & paste)를 하신 ~~게으름뱅이~~분들은 동일한 오타가 있을 겁니다. **migration 파일은 가급적 수정 및 삭제를 해서는 안됩니다**. migration 파일을 수동으로 변경할 경우 이후의 스키마 동기화 작업에 문제가 생길 수 있습니다.

### 모델 수정
모델 파일에서 오타난 부분을 수정합니다.
```python
# bbs.models.py

from django.db import models

class Article(models.Model):
    title      = models.CharField('제목', max_length=126, null=False)
    content    = models.TextField('내용', null=False)
    author     = models.CharField('작성자', max_length=16, null=False) # auther => author 로 수정
    created_at = models.DateTimeField('작성일', auto_now_add=True)
```

`makemigrations` 커맨드를 다시 실행합니다. 이 때 이름을 변경할 것인지를 묻는데 `y`(yes)를 입력합니다.
```bash
(test-venv-36) $ ./manage.py makemigrations
Did you rename article.auther to article.author (a CharField)? [y/N] y
Migrations for 'bbs':
  bbs/migrations/0002_auto_20181121_1545.py
    - Rename field auther on article to author
```

`bbs/migrations/0002_auto_20181121_1545.py` 파일이 생성되고 수정된 내용의 요약이 출력됩니다. 새로 생성된 migration 파일을 다시 확인합니다.

> 파일명에 생성 날짜와 시간(0002_auto_yyyymmdd_HHMM.py)이 기록되기 때문에 완전히 동일한 파일명이 아닙니다.

```python
# bbs/migrations/0002_auto_20181121_1545.py

from django.db import migrations


class Migration(migrations.Migration):

    dependencies = [
        ('bbs', '0001_initial'),
    ]

    operations = [
        migrations.RenameField(
            model_name='article',
            old_name='auther',
            new_name='author',
        ),
    ]
```
~~아직 `migrate`를 하지 않은 상태이기 때문에 migration에 익숙하신 분들은 삭제하거나 모델파일과 마이그레이션 파일의 오타부분을 수정하셔도 됩니다.~~

manage.py의 `migrate` 커맨드를 이용하면 모든 migration 파일을 추적 후 아직 적용하지 않은 migration을 적용시켜 줍니다. 변경된 migration 파일이 없다면 아무런 작업도 하지 않을 것입니다.
```bash
(test-venv-36) $ ./manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, bbs, contenttypes
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
  Applying bbs.0001_initial... OK
  Applying bbs.0002_auto_20181121_1545... OK
  Applying sessions.0001_initial... OK
```
bbs 뿐만 아니라 등록된 모든 앱들의 모델들도 migration 됐습니다.

## 3. 장고 쉘에서 테스트
manage.py의 shell 커맨드를 실행하면 파이썬 쉘이 실행되고, 현재 장고가 실행할 때 가져오는 환경변수들을 동일하게 가져옵니다. 이 쉘을 통해 간단하게 모델이 정상적으로 동작하는 지 확인할 수 있습니다.
```bash
(test-venv-36) $ ./manage.py shell

>>>
```
> ipython을 설치하시면 좀더 강력한 파이썬 쉘을 경험할 수 있습니다. 문법에 맞게 코드에 색깔을 입혀주기도 하고, tab 키를 누르면 자동완성도 제공합니다. 설치는 pip로 간단하게 할 수 있습니다.
> 
> (test-venv-36) $ pip install ipython

### 데이터 저장
파이썬 쉘 프롬프트가 출력되면 Article 모델을 불러와서 CRUD 명령을 실행해볼 수 있습니다.
```python
# 데이터 저장
>>> from bbs.models import Article
>>> article = Article.objects.create(title='How to create a article', content='1. import Article class\n2. invoke \'create\' method of Article\'s manager.', author='swarf00', created_at='2018-11-22')
>>> print(article)
Article object (1)
>>> print('{} title: {}, content: {}, author: {} created_at: {}'.format(article.id, article.title, article.content, article.author, article.created_at))
title: How to create a article, content: 1. import Article class
2. invoke 'create' method of Article's manager., author: swarf00 created_at: 2018-11-22 01:15:21.135315
```
Article 모델을 관리하는 objects 매니저는 Article 클래스가 상속받은 models.Model 클래스에 기본 내장되어 있습니다 기본 매니저를 통해 CRUD를 실행할 수 있는데 데이터 생성은 create 메소드를 이용합니다.

create 함수는 positional argument도 지원하지만 인자가 여러개이거나 한눈에 보기 어려운 경우 keyword arguemnt로 전달하는 것이 보기 좋습니다.

### 데이터 표시 형식 변경
article은 생성된 데이터 객체입니다. print 함수로 출력해보면 `Article object (1)`이라고 출력됩니다. 
`string formatter`를 이용해 저장된 데이터를 확인해보니 created_at값이 입력한 값과 다르게 출력이 되었습니다. `created_at` 필드에 `auto_now_add`를 `True`로 설정하면 `create`메소드가 호출될 때 항상 현재시간(01시 15분 실화냐?)이 기록됩니다.
앞으로 자주 디버깅해야 할 오브젝트인데 매번 `string formatter`를 이용하는게 불편합니다. 그래서 모델에 `__str__`메소들르 오버라이드 해줍니다.
```python
# bbs/models.py

from django.db import models

class Article(models.Model):
    title      = models.CharField('타이틀', max_length=126, null=False)
    content    = models.TextField('내용', null=False)
    author     = models.CharField('작성자', max_length=16, null=False)
    created_at = models.DateTimeField('작성일', auto_now_add=True)

    def __str__(self):
        return '[{}] {}'.format(self.id, self.title)
```

### 데이터 검색, 수정, 저장
field가 변경된 것이 아니고 메소드만 변경된 것이니 migration을 해줄 필요는 없습니다.
다시 `shell` 커맨드를 실행하셔서 아까 전에 생성한 데이터를 검색해서 `created_at` 값을 변경해줍니다.
```python
>>> from bbs.models import Article
>>> article = Article.objects.get(id=1)     # id가 1인 Article 데이터 검색. 없거나 2개 이상일 경우 에러발생
>>> print(article)
<Article: [1] How to create a article>
>>> article.created_at = '2018-11-22 01:15'
>>> article.save()                          # 변경된 값 저장. `time formatter('%Y-%m-%d %H:%M')` 형식의 문자열은 DateTimeField에서 자동으로 시간 데이터로 변환해줍니다.
>>> article.created_at.strftime('%Y-%m-%d') # 변경된 created_at 값을 time fomatter를 이용해 출력해보지만 에러발생
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-16-560946d1936a> in <module>
----> 1 article.created_at.strftime('%Y-%m-%d')

AttributeError: 'str' object has no attribute 'strftime'
>>> article.refresh_from_db()               # db로 부터 새로 검색
>>> article.created_at.strftime('%Y-%m-%d') # 정상출력
'2018-11-22'
```
모델의 데이터 검색은 기본 매니저의 `get` 메소드로 검색할 수 있습니다. sql의 `where`에 해당하는 내용을 keyword argument로 전달하면 됩니다. 만일 데이터가 아무것도 발견되지 않거나 2개 이상이 발견될 경우 에러가 발생합니다. get 메소드 대신 `QuerySet`이라는 이터레이터를 반환하는 `filter` 메소드를 이용하면 레코드가 0개 이거나 2개 이상이더라도 오류가 발생하지 않습니다. 

`QuerySet` 객체는 이터레이터의 모든 기능을 사용할 수도 있습니다. 특별히 첫번째 데이터와 마지막 데이터의 경우 `first`, `last` 메소드로도 접근이 가능하고 `QuerySet`이 검색결과가 0개인 경우 `None`을 반환합니다..
```python
>>> Article.objects.filter(author='swarf00').first() # author='swarf00'인 첫번째 레코드 검색
>>> Article.objects.filter(author='swarf00').last()  # author='swarf00'인 마지막 레코드 검색
>>> Article.first()                                  # Article 테이블에서 조건없이 첫번째 레코드 검색
>>> Article.last()                                   # Article 테이블에서 조건없이 마지막 레코드 검색
```

## 4. Admin 사이트

### 관리자 등록하기
manage.py의 `shell` 커맨드도 database를 관리하기에 편리하지만 장고에서는 웹기반의 **admin 사이트**를 제공합니다. **장고ORM**이 익숙하지 않거나 불편하실 경우 **admin 사이트**를 이용하세요.

admin 사이트는 데이터베이스 내의 각종 데이터를 접근 및 수정이 가능하기 때문에 인증기능이 필수로 구현되어 있습니다.
manage.py의 `createsuperuser` 커맨드를 이용하면 간단하게 `superuser` 권한의 사용자를 생성할 수 있습니다.
```bash
(test-venv-36) $ ./manage.py createsuperuser
Username (leave blank to use 'swarf00'):      
Email address:
Password: 
Password (again): 
Superuser created successfully.
```
4가지를 적절하게 입력하면 admin 사이트로 `username` 과 `password` 를 이용해서 접속할 수 있습니다.

### Admin 사이트 접속
장고를 재시작하고 `http://127.0.0.1:8000/admin` 으로 접속합니다.

![로그인 화면]({{ site.url }}/snapshots/admin_login.png){:.border .rounded .shadow}

superuser 계정으로 로그인하시면 현재 admin 사이트에 등록된 앱과 모델들이 나타납니다.
장고2부터는 admin 사이트도 ResponsiveWebDesign이 지원됩니다. pc화면에서는 좀 더 시원한 화면을 볼 수 있습니다.

![로그인 후 화면]({{ site.url }}/snapshots/admin_home.png){:.border .rounded .shadow}

로그인을 하고 admin 사이트의 웰컴페이지가 나타납니다. admin 사이트에 등록된 모든 모델들이 앱별로 보여줍니다. 아직까지 bbs의 모델은 admin 사이트에 등록하지 않은 상태여서 보이지 않습니다. 기본 등록된 모델을 빠르게 보겠습니다.

AUTHENTICATION AND AUTHORIZATION(인증과 권한) 앱에 `Groups`, `Users` 두 모델이 등록되어 있는 것이 확인됩니다. Recent actions라고 가장 최근 작업에 대한 기록도 출력됩니다.

![User 모델 화면]({{ site.url }}/snapshots/admin_aa_users.png){:.border .rounded .shadow}

`Groups` 에는 아무런 레코드도 없고, `Users`를 보니 아까 생성한 `superuser`의 정보가 보입니다. 다행히 비밀번호는 보이지 않습니다.(사실 비밀번호는 해시로 인코딩 되어 있어 안전합니다.)

### Admin 사이트에 등록
`Article` 모델도 admin 사이트에 등록하기 위해 `bbs/admin.py`를 수정합니다.
```python
# bbs/admin.py

from django.contrib import admin

from .models import Article

admin.site.register(Article) 
```

`admin.site.register` 함수를 이용하면 간단하게 `Article` 모델을 admin 사이트에 등록할 수 있습니다. 장고를 재시작하고 다시 admin 사이트에 접속합니다.

![Article 모델 추가화면]({{ site.url }}/snapshots/admin_home_article.png){:.border .rounded .shadow}

BBS 앱에 `Article` 모델이 추가된 걸 확인 할 수 있습니다. `Article` 모델을 클릭하면 레코드리스트가 보입니다.

![Article 리스트 변경전]({{ site.url}}/snapshots/admin_article.png){:.border .rounded .shadow}

### Admin 사이트 커스터마이징
하나의 데이터만 생성시켰었기 때문에 하나의 레코드만 보입니다. 데이터가 문자열형태로 표현되는데 보기가 불편합니다. admin 사이트는 손쉽게 출력화면을 커스터마이징할 수 있는 방법을 제공합니다.
```python
# bbs/admin.py

from django.contrib import admin
  
from .models import Article

@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_display = ('id', 'title', 'author', 'date_created')  # date_created는 아래 정의한 메소드
    list_display_links = ('id', 'title')                      # 상세페이지로 이동할 수 있는 필드 리스트

    def date_created(self, obj):                              # create_at 필드의 출력형식을 변경해주는 메소드
        return obj.created_at.strftime("%Y-%m-%d")

    date_created.admin_order_field = 'created_at'             # date_created 컬럼 제목을 클릭시 실제 어떤 데이터를 기준으로 정렬할 지 결정
    date_created.short_description = '작성일'                   # date_created 컬럼 제목에 보일 텍스트
```

![Article 리스트 변경후]({{ site.url }}/snapshots/admin_article_new.png){:.border .rounded .shadow}

이전과 다르게 ArticleAdmin이란 클래스를 생성 후 @admin.register(Article) 데코레이터로 wrapping을 했습니다. admin.site.register 함수를 사용하는 것보다 번거롭지만 상세한 설정이 가능합니다.

`list_display` 속성에 필드명을 문자열의 튜플로 저장해주면 admin 사이트의 리스트에서 튜플의 순서대로 테이블의 컬럼을 생성해줍니다. `created_at` 필드를 그대로 사용하면 장고의 기본 포멧대로 출력되기 때문에 `date_created` 함수를 생성해서 **출력형식을 변경**해줬습니다. `date_created` 함수는 admin 사이트에서 필드처럼 사용됩니다. 하지만 모델에 존재하는 필드가 아니기 때문에 **컬럼에 대한 속성을 새로 지정**해줘야 합니다.

`list_display_links` 는 레코드의 상세내용을 확인하거나 수정하기 위해 이동할 수 있는 링크를 어떤 필드에 제공할 것인가에 대한 속성입니다. 기본은 첫번째 필드인 `id`에만 링크가 걸려있지만 좀 더 클릭하기 편하게 `id`, `title` 에 클릭할 수 있도록 설정했습니다.

변경한 대로 작성일 컬럼이 추가되었고 출력형식은 `%Y-%m-%d` 형식으로 출력됩니다. 또한 제목을 클릭하면 작성일을 기준으로 정렬을 할 수 있습니다. 제목을 클릭하여 상세 페이지로 이동할 수 있습니다.

![Article 리스트 변경후]({{ site.url }}/snapshots/admin_article_detail.png){:.border .rounded .shadow}

상세 페이지에서 작성일이 보이지 않는데 `DateTimeField`의 `auto_now_add` 속성이 `True`로 설정이 되면 `editable` 속성이 자동으로 `False`가 설정됩니다. `editable` 속성을 `True`로 변경해 주면 디테일 화면에서 확인 및 수정이 가능해집니다.
```python
# bbs/models.py

from django.db import models
  
class Article(models.Model):
    title      = models.CharField('제목', max_length=126, null=False)
    content    = models.TextField('내용', null=False)
    author     = models.CharField('작성자', max_length=16, null=False)
    created_at = models.DateTimeField('작성일', auto_now_add=True)
    created_at.editable = True                                     # created의 editable 속성에 True를 설정했습니다.

    def __str__(self):
        return '[{}] {}'.format(self.id, self.title)
```

![Article 상세 페이지]({{ site.url }}/snapshots/admin_article_detail_new.png){:.border .rounded .shadow}

> 모델만들기에 약간의 시간을 허비했지만 admin 사이트를 더 많은 방법으로 커스터마이징 하는 것이 이번 튜토리얼의 목표에 대해서 불필요하므로 임시로 덮어 두기로 결정하였다.
>
> -- swarf00, 모델만들기에 대한 간단한 예제에서...
