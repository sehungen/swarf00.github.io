---
layout: article
title: 뷰 만들기
excerpt: 장고(Django) 웹프레임워크의 FBV와 CBV를 통해 뷰를 구현하는 방법을 설명합니다. CBV는 제네릭 뷰 중 가장 단순한 구현체인 TemplateView를 통해 개발하는 방법을 설명합니다.
key: django-build-view
aside:
  toc: true
sidebar:
  nav: docs-ko
pageview: true
tags: 장고 Django 뷰 CBV FBV CSRF TemplateView
metadata:
  og_title: 장고(Django) 뷰 정의하기
  og_type: article
  og_locale: 'ko_KR'
  og_description: 장고(Django) 웹프레임워크의 FBV와 CBV를 통해 뷰를 구현하는 방법을 설명합니다. CBV는 제네릭 뷰 중 가장 단순한 구현체인 TemplateView를 통해 개발하는 방법을 설명합니다.
  og_site_name: 장고(Django) 핥짝 맛보기
---

## 1. 뷰 설계
모델을 설계할 때 가정한 사용자들의 행동들을 화면 단위로 상상해봅니다.

> 1. 게시글 목록 - 게시판(게시글 목록)에는 게시글들의 목록이 나열됩니다.
> 2. 게시글 목록 - 게시글들은 제목과 작성자 표시됩니다. 
> 3. 게시글 상세 화면 - 게시글을 (클릭해서) 들어가면 게시글 상세화면으로 이동하고 제목, 내용, 작성일이 출력합니다.
> 4. 게시글 수정 화면 - 게시글 상세화면에서 수정하기 버튼을 누르면 수정하는 화면으로 이동합니다.
> 5. 게시글 수정 화면 - 게시글 수정화면에서 저장하기 버튼을 누르면 수정된 내용이 저장되고 게시판으로 이동합니다.
> 6. 게시글 수정 화면 - 게시글 수정화면에서 삭제하기 버튼을 누르면 게시글이 삭제되고 게시판으로 이동합니다.
> 7. 게시글 추가 화면 - 게시판에서 새글쓰기 버튼을 누르면 새로운 게시글을 입력할 수 있는 화면이 출력됩니다.
> 8. 게시글 추가 화면 - 게시글을 작성하고 저장하기 버튼을 누르면 수정된 내용이 저장되고 게시판으로 이동합니다.

화면이 크게 4가지(목록, 상세, 수정, 추가)이고 수정과 추가화면은 동일한 화면을 사용해도 될 것 같습니다. 추가화면에는 각 입력값이 빈 상태로 나타나고, 수정화면은 추가화면에 데이터베이스에 저장된 값으로 초기화해서 보여주면 될 것 같습니다.

