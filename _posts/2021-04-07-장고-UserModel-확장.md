---
title: ์ฅ๊ณ  User Model ํ์ฅ (feat.AbstractUser)
author: Huey-J
date: 2021-04-07 12:00:00 +0800
categories: [backend, Django]
tags: [django, python]
---

# ๐ข ์์ํ๊ธฐ ์ ์...

### ๐ ํ์ฅ์ด ์ ํ์ํด?

`Django`์๋ **๊ถํ ๋ฐ ์ธ์ฆ์ ๋ํ ๊ธฐ๋ณธ์ ์ธ ๊ธฐ๋ฅ**๋ค์ ์ ๊ณตํ๊ณ  ์๋๋ฐ์.
์ฌ์ฉ์ ์ ๋ณด๊ฐ ๋ด๊ธธ `User Model` ์ญ์ ์ด๋ฏธ ๊ตฌํ์ด ๋์ด์๊ธฐ ๋๋ฌธ์ **์ฝ๊ฒ ๋ก๊ทธ์ธ ๊ธฐ๋ฅ์ ๊ตฌํ**ํ  ์ ์์ต๋๋ค.
**ํ์ง๋ง** ์ค์  ์๋น์ค์์ ์ ์ฅํด์ผ ํ๋ ์ฌ์ฉ์ ๋ฐ์ดํฐ๋ค์ด ๋๋ถ๋ถ ๋ค๋ฅด๊ธฐ ๋๋ฌธ์ **ํด๋น ๊ธฐ๋ฅ๋ค์ ์์ ํด์ผํ  ํ์**๊ฐ ์์ต๋๋ค.

์ด๊ฒ ๋ํ `Django`์ ์ค๋น๊ฐ ๋์ด์์ต๋๋ค.
๋ฌด๋ ค ์์ ์ด **ํ์ํ ์ ๋์ ๋ฐ๋ผ 4๊ฐ์ง**๋ก ๊ตฌ๋ถ๋์ด์ ๋ง์ด์ฃ ..!
์ด 4๊ฐ์ง ๋ฐฉ๋ฒ์ ๋ํ ๊ฒ์ **[์๊ธฐ](/posts/์ฅ๊ณ -UserModel-ํ์ฅ-๋ฐฉ๋ฒ)**์ ์ ๊ฐ ๊ฐ๋จํ๊ฒ ์ ๋ฆฌํ์ผ๋ ํ์ธํ๊ณ  ์ค์๋ ์ข์ต๋๋ค!

>
ํด๋น ๊ธ์์๋ `AbstractUser`๋ฅผ ํ์ฉํฉ๋๋ค. `AbstractUser`๋ `Django`์ **๋ก๊ทธ์ธ ๊ธฐ๋ฅ์ ์ฌ์ฉํ๋ฉด์ User Model์ ์นผ๋ผ(๋ฐ์ดํฐ)๋ค์ ์์ **ํ  ์ ์๊ฒ ๋ฉ๋๋ค.


### ๐ง ํ์ฌ ์ํฉ์ ์ด๋ ์ ๊ฐ์

`AbstractUser`๋ ์ฅ๊ณ ์์ ๊ธฐ๋ณธ์ ์ผ๋ก ์ ๊ณต๋๋ ๋ก๊ทธ์ธ ๊ธฐ๋ฅ์ ์์ ํ๋ ๊ฒ์๋๋ค.
์ฆ `User Model` ์ญ์ ๊ธฐ๋ณธ์ ์ผ๋ก ์ ๊ณต๋์ด ์๋์ผ๋ก DB์ ํ์ด๋ธ์ ๋ง๋ค๊ธฐ ๋๋ฌธ์ `Django`๊ฐ ์ด **ํ์ด๋ธ์ ๋ง๋ค๊ธฐ ์ (migrate ์ )์! ์์**์ ํด์ผํฉ๋๋ค.

๊ทธ๋ผ ์ด๋ฏธ `Django`๊ฐ ์ด๋ฏธ ํ์ด๋ธ์ ๋ง๋ค์๋๋ฐ ์  `AbstractUser` ์ฌ์ฉ ์๋๋์??!?!?!๐ฑ

๊ทธ๋๋ ๊ด์ฐฎ์ต๋๋ค. ๋น์ฐํ ํ๋ก์ ํธ ์ด๊ธฐ์ ํ๋๊ฒ ๊น๋ํ์ง๋ง ๋ฐฉ๋ฒ์ ์์ฃ ! ์๋์ **์ํฉ์ ๋ฐ๋ผ ๊ตฌ๋ถ์ง์ด์ ์งํ**ํ  ๊ฒ์ด๋ ์ ๋ฐ๋ผ์ ์ฃผ์ธ์!

