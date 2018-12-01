---
layout: article
title: 인증과 권한
aside:
  toc: true
sidebar:
  nav: docs-ko
metadata:
  'og:title': 장고(Django) 사용자인증 활용하기
  'og:type': article
  'og:locale': 'ko_KR'
  'og:description': 장고(Django) 웹프레임워크에서 기본 제공하는 auth 프레임워크를 이용하여 사용자인증을 구현하는 방법을 설명합니다.
  'og:site_name': 장고(Django) 핥짝 맛보기
---

# 1. 사용자인증

기본적인 게시판 기능이 완성되었는데, 누구라도 작성이 가능하고 누구라도 자신이 작성하지 않은 게시글까지 수정이 가능한 상황입니다. 또한 어뷰징을 하는 사용자가 있다면 그런놈들은 영구적으로 사용을 하지 못하도록 하고 싶습니다. 이럴 때 접속자들에게 꼬리표를 붙여주면 됩니다. 만일 꼬리표를 마음대로 떼버린다면 기능에 제한해버리면 됩니다. 이것을 `사용자인증` 이라고 합니다.

## 장고 auth 프레임워크
장고 admin 사이트에 접속할 때 생성했던 슈퍼유저가 기억나실 겁니다. 이것이 장고에서 기본적으로 제공하는 인증기능입니다. id와 비밀번호를 포함한 모든 사용자정보는 데이터베이스에 기록이 되고 로그인을 할 때 입력한 id와 비밀번호가 동일한 경우 해당 id의 사용자가 맞다고 판단하게 됩니다. 
장고 auth 프레임워크는 크게 가입, 로그인, 로그아웃 세가지의 기능을 제공합니다. 앞으로 각 기능이 어떻게 구현되어 있는지 메커니즘을 이해하며 공부를 하면 좀 더 안전한 웹서비스를 구축할 수 있습니다.

<!-- Global site tag (gtag.js) - Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=UA-129815004-1"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'UA-129815004-1');
</script>

###


### 로그인


### 로그아웃


## 회원가입

## 로그인

## 로그아웃

# 2. 권한부여

# 3. 