> DRY (Don't Repeat Yourself) principle
> * 똑같은 일을 두번 하지 않는다. 
> * 중복되는 함수나 코드는 하나의 공통의 콤포넌트(또는 함수)에 넣고 사용한다.
> * 큰 시스템을 여러 조각으로 나누고 서로 참조한다.

## 2. 뷰 생성
### 화면 핸들러 정의하기
우선 각 화면들을 표시할 핸들러들을 정의합니다.
```python
# bbs/views.py

from django.http import HttpResponse
  
def hello(request, to):
    return HttpResponse('Hello {}.'.format(to))

def list_article(request):                          # 목록보기
    return HttpResponse('list')

def detail_article(request, article_id):            # 상세보기, 상세보기할 article의 id 필요
    return HttpResponse('detail {}'.format(article_id))

def create_or_update_article(request, article_id):  # 생성 및 수정하기, 수정할 때는 article의 id 필요
    if article_id:
        return HttpResponse('update {}'.format(article_id))
    else:
        return HttpResponse('create')
```
url에 핸들러를 연결시켜 줍니다.
```python
# minitutorial/urls.py

from django.contrib import admin
from django.urls import path

from bbs.views import hello, list_article, detail_article, create_or_update_article

urlpatterns = [
    path('hello/<to>', hello),
    
    path('article/', list_article),
    path('article/create/', create_or_update_article, {'article_id': None}), # {'article_id':None} 필수
    path('article/<article_id>/', detail_article),
    path('article/<article_id>/update/', create_or_update_article),

    path('admin/', admin.site.urls),
]
```
`create_or_update_article` 핸들러에서 article_id 함수의 기본값이 없기 때문에 반드시 핸들러에 추가 파라미터 {'article_id':None}을 넣어줘야 합니다.

장고를 실행하고 아래와 같이 접속해보고 화면에 정상적으로 출력이 되는 지 확인을 합니다.
* http://127.0.0.1/article/           # list 출력
* http://127.0.0.1/article/create/    # create 출력
* http://127.0.0.1/article/10/        # detail 10 출력
* http://127.0.0.1/article/11/update/ # update 10 출력

### 액션 핸들러 정의하기
화면은 일단 더미 핸들러를 만들어 출력했으니 이제 화면 내에서의 사용자 입력(수정하기, 생성하기 등)을 처리하는 핸들러를 정의합니다.
```python
# bbs/views.py

# 생략
def do_create_article(request):
    return HttpResponse(request.POST)

def do_update_article(request):
    return HttpResponse(request.POST)
```
RESTful API를 제공하면 좋겠지만 본 미니튜토리얼에서는 전통적인 POST 방식으로 액션을 처리할 예정입니다. 걸음마는 첫걸음부터, 프로그래밍은 헬로월드부터 사용자액션은 POST방식부터...순서대로 합니다.

액션 핸들러도 url은 따로 연결하지 않고 create_or_update 핸들러에서 메소드가 GET 일 경우 화면을 보여주고, POST 일 경우 액션핸들러를 호출합니다.
```python
# bbs/views.py

from django.http import HttpResponse, HttpResponseNotAllowed

# 생략
def create_or_update_article(request, article_id):
    if article_id: # 수정하기
        if request.method == 'GET':
            return HttpResponse('update {}'.format(article_id))
        elif request.method == 'POST':
            return do_create_article(request)
        else:
            return HttpResponseNotAllowed(['GET', 'POST'])
    else:          # 생성하기
        if request.method == 'GET':
            return HttpResponse('create')
        elif request.method == 'POST':
            return do_update_article(request)
        else:
            return HttpResponseNotAllowed(['GET', 'POST'])

def do_create_article(request):
    return HttpResponse(request.POST)

def do_update_article(request):
    return HttpResponse(request.POST)
```

HttpResponseNotAllowed 클래스는 HttpResponse와 다르게 status_code가 405이고 허용되지 않는 메소드로 요청했다는 의미를 가지고 있습니다. POST 방식이므로 GET과 POST만 허용합니다.

POST에 대해서는 아직 테스트를 하지 않고 개발을 진행하면서 테스트합니다.

### CBV로 변환
create_or_update_article 핸들러는 작성하면서도 느꼈지만 **DRY**원칙이 떠오릅니다. 장고에서는 FBV(Function Based View)와 CBV(Class Based View) 두가지의 뷰를 개발할 수 있는 방법을 제공합니다. 현재까지는 FBV로만 개발했지만 CBV를 이용한다면 중복된 코드를 최소화할 수 있습니다. ~~원래는 다 구현하고 리팩토링 하려했으나~~ 더 늦기 전에 CBV로 변환합니다. ^^;
```python
# bbs/views.py

from django.http import HttpResponse
from django.views.generic import TemplateView


class ArticleListView(TemplateView):         # 게시글 목록
    template_name = 'base.html'

    def get(self, request, *args, **kwargs):
        ctx = {}                             # 템플릿에 전달할 데이터
        return self.render_to_response(ctx)


class ArticleDetailView(TemplateView):       # 게시글 상세
    template_name = 'base.html'

    def get(self, request, *args, **kwargs):
        ctx = {}
        return self.render_to_response(ctx)


class ArticleCreateUpdateView(TemplateView):  # 게시글 추가, 수정
    template_name = 'base.html'

    def get(self, request, *args, **kwargs):  # 화면 요청
        ctx = {}
        return self.render_to_response(ctx)

    def post(self, request, *args, **kwargs): # 액션
        ctx = {}
        return self.render_to_response(ctx)


def hello(request, to):
    return HttpResponse('Hello {}.'.format(to))
```
장고에서는 CBV를 지원하기 위해 `제네릭 뷰`라고 부르는 다양한 클래스들을 제공있습니다. 대표적으로 `TemplateView`, `ListView`, `DetailView`, `CreateView`, `UpdateView` 등이 있습니다. 그 중 가장 간단한 `TemplateView`를 이용해 변환했습니다.

모든 뷰에 클래스변수로 template_name 속성을 추가해서 모두 'base.html'이라고 정의했습니다. base.html은 템플릿 파일의 이름입니다. template_name을 정의하면 장고에서는 자동으로 앱 디렉토리의 templates 디렉토리에서 참조해 파일명이 template_name인 파일을 템플릿으로 사용합니다. 

제네렉 뷰에서는 http메소드에 따라 해당 이름의 클래스 매소드를 호출합니다. 화면요청의 경우 항상 http get으로 요청할 것이고 액션의 경우 항상 http post로 요청할 것입니다. http get은 데이터를 url에 query 파라미터로만 보낼 수 있어서 제한적이지만, http post는 데이터를 body에 보낼 수 있어서 사이즈에 제한 없이 다양한 종류의 데이터를 전송할 수 있기 때문입니다.

모든 뷰에 공통적으로 get 핸들러가 정의되어 있지만 post 핸들러는 액션이 필요한 ArticleCreateUpdateView에서만 정의했습니다. get 핸들러와 post 핸들러를 언제 사용해야 하는 지 헷갈린다면 간단하게 **화면을 보여줘 = get, 서버에서 처리해줘 = post** 라고 생각하시면 됩니다. **게시글을 작성할 화면을 보여줘 = get, 게시글을 서버에 저장해줘 = post** 이런 식입니다. 여러가지의 뷰들을 처리하다보면 익숙해질 수 있습니다.

모든 핸들러에 공통적으로 self.render_to_response로 반환하도록 되어 있습니다. render_to_response는 제네릭 뷰에서 제공하는 함수로서, 템플릿을 자동적으로 기본 템플릿 엔진을 이용해서 html로 변환해주는 함수입니다. 이 때 템플릿 내부에 변수를 사용해야 한다면 인자로 ctx(CONTEXT) 객체를 전달해 줄 수 있습니다. 공통된 템플릿을 사용하더라도 ctx 값은 각 뷰마다 적절하게 사용하면 됩니다. 단 반드시 dict 형태로 정의해야 합니다.

핸들러가 함수에서 클래스로 변경되었으니 url 연결도 변경해줘야 합니다.

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

    path('admin/', admin.site.urls),
]
```
path 함수의 두번째 인자로 핸들러 함수가 전달되던 것이 뷰 클래스의 as_view()메소드 실행 결과값으로 변경되었습니다. as_view메소드는 간단하게 설명하면 뷰클래스의 초기화와 핸들러를 반환하는 기능을 제공합니다. 

마지막으로 뷰에서 호출할 template을 작성을 합니다. template은 우선 dummy로 만들고 모든 화면이 정상적으로 동작하는 것을 확인하면 하나씩 변경할 것입니다.
```html
<!-- bbs/templates/base.html -->

