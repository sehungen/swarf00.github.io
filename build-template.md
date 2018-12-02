---
layout: article
title: 템플릿 만들기
key: django-build-template
aside:
  toc: true
sidebar:
  nav: docs-ko
pageview: true
tags: 장고 Django 템플릿
metadata:
  og_title: 장고(Django) 템플릿 만들기
  og_type: article
  og_locale: 'ko_KR'
  og_description: 장고(Django) 웹프레임워크의 템플릿태그와 템플릿변수를 활용한 템플릿을 작성하는 방법을 설명합니다.
  og_site_name: 장고(Django) 핥짝 맛보기
---

## 1. 템플릿 설계
템플릿도 설계가 필요합니다. 많이 봐오신 웹사이트들을 보면 웹사이트는 대부분의 페이지들이 항상 일정한 헤드, 메뉴, 푸터 등을 표시하는 것을 볼 수 있습니다. 현재 이 웹사이트도 헤드(상단)과 사이드바(좌측), 푸터 등이 항상 일정하게 나타나고 있습니다. 단지 현재의 페이지에 따라서 특정 텍스트들이 강조되어 있습니다. 현재 사이드바의 Template 만들기가 강조되어 있습니다. **템플릿을 기능별로 구분한다면 재활용성이 높고, 개발할 때 단순함을 더할 수 있습니다.**

게시판의 모든 화면은 크게 두가지로 나눌 것 입니다. **기본구조와 실제 화면내용**으로 구분됩니다. `기본구조`는 html의 공통적인 head와 body에서 화면내용이 삽입될 `틀`입니다. `화면내용`은 뷰마다 제공하는 데이터를 사용자가 알아볼 수 있게 표현하는 부분입니다. 말로만 하면 설명을 이해하기 어려우니 일단 따라합니다.
```html
<!-- bbs/templates/base.html -->

{% raw %}
<!DOCTYPE html>
<html lang="ko">
    <head>
    
        {% block title %}      <!-- 페이지별 타이틀 공간 -->
        <title>bbs - minitutorial</title>
        {% endblock title %}

        {% block meta %}    <!-- 페이지별 메타 데이터 공간 -->
        {% endblock meta %}

        {% block scripts %}    <!-- 페이지별 스크립트 공간 -->
        {% endblock scripts %}

        {% block css %}        <!-- 페이지별 css -->
        {% endblock css %}
    </head>
    <body>
    {% block content %}
    view: {{ view }} <!-- ctx['view'] -->
    <br>
    data: {{ data }}  <!-- ctx['data'] -->
    {% endblock %}
    </body>
</html>
{% endraw %}
```

### 템플릿 태그와 템플릿 변수
우선 기본구조를 먼저 생성합니다. 템플릿은 단순한 텍스트(MarkUp) 파일입니다. 템플릿엔진은 내부가 html인지 csv인지 뭔지 아무런 관심도 없고 ~~여러분처럼~~ 이해하지도 못합니다. 템플릿엔진이 알아볼 수 있는건 딱 두가지 템플릿태그와 템플릿변수 입니다. 

각 block 태그들은 페이지마다 끼워넣거나 대체해넣을 수 있는 공간입니다. 해당 블록에 **`title block` 처럼 block의 시작과 종료(endblock)사이에 값(문자열)을 넣으면 해당하는 값의 기본값이 설정**됩니다. 어떤 템플릿이든 이 템플릿(base.html)을 상속받으면 해당 데이터를 덮어쓰거나 추가할 수 있습니다.

> 템플릿태그와 템플릿 변수
> * 템플릿태그는 for-in 반복문, if-elif-else 조건문 등의 템플릿 엔진이 이해하는 몇가지 기능들을 수행합니다. {% raw %}{% %}{% endraw %} 으로 표현하고 for-in, if-elif-else 처럼 처리해야 할 텍스트가 2줄이상이 될 수 있는 경우 여는 태그({% raw %}{% %}{% endraw %})와 닫는 태그({% raw %}{% %}{% endraw %})로 이루어집니다. 여는 태그와 닫는 태그 사이의 텍스트를 제어하는 것입니다. for-in 태그는 endfor 태그로 반드시 종료가 되어야 합니다.
> * 템플릿변수는 뷰로부터 전달받은 객체의 값을 인용할 때 사용합니다. {% raw %}{{ }}{% endraw %}로 표현하며 표현할 변수의 값이 딕셔너리일 경우에도 getattr 연산자(.)으로 key에 접근할 수 있습니다. 리스트나 튜플일 때도 인덱스를 대괄호가 아니라 .으로 접근할 수 있습니다. 템플릿엔진은 일부 문법에 대해 파이썬 문법보다 좀더 유연함을 제공합니다.

템플릿은 수정을 해도 장고를 재시작할 필요가 없습니다. 이것은 매 요청 때마다 템플릿 렌더링을 한다는 것을 의미하고, 성능적으로 그리 좋지 않다는 것을 의미합니다. ~~여기선 안 가르져주니~~나중에 캐시도 공부해보시기 바랍니다.

이 상태에서 뷰 테스트했을 때와 같이 접속해보면 동일한 결과를 볼 수 있습니다.
* http://127.0.0.1/article/
* http://127.0.0.1/article/create/
* http://127.0.0.1/article/10/
* http://127.0.0.1/article/11/update/

