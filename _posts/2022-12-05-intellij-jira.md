---
title: Intellij Jira 연동
author: Huey-J
date: 2022-12-05 12:00:00 +0800
categories: [팁, 인텔리제이]
tags: [IntelliJ, Jira]
---

# 1. Jira Api Token 발급

- 계정관리 → 보안 → API 토큰 만들기 및 관리 → 토큰 만들기 → 토큰 복사

![intellij myaccount1](/assets/img/intellij_myaccount1.png)

![intellij myaccount2](/assets/img/intellij_myaccount2.png)


# 2. 인텔리제이 Jira 연동

- `shift shift` 전체 검색 → configure servers

![intellij configure server](/assets/img/intellij_configureserver.png)

- `+` 버튼 → Jira → 입력값 입력
- Search 내용에 따라 이슈가 필터링 됨

![intellij jira setting](/assets/img/intellij_jira_setting.png)


# 3. 브랜치 네이밍 설정

- Preferences → Tools → Tasks → Feature branch name format 수정

![intellij jira branchname](/assets/img/intellij_jira_branchname.png)


# + 이슈 진행중 처리 및 브랜치 생성

- opt + shift + n → 이슈 검색 및 선택
- 변경할 상태 선택, 브랜치 명 확인

![intellij jira create branch](/assets/img/intellij_jira_createbranch.png)

- 인텔리제이 오른쪽 위에서도 확인 가능