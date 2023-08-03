---
title: SonarCloud로 코드 분석하기
author: Huey-J
date: 2023-08-03 12:00:00 +0800
categories: [backend, spring]
tags: [spring, java, github actions, sonar cloud]
---

# 1. 목표

SonarCloud를 이용하여 코드의 품질과 테스트 커버리지 등을 측정하고, Github Actions로 PR 시 정적 분석 결과를 코멘트로 남겨주도록 설정한다.


# 2. SonarCloud 란

비슷한 제품으로는 Sonar Qube, Sonar Cloud, Sonar Lint가 있으며

간단히 요약하면 Sonar Cloud는 정정 분석 도구이다.

실행 중인 프로그램을 분석하는 "동적 프로그램 분석"과

## SonarQube 지표

- Bugs(Reliability): 잠재적인 버그 혹은 실행시간에 예상되는 동작을 하지 않는 코드
- Vulnerabilities(Security): 보안상 이슈가 있는 코드
- Code Smells(Maintainability): 심각한 이슈는 아니지만 유지관리를 위해 개선하면 좋을 코드
- Coverage: 테스트 커버리지
- Duplications: 코드 중복
- Security Hotspots: 취약성이 존재하는지 여부를 평가하기 위해 수동 검토가 필요한 코드

# 3. 기본 연동

## 프로젝트 연결

[SonarCloud](https://sonarcloud.io/) 에서 깃허브 계정으로 로그인 후 organization을 연결한다.

1.우측 상단 create new organization -> Import an organization from Github

![create_organization](/assets/img/2023-08-03/sonarcloud_create_organization.png)

2.Organization 선택 후 SonarCloud 에 업로드하고자 하는 Repository를 선택해 준다.

![import_repository](/assets/img/2023-08-03/sonarcloud_import_repository.png)

3.free plan을 선택하고 넘어가자. organization name, organization key는 이후에도 변경 가능하니 일단 넘어가도 괜찮다!

![import_repository_2](/assets/img/2023-08-03/sonarcloud_import_repository_2.png)

이후 분석할 repository를 다시 한번 선택하면 SonarCloud에 프로젝트가 생긴 걸 볼 수 있다.
분석 이력을 보면 아래 사진처럼 첫 분석이 Success로 잘 수행된 것을 볼 수 있다!!

![first_background_task](/assets/img/2023-08-03/sonarcloud_first_background_task.png)

## PR 코멘트 확인

PR에 코멘트를 달아주는지 확인해보자. 새로운 브랜치를 만들고 아래 코드를 추가해 주자.

```java
import java.math.BigDecimal;   // 사용하지 않는 import 문

public class MyCalculator {

  public int add(int a, int b) {
    return a + b;
  }

  public int minus(int a, int b) {
    return a - b;
  }
}
```

```java
class MyCalculatorTest {

  private final MyCalculator myCalculator = new MyCalculator();

  @Test
  void addTest() {
    int result = myCalculator.add(1, 2);
    assertEquals(3, result);
  }
}
```

간단하게 계산기 클래스와 테스트 코드를 만들어 PR을 올렸다.

![first_pullrequest](/assets/img/2023-08-03/sonarcloud_first_pullrequest.png)

SonarCloud에서 자동으로 코드 분석을 해주어 코멘트를 달아주었다.
버그는 없지만 Code Smell이 하나 있다. 이를 SonarCloud에서 확인해 보자.

![first_pullrequest_code_smell](/assets/img/2023-08-03/sonarcloud_first_pullrequest_code_smell.png)

사용하지 않는 import문이 있으니 삭제를 권고하고 있다. ~~기특하다.~~

여기까지가 가장 간단하게 연동하는 방법이다. 하지만 뭔가 많이 부족하다.

| 나는 테스트 코드의 커버리지도 알고싶다!!


# 코드 커버리지 적용

SonarCloud는 자체적으로 테스트 코드의 커버리지를 분석하지는 않고, jacoco와 같은 커버리지 툴이 만든 보고서를 이용해 보여주는 방식이다.
즉, 커버리지 적용을 위해서는 jacoco가 필요하고, 이러한 xml 보고서를 전달하기 위해 현재의 Auto 방식이 아닌 CI 방식으로 변경해야 한다.

좌측 하단의 Analysis Method 메뉴로 들어가 Automatic Analysis를 끄고, 우리가 사용할 github action을 선택해 준다.

![coverage_setting](/assets/img/2023-08-03/sonarcloud_coverage_setting.png)

그리고 굉장히 친절하게 알려주는 설정 방법을 통해 Secret을 설정한다.

![coverage_setting_2](/assets/img/2023-08-03/sonarcloud_coverage_setting_2.png)

그리고 workflow와 build.gradle을 수정하는데, 아래와 같이 Jacoco와 xml report를 생성하도록 설정한다.

```yaml
plugins {
    id 'jacoco'    // jacoco 설정
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
          java-version: 17    # JDK 버전 확인
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

위와 같이 수정 후 커밋, 푸시 하면 Github Actions가 정상적으로 수행되고, 테스트 커버리지가 잘 반영되는 것을 볼 수 있다.

![coverage_result_1](/assets/img/2023-08-03/sonarcloud_coverage_result_1.png)

![coverage_result_2](/assets/img/2023-08-03/sonarcloud_coverage_result_2.png)

그리고 PR 코멘트에도 정상적으로 커버리지가 적용되는걸 볼 수 있다.

![second_pullrequest](/assets/img/2023-08-03/sonarcloud_second_pullrequest.png)


# 세부 설정

## SonarCloud PR approve 조건 설정

SonarCloud Bot이 PR에 코멘트를 남길 때 approve, reject에 대한 조건이 있는데, 이를 수정해 보자.

Organization -> Quality Gates 에 들어가면 Organization에 속한 프로젝트들에 대해 적용할 조건들의 그룹이 있다.

아래 이미지에서는 Default 설정을 복사하여 Coverage 통과 조건만 80에서 60으로 낮추었다.

그리고 맨 아래에 있는 Projects에 해당 Quality Gate를 적용할 프로젝트를 체크하면 이후 검증 작업부터 적용이 된다.

![quality_gate_setting](/assets/img/2023-08-03/sonarcloud_quality_gate_setting.png)


## PR Merge 조건에 SonarCloud 연동



![pr_merge_setting](/assets/img/2023-08-03/sonarcloud_pr_merge_setting.png)


# References

- [https://docs.sonarcloud.io/enriching/test-coverage/java-test-coverage/](https://docs.sonarcloud.io/enriching/test-coverage/java-test-coverage/)
- [https://community.sonarsource.com/t/jacoco-import-troubleshooting/35755](https://community.sonarsource.com/t/jacoco-import-troubleshooting/35755)