## 2. 리스트 템플릿 구현
### 템플릿의 상속
정상적으로 출력이 된다면 각 화면별로 템플릿을 작성합니다. 반드시 base.html 템플릿을 상속받도록 구현하는 것이 좋습니다. 그렇게 해야 중복된 코드를 줄이고 실수로 공통 코드를 빠뜨리지 않을 수 있습니다.
```html
<!-- bbs/templates/article_list.html -->

{% raw %}
{% extends 'base.html' %}

{% block title %}<title>게시글 목록</title>{% endblock %}

{% block content %}
view: {{ view }}
<br>
data: {{ data }}
{% endblock %}
{% endraw %}
```
article_list.html 템플릿은 extends의 인자인 'base.html' 파일을 상속받습니다. **템플릿에서 상속이란 기본 뼈대를 부모 템플릿으로 두고 각 block을 오버라이드 한다는 의미**입니다. python의 클래스 상속과 마찬가지로 부모템플릿의 내용을 인용하고 싶다면 {% raw %}`{{ block.super }}`{% endraw %} 변수를 사용하면 됩니다. 

ArticleListView의 템플릿이 base.html에서 article_list.html로 변경되었으니 뷰의 template_name 클래스변수를 변경합니다.
```python
# bbs/views.py

class ArticleListView(TemplateView):
    template_name = 'article_list.html'    # 뷰 전용 템플릿 생성.
    queryset = Article.objects.all()

    def get(self, request, *args, **kwargs):
        print(request.GET)
        ctx = {
            'view': self.__class__.__name__,
            'data': self.queryset
        }
        return self.render_to_response(ctx)
```
> 템플릿 파일명을 article_list.html로 지은 이유가 있습니다. 모델을 기반으로 하는 ListView나 DetailView등은 클래스 변수 model을 정의할 경우 자동으로 모델명(소문자) + '_list.html' 또는 '_detail.html'로 템플릿파일을 자동으로 생성합니다. 파일명을 이런식으로 작명한다면 나중에 더 복잡한 제네릭뷰를 사용할 때 편리하고 오류를 줄일 수 있습니다.~~뿌듯해. 간만에 꿀팁 지렸다.^^~~

### 템플릿 반복문

리스트 템플릿은 0개 이상의 데이터를 표현해야 합니다. 0개일 경우를 따로 구현하진 않을 예정이지만 1개이상인 경우 테이블형태로 표현되도록 할 예정입니다. 카드형태로 ~~구현하는 것은 숙제입니다.~~구현하실 분들은 따로 해보시기 바랍니다.
```html
<!-- bbs/templates/article_list.html -->
{% raw %}
{% extends 'base.html' %}

{% block title %}<title>게시글 목록</title>{% endblock %}

{% block content %}
<table>
    <thead>
        <th>게시글번호</th><th>제목</th><th>작성자</th>
    </thead>
    <tbody>
        <tr>
            <td>3</td><td>제목3</td><td>작성자</td>
        </tr>
        <tr>
            <td>2</td><td>제목2</td><td>작성자</td>
        </tr>
        <tr>
            <td>1</td><td>제목1</td><td>작성자</td>
        </tr>
    </tbody>
</table>
{% endblock %}
{% endraw %}
```
이런식으로 테이블로 표현할 것입니다. 아직은 실제 데이터를 넣지 않고 가짜데이터로 틀만 만들어봤습니다. 우선 `table` 태그를 간단히 살펴보겠습니다. 들여쓰기로 구분하여 보시면 좋습니다.

`thead`와 `tbody`로 나뉩니다. `thead`는 칼럼의 제목들이 표시될 것입니다. `thread` 안에는 `th`태그들이 있는데 각 태그들은 칼럼들의 제목이 표시됩니다.
`tbody`는 여러개의 `tr`로 구성됩니다. `tr`은 데이터의 갯수만큼 출력됩니다. 위의 코드에서는 3개의 `tr`이 있는 걸 보니 데이터의 총갯수가 3개입니다. 각 `tr`태그에는 각각 3개의 `td`태그들이 있습니다. 이 태그들 안에 데이터의 적절한 속성값을 넣어주면 됩니다.

위 예를 보면 `tr`태그 단위로 하나의 데이터라는 것을 알 수 있습니다. 즉 **`for 루프`가 한 사이클이 돌 때마다 `tr`태그를 생성**해 주면 됩니다. 물론 `tr` 태그 내부에 있는 `td`의 데이터도 채워서 생성해야 합니다.
```html
<!-- bbs/templates/article_list.html -->

{% raw %}
{% extends 'base.html' %}

{% block title %}<title>게시글 목록</title>{% endblock %}

{% block content %}
<table>
    <thead>
        <th>번호</th><th>제목</th><th>작성자</th>
    </thead>
    <tbody>
        {% for article in articles %}         <! -- for tag 시작 -->
        <tr>
            <td>3</td><td>제목3</td><td>작성자</td>
        </tr>
        {% endfor %}                          <! -- for tag 종료 -->
    </tbody>
</table>
{% endblock %}
{% endraw %}
```
{% raw %}
한번 더 가짜데이터로 출력을 해봤습니다. 이번에 달라진 점은 for 태그가 사용되었다는 것입니다. python의 for-in 루프와 닮았습니다. **다른 점은 {% %}로 감싸 있다는 것이고 {% endfor %}로 for-in루프의 블럭의 끝을 표시**했다는 것입니다. for 태그는 반복문을 실행하되 {% for ~ in ~ %} 에서부터 {% endfor %} 사이의 텍스트를 출력해줍니다. 
{% endraw %}

