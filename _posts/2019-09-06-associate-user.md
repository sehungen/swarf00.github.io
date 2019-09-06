---
layout: article
title: 사용자인증(5)
excerpt: 장고ORM 의 ForeignKey 사용법을 배워봅니다. ForeignKey는 다소 많은 옵션을 제공하지만 자주 사용되는 옵션은 몇가지 되지 않으니 소개해드리는 것만은 꼭 기억하시길 바랍니다.
key: django-authentication-05
aside:
  toc: true
sidebar:
  nav: docs-ko
mermaid: true
pageview: true
date: 2019/09/06
tags: 장고 Django 인증 ForeignKey
metadata:
  og_title: 장고(Django) 게시글 사용자 연동
  og_type: article
  og_locale: ko_KR
  og_description: 장고ORM 의 ForeignKey 사용법을 배워봅니다.
  og_site_name: 장고(Django) 핥짝 맛보기
---
## 1. 사용자가 작성하는 게시글

지금까지의 게시글은 로그인만 되어 있으면 누구라도 작성하도록 되어 있습니다. 게시글에 사용자의 이름을 남겨서 작성자가 누구인지만 알게 해뒀습니다. 그런데 만약 사용자의 이름이 겹친다면 어떻해야 하나요? 또 내가 작성한 글을 나만 수정하고 싶은데 이름만 같다면 누구라도 내 글을 수정할 수 있나요? 누가 작성한 글인지에 대한 식별을 작성자가 입력한 사용자 이름으로만 구별하는 것은 좋은 방법이 아니란 것이 드러났습니다. 

그럼 author 라는 필드를 수정해서 User 모델의 pk 값인 id 값을 저장하도록 하면 어떨까요? 그렇게 하면 실제로 저장된 사용자는 User 모델에 저장된 사람에 한정되게 됩니다. 한번 적용하고 다시 설명하도록 하죠.
```python
# bbs/models.py

from django.db import models
from django.utils import timezone


class Article(models.Model):
    title      = models.CharField('제목', max_length=126, null=False)
    content    = models.TextField('내용', null=False)
    # author     = models.CharField('작성자', max_length=16, null=False)
    author     = models.PositiveIntegerField('작성자')
    created_at = models.DateTimeField('작성일', default=timezone.now)

    def __str__(self):
        return '[{}] {}'.format(self.id, self.title)
```