<!DOCTYPE html>
<html lang="ko">
    <head>
        <title>base</title>
    </head>
    <body>
    base....
    </body>
</html>
```
base.html은 아무런 템플릿태그나 템플릿변수도 사용하지 않는 dummy template 입니다. 이럴때 장고 템플릿은 아무런 변경을 하지 않습니다.

FBV로 테스트했을 때처럼 장고를 실행하고 url에 접속해서 정상적으로 화면이 출력되는 지 확인합니다.
* http://127.0.0.1/article/           # base... 출력
* http://127.0.0.1/article/create/    # base... 출력
* http://127.0.0.1/article/10/        # base... 출력
* http://127.0.0.1/article/11/update/ # base... 출력

### 데이터 검색

이제 실제로 데이터를 불러와서 화면에 출력해야 하는 뷰에서 데이터를 검색 후 어떻게 템플릿에 데이터를 전달하는 지 확인할 차례입니다.
TemplateView는 데이터 검색에 대한 메소드를 제공해주지 않기 때문에 상속받은 클래스에 직접 구현해야 합니다. 

먼저 ArticleListView부터 하나씩 수정해 나갑니다.
```python
# bbs/views.py

# 생략
from bbs.models import Article


class ArticleListView(TemplateView):
    template_name = 'base.html'
    queryset = Article.objects.all()         # 모든 게시글

    def get(self, request, *args, **kwargs):
        ctx = {
            'view': self.__class__.__name__, # 클래스의 이름
            'data': self.queryset            # 검색 결과
        }
        return self.render_to_response(ctx)
