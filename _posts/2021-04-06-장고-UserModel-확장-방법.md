---
title: 장고 User Model 확장 방법
author: Huey-J
date: 2021-04-06 12:00:00 +0800
categories: [backend, Django]
tags: [django, python]
---

# 'User Model'이란?

`Django`는 백엔드에서 꽤 중요한 부분을 차지하고 있는 **권한과 인증**에 대해 구현이 되어있습니다.
이때 `User Model`이 **사용자들의 데이터를 저장**합니다.

물론 제공되어 있는 그대로의 `User Model`을 사용해도 좋지만 서비스를 개발할 땐 **더 다양한 기능과 정보**들을 필요로 할 때가 많기 때문에 서비스를 개발 할 때 `User Model`을 그대로 사용하는 경우는 거의 없다고 합니다.

> 아래처럼 `settings.py`에 자동으로 적용되어 있다.
``` python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.sessions',
    ...
]
```


# User Model 확장 방법

## 1. 프록시 모델 사용(Proxy Model)

- **테이블 추가, 변경 없이** 단순히 상속만 하는 방식
- **정렬순서**나 필요한 **메소드**만 추가하기 위해 사용
- 기존의 `User Model`에 추가적인 사용자 정보를 저장할 필요가 없을 때 사용하는 **가장 간단한** 방법

``` python
class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    user_pk = models.IntegerField(blank=True)
    nickname = models.CharField(max_length=200, blank=True)
    phone = models.CharField(max_length=200, blank=True)
    
    ...
```

## 2. 일대일 연결(one-to-one)

- **모델(테이블)을 추가**하여 기존 `User Model`과 일대일로 연결시켜서 사용자에 대한 정보를 저장
- `Django`의 인증 시스템을 그대로 활용하고 로그인, 권한 부여 등과 상관이 없는 사용자 데이터를 저장하고자 할 때 사용하는 간단한 방법

``` python
from django.db import models
from django.contrib.auth.models import User
    
class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    user_pk = models.IntegerField(blank=True)
    nickname = models.CharField(max_length=200, blank=True)
    phone = models.CharField(max_length=200, blank=True)
    
    ...
```

## 3. AbstractUser 모델 사용

- `AbstractUser Model`을 상속한 `User Model`을 **새로 정의**하여 사용 (settings.py 수정 필요)
- 이 기법의 사용 여부는 **프로젝트 시작 전**에 하는 것이 좋음
- 기존의 `User Model`을 그대로 사용하므로 기본 **로그인 인증 처리 부분은 `Django`의 것을 이용**하면서 몇몇 **사용자 정의 필드를 추가**할 때 유리함

``` python
from django.db import models
from django.contrib.auth.models import AbstractBaseUser

class User(AbstractBaseUser, PermissionsMixin):
    objects = UserManager()

    email = models.EmailField(verbose_name = "email id", max_length = 64, unique = True)
    username = models.CharField(max_length=30)
    USERNAME_FIELD = 'email'
    date_joined = models.DateTimeField(_('date joined'), default=timezone.now)
    
    ...

```

## 4. AbstractBaseUser 모델 사용

- `AbstractBaseUser Model`을 상속한 `User Model`을 **새로 정의**하여 사용 (settings.py 수정 필요)
- 이 기법의 사용 여부는 **프로젝트 시작 전**에 하는 것이 좋음
- 로그인 아이디로 이메일 주소를 사용하도록 하거나 `Django` 로그인 절차가 아닌 **인증 절차를 직접 구현**하고자 할 때 사용 (가장 자유도가 높음)

``` python
from django.db import models
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    bio = models.TextField(max_length=500, blank=True)
    location = models.CharField(max_length=30, blank=True)
    birth_date = models.DateField(null=True, blank=True)
    
    ...
```