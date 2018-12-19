---
layout: article
title: 사용자인증(4)
excerpt: 장고(Django) 웹프레임워크에서 소셜로그인(oauth)를 이용하여 로그인 및 회원가입을 구현하는 방법을 알아봅니다.모델폼을 이용하여 쉽게 템플릿을 구현하는 방법을 알아봅니다.
key: django-authentication-04
aside:
  toc: true
sidebar:
  nav: docs-ko
mermaid: true
pageview: true
date: 2018/12/19
tags: 장고 Django auth 간편로그인 소셜로그인 oauth naver kakao 네이버 네아로 django-allauth 인증백엔드 login authenticate ModelBackend 싱글턴 singleton requests
metadata:
  og_title: 장고(Django) 사용자인증 제 4 편
  og_type: article
  og_locale: ko_KR
  og_description: 장고(Django) 웹프레임워크에서 소셜로그인(oauth)를 이용하여 로그인 및 회원가입을 구현하는 방법을 알아봅니다.
  og_site_name: 장고(Django) 핥짝 맛보기
---
## 1. 소셜로그인

앞서 인증기능을 구현하는 방법에 대해 살펴봤는데 로컬에서 회원가입과 로그인은 사용자에게 불편함과 불안함을 가져다 주기도 합니다. 여러분은 사용자의 정보를 안전하게 관리하고 있더라도 사용자는 그것을 알 수 없기에 자기 자신의 정보를 여러분의 서비스에 저장하는 것에 부담감이 있을 수 있습니다. 게다가 모바일로 접속한 사용자라면 이메일 주소 하나만 입력하더라도 피곤하다고 느낄 것이고 모바일의 터치키보드 떄문에 오타로 인한 스트레스를 받을 수 있기 때문에 좀 더 빠르고 편한 인증 방식을 제공해주는 것이 좋습니다.  
인증이 반드시 필요한 서비스라면 이메일 기반의 로컬인증 시스템만 제공하는 것은 사용자 입장에서는 난감할 수 있습니다. 이럴 때 이미 많은 **사용자들에게 신뢰를 받고 있으며 충분히 많은 사용자를 보유한 서비스에게 인증기능을 위임해서 인증결과만 받아볼 수 있는 방법**이 있습니다. `간편로그인`이라고 부르기도 하고 `소셜로그인`이라고도 부르기도 하는 기능인데 이름은 어찌되었든 사용자와 개발자 입장에서 간편하고 안전하게 인증을 할 수 있는 방법입니다. 이성적으로 간편로그인이라는 용어가 맞을 것 같은데 장고 커뮤니티에서 주로 소셜로그인이라 부르기 때문에 여기서도 소셜로그인이라고 하겠습니다.

### oauth 소개

소셜로그인은 `oauth` 라는 인증 프로토콜을 구현한 api를 외부에 공개해서 **누구라도`인증(authentication)`과 `권한허가(authorization)`를 사용할 수 있도록 제공하는 api** 입니다. 현재 oauth는 2.0 버전까지 나와 있지만 소셜로그인 api 제공 서비스마다 구현된 프로토콜이 다양합니다. 국내보다 해외에서 oauth 2.0 버전을 지원하는 곳이 더 많습니다. 아무래도 1.0 버전의 보안 취약성을 개선한 2.0 버전을 선호하기 때문인데 국내 서비스들도 2.0 버전을 지원하는 사례들이 많아지고 있습니다.