# 생략
```
`queryset` 이라는 클래스 변수를 정의를 했는데 `queryset`은 `TemplateView`를 제외한 다른 제네릭뷰들에서 공통적으로 정의된 클래스변수입니다. 제네릭뷰에서 데이터 검색관련 클래스 변수가 필요하다면 ~~꼭 이렇게 하지 않아도 되지만~~ `queryset`으로 정의하는 것을 추천합니다. `queryset` 뿐만 아니라 다른 클래스 변수들도 장고에서 사용하는 일반적인 변수명을 ~~아는만큼~~ 사용하는 것을 추천합니다.

ctx 값은 `view`와 `data`로 키를 구성했습니다. `view`의 값 `self.__class__.__name__`는 해당 제네릭뷰 인스턴스의 클래스 이름으로서, 현재 보여지는 화면을 처리하는 뷰의 이름을 전달했습니다. `data`는 검색된 데이터 그대로 전달하는데 템플릿에서는 장고ORM의 `QuerySet`을 잘 이해하기 때문에 그대로 사용가능 합니다.

> 다른 제네릭뷰에서는 `SingleObjectMixin`, `MultipleObjectMixin` 등을 상속받아 정의가 되어 있습니다. 두 믹스인의 
클래스변수를 미리 알아두면 좋을 것 같습니다. 메소드는 소스코드를 참고해서 공부해보세요.
> * **SingleObjectMixin**
>   * model = None                # 뷰에서 사용할 모델
>   * queryset = None             # 검색 객체
>   * slug_field = 'slug'         # 모델에 정의된 슬러그 필드 이름
>   * context_object_name = None  # 템플릿에 전달될 검색 데이터 이름
>   * slug_url_kwarg = 'slug'     # path 함수로부터 전달받을 슬러그의 키워드 이름
>   * pk_url_kwarg = 'pk'         # path 함수로부터 전달받을 pk의 키워드 이름
>   * query_pk_and_slug = False   # 슬러그와 pk를 데이터 검색에서 사용할 지 여부
> * **MultipleObjectMixin**
>   * allow_empty = True          # 검색결과가 없어도 되는 지 여부
>   * queryset = None             # 검색 객체
>   * model = None                # 뷰에서 사용할 모델
>   * paginate_by = None          # 검색데이터가 많을 때 한 페이지당 보여줄 데이터 수량
>   * paginate_orphans = 0        # 마지막 페이지의 최소 데이터 수량
>   * context_object_name = None  # 템플릿에 전달된 검색 데이터 이름
>   * paginator_class = django.core.paginator.Paginator  # 페이지화를 작동시킬 구현체
>   * page_kwarg = 'page'         # 검색할 페이지 번호에 대한 키워드 이름
>   * ordering = None             # 검색시 사용할 정렬방식. ORM의 `order_by`

그리고 템플릿을 수정해서 ctx의 값들을 간단하게 출력해봅니다.
```html
<!-- bbs/templates/base.html -->

<!DOCTYPE html>
<html lang="ko">
    <head>
        <title>base</title>
    </head>
    <body>
    view: {{ "{{ view " }}}} <!-- ctx['view'] -->
    <br>
    data: {{ "{{ data " }}}}  <!-- ctx['data'] -->
    </body>
</html>
```
템플릿에서 ctx의 값을 사용하는 방법은 간단합니다. ctx 객체에 저장된 데이터의 key 이름을 `{{ "{{ " }}}}` 안에 넣어주기만 합니다. 템플릿 엔진은 `{{ "{{ " }}}}`로 표시를 안에 있는 key에 하당하는 값으로 치환해줍니다.

![ArticleListView 결과]({{ site.url }}/snapshots/result_articlelist_01.png){:.border .rounded .shadow}

출력결과가 위와 같이 나온다면 성공입니다. `<QuerySet [<Article: [1] How to create a article>]>`는 검색된 `QuerySet`오브젝트이고 리스트 안에 보이는 객체들이 검색결과입니다.

이제 나머지 ArticleDetailView 와 ArticleCreateUpdateView의 화면요청에 대한 데이터 검색을 구현합니다.
```python
# bbs/views.py

class ArticleDetailView(TemplateView):
    template_name = 'base.html'
    queryset = Article.objects.all()
    pk_url_kwargs = 'article_id'                 # 검색데이터의 primary key를 전달받을 이름

    def get_object(self, queryset=None):
        queryset = queryset or self.queryset     # queryset 파라미터 초기화
        pk = self.kwargs.get(self.pk_url_kwargs) # pk는 모델에서 정의된 pk값, 즉 모델의 id
        return queryset.filter(pk=pk).first()    # pk로 검색된 데이터가 있다면 그 중 첫번째 데이터 없다면 None 반환

    def get(self, request, *args, **kwargs):
        article = self.get_object()
        if not article:
            raise Http404('invalid article_id')  # 검색된 데이터가 없다면 에러 발생

        ctx = {
            'view': self.__class__.__name__,
            'data': article
        }
        return self.render_to_response(ctx)