아직 마이그레이션 하시지 마시고 일단 아래 설명을 들어보세요.  
이렇게 하면 실제 데이터가 게시글이 등록될 때 등록하는 사용자의 id 값이 저장되고, 해당 게시글을 수정할 때는 로그인된 사용자의 id 값을 author 와 필드와 비교해서 동일할 경우만 수정이 되도록 할 수도 있습니다. 또한 Article 모델에서 자기가 작성한 게시글만 필터링 해서 볼 수도 있겠네요. 그리고 게시글 목록에서도 author 값으로 User 테이블을 검색해서 작성자의 이름을 표시하도록 할 수 있습니다. 이렇게만 해도 충분히 원하는 기능들을 사용할 수 있을 것 같습니다. author 에 사용자의 id 값이 저장되어 있다는 것을 우리는 잘 알고 있기 때문에 실수할 일이 없겠죠. 하지만 두명이 이상이 같이 작업을 하거나, 1년이 지나고 2년이 지나서 여러 개발자들을 거쳐가면서 데이터베이스 모델이 복잡해지고 소스코드가 커지면 author가 User 모델의 id 값이라는 것을 알아차리기 어려울 수 있습니다. 그렇게 되면 실제 존재하지 않는 User의 id를 입력하거나 잘못된 id 값을 입력하는 등의 오류가 발생할 수가 있죠. 물론 소스코드에서 무결성을 유지되도록 잘 처리할 수도 있겠지만 성능적인 부분은 고려하지 않더라도 그것 또한 또하나의 문제가 발생할 수 있는 지점이 됩니다.  
이렇게 모델의 필드값을 다른 모델의 id로 사용할 때의 장점이 있는 반면 단점도 분명히 존재합니다. 일반적으로 RDB(관계형 데이터베이스)에서 이런 경우 모델과 모델에 제약조건을 추가하여 두 모델 간의 관계성을 데이터베이스에 주입하고, 데이터베이스가 알아서 무결성을 보장하도록 설정합니다. Foreign Key(외래키)라고 하는 것이 바로 그것인데, ForeignKey에 대한 설명은 [여기](https://namsieon.com/entry/SQL-%EC%B0%B8%EC%A1%B0%ED%82%A4-Foreign-key-%EC%99%B8%EB%9E%98%ED%82%A4-%EA%B7%B8%EB%A6%AC%EA%B3%A0-%EC%A0%9C%EC%95%BD%EC%A1%B0%EA%B1%B4)를 참고하시길 바랍니다.

## ForeignKey(외래키)
데이터베이스(RDB)에서는 ForeignKey 를 지정해서 테이블 간의 관계를 표현하는 것이 가능합니다. 이 ForeignKey를 장고ORM에서도 상당히 편리한 방법으로 표현할 수 있습니다. ForeignKey 라는 필드타입을 제공하는데 sql의 그 ForeignKey 를 생성하는 기능을 제공합니다. ForeignKey 필드는 연결되어지는 모델과 해당 모델이 1:N 관계가 됩니다. 당연히 하나의 Article 은 1명의 User가 작성을 할 수 있고, 1명의 User는 다수의 Article을 작성할 수 있으니 User 와 Article 은 1:N 관계가 됩니다. 이해가 안되면 닥공😏 일단 Article 모델의 author 필드를 ForeignKey 로 변경하도록 하겠습니다.
```python
# bbs/models.py

from django.db import models
from django.utils import timezone


class Article(models.Model):
    title      = models.CharField('제목', max_length=126, null=False)
    content    = models.TextField('내용', null=False)
    # author     = models.CharField('작성자', max_length=16, null=False)
    author     = models.ForeignKey('user.User', related_name='articles', on_delete=models.CASCADE)
    created_at = models.DateTimeField('작성일', default=timezone.now)

    def __str__(self):
        return '[{}] {}'.format(self.id, self.title)
```

### 모델 수정
ForeignKey 필드를 가장 전형적인 방식으로 선언을 했습니다. 첫번째 인자로 ForeignKey로 연결할 모델의 식별자를 전달합니다. 직접 모델 클래스를 전달하는 방법도 있고, 저렇게 '앱label.모델이름'으로 표현하는 방법도 있습니다. 어느 방법을 사용하셔도 동일하지만 앱이 많아지면(실무에선) 이런 방식으로 선언하는 것을 추천합니다.(소곤소곤🤫 앱끼리 모델명이 중복되는 경우가 있어서 헷갈리드라고욤)
두번째 인자 `related_name` 은 실제 데이터베이스 상에 추가되는 속성은 아니고 ForeignKey 로 설정되어지는 모델(User)에서 Article 객체를 참조해야 할 때 사용하는 속성으로 사용됩니다. 예를 들어 특정 사용자가 작성한 모든 게시글을 검색할 때 `user.articles.all()` 이라고 하면 User 모델에 정의한 적 없는 articles 라는 속성으로 추가되어 연관된 Article 모델을 검색할 수 있게 됩니다. `related_name` 이라는 속성을 추가하지 않으면 기본적으로 클래스이름(소문자)+'_set' 으로 `related_name` 이 추가됩니다. `related_name` 을 설정하지 않았다면 User 모델의 인스턴스에서 `user.article_set.all()`이라고 검색할 수 있습니다. 굳이 `related_name` 속성을 정의해주는 이유는 간혹 두 모델 사이에 ForeignKey 가 두개 이상 존재하는 경우도 있습니다. 이럴 때 자동 생성된 `related_name`이 겹쳐 오류가 생길 수 있으니 습관적으로 `related_name` 을 추가해주면 오류를 줄일 수 있습니다.
`on_delete` 속성은 ForeignKey 로 연결되는 모델(User)의 데이터가 삭제될 경우 해당 ForeignKey 필드의 값을 어떻게 할 지에 대한 설정입니다. 기본값은 `CASCADE` 입니다. 윗물이 맑으면 아랫물이 맑다는 말이 있죠. `CASCADE는` 연결된 모델(User)의 인스턴스가 삭제되면 해당 인스턴스를 ForeignKey로 연결한 Article의 인스턴스도 같이 삭제해버립니다. 만약 연결된 모델(User)의 인스턴스가 삭제되더라도 ForeignKey로 연결된 Article의 인스턴스를 삭제하지 않아야 할 경우 선택할 수 있는 방법도 있습니다. SET_NULL, SET_DEFAULT, SET, PROTECT, DO_NOTHING 입니다. 친절하게도 [문서](https://docs.djangoproject.com/ko/2.2/ref/models/fields/#foreignkey)에 자세한 설명이 나와있으니 확인해보세요. 번역기 돌려보시고 이해가 가지 않으면 댓글을 남겨주세요😂

이제 실제 migration 해보겠습니다.

```bash
$ ./manage.py makemigrations
$ ./manage.py migrate

Operations to perform:
  Apply all migrations: admin, auth, bbs, contenttypes, sessions
Running migrations:
  Applying auth.0010_alter_group_name_max_length...Traceback (most recent call last):
  File "./manage.py", line 15, in <module>
    execute_from_command_line(sys.argv)
  File "/Users/sehunkim/minitutorial/venv/lib/python3.7/site-packages/django/core/management/__init__.py", line 381, in execute_from_command_line
    utility.execute()
  File "/Users/sehunkim/minitutorial/venv/lib/python3.7/site-packages/django/core/management/__init__.py", line 375, in execute
    self.fetch_command(subcommand).run_from_argv(self.argv)
  File "/Users/sehunkim/minitutorial/venv/lib/python3.7/site-packages/django/core/management/base.py", line 323, in run_from_argv
    self.execute(*args, **cmd_options)
  File "/Users/sehunkim/minitutorial/venv/lib/python3.7/site-packages/django/core/management/base.py", line 364, in execute
    output = self.handle(*args, **options)
  File "/Users/sehunkim/minitutorial/venv/lib/python3.7/site-packages/django/core/management/base.py", line 83, in wrapped
    res = handle_func(*args, **kwargs)
  File "/Users/sehunkim/minitutorial/venv/lib/python3.7/site-packages/django/core/management/commands/migrate.py", line 234, in handle
    fake_initial=fake_initial,
  File "/Users/sehunkim/minitutorial/venv/lib/python3.7/site-packages/django/db/migrations/executor.py", line 117, in migrate
    state = self._migrate_all_forwards(state, plan, full_plan, fake=fake, fake_initial=fake_initial)
  File "/Users/sehunkim/minitutorial/venv/lib/python3.7/site-packages/django/db/migrations/executor.py", line 147, in _migrate_all_forwards
    state = self.apply_migration(state, migration, fake=fake, fake_initial=fake_initial)
  File "/Users/sehunkim/minitutorial/venv/lib/python3.7/site-packages/django/db/migrations/executor.py", line 247, in apply_migration
    migration_recorded = True
  File "/Users/sehunkim/minitutorial/venv/lib/python3.7/site-packages/django/db/backends/sqlite3/schema.py", line 34, in __exit__
    self.connection.check_constraints()
  File "/Users/sehunkim/minitutorial/venv/lib/python3.7/site-packages/django/db/backends/sqlite3/base.py", line 318, in check_constraints
    bad_value, referenced_table_name, referenced_column_name
django.db.utils.IntegrityError: The row in table 'bbs_article' with primary key '1' has an invalid foreign key: bbs_article.author_id contains a value 'swarf00' that does not have a corresponding value in user_user.id.
```

이제는 익숙하시겠죠. 역시나 오류가 발생합니다. 침착하게 오류메시지를 살펴봅니다. ForeignKey로 참조하는 User 모델의 필드가 `User.id`(int) 인데 ForeignKey 에 저장된 값이 swarf00(str) 이라 무결성이 맞지 않아 마이그레이션이 되지 않는다고 합니다. ForeignKey 필드로 설정된 필드에 이미 문자열 데이터가 들어가 있으니 User 모델의 `id` 의 자료형이 맞지 않아 오류가 발생하는 겁니다. 이럴 때 깨끗이 데이터를 날리고 시작하면 깔끔하고 편한데 실무에서는 책상도 깨끗하게 날라가는 겁니다.(데이터베이스 작업을 할 때는 꼭 백업을 해둡시다)

일단 데이터베이스가 변경되었는지 오류로 인해 롤백이 되었는 지 확인해봅니다.
```sql
sqlite> pragma table_info(bbs_article);
0|id|integer|1||1
1|title|varchar(126)|1||0
2|content|text|1||0
3|author_id|integer|1||0
4|created_at|datetime|1||0
```

일단 author 필드가 author_id 필드로 변경되었고 자료형이 integer로 변경된 것이 확인됩니다. ForeignKey 필드 이름로 author 를 사용했는데 실제 데이터베이스에 `author_id` 로 필드명이 변경된 이유는 ForeignKey 필드의 또다른 속성 `to_field` 값을 설정하지 않아 기본값인 참조하는 모델의 pk 필드인 id로 설정이 된 것 입니다. 잘 사용하지 않는 속성이니 대충 넘어갑니다.

어쨋든 데이터베이스가 잘 마이그레이션되었으니 `author_id` 의 값에 적절한 값을 넣어 수정합니다. 강제로 `user_id` 에 1을 설정하도록 하겠습니다.
```sql
sqlite> update bbs_article set author_id = 1;
```

`shell` 커맨드를 실행하여 ForeignKey 가 잘 작동되는 지 몇가지 테스트를 해보겠습니다.
```python
>>> from bbs.models import Article
>>> article = Article.objects.first()
>>> article.author
<User: swarf00@gmail.com>
>>> user = article.author
>>> user.articles.all()
<QuerySet [<Article: [1] How to create a article>, <Article: [2] foo>, <Article: [3] 새로운 게시글>]>
```

> 실제 사용중인 데이터베이스에서는 이런식으로 ForeignKey를 변경하는 경우는 없다고 생각하셔야 합니다. 정확하게는 말씀드리면 이렇게 변경하도록 설계를 하면 안됩니다. 예제를 위해 일부러(진짜로🤣) 이렇게 설계한 것 입니다. 이런 경우 일반적으로 ForeignKey를 제대로 설정하지 못해 버리게 되는 데이터가 대량으로 발생할 수 있습니다.

### 뷰 수정
실제 ForeignKey 연결이 잘된 것이 확인되었습니다. 이제 뷰와 템플릿을 순차적으로 수정해 나가겠습니다. 지금까지는 author가 문자열였지만 이제는 ForeignKey 즉 숫자형이기 때문에 문자열이 저장이 되거나 문자열로 처리를 할 경우 오류가 발생할 수 있습니다. 먼저 뷰부터 수정을 해보도록 하죠.
현재 뷰는 3가지가 있죠. `ArticleListView`, `ArticleDetailView`, `ArticleCreateUpdateView` 이 세가지가 있는데 author 를 참조하는 건 다행히 `ArticleCreateUpdateView` 뿐 입니다.(이것 역시 나의 Big Picture!!). POST 로 입력받은 author를 처리하지 않도록 수정하고, 저장할 때 로그인한 사용자의 인스턴스를 article.author 에 저장해주도록 수정합니다. 또 한가지 더 해야 할 것이 있는데, 작성된 글을 수정할 때는 작성자가 수정하려는 사용자와 동일한 사용자인지 확인하는 절차를 거쳐야 합니다. 아래와 같이 ~복붙~수정해 봅니다.

```python
# bbs/views.py

class ArticleCreateUpdateView(LoginRequiredMixin, TemplateView):
    login_url = settings.LOGIN_URL
    template_name = 'article_update.html'
    queryset = Article.objects.all()
    pk_url_kwargs = 'article_id'

    def get_object(self, queryset=None):
        queryset = queryset or self.queryset
        pk = self.kwargs.get(self.pk_url_kwargs)
        article = queryset.filter(pk=pk).first()

        if pk:
          if not article:
            raise Http404('invalid pk')
          elif article.author != self.request.user:                             # 작성자가 수정하려는 사용자와 다른 경우
            raise Http404('invalid user')
        return article

    def get(self, request, *args, **kwargs):
        article = self.get_object()

        ctx = {
            'article': article,
        }
        return self.render_to_response(ctx)

    def post(self, request, *args, **kwargs):
        action = request.POST.get('action')
        post_data = {key: request.POST.get(key) for key in ('title', 'content')} # 작성자를 입력받지 않도록 수정
        for key in post_data:
            if not post_data[key]:
                messages.error(self.request, '{} 값이 존재하지 않습니다.'.format(key), extra_tags='danger')

        post_data['author'] = self.request.user                                  # 작성자를 현재 사용자로 설정

        if len(messages.get_messages(request)) == 0:
            if action == 'create':
                article = Article.objects.create(**post_data)
                messages.success(self.request, '게시글이 저장되었습니다.')
            elif action == 'update':
                article = self.get_object()
                for key, value in post_data.items():
                    setattr(article, key, value)
                article.save()
                messages.success(self.request, '게시글이 저장되었습니다.')
            else:
                messages.error(self.request, '알 수 없는 요청입니다.', extra_tags='danger')

            return HttpResponseRedirect('/article/')

        ctx = {
            'article': self.get_object() if action == 'update' else None
        }
        return self.render_to_response(ctx)
```

실제로 뷰는 이렇게만 해도 충분해 보입니다. 실제로 테스트로 작성된 글을 수정하기도 하고 새로운 글을 작성해보면 의도한 대로 동작되는 것이 확인됩니다. 아직 템플릿을 수정하지 않은 상태이기 때무에 작성자 입력란에 어떠한 텍스트를 입력하거나 공백으로 두어도 정상적으로 로그인한 사용자로 게시글의 작성자가 저장되는 것을 확인할 수 있습니다. 또한 작성자와 로그인한 사용자가 다를 경우 오류 페이지를 보여주게 되면 성공입니다.

### 템플릿 수정

이제 템플릿을 변경할 차례입니다. 크게 두가지를 수정하면 됩니다. 다행히 User 모델에 `__str__` 함수가 미리 정의되어 있고, `article.author` 라고 출력하더라도 이메일만 출력이 되어 기존의 `article.author` 템플릿변수를 수정할 필요는 없습니다. 그렇지 않다면 `article.author` 를 출력할 경우 `<User: swarf00@gmail.com>` 식으로 출력이 되어 보기에 좀 불편하기 때문에 `article.author.email` 이라고 변경해줘야 합니다. 템플릿에서 해야 할 일은 단지 article_update.html 에서 작성자 입력란을 삭제하는 것 입니다. 이건 딱히 아래 수정된 코드를 보지 않아도 척척 잘 하시겠지만 복붙하시는 분들이 아직도 많이 계시리라 짐작되기 때문에 친절하게 수정된 코드를 올려드립니다.

```html
<!-- bbs/templates/article_update.html -->

{% raw %}
{% extends 'base.html' %}

{% block title %}
{% if article %}
<title>게시글 수정 - {{ article.pk }}. {{ article.title }}</title>
{% else %}
<title>게시글 작성</title>
{% endif %}
{% endblock title %}

{% block content %}
<form action="." method="post" class="form-horizontal">
{% csrf_token %}
<input type="hidden" name="action" value="{% if article %}update{% else %}create{% endif %}">
<table class="table table-striped table-bordered">
	<tr>
		<th>번호</th>
		<td>{{ article.pk }}</td>
	</tr>
	<tr>
		<th>제목</th>
		<td><input type="text" class="form-control" name="title" value="{{ article.title }}"></td>
	</tr>
	<tr>
		<th>내용</th>
		<td><textarea rows="10" class="form-control" name="content">{{ article.content }}</textarea></td>
	</tr>
  <!--<tr>-->
		<!--<th>작성자</th>-->
		<!--<td><input type="text" class="form-control" name="author" value="{{ article.author }}"></td>-->
	<!--</tr>-->
    <tr>
		<th>작성일</th>
		<td>{{ article.created_at | date:"Y-m-d H:i" }}</td>
	</tr>
</table>

<button class="btn btn-primary" type="submit">게시글 저장</button>
</form>
{% endraw %}
```

첨부터 모델 설명할 때 ForeignKey로 설명하면 되는데 굳이~ 첨부터 돌아돌아 여기까지 오도록 삽질을 끌어내, 여러분들의 값진 피와 땀으로 성장을 보게되어 뿌듯합니다. 이 정도로 가볍게 핥짝거리는 걸로 만족하지 못하시는 ~변태~분들은 다음 포스트를 기다려주세요.

> Talk is cheap. Show me the code.(말은 쉽지. 코드를 보여줘.)
>
> Linus Torvalds