>
### ๊ธฐ๋ณธ์ ์ธ User Model
>
| | ์นผ๋ผ๋ช | ์ค๋ช | ํ์์ฌ๋ถ | ๋ฐ์ดํฐํ์ | ํฌ๊ธฐ |
| :-: | :-: | :-: | :-: | :-: |
| 1 | **id** | PK | O | int | 11 |
| 2 | **username** | ์ด๋ฆ(์ ์ฒด) | O | char | 150 |
| 3 | **first_name** | ์ฑ | X | char | 30 |
| 4 | **last_name** | ์ด๋ฆ | X | char | 150 |
| 5 | **email** | ์ด๋ฉ์ผ | X | char | 254 |
| 6 | **password** | ์ํธํ๋ ๋น๋ฐ๋ฒํธ | O | char | 128 |
| 7 | **is_staff** | admin์ ์ ๊ฐ๋ฅ ์ฌ๋ถ | O | bool | 1 |
| 8 | **is_activate** | ๊ณ์  ํ์ฑ ์ฌ๋ถ | O | bool | 1 |
| 9 | **is_superuser** | ๋ชจ๋  ๊ถํ ํ์ฑ ์ฌ๋ถ | O | bool | 1 |
| 10 | **last_login** | ๋ง์ง๋ง์ผ๋ก ๋ก๊ทธ์ธํ ์๊ฐ | O | datetime | 6 |
| 11 | **date_joined** | ๊ณ์ ์ด ์์ฑ๋ ๋ ์ง | O | datetime | 6 |

---

# User Model ํ์ฅ!

> ### ํ์ด๋ธ ์์ฑ์ด ์ด๋ฏธ ๋์ด ์๋ค๋ฉด (migate๋ฅผ ์ด๋ฏธ ํ ๊ฒฝ์ฐ)
>
- ์์ฑ๋์ด ์๋ ํ์๊ณผ ๊ด๋ จ๋ ์ฑ์ migrationsํด๋์์ `__pycache__ํด๋`์ `__init__.py`๋ฅผ ์ ์ธํ๊ณ  ๋ค ์ง์์ฃผ์ธ์.
- DB์์ ์ด๋ ค์ผํ  ๋ฐ์ดํฐ๋ฅผ ๋ฐฑ์ํด ์ฃผ์  ๋ค์ ๊ณผ๊ฐํ๊ฒ `DROP`ํด ์ค๋๋ค.
- ์๋ ์์์ ํ ๋ค์ ๋ฐฑ์ํ ๋ฐ์ดํฐ๋ฅผ ์ DB์ ๋ฃ์ด์ฃผ์๋ฉด ๋!
>

#### 1. ์ฐ์  **์ฐ๋ฆฌ๋ง์ User Model**์ ๋ง๋ค์ด ๋ด์๋ค.

- `accounts` ์ฑ ์์ ์๋ `models.py`์ ๋ค์์ ์ถ๊ฐํด ์ฃผ์ธ์.
- ์์์ ์ธ๊ธํ ์นผ๋ผ์์ ์ฌ์ฉํ์ง ์์ ๋ฐ์ดํฐ๋ `None`์ผ๋ก ์ฒ๋ฆฌํ  ์ ์์ต๋๋ค.
  (์ ๋ ์ฑ ์ด๋ฆ์ accounts, ๋ชจ๋ธ ์ด๋ฆ์ User๋ก ํ๋๋ฐ ๋ฐ๊พธ์๋ ๋ฉ๋๋ค:)

``` python
from django.db import models
from django.contrib.auth.models import AbstractUser	# AbstractUser ๋ถ๋ฌ์ค๊ธฐ

class User(AbstractUser):
    test = models.CharField(max_length=20, default="")
    test2 = models.CharField(max_length=20, null=True)
    first_name = None
```

#### 2. ์ด์  `Django`ํํ **์ฐ๋ฆฌ๊ฐ User Model์ ๋ฐ๋ก ์ ์ํ๋ค๊ณ  ์๋ ค์ค์๋ค.**

- `settings.py`์ ๋ค์์ ์ถ๊ฐํด ์ฃผ์ธ์.

```
AUTH_USER_MODEL = 'accounts.User'
```

- accounts์ฑ ํด๋ ์์ admin.py์ ๋ค์์ ์ถ๊ฐํด์ฃผ์ธ์.
  admin ํ์ด์ง์์๋ ํ์ธํ  ์ ์์ด์ผ ํ๋๊น์

``` python
from django.contrib import admin
from .models import User

admin.site.register(User)
```

#### 3. ๊ทธ๋ผ DB๋ฅผ ์์ฑํด ๋ณผ๊น์?

- ์ฝ์์ฐฝ์ ์๋์ ๊ฐ์ด ์คํํด ์ค๋๋ค.

```
python manage.py makemigrations
python manage.py migrate
```

#### 4. ๊ด๋ฆฌ์๊ณ์ ์ ๋ง๋ค๊ณ  ์์ฑ๋ DB๋ฅผ ํ์ธํด๋ณด์ธ์!

```
python manage.py createsuperuser
python manage.py runserver
```

>
![](https://images.velog.io/images/wkdgus7113/post/2f7add0a-6266-4a1c-86ca-8f347678539e/image.png)![](https://images.velog.io/images/wkdgus7113/post/8c2f6ad9-18aa-4b29-a92a-ed5d2eff98c2/image.png)

## ์์ฑ์๋๋ค! ๐

์ด์  `User Model`์ ๋ฐ์ดํฐ๋ฅผ ๋ณ๊ฒฝํ๋ฉด์ `Django`์ ๋ก๊ทธ์ธ ๊ธฐ๋ฅ์ ์ฌ์ฉํ  ์ ์๊ฒ ๋์์ต๋๋ค.๐๐๐๐

> ์ถ๊ฐ๋ก mysql์ ์ฌ์ฉํ๋ ๊ฒฝ์ฐ ํ๊ธ์ ์ ์ฅํ  ๋ charset ๋ฌธ์ ๊ฐ ๋ฐ์ํ  ์ ์์ผ๋ DB์์ฑ์ utf8๋ก ๊ผญ ์ค์ ํด ์ฃผ์ธ์!