class ArticleCreateUpdateView(TemplateView):
    template_name = 'base.html'
    queryset = Article.objects.all()
    pk_url_kwargs = 'article_id'

    def get_object(self, queryset=None):
        queryset = queryset or self.queryset
        pk = self.kwargs.get(self.pk_url_kwargs)
        return queryset.filter(pk=pk).first()

    def get(self, request, *args, **kwargs):
        article = self.get_object()
        if not article:
            raise Http404('invalid article_id')

        ctx = {
            'view': self.__class__.__name__,
            'data': article
        }
        return self.render_to_response(ctx)

    def post(self, request, *args, **kwargs):
        ctx = {}
        return self.render_to_response(ctx)
```
`pk_url_kwargs` 클래스변수를 정의할 필요없이 urls.py에서 url의 `<article_id>`를 `<pk>`로 변환해줘도 상관없습니다. 보통은 `<pk>`을 사용하지만 다른 방법도 있다는 것을 알려드리고 싶어서 굳이 이렇게 ~~뻘짓~~full git을 하고 있습니다.

화면출력을 위한 데이터 검색은 이정도면 될 것 같습니다. 추가 작업은 더 필요할 때 구현하도록 합니다.

이제 액션을 구현하도록 합니다. 액션은 `create`, `update` 두 가지가 있는데 클라이언트(브라우저)로부터 데이터를 전달받아야 합니다. 전달된 데이터는 request.POST라는 `QueryDict`객체에 저장됩니다. 전달받을 데이터는 title(제목), content(내용), author(작성자) 이 세가지 입니다. 모두 필수로 입력받고 데이터가 없거나 문자열인 경우 오류를 발생시킵니다.
```python
# bbs/views.py

from django.utils.decorators import method_decorator
from django.views.decorators.csrf import csrf_exempt

# 생략

class ArticleCreateUpdateView(TemplateView):
    template_name = 'base.html'
    queryset = Article.objects.all()
    pk_url_kwargs = 'article_id'

    def get_object(self, queryset=None):
        queryset = queryset or self.queryset
        pk = self.kwargs.get(self.pk_url_kwargs)
        return queryset.filter(pk=pk).first()

    def get(self, request, *args, **kwargs):
        article = self.get_object()
        if not article:
            raise Http404('invalid article_id')

        ctx = {
            'view': self.__class__.__name__,
            'data': article
        }
        return self.render_to_response(ctx)

    def post(self, request, *args, **kwargs):
        action = request.POST.get('action')           # request.POST 객체에서 데이터 얻기
        post_data = {key: request.POST.get(key) for key in ('title', 'content', 'author')}
        for key in post_data:                         # 세가지 데이터 모두 있어야 통과
            if not post_data[key]:
                raise Http404('no data for {}'.format(key))

        if action == 'create':                        # action이 create일 경우
            article = Article.objects.create(title=title, content=content, title=author)
        elif action == 'update':                      # action이 update일 경우
            article = self.get_object()
            if not article:
                raise Http404('invalid article_id')

            for key, value in post_data.items():
                setattr(article, key, value)
            article.save()
        else:                                         # action이 없거나 create, update 중 하나가 아닐 경우
            raise Http404('invalid action')

        ctx = {
            'view': self.__class__.__name__,
            'data': article
        }
        return self.render_to_response(ctx)           # 액션 작업 후 화면을 보냄