oauth 2.0 버전이 내용을 이해하기는 그리 어렵지 않습니다. api 제공해야 하는 프로바이더(authorization server or resource server) 입장에서는 복잡하지만 각 프로바이더들이 제공하는 api를 사용하는 여러분의 서비스(client) 입장에서는 몇 가지만 구현해주면 됩니다. **먼저 oauth 2.0 에 대해 잘 정리된 [문서](https://meetup.toast.com/posts/105)를 읽어 보시고** 아래 내용을 따라하시길 바랍니다.  
한가지 소셜로그인을 구현해 보면 나머지 서비스들도 비슷하게 구현해보실 수 있을 겁니다. 먼저 우라나라에서 가장 많은 사용자를 보유한 네이버 로그인 api로 소셜로그인 기능을 구현해보도록 하겠습니다.


### 네이버 소셜로그인 [네아로] 앱 등록
[django-allauth](https://django-allauth.readthedocs.io/en/latest/) 에서는 네이버, 카카오 등 다른 여러 서비스들에 대한 소셜로그인 기능을 제공하는데 빠르고 편하게 여러 소셜로그인 기능을 제공하고 싶으신 분들이 이 라이브러리를 사용하시고 소셜로그인의 구현방법을 좀 더 자세히 알고 싶으신 분들은 여기에 나온 방식을 따라하면서 공부해보실 수 있습니다.

> django-aullauth 는 설정을 database(admin 사이트) 로 합니다. 이렇게 하면 다양한 프로바이더를 각각 설정하기에 용이한 면이 있습니다. 여기에서는 네이버의 소셜로그인만 구현하기 때문에 설정파일에 설정하는 것으로 구현했습니다. 여러분의 편의에 따라 설정값을 데이터베이스에 저장하도록 변경하셔도 좋습니다.

먼저 네이버 소셜로그인을 사용하려면 사용권한을 얻어야 합니다. 네이버 [개발자센터](https://developers.naver.com/apps/#/register)에서 여러분의 앱을 등록하고 몇가지 설정을 하면 사용허가가 됩니다. 

1. 애플리케이션 이름 입력
2. 사용 API 선택 - 네아로 (네이버 아이디로 로그인)
3. 필수권한 선택 - 회원이름, 이메일 (그 외에 여러분이 필요한 것들)
4. 서비스 환경 추가 - PC 웹
5. 서비스 URL 입력 - http://localhost:8000
6. Callback URL 입력 - http://localhost:8000/user/login/social/naver/callback/
7. 등록하기 버튼 클릭

![네이버 앱등록 01]({{ site.url }}/snapshots/naver_app_register_01.png)

사용 API 와 서비스 환경 추가는 원하는 것들을 선택할 때마다 추가가 됩니다. 여기에서 설명하는 부분은 네이버 소셜로그인 이므로 네아로만 추가하셔도 됩니다. 서비스 URL 과 Callback URL 의 도메인은 현재 개발환경이기 때문에 localhost:8000로 설정했습니다. 여러분의 도메인이 있고 해당 **도메인으로 연결된 서버에 배포할 때 반드시 수정**해주셔야 합니다. Callback URL 은 `input` 칸 옆에 있는 + 버튼을 누르셔서 두개 이상의 URL을 입력가능합니다. 나중에 장고앱에서 로그인을 연동할 때 반드시 여기에 등록된 Callback URL 중에 하나를 사용해야 됩니다. 

![네이버 앱등록 02]({{ site.url }}/snapshots/naver_app_register_02.png)

> Callback URL 을 여러개 등록하면 케이스마다 네이버 로그인 이후의 처리방식을 달리 할 수 있습니다. 예를 들어 네이버 javascript sdk를 사용하면 브라우저에서 편리하게 access_token 을 받아올 수 있습니다. 이렇게 되면 서버에서 access_token에 대해서 검증을 할 필요가 없게 됩니다(사실은 검증할 수 없습니다). 여기서는 네이버 sdk를 사용하지 않고 서버에서 인증처리를 하도록 했습니다. 경우에 따라서 두 가지 방식 모두 제공해야 한다면 각 케이스마다 Callback URL 분리해서 사용하는 것도 좋은 방법입니다.

앱이 잘 등록이 되었다면 [앱 목록](https://developers.naver.com/apps/#/list)에서 여러분이 등록한 앱을 선택해서 앱 정보를 확인합니다. Client ID 와 Client Secret 두가지를 복사해서 설정파일에 설정합니다. Client Secret 는 보기 버튼을 클릭하셔야 내용을 볼 수 있습니다. **Client Secret 는 소스코드 외에는 복사를 하셔도 안 되고 외부에 노출되지 않도록 주의**하셔야 합니다.

![네이버 앱등록 03]({{ site.url }}/snapshots/naver_app_register_03.png)

```python
# minitutorial/settings.py

# 생략

NAVER_CLIENT_ID = 'your client id'
NAVER_SECRET_KEY = 'your secret key'

# 생략
```

> 공개된 git repository에 관리할 경우 설정파일에 민감한 내용은 푸시되지 않도록 주의하세요.

네이버 로그인에 사용되는 버튼이미지도 [네이버](https://developers.naver.com/inc/devcenter/downloads/naveridro/2014_Login_with_NAVER_button_png.zip)에서 제공하고 있습니다. sdk 를 사용하신다면 필요없겠지만 여기에서는 sdk를 사용하지 않으므로 로그인버튼을 만들어줘야 합니다(css로 잘 디자인 하셔도 됩니다.). 따로 로그인버튼을 만들기 어려운 상황이라면 [이걸](https://developers.naver.com/inc/devcenter/downloads/naveridro/2014_Login_with_NAVER_button_png.zip) 이용하셔도 됩니다. 다운로드해서 압축을 풀어보면 다양한 버튼이미지들이 있습니다. 저는 완성형 버튼을 사용할 예정인데 평상시에는 녹색이었다가 hover 상태가 되면 흰색버튼으로 바뀌게 할 예정입니다.`네이버 아이디로 로그인_완성형_Green.PNG`과 `네이버 아이디로 로그인_완성형_White.PNG` 두 이미지를 사용할 것인데 파일이름에 한글과 공백문자가 포함되어 이름을 변경할 것을 추천합니다. 각각 파일이름을 `naver_login_green.png`, `naver_login_white.png` 으로 변경해서 user/static/user/img 디렉토리에 저장합니다. 

### 네이버 소셜로그인 템플릿 생성
여기서는 뷰보다 먼저 템플릿을 만들 것입니다. 기존 로그인 기능과 함께 소셜로그인을 제공할 예정이어서 `login_form.html` 의 content 블록 상단에 `include` 템플릿태그를 추가합니다.

```html
<!-- user/templates/user/login_form.html -->

{% raw %}
{% extends 'base.html' %}
{% load i18n %}
{% load static %}

{% block title %}<title>로그인</title>{% endblock %}

{% block css %}
{{ block.super }}
<link rel="stylesheet" href="{% static 'user/css/user.css' %}">
{% endblock css %}

{% block content %}

{% include 'user/partials/social_login_panel.html' %}

<div class="panel panel-default user-panel">
    <div class="panel-heading">
        로그인하기
    </div>
    <div class="panel-body">
        <form action="." method="post">
            {% csrf_token %}
            {% include 'user/partials/form_field.html' with form=form %}
            <div class="form-actions">
                <button class="btn btn-primary btn-large" type="submit">로그인하기</button>
            </div>
            <a href="/user/resend_verify_email/">
                <div class="link-below-button">인증이메일 재발송</div>
            </a>
        </form>
    </div>
</div>
{% endblock content %}
{% endraw %}
```

기존 로그인과 동일한 화면에서 보여지지만 분리해두는게 소스코드를 보기에도 좋고, 로그인 외의 다른 화면에서도 사용하기에 좋습니다. 나중에 가입하기 화면에서도 소셜로그인 기능을 추가할 건데 동일하게 `include` 템플릿태그만 추가하면 됩니다. `social_login_panel.html` 파일을 하나 만드셔서 아래와 같이 추가합니다.

```html
<!-- user/templates/user/partials/form_field.html -->

{% raw %}
{% load static %}

{% static 'user/img/kakao_login.png' as kakao_button %}
{% static 'user/img/kakao_login_ov.png' as kakao_button_hover %}
{% static 'user/img/naver_login_green.png' as naver_button %}
{% static 'user/img/naver_login_white.png' as naver_button_hover %}

<div class="panel panel-default user-panel">
    <div class="panel-heading">
        소셜로그인
    </div>
    <div class="panel-body text-center">
        <div class="pull-left">
            <a>
                <img src="{{ kakao_button }}"
                     onmouseover="this.src='{{ kakao_button_hover }}'"
                     onmouseleave="this.src='{{ kakao_button }}'"height="34">
            </a>
        </div>
        <div class="pull-right">
            <a href="#" onclick="naverLogin()">
                <img src="{{ naver_button }}"
                     onmouseover="this.src='{{ naver_button_hover }}'"
                     onmouseleave="this.src='{{ naver_button }}'"height="34">
            </a>
        </div>
    </div>
</div>
{% endraw %}
```

먼저 상단에 `static` 템플릿태그의 렌더링 된 문자열을 as 변수명 으로 변수에 저장을 하고 이후 img 태그의 `src` 속성에는 이 변수를 사용했습니다. `static` 템플릿태그가 반복되는 경우에는 이렇게 변수에 저장하면 좀 더 깔끔하게 코드를 관리할 수 있습니다. 아까 저장한 네이버 버튼을 화면에 출력해보니 빈 공간이 너무 많이 남아서 카카오 소셜로그인 버튼도 추가했습니다. 카카오 로그인 버튼은 [여기](https://developers.kakao.com/assets/img/about/logos/login/kr/kakao_account_login_btn_medium_narrow.zip) 에서 다운로드 하시고 user/static/user/img 디렉토리에 각각` kakao_login.png`, `kakao_login_ov.png` 라는 이름으로 저장하시면 됩니다.

각 로그인 버튼의 `src`, `onmouseover`, `onmouseleave` 속성값을 설명해 드리면 src 는 img 의 기본 이미지 url 입니다. 마우스가 버튼 위에 올라가면 `onmouseover`에 등록된 스크립트가 실행되어 이미지의 `src` 를 변경하게 되어 있습니다. 마우스가 버튼에서 벗어나면 `onmouseleave`에 등록된 스크립트가 실행되어 원래의 이미지로 변경이 됩니다.

> 여기서는 카카오 소셜로그인은 구현하지 않을 예정이지만 네이버 소셜로그인을 배우시고 개인적으로 구현해보시길 권합니다.

네이버 소셜로그인 버튼이미지가 `a` 태그로 감싸있고 `onclick` 속성에 `naverLogin()` 를 호출하도록 바인드했습니다. **`naverLogin` 함수는 네이버 인증페이지로 화면을 이동시켜 사용자롤 로그인하고, 사용자로부터 사용권한을 제공**받도록 하는 기능을 합니다. `naverLogin` 함수 구현에 앞서 [네이버 기술문서](https://developers.naver.com/docs/login/api/)를 먼저 읽어보시길 바랍니다.

![네이버 소셜로그인 01]({{ site.url }}/snapshots/naver_social_login_01.png)

이정도만 해도 그럴싸한 소셜로그인 UI 가 완성되었습니다. 여기서는 소셜로그인을 로컬로그인(이메일)보다 위에 위치하도록 했습니다. UX에서 보통 더 선호하는 기능이 있을 때 덜 중요한 요소보다 위쪽에 위치하도록 합니다. 여기서도 소셜로그인을 장려하는 마음으로 상단에 위치시켰으니 혹시 여러분은 다른 생각이 있으시다면 위치를 바꾸셔도 상관없습니다.

### 네이버 로그인 javascript 구현
아까 보신대로 네이버 소셜로그인을 하기 위해서는 javascript 코드도 약간 필요합니다. 이번에 구현할 javascript 는 서버에서 렌더링할 부분이 없기 때문에 static 디렉토리에 파일을 분리합니다. 우선 `social_login_panel.html` 파일 하단에 `script` 태그를 추가합니다.

```html
<!-- user/templates/user/partials/form_field.html -->

<!-- 생략 -->

{% raw %}
<script src="{% static 'user/js/social_login.js' %}"></script>
{% endraw %}
```
그리고 `user/static/user/js/social_login.js` 파일을 하나 생성하시고 아래 코드를 추가해주세요.
```javascript

// user/static/user/js/social_login.js


function buildQuery(params) {
    return Object.keys(params).map(function (key) {return key + '=' + encodeURIComponent(params[key])}).join('&')
}
function buildUrl(baseUrl, queries) {
    return baseUrl + '?' + buildQuery(queries)
}

function naverLogin() { // 네이버 로그인
    params = {
        response_type: 'code',
        client_id:'nfenn0pzKTlihOzu_h8S',
        redirect_uri: location.origin + '/user/login/social/naver/callback/',
        state: document.querySelector('[name=csrfmiddlewaretoken]').value
    }
    url = buildUrl('https://nid.naver.com/oauth2.0/authorize', params)
    location.replace(url)
}
```
설명드려야 할 부분은 `naverLogin` 함수에서 `params` 객체의 각 값들입니다.  
먼저 `reponse_type` 은 항상 `code` 입니다. 네이버 sdk 에서는 `token` 으로 설정되어 있는데 이렇게 하면 어러분의 서버에서 인증토큰을 받는 것이 아니라 브라우저에서 전달하는 인증토큰을 사용해야 하기 때문에 보안에 취약할 수 있습니다. 즉 이 방식으로 전달받은 것들로 사용자 가입을 한다면 악의적으로 다른 사람의 `access_token` 을 도용해도 확인할 수 있는 방법이 없습니다. 여러분이 소셜로그인된 사용자의 회원정보를 서버에 저장하지 않는 경우에 적합한 방식입니다. 여기서는 **네이버 소셜로그인을 하고 회원정보를 이용해서 서버에 회원을 가입시키고 회원정보를 저장할 것이기 때문에 `code` 타입으로 설정**하셔야 합니다.  
`client_id` 는 아까 앱등록하고 발급받은 `client_id` 입니다. `client_id` 는 `secret` 과는 달리 공개되어도 되는 부분입니다.  
`redirect_uri` 는 앱등록할 때 입력한 Callback URL 중의 하나를 입력하면 됩니다. 개발단계에서는 `hostname` 이 `localhost` 이지만 실제 서비스에 배포를 하면 해당 도메인으로 설정되도록 했습니다.  
`state` 는 아주 중요한 값입니다. **csrf 등의 공격으로 사용자가 해당 서비스를 접속하지 않고 소셜로그인을 시도하는 경우를 차단**하기 위해서 필요한 값입니다. 임의의 문자열이 필요한데 서버에서도 올바른 값인지 비교할 수 있도록 로그인 `csrfmiddlewaretoken` 을 이용했습니다. `state` 값을 서버에서 이용하는 방법은 뷰를 개발할 때 설명하도록 하겠습니다.  

`naverLogin` 함수는 결국 생성된 url 로 화면이 전환하도록 하는 기능을 제공합니다. 전환된 화면은 네이버에서 제공하는 화면으로 여러분이 컨트롤할 수 없습니다. 다만 앱설정 화면 몇가지 설정만 변경할 수 있습니다.

이 상태에서 네이버 아이디로 로그인 버튼을 클릭하면 화면이 네이버에서 제공하는 화면으로 전환되고 아직 네이버에 로그인되지 않은 상태일 경우 로그인화면이 나타나고 로그인을 하면 앱 권한설정하는 화면으로 전환됩니다.(로그인이 되어 있을 때는 곧바로 앱 권한설정하는 화면으로 이동됩니다.) 

![네이버 소셜로그인 02]({{ site.url }}/snapshots/naver_social_login_02.png)

필수 제공 항목에 아까 설정한 필수권한으로 설정한 이름과 이메일이 선택할 수 있게 나타납니다. 사용자가 이 항목들을 제공하지 않고 싶을 경우에 선택을 해제할 수 있습니다. 여기서는 이름과 이메일을 이용해 회원가입을 구현해야 하기 때문에 어느 하나라도 선택을 해제할 경우 이후 callback 으로 이동된 뷰에서 로그인을 허용하지 않을 것입니다. 동의하기 버튼을 누르면 `redirect_uri` 로 설정한 주소로 이동합니다. 물론 아직 뷰가 없으니 오류가 날 것입니다.

### 네이버 로그인 callback 뷰 구현
현재는 네이버의 소셜로그인만 제공하고 있지만 향후 카카오나 페이스북 등의 소셜로그인도 기능을 제공할 수 있기 때문에 callback 뷰는 하나만 제공하고, 프로바이더 별로 믹스인을 추가하도록 하는 방법을 선택했습니다. 프로바이더 별로 callback 뷰를 생성할 수도 있으나 여러분의 선택입니다. ~~분리하는 게 저 개인적으로 선호하는 방법입니다.~~

`SocialLoginCallbackView` 라는 이름으로 뷰를 하나 생성합니다. 
```python
# user/views.py

from django.conf import settings
from django.views.generic.base import TemplateView, View
from django.middleware.csrf import _compare_salted_tokens
from user.oauth.providers.naver import NaverLoginMixin

# 생략

class SocialLoginCallbackView(NaverLoginMixin, View):

    success_url = settings.LOGIN_REDIRECT_URL
    failure_url = settings.LOGIN_URL
    required_profiles = ['email', 'name']
    model = get_user_model()

    def get(self, request, *args, **kwargs):

        provider = kwargs.get('provider')

        if provider == 'naver': # 프로바이더가 naver 일 경우
            csrf_token = request.GET.get('state')
            code = request.GET.get('code')
            if not _compare_salted_tokens(csrf_token, request.COOKIES.get('csrftoken')): # state(csrf_token)이 잘못된 경우
                messages.error(request, '잘못된 경로로 로그인하셨습니다.', extra_tags='danger')
                return HttpResponseRedirect(self.failure_url)
            is_success, error = self.login_with_naver(csrf_token, code)
            if not is_success: # 로그인 실패할 경우
                messages.error(request, error, extra_tags='danger')
            return HttpResponseRedirect(self.success_url if is_success else self.failure_url)

        return HttpResponseRedirect(self.failure_url)

    def set_session(self, **kwargs):
        for key, value in kwargs.items():
            self.request.session[key] = value
```

`SocialLoginCallbackView` 뷰는 화면이 필요없고 **오직 서버단에서 네이버 인증토큰을 받고 인증처리**를 하는 기능만 합니다. 그래서 기본 제네릭뷰를 상속받았습니다. 로그인에 설정할 경우 `settings.LOGIN_REDIRECT_URL` 로 이동하고 로그인에 실패할 경우 `settings.LOGIN_URL` 에 이동하도록 설정했습니다. 만일 변경하고 싶으시다면 `success_url`, `failure_url` 클래스변수를 수정하시면 됩니다.(해당 값들은 설정파일에 설정하는 것이 재사용하는 데 편리합니다.) 

`SocialLoginCallbackView` 는 네이버 소셜로그인 기능을 구현한 `NaverLoginMixin` 을 추가했습니다. [`NaverLoginMixin`]({{ page.url}}#네이버-로그인-믹스인-구현) 는 조금 뒤에 설명드리겠습니다. 

먼저 살펴봐야 할 부분은 `provider = kwargs.get('provider')` 부분인데 `SocialLoginCallbackView` 에서는 url 라우터로부터 프로바이더 이름을 인자로 전달받습니다. 이렇게 하면 하나의 callback 뷰에서 여러 개의 프로바이더 를 처리할 수 있습니다. 현재는 하나의 프로바이더 만 제공하기 때문에 이렇게 했지만 2개 이상의 프로바이더 만 제공하더라도 뷰클래스를 분리하는 것도 더 효율적일 것 같습니다.

**`_compare_salted_tokens` 함수를 통해 요청된 url의 query 값의 `state` 값과 쿠키의 `csrftoken` 을 비교**합니다. state 값은 `naverLogin()` 함수에서 전달한 `state` 값입니다. 만일 정상적인 로그인페이지에서 네이버 소셜로그인 버튼을 누른 거라면 `state` 값이 쿠키의 `csrftoken`과 동일한 값이어야 합니다. `_compare_salted_tokens` 함수로 비교하면 동일한 지 아닌 지 알 수 있습니다. `state` 값이 정상적인 값이라면 `NaverLoginMixin` 에 정의한 `login_with_naver` 메소드를 통해서 로그인을 시도합니다. 로그인이 정상적으로 되었다면 `success_url` 로 이동하도록 하고 로그인에 실패했다면 `failure_url` 로 이동합니다.

### 네이버 로그인 믹스인 구현

`NaverLoginMixin` 는 장고에서 제공하는 클래스가 아니라 여러분이 구현할 클래스입니다. user 앱에 `oauth` 라는 패키지를 생성하고 그 안에 `providers` 라는 패키지를 생성합니다. `providers` 패키지에 `naver.py` 라는 파일을 생성 후 아래와 같이 코드를 추가합니다.

> 일반디렉토리와 달리 패키지는 내부에 `__init__.py` 모듈(파일)이 있어야 합니다.

```python
# user/oauth/providers/naver.py

from django.conf import settings
from django.contrib.auth import login

class NaverLoginMixin:
    naver_client = NaverClient()

    def login_with_naver(self, state, code):
        
        # 인증토근 발급
        is_success, token_infos = self.naver_client.get_access_token(state, code)

        if not is_success:
            return False, '{} [{}]'.format(token_infos.get('error_desc'), token_infos.get('error'))

        access_token = token_infos.get('access_token')
        refresh_token = token_infos.get('refresh_token')
        expires_in = token_infos.get('expires_in')
        token_type = token_infos.get('token_type')

        # 네이버 프로필 얻기
        is_success, profiles = self.get_naver_profile(access_token, token_type)
        if not is_success:
            return False, profiles

        # 사용자 생성 또는 업데이트
        user, created = self.model.objects.get_or_create(email=profiles.get('email'))
        if created: # 사용자 생성할 경우
            user.set_password(None)
        user.name = profiles.get('name')
        user.is_active = True
        user.save()

        # 로그인
        login(self.request, user, 'user.oauth.backends.NaverBackend')

        # 세션데이터 추가
        self.set_session(access_token=access_token, refresh_token=refresh_token, expires_in=expires_in, token_type=token_type)

        return True, user

    def get_naver_profile(self, access_token, token_type):
        is_success, profiles = self.naver_client.get_profile(access_token, token_type)

        if not is_success:
            return False, profiles

        for profile in self.required_profiles:
            if profile not in profiles:
                return False, '{}은 필수정보입니다. 정보제공에 동의해주세요.'.format(profile)

        return True, profiles
```

`NaverLoginMixin` 에서 네이버의 api를 구현한 네이버 클라이언트를 `naver_client` 클래스변수로 추가했습니다. 네이버의 인증토큰 발급과 프로필 정보를 가져오는 두 가지의 기능을 제공합니다. [네이버 클라이언트]({{ page.url }}#네이버-클라이언트-구현)는 나중에 설명하기로 하고 `login_with_naver` 메소드를 설명드리겠습니다. `login_with_naver` 메소드는 `naver_client`로부터 `token_infos` 객체를 전달받는데 `token_infos` 객체는 아래와 같은 키를 갖는 딕셔너리 객체입니다.

1. error - 에러코드
2. error_description - 에러메시지
3. access_token - 인증토큰
4. refresh_token - 인증토큰 재발급토큰
5. expires_in - 인증토큰 만료기한(초)
6. token_type - 인증토큰 사용하는 api 호출시 인증방식(Authorization 헤더 타입)

만일 인증토큰을 받아오는 데 실패했다면 에러메시지와 함께 함수를 바로 종료합니다. 

인증토큰이 정상적으로 발급되었다면 회원가입을 위해 이메일과 사용자의 이름을 받아야 하는데, 네이버에서 profile api도 제공해주기 때문에 이것을 이용해서 받아오면 됩니다. **`get_naver_profile` 메소드는 api를 통해 받아 온 프로필 정보를 검증하는 역할**을 합니다. 프로필 정보는 사용자가 제공항목에 선택한 값들과 사용자의 id 값만 전달되는데 만일 **이메일이나 이름을 선택하지 않은 경우 에러메시지를 반환**하도록 했습니다.

프로필정보까지 정상적으로 받아오면 사용자 모델에서 `get_or_create` 메소드를 통해 동일한 이메일의 사용자가 있는 지 확인 후 없으면 새로 생성합니다. 소셜로그인은 가입과 로그인을 동시에 제공하는 것이 더 좋습니다. 이미 **가입되어 있는 사용자라면 회원정보(이름)만 수정**하면 되고, **가입되어 있지 않은 케이스라면 새로 회원정보를 생성해서 가입**시켜 줍니다. 소셜로그인은 로컬 비밀번호가 필요없기 때문에 새로 사용자 데이터가 추가되는 경우라면 `set_password(None)` 메소드를 통해 랜덤한 비밀번호를 생성해서 저장합니다. 이미 소셜로그인을 통해서 이메일에 대한 인증도 되었으니 `is_active` 값도 활성화 시켜주고 저장을 하면 가입이 완료입니다. 만일 **이미 가입되어 있던 사용자라면 이메일과 비밀번호로도 로그인이 가능하고 네이버 소셜로그인으로도 로그인이 가능**합니다.

가입된 이후에 로그인처리까지 해줘야 합니다. 로그인은 `auth` 프레임워크의 `login` 함수를 이용합니다. `login` 함수는 사용자 데이터와 로그인처리를 해줄 인증백엔드의 경로가 필요합니다. 기본 인증모듈인 `'django.contrib.auth.backends.ModelBackend'` 는 `username(email)` 과 비밀번호를 이용해서 인증처리를 하는데 소셜로그인은 비밀번호를 전달받을 수가 없습니다. 어쩔 수 없이 소셜로그인을 위한 인증백엔드를 추가로 구현해줘야 합니다. [인증백엔드]({{ page.url }}#인증백엔드-구현)의 구현은 조금 뒤에 설명하겠습니다. 

소셜로그인의 마지막은 **세션정보에 인증토큰정보를 추가**하는 것입니다. 현재는 인증토큰이 필요없지만 네이버 api를 이용한 기능을 제공할 경우도 있습니다. 이 때 사용자의 인증토큰이 있어야만 사용자의 권한으로 네이버 서비스 api 기능들을 제공할 수 있는데 매번 재로그인을 할 수 없으니 인증토큰과 그 외 정보들을 세션에 저장합니다. 인증토큰 재발급토큰(refresh_token)도 함께 저장을 해야 인증토큰이 만료가 되더라도 재발급토큰으로 다시 인증토큰을 갱신할 수 있습니다. 만일 재발급토큰도 만료가 되었거나 문제가 있어서 인증토큰을 갱신할 수 없다면 로그아웃 처리 해주면 됩니다.

### 네이버 클라이언트 구현
네이버 api 를 호출하는 모듈이 `NaverLoginMixin` 에서 필요합니다. 네이버 api 는 인증과 관련된 부분과 서비스 api를 통칭하지만 현재 여러분은 인증과 관련된 api 만 구현할 것입니다. 그래서 네이버 로그인 믹스인과 동일한 패키지에 구현해도 괜찮을 듯 합니다. 하지만 나중에 네이버 서비스 api도 구현하게 된다면 이것을 외부로 분리시키는 것이 바람직합니다. 

네이버의 api를 호출할 때 `requests` 라이브러리를 사용하여 호출하도록 했습니다. `requests` 는 파이썬의 표준 http 클라이언트보다 사용하기 간편하고, 무엇보다 직관적입니다. `requests` 라이브러리를 먼저 설치하세요.
```bash
(test-venv-36) $ pip install requests
```

`NaverLoginMixin` 이 정의된 `oauth/providers/naver.py` 에 `NaverClient` 라는 클래스를 추가하고 아래와 같이 정의합니다.
```python
# user/oauth/providers/naver.py

import requests


class NaverClient:
    client_id = settings.NAVER_CLIENT_ID
    secret_key = settings.NAVER_SECRET_KEY
    grant_type = 'authorization_code'

    auth_url = 'https://nid.naver.com/oauth2.0/token'
    profile_url = 'https://openapi.naver.com/v1/nid/me'

    __instance = None

    def __new__(cls, *args, **kwargs):
        if not isinstance(cls.__instance, cls):
            cls.__instance = super().__new__(cls, *args, **kwargs)
        return cls.__instance

    def get_access_token(self, state, code):
        res = requests.get(self.auth_url, params={'client_id': self.client_id, 'client_secret': self.secret_key,
                                                  'grant_type': self.grant_type, 'state': state, 'code': code})

        return res.ok, res.json()

    def get_profile(self, access_token, token_type='Bearer'):
        res = requests.get(self.profile_url, headers={'Authorization': '{} {}'.format(token_type, access_token)}).json()

        if res.get('resultcode') != '00':
            return False, res.get('message')
        else:
            return True, res.get('response')
```
[네이버 소셜로그인 튜토리얼](https://developers.naver.com/docs/login/web/) 문서를 참고해서 구현했습니다. 특별히 어려운 내용은 없고 간단히 requests 모듈의 사용법을 알려드리면 `get`, `post`, `put`, `delete` 등의 함수들이 구현되어 있고, 각각의 함수는 함수명과 동일한 http 메소드로 요청을 합니다. 첫번째 위치 인자는 url 이고 그 외 파라미터는 keyword 인자로 전달하면 됩니다. `get_profile` 메소드에서 `headers` 라는 파라미터가 사용되는데 http 헤더의 값을 딕셔너리 형태로 전달하면 됩니다. `Authorization` 헤더를 `token_type(bearer)` 와 인증토큰을 조합한 값으로 추가했습니다. 각 함수 반환데이터는 json 메소드를 통해 본문의 내용을 딕셔너리 형태로 반환해 줄 수도 있습니다. 물론 본문이 json 타입이 아닐 경우 에러가 발생합니다.

눈치채셨겠지만 여기서 약간 재미있는 프로그래밍(디자인) 패턴을 사용했습니다. **`singleton` 이라는 패턴인데 첫번째 생성자 호출 때만 객체만 생성시키고 이후 생성자 호출부터는 먼저 생성된 객체를 공유**하게 하는 방식입니다. `NaverClient` 클래스를 `NaverLoginMixin` 뿐만 아니라 다른 클래스에서도 공유하며 사용할 수 있습니다. `NaverClient` 객체는 인스턴스변수가 없기 때문에 하나의 객체를 서로 공유하더라도 문제가 발생하지 않습니다. 이렇게 인스턴스변수가 존재하지 않으나 여러 클래스에서 유틸리티처럼 사용하는 클래스의 경우 싱글턴 패턴을 많이 사용합니다. 객체를 생성하는 비용이 줄어 서버의 가용성을 높이는 좋은 패턴이니 구현방법을 알아두세요. 파이썬에서 여러가지 방법이 있으나 가장 간단한 방법으로 구현했습니다.

> 일반적으로 싱글턴은 생성자가 아니라 명시적으로 `getInstance` 라는 static 메소드를 제공해서 객체를 생성합니다. `getInstance` 를 사용하지 않고 생성자를 사용해 객체를 생성하면 에러를 발생시켜 싱글턴으로 구현되었음을 개발자에게 알려주는 것이죠. 싱글턴 객체에 인스턴스변수를 추가하거나 클래스변수를 변경하면 벌받습니다. ㅠㅠ

### 인증백엔드 구현

`NaverLoginMixin` 에서 로그인할 때 인증백엔드를 `'user.oauth.backends.NaverBackend'` 로  전달했습니다. 인증백엔드의 경로대로 `oauth` 패키지에 `backends.py` 파일을 추가하고 아래의 클래스를 생성해줍니다.
```python

# user/oauth/backends.py

from django.contrib.auth import get_user_model
from django.contrib.auth.backends import ModelBackend
from django.contrib.auth.models import AnonymousUser

UserModel = get_user_model()


class NaverBackend(ModelBackend):
    def authenticate(self, request, username=None,**kwargs):
        if username is None:
            username = kwargs.get(UserModel.USERNAME_FIELD)
        try:
            user = UserModel._default_manager.get_by_natural_key(username)
        except UserModel.DoesNotExist:
            pass
        else:
            if self.user_can_authenticate(user):
                return user

```

`NaverBackend` 백엔드는 기본인증백엔드(`ModelBackend`) 를 상속받아 대부분의 기능들을 그대로 사용합니다. **`authenticate` 메소드에서 비밀번호를 비교하여 인증하는 부분이 있는데 이 부분을 삭제하고 소셜로그인으로 `email` 만 비교**하도록 했습니다. 저는 예전에 `naverid` 도 데이터베이스에 저장하고 `email` 과 `naverid` 도 같이 비교하도록 구현한 적이 있는데 naverid 가 서비스에 필요하다면 저장하는 것이 맞으나 불필요하다면 굳이 데이터베이스에 저장할 필요는 없습니다. 만일 `email`이 사용자 테이블에 존재하지 않는다면 `None` 을 반환해주면 됩니다. 함수에서 아무것도 반환하지 않으면 `None` 을 리턴하므로 사용자 데이터 검색에 실패할 경우 아무것도 하지 않도록(`pass`) 했습니다.

`user_can_authenticate` 메소드는 사용자 데이터의 `is_active` 가 `True` 인지 확인하는 기능을 제공합니다. 비밀번호와 관계가 없으니 이것을 확인하는 것으로 인증백엔드의 인증테스트를 종료합니다.

소셜로그인은 이미 프로바이더에게 인증을 위임했기 때문에 인증백엔드에서 추가로 인증할 것이 별로 없습니다. 다만 사용자 모델의 정의가 이전 예제와 같지 않고 무언가 추가로 인증해야 할 필드들이 있을 경우에만 해당 필드를 이용해서 추가 인증을 하면 됩니다.

### next query 파라미터 처리

로그인되지 않은 상태에서 게시글 작성 화면 접근시 자동으로 로그인페이지로 이동하고 url 의 쿼리에 next query 파라미터가 추가됩니다.

![네이버 소셜로그인 03]({{ site.url }}/snapshots/naver_social_login_03.png)

현재 소셜로그인을 성공할 경우 무조건 settings.LOGIN_REDIRECT_URL 로 이동하는데 next query 파라미터가 있는 경우 소셜로그인 이후 해당 url로 이동하도록 수정하겠습니다.

우선 javascript 의 redirect_uri 를 통해 next query 파라미터를 전달하겠습니다.
```javascript
// user/static/user/js/social_login.js

function buildQuery(params) {
    return Object.keys(params).map(function (key) {return key + '=' + encodeURIComponent(params[key])}).join('&')
}
function buildUrl(baseUrl, queries) {
    return baseUrl + '?' + buildQuery(queries)
}

function naverLogin() {
    params = {
        response_type: 'code',
        client_id:'nfenn0pzKTlihOzu_h8S',
        redirect_uri: location.origin + '/user/login/social/naver/callback/' + location.search,
        state: document.querySelector('[name=csrfmiddlewaretoken]').value
    }
    url = buildUrl('https://nid.naver.com/oauth2.0/authorize', params)
    location.replace(url)
}
```
query 파라미터가 있을 경우 redirect_uri 에도 그대로 추가하도록 했습니다. 다음으로는 callback view 에서 next query 파라미터를 읽고 소셜로그인이 성공했을 경우 해당 url로 이동하도록 수정합니다.

```python
# user/views.py

# 생략

class SocialLoginCallbackView(NaverLoginMixin, View):

    success_url = settings.LOGIN_REDIRECT_URL
    failure_url = settings.LOGIN_URL
    required_profiles = ['email', 'name']

    model = get_user_model()

    def get(self, request, *args, **kwargs):

        provider = kwargs.get('provider')
        success_url = request.GET.get('next', self.success_url)

        if provider == 'naver':
            csrf_token = request.GET.get('state')
            code = request.GET.get('code')
            if not _compare_salted_tokens(csrf_token, request.COOKIES.get('csrftoken')):
                messages.error(request, '잘못된 경로로 로그인하셨습니다.', extra_tags='danger')
                return HttpResponseRedirect(self.failure_url)
            is_success, error = self.login_with_naver(csrf_token, code)
            if not is_success:
                messages.error(request, error, extra_tags='danger')
            return HttpResponseRedirect(success_url if is_success else self.failure_url)

        return HttpResponseRedirect(self.failure_url)

    def set_session(self, **kwargs):
        for key, value in kwargs.items():
            self.request.session[key] = value
```

간단하게 `success_url = request.GET.get('next', self.success_url)` 로 지역변수 success_url 을 정의했습니다. 소셜로그인이 성공한 이후에 이 success_url 지역변수를 이용해 이동하도록 변경했습니다. 

이제 로그아웃 이후 새 게시글 작성 버튼을 클릭하면 정상적으로 로그인 화면으로 이동되고, 소셜로그인을 하더라도 로그인 이후 게시글 작성 화면으로 이동됩니다.

### urlpatterns 추가 등록
`SocialLoginCallbackView` 는 앞서 살펴 본 대로 모든 프로바이더들을 모두 처리하도록 구현되어 있습니다. 물론 네이버 이외의 다른 프로바이더는 아직 미구현 상태이지만 여러분들이 곧바로 추가할 테니까요.^^ `urlpatterns` 에 여러분들이 개발한 `SocialLoginCallbackView` 를 등록하면 소셜로그인 기능이 완료됩니다.
```python
# minitutorial/urls.py

from django.contrib import admin
from django.contrib.auth.views import LogoutView
from django.urls import path

from bbs.views import hello, ArticleListView, ArticleDetailView, ArticleCreateUpdateView
from user.views import UserRegistrationView, UserLoginView, UserVerificationView, ResendVerifyEmailView, SocialLoginCallbackView

urlpatterns = [
    path('hello/<to>', hello), 

    path('article/', ArticleListView.as_view(), name='article-list'),
    path('article/create/', ArticleCreateUpdateView.as_view()),
    path('article/<article_id>/', ArticleDetailView.as_view()),
    path('article/<article_id>/update/', ArticleCreateUpdateView.as_view()),

    path('user/create/', UserRegistrationView.as_view()),
    path('user/<pk>/verify/<token>/', UserVerificationView.as_view()),
    path('user/resend_verify_email/', ResendVerifyEmailView.as_view()),
    path('user/login/', UserLoginView.as_view()),
    path('user/logout/', LogoutView.as_view()),
    path('user/login/social/<provider>/callback/', SocialLoginCallbackView.as_view()),

    path('admin/', admin.site.urls),
]
```

굳이 하나의 클래스에 구현한 이유는 클래스를 좀 더 일반적(general)인 형태로 구현하려고 설계하다보니 이렇게 되었습니다. 지금이라도 네이버 전용 콜백 클래스로 변형해도 나쁘지 않지만 여러분들에게 숙제로 남겨드립니다.

이제 아까 callback 페이지가 없어서 오류가 난 화면에서 새로고침을 해보시거나 다시 로그인화면으로 이동하셔서 네이버 소셜로그인을 테스트해보세요. 정상적으로 로그인이 되었다면 ~~복붙~~ 잘 따라하신 것입니다.

## 2. 회원가입 화면에 소셜로그인 추가

회원가입할 때도 소셜로그인 기능을 제공해주면 사용자들의 가입이 훨씬 편해질 것입니다. 이미 소셜로그인 기능은 다 구현된 상태이기 때문에 **회원가입 템플릿만 수정**해주면 됩니다.
```html
<!-- user/templates/registration_form.html -->

{% raw %}
{% extends 'base.html' %}
{% load i18n %}
{% load static %}

{% block title %}<title>회원 가입</title>{% endblock %}

{% block css %}
{{ block.super }}
<link rel="stylesheet" href="{% static 'user/css/user.css' %}">
{% endblock css %}

{% block content %}

{% include 'user/partials/social_login_panel.html' %}

<div class="panel panel-default user-panel">
    <div class="panel-heading">
        가입하기
    </div>
    <div class="panel-body">
        <form action="." method="post">
            {% csrf_token %}
            {% include 'user/partials/form_field.html' with form=form %}
            <div class="form-actions">
                <button class="btn btn-primary btn-large" type="submit">가입하기</button>
            </div>
        </form>
    </div>
</div>
{% endblock content %}
{% endraw %}
```
`include` 템플릿태그 하나로 소셜로그인을 추가했습니다.😏 회원가입 화면에서는 소셜로그인 화면을 조금 다르게 하고 싶다면 `include` 하지 않고 새로 구현하셔도 상관없습니다. 만약 회원가입 화면에서도 패널이름으로 소셜로그인 이라고 표시되는 것이 보기 싫으시다면 인자로 `include` 템플릿 태그에 `with` 키워드로 패널이름을 넘겨주셔도 좋습니다. 기왕 말이 나왔으니 리팩토링을 하도록 하죠. 먼저 `social_login_panel.html` 템플릿에서 `panel_name` 이라는 변수로 소셜로그인 이라는 텍스트를 대체합니다.

```html
<!-- user/templates/user/partials/form_field.html -->

{% raw %}
{% load static %}

{% static 'user/img/kakao_login.png' as kakao_button %}
{% static 'user/img/kakao_login_ov.png' as kakao_button_hover %}
{% static 'user/img/naver_login_green.png' as naver_button %}
{% static 'user/img/naver_login_white.png' as naver_button_hover %}

<div class="panel panel-default user-panel">
    <div class="panel-heading">
        {{ panel_name }}
    </div>
    <div class="panel-body text-center">
        <div class="pull-left">
            <a>
                <img src="{{ kakao_button }}"
                     onmouseover="this.src='{{ kakao_button_hover }}'"
                     onmouseleave="this.src='{{ kakao_button }}'"height="34">
            </a>
        </div>
        <div class="pull-right">
            <a href="#" onclick="naverLogin()">
                <img src="{{ naver_button }}"
                     onmouseover="this.src='{{ naver_button_hover }}'"
                     onmouseleave="this.src='{{ naver_button }}'"height="34">
            </a>
        </div>
    </div>
</div>
{% endraw %}
```

이제 `login_form.html` 과 r`egistration_form.html` 의 `include` 템플릿태그를 수정합니다.

```html
<!-- user/templates/login_form.html -->

<!-- 생략 -->

{% raw %}
{% include 'user/partials/social_login_panel.html' with panel_name='소셜로그인' %}

<!-- 생략 -->
```

```html
<!-- user/templates/registration_form.html -->

<!-- 생략 -->

{% include 'user/partials/social_login_panel.html' with panel_name='소셜회원가입' %}
{% endraw %}

<!-- 생략 -->
```

소셜로그인, 소셜회원가입으로 각각 인자를 넘겨주도록 수정하고 로그인과 회원가입 화면을 다시 한번 접속해보시면 정상적으로 출력이 됩니다.

## 3. 소셜로그인 앱 분리

소셜로그인 기능을 user 앱과는 별도의 앱으로 분리해도 되겠지만 여기서는 user 앱 내부에 소셜로그인 기능을 내장하도록 했습니다. 소셜로그인 기능만 다른 프로젝트에서 재사용하고 싶으시다면 아까 생성한 static 파일들과 뷰, 그리고 `oauth` 패키지를 따로 앱으로 분리하셔도 됩니다. 

> 사용방법은 3가지만 하시면 됩니다.
> 1. 설정파일에 `AUTHENTICATION_BACKENDS`, `NAVER_CLIENT_ID`, `NAVER_SECRET_KEY` 설정
> 2. `urlpatterns` 에 `CallbackView` 를 등록합니다.
> 3. 템플릿 생성 및 `naverLogin()` 호출. 템플릿은 어쩔 수 없으니 해당 프로젝트에 맞게 생성하시고 네이버 로그인 버튼의 `onclick` 속성을 `naverLogin()` 으로 설정해주시면 됩니다.

여기서는 앱을 분리하지 않고 조금 복잡하지만 소셜로그인과 사용자 앱을 하나의 앱으로 관리할 예정입니다. 하나의 앱으로 관리하면 좋은 점은 소셜로그인 기능이 사용자 모델에 어느정도 의존성이 있기 때문에 문제의 여지가 줄어듭니다. 예를들어 장고 `auth` 프레임워크에서 기본으로 제공하는 사용자 모델은 사용자의 이름이 `first_name`, `last_name` 으로 분리되어 있으나 새로운 사용자 모델에서는 `name` 이라는 하나의 필드로만 제공하고, 소셜로그인할 때도 `name` 이라는 필드를 사용합니다. 만일 사용자 모델에 `name` 이라는 필드가 없다면 오류가 생길테니 `NaverLoginMixin` 을 수정해주셔야 합니다. `email` 필드의 이름이 변경될 경우도 마찬가지이구요.

사용자 앱이 인증 기능 제공을 위한 앱이기 때문에 소셜로그인 기능을 추가해도 큰 문제는 없습니다. 그렇더라도 소셜로그인 기능을 별도의 앱으로 분리하는 것이 여전히 좋다고 생각하는 것이 잘못된 생각은 아닙니다. 기능별로 가급적 **앱을 분리하는 것은 좋은 아이디어이지만 분리된 앱을 재사용할 때 문제가 되는 점이 있는 지 없는 지 면밀히 검토하고 문제가 발생하지 않도록 확실한 처리를 하도록 주의**하셔야 합니다.

이것으로 인증기능은 여기서 마무리 합니다. 다음번에 배워볼 기능은 여러가지를 생각하고 있는데 권한관리(Authorization), 로깅, 파일업로드(썸네일 포함) 등의 장고 기본기능의 활용이 있습니다. 가능한 실무에서 많이 사용하는 기능을 실무에서 많이 사용하는 형태로 소개해드리겠습니다. 

> 창조적인 개발을 하려면 내가 틀릴지도 모른다는 공포를 버려야 한다
>
> swarf00, 공포는 떠나가고...창조적인 개발도 하는데...야근은 해야 하고...눈물이...뚝뚝...