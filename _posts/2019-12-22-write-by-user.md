---
layout: article
title: 사용자인증(5)
excerpt: 장고(Django) 웹프레임워크에서 소셜로그인(oauth)를 이용하여 로그인 및 회원가입을 구현하는 방법을 알아봅니다.모델폼을 이용하여 쉽게 템플릿을 구현하는 방법을 알아봅니다.
key: django-authentication-05
aside:
  toc: true
sidebar:
  nav: docs-ko
mermaid: true
pageview: true
date: 2018/12/22
tags: 장고 Django auth 간편로그인 소셜로그인 oauth naver kakao 네이버 네아로 django-allauth 인증백엔드 login authenticate ModelBackend 싱글턴 singleton requests
metadata:
  og_title: 장고(Django) 사용자인증 제 5 편
  og_type: article
  og_locale: ko_KR
  og_description: 장고(Django) 웹프레임워크에서 소셜로그인(oauth)를 이용하여 로그인 및 회원가입을 구현하는 방법을 알아봅니다.
  og_site_name: 장고(Django) 핥짝 맛보기
---
## 1. 사용자가 작성하는 게시글

지금까지의 게시글은 로그인만 되어 있으면 누구라도 작성하도록 되어 있습니다. 게시글에 사용자의 이름을 남겨서 작성자가 누구인지만 알게 해뒀습니다. 그런데 만약 사용자의 이름이 겹친다면 어떻해야 하나요? 또 내가 작성한 글을 나만 수정하고 싶은데 이름만 같다면 누구라도 내 글을 수정할 수 있나요? 작성자의 표시를 사용자 이름으로만 하는 것은 좋은 방법이 아니란 것이 드러났습니다. 

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
이렇게 모엘의 필드값을 다른 모델의 id로 사용할 때의 

## ForeignKey(외래키)
데이터베이스(sql)에서는 ForeignKey 를 지정해서 테이블 간의 관계를 표현하는 것이 가능했습니다. 장고ORM에서도 ForeignKey 라는 필드타입을 제공하는데 sql의 그 ForeignKey 를 생성하는 기능을 제공합니다. ForeignKey 필드는 연결되어지는 모델과 해당 모델이 1:N 관계가 됩니다. 당연한 얘기같지만 Article 모델에서 사용자 모델을 ForeignKey 로 연결할 경우 하나의 게시글은 한명의 사용자만이 작성자로 등록될 수 있습니다. 일단 Article 모델의 author 필드를 ForeignKey 로 변경하도록 하겠습니다.
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

ForeignKey 필드를 가장 전형적인 방식으로 선언을 했습니다. 첫번째 인자로 ForeignKey로 연결할 모델의 식별자를 전달합니다. 직접 모델 클래스를 전달하는 방법도 있고, 저렇게 '앱label.모델이름'으로 표현하는 방법도 있습니다. 어느방법을 사용하셔도 동일합니다.  
두번째 인자 related_name 은 실제 데이터베이스 상에 추가되는 속성은 아니고 ForeignKey 로 설정되어지는 모델(User)에서 Article 객체를 참조해야 할 때 사용하는 속성으로 사용됩니다. 예를 들어 특정 사용자가 작성한 모든 게시글을 검색할 때 `user.articles.all()` 이라고 하면 User 모델에 정의한 적 없는 articles 라는 속성으로 추가되어 연관된 Article 모델을 검색할 수 있게 됩니다. related_name 이라는 속성을 추가하지 않으면 기본적으로 필드명+'_set' 으로 related_name 이 추가됩니다. 때로는 두 모델 사이에 ForeignKey 가 두개 이상 존재하는 경우도 있습니다. 이럴 때 related_name이 겹쳐 오류가 생길 수 있으니 습관적으로 related_name 을 추가해주면 오류를 줄일 수 있습니다.  
on_delete 속성은 ForeignKey 로 연결되는 모델(User)의 데이터가 삭제될 경우