```
http post의 경우 request.body 객체에 데이터 내용이 문자열 형태로 전달됩니다. 이 데이터가 딕셔너리로 변환이 가능할 경우 장고의 미들웨어가 자동으로 request.POST 객체에 변환된 값을 저장합니다. 변환된 값은 딕셔너리와 동일하게 읽을 수 있습니다. 하지만 immutable 객체이기 때문에 수정이 불가합니다.

실제 동작이 작동하는 지 curl또는 사용하시는 rest client로 테스트를 해봅니다.
```bash
(test-venv-36) $ curl -X POST http://127.0.0.1:8000/article/create/ -d "title='test title'&content='test content'"
```

![ArticleCreateUpdate 뷰의 create 결과]({{ site.url }}/snapshots/result_articlecreate_01.png){:.border .rounded .shadow}

### CSRF verification

`CSRF verification failed. Request.aborted.` 라는 오류메시지가 반환됐습니다. 장고의 여러 보안관련 기능 중 `CSRF verification` 이라는 것이 있는데, 모든 http post 요청은 장고에서 자동생성한 `csrftoken`을 body 데이터에 포함하고 있어야 합니다. `csrfmiddlewaretoken`이라는 값이 없을 경우 CSRF 공격으로 인식하고 미들웨어에서 에러를 발생시킵니다. 정상적인 템플릿으로 테스트하면 `csrfmiddlewaretoken`을 보낼 수 있지만 이번은 간단하게 뷰에서 **CSRF verification 기능을 예외처리**하고 테스트하겠습니다. 뷰 테스트가 종료되고 본격적으로 템플릿을 개발하기 전까지 예외처리해두겠습니다.

> curl 설치법
> * 실행환경이 윈도우라면 https://curl.haxx.se/download.html 에서 다운로드받으세요.
> * centos - yum install curl
> * ubuntu - sudo apt install curl
> * macos - 설치되어 있음.
> 
> postman 설치법
> 1. 크롬 실행
> 2. 메뉴 > 창 > 확장 프로그램
> 3. postman 검색 및 설치

CSRF verification을 예외처리할 때 계속 거슬리던 반복 코드를 정리하겠습니다. `get_objct` 메소드 호출 뒤에 항상 데이터가 None인지 아닌지를 확인하고 None일 경우 동일한 작업을 합니다. 차라리 `get_object`에서 `pk`가 `None`이 아닌데 결과값이 `None`인지 체크를 하도록 수정하겠습니다. `kwargs`에 `pk`가 있다는 것은 `update`를 의미하고, `pk`가 없다는 건 `create`를 의미합니다.
> 반복되는 코드들을 볼 때마다 가슴 한쪽에 고구마가 틀어막고 있음을 느낀다면 대단히 정상적입니다.

```python
# bbs/views.py

from django.utils.decorators import method_decorator
from django.views.decorators.csrf import csrf_exempt

#생략

@method_decorator(csrf_exempt, name='dispatch')   # 모든 핸들러 예외 처리
class ArticleCreateUpdateView(TemplateView):
    template_name = 'base.html'
    queryset = Article.objects.all()
    pk_url_kwargs = 'article_id'

    def get_object(self, queryset=None):
        queryset = queryset or self.queryset
        pk = self.kwargs.get(self.pk_url_kwargs)
        article = queryset.filter(pk=pk).first()
        
        if pk and not article:                    # 검색결과가 없으면 곧바로 에러 발생
            raise Http404('invalid pk')
        return article

    def get(self, request, *args, **kwargs):
        article = self.get_object()

        ctx = {
            'view': self.__class__.__name__,
            'data': article
        }
        return self.render_to_response(ctx)

    def post(self, request, *args, **kwargs):
        action = request.POST.get('action')
        post_data = {key: request.POST.get(key) for key in ('title', 'content', 'author')}
        for key in post_data:
            if not post_data[key]:
                raise Http404('no data for {}'.format(key))

        if action == 'create':
            article = Article.objects.create(title=title, title=content, title=author)
        elif action == 'update':
            article = self.get_object()
            for key, value in post_data.items():
                setattr(article, key, value)
            article.save()
        else:
            raise Http404('invalid action')

        ctx = {
            'view': self.__class__.__name__,
            'data': article
        }
        return self.render_to_response(ctx)
```
모든 제네릭 뷰에는 dispatch라는 메소드가 내장되어 있습니다. 이 메소드에서 get, post 핸들러로 분기시켜주는 역할을 합니다. 이 dispatch 함수에 csrf_exempt 데코레이터로 예외처리를 해주면 되는데, 그러면 dispatch함수를 한번 더 오버라이딩해줘야 하는 수고가 있습니다. 수고로움을 견디지 못하는 탓에 method_decorator 데코레이터를 사용했습니다. 클래스에서 오버라이딩하지 않은 메소드에 데코레이팅 할 때 편리하게 사용할 수 있습니다.

여기까지 이해하셨다면 이제 템플릿만 남았습니다. 모델과 뷰를 개발하면서 모델은 이런 걸 하고, 뷰는 저런 걸 하는구나 하는 감이 와야 합니다. ~~첨부터 한번 더 본다고 돈 들지 않습니다.~~ 그렇지 않다면 저의 탓입니다. 못난 독자를 둔 저의 탓으로 넘기고 가벼운 마음으로 다음 페이지로 넘어가십시오.

> 모델과 뷰와 책임감을 수반하는 고통은 또한 템플릿을 주기도 한다.
>
> -- swarf00, ~~못난~~ 불쌍한 독자들을 생각하며...