실제로 테스트하기 위해 뷰의 ctx를 수정합니다.
```python
class ArticleListView(TemplateView):
    template_name = 'article_list.html'
    queryset = Article.objects.all()

    def get(self, request, *args, **kwargs):
        print(request.GET)
        ctx = {
            'articles': self.queryset
        }
        return self.render_to_response(ctx)
```
템플릿의 `title` 태그로 페이지 제목을 알 수 있으니 ctx의 view 값을 제거했습니다. `data`라는 이름으로 모호했던 이름을 템플릿에서 사용하는 `articles`로 변경합니다. 현재 저의 데이터베이스에는 2개의 레코드가 저장된 상태라서 `articles.count()`는 2입니다.

![ArticleList for 태그 적용]({{ site.url }}/snapshots/result_articlelist_02.png){:.border .rounded .shadow}

게시글이 총 2개이기 때문에 `tr` 태그가 2번 반복해서 출력이 되었습니다. 그럼 마지막으로 `td`의 값을 실제 값으로 채워넣으면 됩니다.
```html
<!-- bbs/templates/article_list.html -->

{% raw %}
{% extends 'base.html' %}

{% block title %}<title>게시글 목록</title>{% endblock %}

{% block content %}
<table>
    <thead>
        <th>번호</th><th>제목</th><th>작성자</th>
    </thead>
    <tbody>
        {% for article in articles %}
        <tr>
            <td>{{ article.pk }}</td><td>{{ article.title }}</td><td>{{ article.author }}</td>
        </tr>
        {% endfor %}
    </tbody>
</table>
{% endblock %}
{% endraw %}
```

템플릿 변수를 사용하면 특정 값으로 치환을 할 수 있습니다. `for 루프`에서 선언한 변수 article의 값을 템플릿변수에서 접근하는데 각각 `pk`, `title`, `author` 속성값으로 치환합니다. `pk`는 전에 설명한 대로 `primarykey`로 설정된 값 즉, id가 반환됩니다.

![ArticleList 템플릿변수 적용]({{ site.url }}/snapshots/result_articlelist_03.png){:.border .rounded .shadow}

