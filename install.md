---
layout: article
title: 설치하기
key: django-install
aside:
  toc: true
sidebar:
  nav: docs-ko
tags: 장고 Django 설치
metadata:
  og_title: 장고(Django) 설치하기
  og_type: article
  og_locale: 'ko_KR'
  og_description: 장고(Django) 웹프레임워크 개발에 앞서 가상환경 세팅과 설치하는 방법을 설명합니다.
  og_site_name: 장고(Django) 핥짝 맛보기
---

1. 파이썬3.6과 virtualenv가 설치되어 있는 것을 간주하고 설명합니다. 혹시 아직 설치가 되지 않았다면 아래 링크를 따라 설치하세요.

   >[python3.6 설치](https://www.python.org/downloads/)
   >
   > [virtualenv 설치](https://docs.python.org/ko/3.6/tutorial/venv.html)

   virtualenv까지 설치하셨다면 아래 명령어로 가상환경을 만드세요.
   ```bash
   $ python3 -m venv test-venv-36
   ```

2. 가상환경을 활성화시키세요.
```bash
$ source test-venv-36/bin/activate
```

3. 가상환경이 활성화되었다면 프롬프트에 아래와 같이 가상환경의 이름이 표시됩니다.
```bash
(test-venv-36) $
```

4. pip로 django를 설치해줍니다.
```bash
(test-venv-36) $ pip install django
```

5. django가 정상적으로 설치되었는 지 python shell에서 확인합니다. 오류가 발생하지 않는다면 정상적인 설치가 되었습니다.
```python
(test-venv-36) $ python
>>> import django
>>> django.get_version()
'2.1.3'
```
본 튜토리얼은 장고 2.1.3을 기준으로 개발하고 있습니다.