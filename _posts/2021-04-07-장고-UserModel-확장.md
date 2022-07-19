---
title: 장고 User Model 확장 (feat.AbstractUser)
author: Huey-J
date: 2021-04-07 12:00:00 +0800
categories: [backend, Django]
tags: [django, python]
---

# 📢 시작하기 전에...

### 🙄 확장이 왜 필요해?

`Django`에는 **권한 및 인증에 대한 기본적인 기능**들을 제공하고 있는데요.
사용자 정보가 담길 `User Model` 역시 이미 구현이 되어있기 때문에 **쉽게 로그인 기능을 구현**할 수 있습니다.
**하지만** 실제 서비스에서 저장해야 하는 사용자 데이터들이 대부분 다르기 때문에 **해당 기능들을 수정해야할 필요**가 있습니다.

이것 또한 `Django`에 준비가 되어있습니다.
무려 수정이 **필요한 정도에 따라 4가지**로 구분되어서 말이죠..!
이 4가지 방법에 대한 것은 **[요기](/posts/장고-UserModel-확장-방법)**에 제가 간단하게 정리했으니 확인하고 오셔도 좋습니다!

>
해당 글에서는 `AbstractUser`를 활용합니다. `AbstractUser`는 `Django`의 **로그인 기능을 사용하면서 User Model의 칼럼(데이터)들을 수정**할 수 있게 됩니다.


### 🧐 현재 상황은 어떠신가요

`AbstractUser`는 장고에서 기본적으로 제공되는 로그인 기능을 수정하는 것입니다.
즉 `User Model` 역시 기본적으로 제공되어 자동으로 DB에 테이블을 만들기 때문에 `Django`가 이 **테이블을 만들기 전(migrate 전)에! 작업**을 해야합니다.

그럼 이미 `Django`가 이미 테이블을 만들었는데 전 `AbstractUser` 사용 안되나요??!?!?!😱

그래도 괜찮습니다. 당연히 프로젝트 초기에 하는게 깔끔하지만 방법은 있죠! 아래에 **상황에 따라 구분지어서 진행**할 것이니 잘 따라와 주세요!

>
### 기본적인 User Model
>
| | 칼럼명 | 설명 | 필수여부 | 데이터타입 | 크기 |
| :-: | :-: | :-: | :-: | :-: |
| 1 | **id** | PK | O | int | 11 |
| 2 | **username** | 이름(전체) | O | char | 150 |
| 3 | **first_name** | 성 | X | char | 30 |
| 4 | **last_name** | 이름 | X | char | 150 |
| 5 | **email** | 이메일 | X | char | 254 |
| 6 | **password** | 암호화된 비밀번호 | O | char | 128 |
| 7 | **is_staff** | admin접속 가능 여부 | O | bool | 1 |
| 8 | **is_activate** | 계정 활성 여부 | O | bool | 1 |
| 9 | **is_superuser** | 모든 권한 활성 여부 | O | bool | 1 |
| 10 | **last_login** | 마지막으로 로그인한 시간 | O | datetime | 6 |
| 11 | **date_joined** | 계정이 생성된 날짜 | O | datetime | 6 |

---

# User Model 확장!

> ### 테이블 생성이 이미 되어 있다면 (migate를 이미 한 경우)
>
- 생성되어 있는 회원과 관련된 앱에 migrations폴더에서 `__pycache__폴더`와 `__init__.py`를 제외하고 다 지워주세요.
- DB에서 살려야할 데이터를 백업해 주신 다음 과감하게 `DROP`해 줍니다.
- 아래 작업을 한 다음 백업한 데이터를 새 DB에 넣어주시면 끝!
>

#### 1. 우선 **우리만의 User Model**을 만들어 봅시다.

- `accounts` 앱 안에 있는 `models.py`에 다음을 추가해 주세요.
- 위에서 언급한 칼럼에서 사용하지 않을 데이터는 `None`으로 처리할 수 있습니다.
  (저는 앱 이름을 accounts, 모델 이름을 User로 했는데 바꾸셔도 됩니다:)

``` python
from django.db import models
from django.contrib.auth.models import AbstractUser	# AbstractUser 불러오기

class User(AbstractUser):
    test = models.CharField(max_length=20, default="")
    test2 = models.CharField(max_length=20, null=True)
    first_name = None
```

#### 2. 이제 `Django`한테 **우리가 User Model을 따로 정의했다고 알려줍시다.**

- `settings.py`에 다음을 추가해 주세요.

```
AUTH_USER_MODEL = 'accounts.User'
```

- accounts앱 폴더 안의 admin.py에 다음을 추가해주세요.
  admin 페이지에서도 확인할 수 있어야 하니까요

``` python
from django.contrib import admin
from .models import User

admin.site.register(User)
```

#### 3. 그럼 DB를 생성해 볼까요?

- 콘솔창에 아래와 같이 실행해 줍니다.

```
python manage.py makemigrations
python manage.py migrate
```

#### 4. 관리자계정을 만들고 생성된 DB를 확인해보세요!

```
python manage.py createsuperuser
python manage.py runserver
```

>
![](https://images.velog.io/images/wkdgus7113/post/2f7add0a-6266-4a1c-86ca-8f347678539e/image.png)![](https://images.velog.io/images/wkdgus7113/post/8c2f6ad9-18aa-4b29-a92a-ed5d2eff98c2/image.png)

## 완성입니다! 🎉

이제 `User Model`의 데이터를 변경하면서 `Django`의 로그인 기능을 사용할 수 있게 되었습니다.👏👏👏👏

> 추가로 mysql을 사용하는 경우 한글을 저장할 때 charset 문제가 발생할 수 있으니 DB생성시 utf8로 꼭 설정해 주세요!

