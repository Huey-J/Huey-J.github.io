---
title: SonarCloud로 코드 분석하기
author: Huey-J
date: 2023-08-03 12:00:00 +0800
categories: [backend, spring]
tags: [spring, java, github actions, sonar cloud]
---

# 1. 목표

SonarCloud를 이용하여 코드의 품질과 테스트 커버리지 등을 측정하고, Github Actions와 연동하여 PR 시 정적 분석 결과를 코멘트로 남겨주도록 설정한다.

> 예제에서 다룬 코드는 [여기](https://github.com/Huey-J/sonarcloud-example)서 확인할 수 있습니다.
{: .prompt-info }


# 2. SonarCloud 란

SonarCloud는 정적 분석 툴인 SonarQube의 SaaS 형태입니다.

여기서 정적 분석이란 애플리케이션을 실제 실행하지 않고 분석하는 것을 의미하며, 반대로는 동적 분석이 있습니다.
간단하게 얘기하면 제 코드를 보고 버그나 취약점, 냄새나는(?) 코드를 찾아주는 ~~배은망덕한~~ 고마운 친구죠 :)

PR에 코멘트도 달아주고, Quality Gates와 Github 설정을 연계해 머지에 대한 제약도 걸 수 있습니다.

그리고 뭐니 뭐니 해도 SonarCloud를 사용해야 할 가장 큰 이유는 분석 대상이 Public Repository인 경우 '무료'이기 떄문입니다! ㅎㅎ

![흐뭇](/assets/img/zzal/이말년_흐뭇.jpeg)
_공짜는 못 참지 크흠 흠..._

> **SonarCloud 지표**
> - Bugs(Reliability): 잠재적인 버그 혹은 실행시간에 예상되는 동작을 하지 않는 코드
> - Vulnerabilities(Security): 보안상 이슈가 있는 코드
> - Code Smells(Maintainability): 심각한 이슈는 아니지만 유지관리를 위해 개선하면 좋을 코드
> - Coverage: 테스트 커버리지
> - Duplications: 코드 중복
> - Security Hotspots: 취약성이 존재하는지 여부를 평가하기 위해 수동 검토가 필요한 코드


# 3. 기본 연동

가장 간단한 방법입니다. 준비물은 분석하고자 하는 Public Repository만 있으면 되고, 이것을 SonarCloud에 등록하여 default 브랜치를 분석합니다.

## 프로젝트 연결

[SonarCloud](https://sonarcloud.io/) 에서 깃허브 계정으로 로그인 후 organization을 연결해 줍니다.

1.우측 상단 create new organization -> Import an organization from Github

![create_organization](/assets/img/2023-08-03/sonarcloud_create_organization.png)

2.Organization 선택 후 SonarCloud 에 분석하고자 하는 Repository를 선택해 줍니다.

![import_repository](/assets/img/2023-08-03/sonarcloud_import_repository.png)

3.free plan을 선택하고 넘어갑시다.(private repo는 유료) 여기서 설정하는 값들은 추후 언제든지 변경 가능하니 참고 해주세요 :)

![import_repository_2](/assets/img/2023-08-03/sonarcloud_import_repository_2.png)

이후 분석할 repository를 다시 한번 선택하면 SonarCloud에 프로젝트가 생긴 걸 볼 수 있습니다.
분석 이력을 보면 아래 사진처럼 첫 분석이 Success로 잘 수행된 것을 확인할 수 있습니다!

![first_background_task](/assets/img/2023-08-03/sonarcloud_first_background_task.png)

## PR 코멘트 확인

간단하게 SonarCloud에 등록만 해준 것 뿐인데 아래 PR처럼 코멘트도 자동으로 달아줍니다.

![first_pullrequest](/assets/img/2023-08-03/sonarcloud_first_pullrequest.png)
_https://github.com/Huey-J/sonarcloud-example/pull/1_

![주목](/assets/img/zzal/주목_움짤.gif)
_짜자잔~_

이는 Automatic Analysis 라는 설정이 기본적으로 켜져 있기 때문인데요! 아래에서 추가로 알아보는 걸로 하고 일단 넘어갑시다.

다시 PR 코멘트로 돌아와 내용을 보면 Bug는 없지만 Code Smell이 하나 있는걸 볼 수 있는데 SonarCloud에서 한번 확인해 볼까요?

![first_pullrequest_code_smell](/assets/img/2023-08-03/sonarcloud_first_pullrequest_code_smell.png)

사용하지 않는 import문이 있으니 삭제하라고 권고하고 있습니다. 말 그대로 냄새 나는 코드군요. 🫣

여기까지가 가장 간단하게 SonarCloud를 연동하는 방법이었습니다. 굉장히 간단하죠? 하지만 뭔가 많이 부족한 것 같은데...


# 3. 테스트 커버리지 적용

위에서 봤던 PR 코멘트를 보면 `No Coverage information`라는 문구를 볼 수 있습니다.
말 그대로 테스트 코드의 커버리지 정보가 없다는 것인데요. 정적 분석에 커버리지가 없다니! 얼른 추가해 줍시다.

## CI 방식으로 전환