[bootstrap](http://bootstrapk.com/css/#tables)을 이용해서 디자인을 좀 입혀보겠습니다. [bootstrap](http://bootstrapk.com/)의 한글 메뉴얼도 있으니 참고해보시면 더 좋은 기능들을 확인할 수 있습니다. 

```html
<!-- bbs/templates/article_list.html -->

{% raw %}
{% extends 'base.html' %}

{% block title %}<title>게시글 목록</title>{% endblock %}

{% block css %}                                    <!-- bootstrap CSS -->
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
{% endblock css %}

{% block content %}
<table class="table table-hover table-responsive"> <!-- hover, responsive -->
    <thead>
        <th>번호</th><th>제목</th><th>작성자</th>
    </thead>
    <tbody>
        {% for article in articles %}
        <tr>
            <td>{{ article.pk }}</td><td>{{ article.title }}</td><td>{{ article.author }}</td>
        </tr>
        {% endfor %}
    </tbody>
</table>
{% endblock %}
{% endraw %}
```

부트스트랩의 자바스크립트는 아직까지 사용할 일이 없으니 추가하지 않습니다. 부트스트랩의 table은 css파일만 가져오면 됩니다. 공개 cdn으로부터 무료로 다운로드해서 사용하실 수 있습니다. `table` 태그에 `.table` `.table-hover` `.table-responsive` 클래스를 추가합니다. .table은 부트스트랩의 테이블 디자인을 사용한다는 의미이고, `.table-hover`는 각 줄(`tr`)에 마우스를 올리면 색깔이 강조되도록 해주는 디자인입니다. `.table-responsive`는 모바일처럼 폭이 좁은 화면에서도 깨짐없이 보일 수 있게 해주는 기능입니다. 현재까지는 모바일에서 부자연스럽지 않지만 제목이 굉장히 길거나 작성자 이름이 굉장히 길 경우 또는 칼럼수가 늘어날 경우 좀 더 모바일 친화적으로 보여지게 됩니다.

### 다른 페이지로 링크
상세페이지와 새게시글 작성 페이지로 이동하는 링크를 추가하면 일단 완료됩니다.
```html
<!-- bbs/templates/article_list.html -->

{% raw %}
{% extends 'base.html' %}

{% block title %}<title>게시글 목록</title>{% endblock %}

{% block css %}
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
<style type="text/css">
    tbody > tr {cursor: pointer;}
</style>
{% endblock css %}

{% block content %}
<table class="table table-hover table-responsive">
    <thead>
        <th>번호</th><th>제목</th><th>작성자</th>
    </thead>
    <tbody>
        {% for article in articles %}
        <tr onclick="location.href='/article/{{ article.pk }}/'">  <!-- 테이블 행 click 시 url 이동 -->
            <td>{{ article.pk }}</td><td>{{ article.title }}</td><td>{{ article.author }}</td>
        </tr>
        {% endfor %}
    </tbody>
</table>

<!-- 버튼 click 시 url 이동 -->
<a href="/article/create/"><button class="btn btn-primary" type="button">새 게시글 작성</button></a>
{% endblock %}
{% endraw %}
```

![ArticleList 최종화면]({{ site.url }}/snapshots/result_articlelist_04.png){:.border .rounded .shadow}

테이블의 행을 클릭하면 그 행의 `pk`를 따라 이동하도록 했습니다. `tr`, `td` 태그에서는 `a` 태그가 적용되지 않아서 `tr`태그에 `onclick` 이벤트를 등록했습니다. 태그의 `onclick` 속성값을 정의하면 해당 태그를 클릭했을 때 정의된 값이 실행됩니다.
새 게시글 작성 버튼은 무조건 `/article/create/`로 이동하도록 했습니다. 게시글 목록 템플릿의 디자인은 여기서 멈춥니다.~~더 이상 설명하면 디자이너들 밥줄 끊깁니다. 갑자기 뭐래...~~

## 3. 게시물 상세보기 템플릿 구현
게시물 상세보기는 게시물 목록 화면보다 간단합니다. 모든 내용을 다 출력해주면 됩니다. 별 내용이 없으니 아예 수정하기 버튼까지 만들면 좋겠습니다.
```html
<!-- bbs/templates/article_detail.html -->

{% raw %}
{% extends 'base.html' %}

{% block title %}<title>게시글 상세 - {{ article.pk }}. {{ article.title }}</title>{% endblock %}

{% block css %}
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
{% endblock css %}

{% block content %}
<table class="table table-striped table-bordered">
	<tr>
		<th>번호</th>
		<td>{{ article.pk }}</td>
	</tr>
	<tr>
		<th>제목</th>
		<td>{{ article.title }}</td>
	</tr>
	<tr>
		<th>내용</th>
		<td>{{ article.content }}</td>
	</tr>
    <tr>
		<th>작성자</th>
		<td>{{ article.author }}</td>
	</tr>
    <tr>
		<th>작성자</th>
		<td>{{ article.created_at }}</td>
	</tr>
</table>

<a href="/article/{{ article.pk }}/update/"><button class="btn btn-primary" type="button">게시글 수정</button></a>
{% endblock %}
{% endraw %}
```
게시글 목록 화면과 같이 뷰도 수정해줍니다.
```python
# bbs/views.py

class ArticleDetailView(TemplateView):
    template_name = 'article_detail.html'
    queryset = Article.objects.all()
    pk_url_kwargs = 'article_id'

    def get_object(self, queryset=None):
        queryset = queryset or self.queryset
        pk = self.kwargs.get(self.pk_url_kwargs)
        article = queryset.filter(pk=pk).first()

        if not article:
            raise Http404('invalid pk')
        return article

    def get(self, request, *args, **kwargs):
        article = self.get_object()

        ctx = {
            'article': article
        }
        return self.render_to_response(ctx)
```
게시글 목록과는 다르게 오직 하나의 데이터만 템플릿에 전달하기 때문에 게시글 객체 이름을 article이라고 정의했습니다. table은 리스트와는 다르게 한 행에 한 속성씩 출력했습니다. 테이블에 마우스 클릭이 필요없으니 hover효과를 빼고 가시성을 높이는 `.table-striped`와 `.table-bordered` 클래스를 추가했습니다.

게시글 수정 버튼을 클릭하면 해당 게시물의 업데이트 화면으로 이동할 수 있게 추가했습니다.

게시글 목록에서 아무 행이나 클릭해서 상세페이지로 이동해 봅니다. 제대로 ~~복붙~~작성했다면 테이블이 나타날 겁니다.

![ArticleDetail 템플릿 구현 후]({{ site.url }}/snapshots/result_articledetail_01.png){:.border .rounded .shadow}

### 템플릿 필터
정상적으로 출력된 듯 합니다. ~~구라고~~ 유심히 살펴보면 두 가지가 불편해 보입니다.
1. 내용의 데이터가 한줄로 출력되었는데, 제가 입력할 때는 줄바꿈이 있었습니다. 즉 `'\n'` 문자가 html에서는 적용이 안됩니다.
2. 작성일이 일반 한국사람이 보기에 적합하지 않습니다. 제가 익숙한 `yyyy-mm-dd HH:MM` 형식으로 출력되면 좋겠습니다.

이럴 줄 알았다는 듯이 장고는 필터라는 기능을 제공하고 있습니다. 이미 장고에서 제공하는 수많은 필터가 있으니 알맞게 가져다 사용만 하면 됩니다. 필터는 템플릿변수 안에서 파이프(`|`)로 연결하여 값을 변경하는 함수를 말합니다. `linebreaksbr`이라는 필터를 사용하면 필터링할 문자열에서 모든 `\n`문자를 `<br>`태그로 변환해주는 기능을 합니다.

날짜시간의 포멧을 변경하는 필터도 역시 존재합니다. `date`이라는 필터인데 이 필터는 인자를 넘겨줄 수도 있습니다. 인자를 넘겨주지 않으면 기본포멧으로 출력되는데 원하는데로 나온다는 보장이 없습니다. PHP의 시간포멧과 비슷한데 `"Y-m-d H:i"`이라고 인자를 넘겨주면 익숙한 형태의 날짜와 시간이 출력됩니다.

```html
<!-- bbs/templates/article_detail.html -->

{% raw %}
{% extends 'base.html' %}

{% block title %}<title>게시글 상세 - {{ article.pk }}. {{ article.title }}</title>{% endblock %}

{% block css %}
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
{% endblock css %}

{% block content %}
<table class="table table-striped table-bordered">
	<tr>
		<th>번호</th>
		<td>{{ article.pk }}</td>
	</tr>
	<tr>
		<th>제목</th>
		<td>{{ article.title }}</td>
	</tr>
	<tr>
		<th>내용</th>
		<td>{{ article.content | linebreaksbr }}</td>
	</tr>
    <tr>
		<th>작성자</th>
		<td>{{ article.author }}</td>
	</tr>
    <tr>
		<th>작성일</th>
		<td>{{ article.created_at | date:"Y-m-d H:i" }}</td>
	</tr>
</table>

<a href="/article/{{ article.pk }}/update/"><button class="btn btn-primary" type="button">게시글 수정</button></a>
{% endblock %}
{% endraw %}
```

> 장고에 내장된 필터가 수십여가지가 있는데 많은 것들이 알아두면 좋습니다. ~~저도 다는 몰라서~~ 모두 설명할 수 없으니 [참고 문서](https://docs.djangoproject.com/en/2.1/ref/templates/builtins/)를 꼭 한번 정독하시기 바랍니다.

## 4. 게시물 업데이트 템플릿 구현
게시물 상세화면을 조금 수정해서 템플릿을 구현할 것입니다. 게시물 업데이트 화면은 번호와 작성일은 수정할 수 없고 제목, 내용, 작성자만 변경할 수 있게 할 것입니다. 각 항목들은 서버에 저장되어 있는 값들을 기본값으로 채워넣은 상태로 보여줍니다. 그래야 사용자가 변경하기가 쉬울테니까요. 
```html
<!-- bbs/templates/article_update.html -->

{% raw %}
{% extends 'base.html' %}

{% block title %}<title>게시글 상세 - {{ article.pk }}. {{ article.title }}</title>{% endblock %}

{% block css %}
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
{% endblock css %}

{% block content %}
                                                     <!-- form -->
<form action="/article/{{ article.pk }}/update/" method="post" class="form-horizontal">
{% csrf_token %}                                     <!-- csrftoken 태그 -->
<input type="hidden" name="action" value="update">   <!-- action -->
<table class="table table-striped table-bordered">
	<tr>
		<th>번호</th>
		<td>{{ article.pk }}</td>
	</tr>
	<tr>
		<th>제목</th>                                     <!-- 제목 입력 -->
		<td><input type="text" class="form-control" name="title" value="{{ article.title }}"></td>
	</tr>
	<tr>
		<th>내용</th>                                     <!-- 내용 입력 -->
		<td><textarea rows="10" class="form-control" name="content">{{ article.content }}</textarea></td>
	</tr>
    <tr>
		<th>작성자</th>                                    <!-- 작성자 입력 -->
		<td><input type="text" class="form-control" name="author" value="{{ article.author }}"></td>
	</tr>
    <tr>
		<th>작성일</th>
		<td>{{ article.created_at | date:"Y-m-d H:i" }}</td>
	</tr>
</table>

<button class="btn btn-primary" type="submit">게시글 저장</button>
</form>
{% endblock %}
{% endraw %}
```
테이블과 버튼을 form 태그로 감싸서 테이블 안의 input, textarea의 데이터들을 전송할 수 있게 했습니다. form 태그부터 속성을 살펴보면, action은 update 액선의 url을 지정했고, method는 post를 지정했습니다. class를 form-horizontal로 지정했는데 html 엘리먼트가 수평으로 잘 정리되도록 하는 bootstrap 속성입니다.

그 아래를 보면 csrf_token 이라는 태그가 있습니다. 이 태그는 `<input type="hidden" name="csrfmiddlewaretoken" value="kjxqvcTIDJ......w2RT9HMHhdF">` 식으로 자동으로 태그를 만들어 줍니다. 장고는 이 post 요청은 csrfmiddlewaretoken이라는 값이 있어야만 정상적인 요청으로 인식합니다. 템플릿엔진은 csrf_token 태그를 만나면 자동으로 csrf verification 프레임워크에서 생성한 csrfmiddlewaretoken의 값을 이용해서 html 태그로 변환해줍니다. 그리고 post요청했을 때 장고의 미들웨어에서 이 값을 검증하고 비정상일 경우 오류를 반환합니다. 템플릿에 {% raw %}`{% csrf_token %}`{% endraw %} 토큰만 넣으면 된다는 것!! 너무 간단해서 잊어버릴 수 있습니다.

그 아래에는 `hidden` 타입의 `input` 태그를 추가했습니다. `action`이라는 이름에 `update`라는 값을 지정했습니다. `hidden` 타입은 사용자에게는 보이지 않는 `input` 태그입니다. action이라는 값을 사용자에게 보여주지도 않고 볼 수 없으니 변경할 수도 없도록 한 것입니다. 물론 사용자가 html에 대한 지식이 있다면 action 값을 확인하거나 수정할 수도 있겠지만 딱히 문제될 내용이 아니기 때문에 hidden타입으로 두는 것으로도 문제가 없습니다.

입력받을 3가지 값이 있는데 1줄로 입력받아도 된다면 input, 2줄 이상으로 입력받아야 한다면 `textarea`를 사용합니다. **`textarea`는 엔터키를 줄바꿈으로 인식**합니다. textarea는 데이터가 매우 길 수 있기 때문에 value라는 속성 대신 여는 태그와 닫는 태그 사이에 작성하도록 되어 있습니다.

마지막으로 버튼은 둘러싸고 있던 a 태그를 제거하고 type을 submit으로 변경했습니다. submit 타입의 버튼은 클릭시 해당 버튼을 둘러싸고 있는 가장 가까운 폼을 서버에 전송합니다. 물론 form 태그 내부에 있는 모든 데이터들을 가지고 전송이 됩니다. 각 태그들의 이름이 key가 되고 value가 값이 되어 서버에 전송이 됩니다. `'key1=value1&key2=value2'` 형태로 전송됩니다. 이렇게 전달된 문자열 데이터는 장고의 미들웨어에서 자동으로 딕셔너리로 변환 후 request.POST 객체에 저장이 됩니다.

뷰의 기능은 이미 구현해 둔 상태인데 이전에 추가했던 데코레이터를 삭제해기만 하면 됩니다. **csrf_exempt 데코레이터는 테스트용으로만 사용**하고 가급적 사용하지 않아야 합니다.

> csrf verification은 이용자가 원치 않는 post요청을 하는 것을 막기 위한 보안 프레임워크입니다. csrf 공격방법은 [나무위키](https://namu.wiki/w/CSRF)에 잘 정리되어 있으니 참고해서 이해하는 것이 좋습니다. 장고의 보안 프레임워크는 만능이 아니고 날이 갈수록 진보된 공격방식이 개발되기 때문에 공격 매카니즘을 이해하는 것이 필요합니다.

![ArticleUpdate 템플릿 구현 후]({{ site.url }}/snapshots/result_articleupdate_01.png){:.border .rounded .shadow}

정상적으로 출력이 됩니다. 그런데 내용을 수정하고 게시글 수정 버튼을 눌렀는데 살짝 깜박임이 보이지만 저장이 잘 되었는 지 의심이 됩니다. 여러번 반복해도 내 눈을 의심하게 될 뿐 저장되었다는 확신이 없습니다. 에러가 발생한 경우도 그냥 에러화면으로 이동이 되는 것이 마음에 걸렸는데 이번 기회에 메시지창을 만들어서 사용자요청이 어떻게 처리되었는 지 알려주는 것이 좋을 것 같습니다.

장고의 `messages` 프레임워크를 사용할 때가 왔습니다. `messages` 프레임워크는 아무 때나 메시지 내용을 기록하면 템플릿에서 데이터를 출력할 때까지 임시로 데이터를 저장해두는 프레임워크입니다. **로그처럼 레벨이 있기 때문에** 오류인지, 단순한 정보인지 구분하기가 좋습니다.

먼저 뷰에서 `messages` 프레임워크에 메시지를 입력하도록 수정합니다.
```python
# bbs/views.py

from django.contrib import messages

# 생략

class ArticleCreateUpdateView(TemplateView):
    template_name = 'article_update.html'
    queryset = Article.objects.all()
    pk_url_kwargs = 'article_id'
    success_message = '게시글이 저장되었습니다.'

    def get_object(self, queryset=None):
        queryset = queryset or self.queryset
        pk = self.kwargs.get(self.pk_url_kwargs)
        article = queryset.filter(pk=pk).first()

        if pk and not article:
            raise Http404('invalid pk')
        return article

    def get(self, request, *args, **kwargs):
        article = self.get_object()

        ctx = {
            'article': article
        }
        return self.render_to_response(ctx)

    def post(self, request, *args, **kwargs):
        action = request.POST.get('action')
        post_data = {key: request.POST.get(key) for key in ('title', 'content', 'author')}
        for key in post_data:
            if not post_data[key]:
                messages.error(self.request, '{} 값이 존재하지 않습니다.'.format(key), extra_tags='danger') # error 레벨로 메시지 저장

        if len(messages.get_messages(request)) == 0:                  # 메시지가 있다면 아무것도 처리하지 않음
            if action == 'create':
                article = Article.objects.create(**post_data)
                messages.success(self.request, self.success_message)  # success 레벨로 메시지 저장
            elif action == 'update':
                article = self.get_object()
                for key, value in post_data.items():
                    setattr(article, key, value)
                article.save()
                messages.success(self.request, self.success_message)  # success 레벨로 메시지 저장
            else:
                messages.error(self.request, '알 수 없는 요청입니다.', extra_tags='danger')     # error 레벨로 메시지 저장
    
        ctx = {
            'article': self.get_object() if action == 'update' else None
        }
        return self.render_to_response(ctx)
```
messages 프레임워크는 뷰에서 사용하는 방법이 간단합니다. messsages 모듈의 `debug`, `info`, `success`, `warning`, `error` 5가지 함수 중 하나를 선택해서 request 객체와 저장할 메시지를 전달하면 됩니다. 성공시 success, 오류시 error 함수를 호출했습니다. 템플릿에서 level에 따라 구분되게 표시할 수 있습니다.

`messages.get_messages(request)` 함수는 현재까지 저장된 메시지들을 반환합니다. 저장된 메시지들이 1개 이상이라면 현재의 코드에서는 반드시 오류가 발생했다는 것이기 때문에 액션로직을 실행하지 않도록 했습니다. article 변수는 액션로직 안에서 정의하기 때문에 만약 오류가 발생하면 action이 `'update'`인 경우 article을 검색해오고 `'create'`인 경우는 None을 저장하도록 했습니다. **if ~ else 문법을 이용해서 3항 연산자처럼 표현한 식**을 익혀두시면 유용하게 사용하실 수 있습니다.

> log레벨과 사용법이 비슷합니다. 몇가지 레벨이 없지만 웹프레임워크로서 충분합니다. 강제적인 규칙은 아니지만 로그레벨은 그 의미대로 사용하는 것이 좋습니다. 결국 사람이 보기 위한 것이기 때문에 반드시 레벨로 메시지의 속성을 표현해셔야 합니다.
> 
> 그 외 `커스텀 레벨`과 `extra_tags`를 이용하는 방법이 있는데 [장고문서](https://docs.djangoproject.com/ko/2.1/ref/contrib/messages/#creating-custom-message-levels)가 어렵지 않으니 간단히 살펴보면 쉽게 따라하실 수 있습니다.

장고에서 템플릿으로 `messages`라는 객체로 저장된 메시지들이 전달됩니다. 뷰에서 context로 전달하지 않아도 됩니다.

```html
<!-- bbs/templates/article_update.html -->

{% raw %}
{% extends 'base.html' %}

{% block title %}<title>게시글 상세 - {{ article.pk }}. {{ article.title }}</title>{% endblock %}

{% block css %}
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
{% endblock css %}

{% block content %}

{% if messages %}                            <!-- message 프레임워크 -->
{% for message in messages %}
<div class="alert alert-{{ message.tags }} alert-dismissible" role="alert">
  {{ message }}
</div>
{% endfor %}
{% endif %}
  
<!-- 생략 -->

{% endblock %}
{% endraw %}
```
if 템플릿 태그로 `messages` 객체가 있는지 확인합니다. `if` 태그는 반드시 `endif` 태그로 종료되어야 한다는 것 주의하셔야 합니다. `messages` 객체는 `iterable` 객체이기 때문에 `for-in` 루프로 반복출력해야 합니다. `for-in` 루프처럼 `iterate`를 진행해야 메시지가 사용된 것으로 변경됩니다. message 그 자체를 출력해도 되고 message.tags 또는 message.level를 이용하셔도 됩니다. message.level은 message를 저장할 때 사용하는 그 레벨이 출력이 되고, **`tags`는 `extra_tags`와 message.level의 조합**입니다. 이 예제에서는 message.tags를 이용했는데 extra_tags를 전달하지 않았기 때문에 level값만 출력이 됩니다. bootstrap의 alert class 를 사용하면 쉽게 강조표시를 할 수 있습니다. `alert-success`, `alert-info`, `alert-warning`, `alert-danger` 등에 따라 색상이 달라지기 때문에 message의 레벨을 적절히 조합하면 손쉽게 일관성있는 강조 표시를 할 수 있습니다. messages 에는 `danger`라는 레벨이 없기 때문에 `error`라는 레벨의 함수에는 `extra_tags`를 이용해서 `error`를 추가해줬습니다. 그러면 message.tags 는 `'danger error'`를 출력합니다.

![ArticleUpdate 메시지 추가]({{ site.url }}/snapshots/result_articleupdate_02.png){:.border .rounded .shadow}

### 부트스트랩 네비게이션바
업데이트까지 정상적으로 되는 것이 확인되었습니다. 이제 마지막으로 게시글 생성 기능을 구현하면 되는데 목록보기로 바로 갈 수 있는 버튼이 없어 불편합니다. 브라우저의 뒤로가기 기능을 사용해도 되지만 여러번 수정했을 경우 여러번 뒤로 가야 하는 문제가 있습니다. 상단의 네비게이션바가 없어서 허전했었는데 네비게이션바에 홈버튼을 추가하면 좀 편리할 것 같습니다.
```html
<!-- bbs/templates/base.html -->

{% raw %}
<!DOCTYPE html>
<html lang="ko">
    <head>
        {% block title %}
        <title>bbs - minitutorial</title>
        {% endblock title %}

        {% block meta %}
        {% endblock meta %}

        {% block scripts %}
        {% endblock scripts %}

        {% block css %}
        {% endblock css %}
    </head>
    <body>
    {% block header %}
    <nav class="navbar navbar-default">
        <div class="container-fluid">
            <div class="navbar-header">
                <a class="navbar-brand" href="/article/">게시글 목록</a>
            </div>
        </div>
    </nav>
    {% endblock%}
    
    {% block content %}
    {% endblock %}
    </body>
</html>
{% endraw %}
```
공통적으로 표시되어야 할 부분이기 때문에 base.html에 `header 블럭`을 추가하고 그 안에 코드를 넣었습니다. 각 페이지에서 네비게이션바를 변형시키고 싶다면 `block 태그`를 이용해서 변경하면 됩니다. base.html은 항상 특정 페이지에서 변경이 있을 수 있다는 점을 염두해두고 **각 부분마다 block으로 정의**해두면 좋습니다. header 블럭도 여러 태그로 구성되어 있는데 각 태그마다 블럭을 정의해도 괜찮습니다. 너무 과하게 block을 지정하면 보기에 불편하니 필요한 만큼만 정의하시기 바랍니다.

## 4. 게시물 작성 템플릿 구현
게시물 작성 페이지는 이전에 `update`와 동일한 뷰와 템플릿을 사용하기로 했습니다. 아무것도 하지 않아도 게시글 목록 화면에서 게시글 작성 버튼을 누르면 게시글 작성 페이지로 잘 이동합니다. 하지만 제목이 게시글 수정으로 되어 있고, 게시글 저장 버튼을 클릭했을 때도 오류가 발생합니다. 왜냐하면 액션이 `update`로 되어 있기 때문입니다. 템플릿이 게시물 작성 페이지에서는 액션부분을 `create`와 `update`로 잘 구분해줘야 합니다.

### create, update 분리
```html
<!-- bbs/templates/article_update.html -->

{% raw %}
{% extends 'base.html' %}

{% block title %}<title>게시글 수정 - {{ article.pk }}. {{ article.title }}</title>{% endblock %}

{% block css %}
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
{% endblock css %}

{% block content %}

{% if messages %}
{% for message in messages %}
<div class="alert alert-{{ message.tags }} alert-dismissible" role="alert">
  {{ message }}
</div>
{% endfor %}
{% endif %}

<form action="." method="post" class="form-horizontal">     # action 변경
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
    <tr>
		<th>작성자</th>
		<td><input type="text" class="form-control" name="author" value="{{ article.author }}"></td>
	</tr>
    <tr>
		<th>작성일</th>
		<td>{{ article.created_at | date:"Y-m-d H:i" }}</td>
	</tr>
</table>

<button class="btn btn-primary" type="submit">게시글 저장</button>
</form>
{% endblock %}
{% endraw %}
```
upate는 url에 article.pk 값이 포함되기 때문에 아직 객체가 생성되지 않은 create 액션은 사용할 수 없는 url입니다. 그래서 현재 url을 의미하는 .을 이용했습니다. 어차피 post나 get이나 모두 같은 뷰에서 처리하니 url이 같아도 상관없습니다.

그리고 `action`의 값은 뷰에서 article 객체가 전달되었으면 `'update'` 그렇지 않으면 `'create'`가 되도록 수정했습니다. 게시글 생성화면에서 article 객체가 전달되지 않지만 `article.pk`, `article.title` 등의 변수는 python과는 달리 오류를 발생하지 않습니다. **None 객체의 속성값에 접근하면 None이 출력됩니다**.

이대로 테스트를 해보면 게시글 저장이 정상적으로 작동되지만, 저장된 내용으로 채워진 게시글 수정화면으로 이동합니다. 저장 전과 후의 화면이 혼동될 수 있으니 아예 새 게시글이 정상적으로 저장이 되면 게시글 목록 화면으로 이동시킵니다.
```python
# bbs/views.py

# 생략

class ArticleCreateUpdateView(TemplateView):
    template_name = 'article_update.html'
    queryset = Article.objects.all()
    pk_url_kwargs = 'article_id'

    def get_object(self, queryset=None):
        queryset = queryset or self.queryset
        pk = self.kwargs.get(self.pk_url_kwargs)
        article = queryset.filter(pk=pk).first()

        if pk and not article:
            raise Http404('invalid pk')
        return article

    def get(self, request, *args, **kwargs):
        article = self.get_object()

        ctx = {
            'article': article,
        }
        return self.render_to_response(ctx)

    def post(self, request, *args, **kwargs):
        action = request.POST.get('action')
        post_data = {key: request.POST.get(key) for key in ('title', 'content', 'author')}
        for key in post_data:
            if not post_data[key]:
                messages.error(self.request, '{} 값이 존재하지 않습니다.'.format(key), extra_tags='danger')

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

            return HttpResponseRedirect('/article/') # 정상적인 저장이 완료되면 '/articles/'로 이동됨

        ctx = {
            'article': self.get_object() if action == 'update' else None
        }
        return self.render_to_response(ctx)
```

이제 기본적인 기능들은 완성되었습니다. 제네릭뷰도 다양하게 구현했다가 너무 많은 걸 설명하다보니 다 빼고 새로 `TemplateView`로만 구현하는 것을 변경했습니다. 모델과 템플릿에서도 좀 더 빙글빙글 꼬아서 여러 탬플릿태그들을 설명하고 싶었는데 ~~여러분의 수준을 고려하여~~ 미니튜토리얼이라 일단 여기에서 마무리하고 **사용자인증**, **페이지네이션** 등의 기능은 추후에 추가하기로 약속! ~~찡끗~~

> 우리가 어느날 마주칠 재난은 우리가 소홀히 테스트한 어느 코드에 대한 보복이다.
>
> -- swarf00, 곧 마주칠 재난을 앞두고...