SonarCloud는 자체적으로 커버리지를 분석하지는 않고, jacoco와 같은 테스트 커버리지 툴이 만든 보고서를 이용하여 보여주는 방식을 사용합니다.
즉, 커버리지 적용을 위해서는 jacoco가 필요하고, 보고서를 전달하기 위해 현재의 Auto 방식이 아닌 CI 방식으로 변경해야 하죠.

1.좌측 하단의 Analysis Method 메뉴로 들어가 Automatic Analysis를 끄고, 우리가 사용할 github action을 선택해 줍니다.

![coverage_setting](/assets/img/2023-08-03/sonarcloud_coverage_setting.png)

2.그리고 굉장히 친절하게 알려주는 설정 방법을 통해 Github Repository의 Secret을 추가해 줍니다.

![coverage_setting_2](/assets/img/2023-08-03/sonarcloud_coverage_setting_2.png)

3.workflow와 build.gradle을 수정하는데, 아래와 같이 Jacoco와 xml report를 생성하도록 설정합니다.
JDK 버전에 주의합시다!

```
plugins {
    id 'jacoco'
    id "org.sonarqube" version "4.2.1.3168"
}

sonar {
    properties {
        property "sonar.projectKey", "Huey-J_sonarcloud-example"
        property "sonar.organization", "huey-j"
        property "sonar.host.url", "https://sonarcloud.io"
    }
}

jacocoTestReport {
    reports {
        xml.enabled true
    }
}
```

```yaml
name: SonarCloud
on:
  push:
    branches:
      - main
  pull_request:
    types: [opened, synchronize, reopened]
jobs:
  build:
    name: Build and analyze
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0  # Shallow clones should be disabled for a better relevancy of analysis
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: 17    # JDK 버전
          distribution: 'zulu' # Alternative distribution options are available
      - name: Cache SonarCloud packages
        uses: actions/cache@v3
        with:
          path: ~/.sonar/cache
          key: ${{ runner.os }}-sonar
          restore-keys: ${{ runner.os }}-sonar
      - name: Cache Gradle packages
        uses: actions/cache@v3
        with:
          path: ~/.gradle/caches
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle') }}
          restore-keys: ${{ runner.os }}-gradle
      - name: Build and analyze
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}  # Needed to get PR information, if any
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: ./gradlew build jacocoTestReport sonar --info    # jacocoTestReport 추가
```

위와 같이 수정 후 커밋, 푸시 하면 Github Actions가 정상적으로 수행되고, 테스트 커버리지가 잘 반영되는 것을 볼 수 있습니다.

![coverage_result_1](/assets/img/2023-08-03/sonarcloud_coverage_result_1.png)

![coverage_result_2](/assets/img/2023-08-03/sonarcloud_coverage_result_2.png)

이후 PR 코멘트에도 정상적으로 커버리지가 적용되는걸 확인할 수 있습니다.

![second_pullrequest](/assets/img/2023-08-03/sonarcloud_second_pullrequest.png)
_https://github.com/Huey-J/sonarcloud-example/pull/2_


# 4. 세부 설정 추가

## SonarCloud PR approve 조건 설정

SonarCloud Bot이 PR에 코멘트를 남길 때 approve, reject에 대한 조건이 있는데, 이를 Quality Gate라 부릅니다. 요걸 수정해 봅시다.

Organization -> Quality Gates 에 들어가면 해당 Organization에 속한 통과 조건들이 있습니다.

![quality_gate_setting](/assets/img/2023-08-03/sonarcloud_quality_gate_setting.png)

위 이미지에서는 Default 설정을 복사하여 Coverage 통과 조건만 80에서 60으로 낮추었습니다.
그리고 맨 아래에 있는 Projects에 해당 Quality Gate를 적용할 프로젝트를 체크하면 이후 검증 작업부터 적용이 됩니다.


## PR Merge 조건에 SonarCloud 연동

깃허브에서 SonarCloud 조건을 통과하지 못할 경우 Merge가 안되도록 설정할 수 있습니다.

Settings -> Branches -> Branch protection rule 머지 조건으로 Sonar Cloud를 추가해 줍니다.

![pr_merge_setting](/assets/img/2023-08-03/sonarcloud_pr_merge_setting.png)


# 5. 마무리

이번 글에서는 SonarCloud를 연동하여 정적 분석을 작업 프로세스에 추가하였습니다.
덕분에 코드의 품질과 테스트 커버리지에 대해 좀 더 고민하며 개발할 수 있지 않을까 라는 자그마한 기대를 해 봅니다. ㅎㅎ

~~테스트 커버리지를 적용하면서 build.yml 마지막 줄에 `jacocoTestReport`를 추가하지 않아 커버리지 적용이 안돼서 삽질한 건 안비밀..~~

![pr_approve](/assets/img/zzal/풀리퀘_승인.jpeg)

# References

- [https://docs.sonarcloud.io/enriching/test-coverage/java-test-coverage/](https://docs.sonarcloud.io/enriching/test-coverage/java-test-coverage/)
- [https://community.sonarsource.com/t/jacoco-import-troubleshooting/35755](https://community.sonarsource.com/t/jacoco-import-troubleshooting/35755)
- [https://hyeon9mak.github.io/sonarcloud-trouble-shooting/](https://hyeon9mak.github.io/sonarcloud-trouble-shooting/)
