---
title: 스프링 서버의 세상 간단한 CI/CD 구축 방법 (with Docker, Github Actions, AWS EC2)
author: Huey-J
date: 2023-01-19 12:00:00 +0800
categories: [데브옵스, 네트워크]
tags: [DevOps, Spring, AWS]
---

CI/CD는 처음하는 분들에게는 꽤 난이도가 높은 작업이라고 생각합니다.

그래서 많은 방법 중에 제가 생각하는 가장 간단한 방법인 도커를 이용하여 CI/CD를 구축하는 것을 소개하고자 합니다.

단계 별로 기본적인 CI를 구성하고, 도커허브에 자동으로 푸시한 이후, AWS에 접속하여, 배포하는 총 4단계로 나누어 진행하겠습니다.

해당 방식의 가장 큰 장점은 쉽다!, 공짜다! 정도가 될 것 같습니다:)


# 준비물

준비물은 아래와 같습니다.

- [도커허브](https://hub.docker.com) 계정
- [AWS](https://aws.amazon.com) 계정
- [Github](https://github.com) 계정
- AWS EC2 (ubuntu)
- Spring Boot 프로젝트

그리고 저는 아래의 환경에서 작업하였습니다.

- Ubuntu Server 22.04
- Gradle
- JDK 11

# 1. 기본 github action CI 설정

우선 가장 간단하게 깃허브 main 브랜치에 푸시가 되었을 때 빌드를 수행하는 워크플로우를 추가해 봅시다.

프로젝트 루트 폴더에서 `.github/workflows/basic-ci.yml` 파일을 추가하고 아래 내용을 입력해 줍니다.

```yaml
name: Basic CI

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      # build
      - name: Setup Java JDK
        uses: actions/setup-java@v1
        with:
          java-version: 11

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Build with Gradle
        run: ./gradlew build
```

- 3-5 Lines : main 브랜치에 push 되었을 때 해당 CI가 시작된다.
- 9 Lines : 우분투 환경에서 실행된다.
- 15-18 Lines : JDK 버전은 11
- 20-21 Lines : gradlew의 권한을 변경한다.
- 23-24 Lines : gradlew를 통해 build 파일을 생성한다.

해당 파일을 main브랜치에 푸시하면 github action탭에서 성공하는 것을 볼 수 있습니다.

![succeed basic ci](/assets/img/docker_cicd_succeed_basic_ci.png)
_succeed basic ci_
![succeed basic ci 2](/assets/img/docker_cicd_succeed_basic_ci_2.png)
_succeed basic ci 2_


# 2. 자동으로 도커 허브에 푸시

이번에는 1번에서 빌드한 jar 파일을 도커에 넣고 도커허브에 푸시하는 것 까지 해보겠습니다.

## 도커파일 생성

루트 폴더에 Dockerfile을 생성하고 아래 내용을 입력해줍니다. (확장자 없이 그냥 만들면 됨)

jar 파일을 컨테이너 안에 넣고 실행시킨다 라는 내용입니다.

```dockerfile
FROM openjdk:11

EXPOSE 8080

# The application's jar file
ARG JAR_FILE=build/libs/demo-0.0.1-SNAPSHOT.jar

# Add the application's jar to the container
ADD ${JAR_FILE} demo.jar

# Run the jar file
ENTRYPOINT ["java", "-jar", "/demo.jar"]
```

## 도커 허브 Repository 생성

도커 허브에 로그인하고 Create Repository로 새 Repository를 만들어 줍니다.

![create docker hub repository 1](/assets/img/docker_cicd_create_repo_1.png)
_create docker hub repository 1_
![create docker hub repository 2](/assets/img/docker_cicd_create_repo_2.png)
_create docker hub repository 2_

## 도커허브 토큰 생성

깃허브 액션에서 도커 허브에 접속할 때 사용할 토큰을 발급받아야 합니다.

Account Settings - Security - New Access Token

![create docker hub token](/assets/img/docker_cicd_create_token.png)
_create docker hub token_

## github secret 추가

public repo라면 비밀번호를 숨길 필요가 있으므로 github secret에 아이디와 repository, 발급받은 토큰을 등록합니다.

깃허브 프로젝트 - Settings - Secrets And Variable - Actions - New Repository Secret

![create docker hub github secret](/assets/img/docker_cicd_create_github_secret.png)
_create docker hub github secret_

## 워크플로우 추가

이제 도커 허브에 푸시를 하기 위한 준비는 끝났습니다. 이제 해당 작업을 워크플로우에 추가해 줍시다.

기존에 작성되어 있는 부분 아래에 추가합니다.

```yaml
      # docker push
      - name: Get current date
        id: date
        run: echo "::set-output name=date::$(date +'%Y-%m-%d')"

      - name: Log in to Docker Hub
        uses: docker/login-action@f054a8b539a109f9f41c372932f1ae047eff08c9
        with:
          username: ${ secrets.DOCKERHUB_ID }
          password: ${ secrets.DOCKERHUB_TOKEN }

      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@98669ae865ea3cffbcbaa878cf57c20bbf1c6c38
        with:
          images: ${ secrets.DOCKERHUB_REPO }

      - name: Build and push Docker image
        uses: docker/build-push-action@ad44023a93711e3deb337508980b4b5e9bcdc5dc
        with:
          context: .
          push: true
          tags: ${ steps.meta.outputs.tags }-${ steps.date.outputs.date }
          labels: ${ steps.meta.outputs.labels }
```

- 2-4 Lines : 도커 태그로 빌드 날짜를 저장하기 위해 현재 날짜 데이터를 불러온다.
- 6-10 Lines : 도커 허브에서 발급받은 토큰을 이용해 로그인한다.
- 12-16 Lines : 도커의 메타정보를 추출한다.
- 18-24 Lines : 도커 허브에 푸시한다.

![succeed docker push ci](/assets/img/docker_cicd_succeed_docker_push_ci.png)
_succeed docker push ci_

# 3. AWS 접속

도커 허브에 빌드파일을 푸시했으니 이제 AWS에서 해당 이미지를 가져와 실행시켜주면 되겠죠.

그 전에 AWS에 먼저 접속을 해봅시다.

## github secret 추가

AWS 접속 정보 역시 숨길 필요가 있으므로 github secret에 AWS EC2의 퍼블릭 IPv4 DNS와 pem키를 등록해줍니다.\
여기서 pem키는 에디터로 열었을 때 나오는 문자들을 넣어주면 됩니다.

![aws ip address](/assets/img/docker_cicd_aws_ip.png)
_aws ip address_

## 워크플로우 추가

기존에 작성되어 있는 부분 아래에 추가해 줍니다.

```yaml
      # server test
      - name: Deploy
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.AWS_HOST }} # EC2 인스턴스 퍼블릭 DNS
          username: ubuntu
          key: ${{ secrets.AWS_SSH_KEY }} # pem 키
          script: |
            touch hello.txt
            echo "hello world" > hello.txt
```

- EC2에 접속하여 루트 폴더에 hello.txt 파일을 생성하고 "hello world"를 저장한다.

EC2에 접속해서 hello.txt가 생겼고, 안에 "hello world"가 입력되어 있다면 성공입니다.


# 4. 도커 pull & run

이제 접속한 EC2에서 도커 허브에 푸시한 이미지를 가져와 실행만 시켜주면 끝입니다!

## AWS 도커 설치

EC2에 도커를 설치해 줍시다. 해당 내용은 [공식 문서](https://docs.docker.com/engine/install/ubuntu/)를 참고해 주세요.

## 도커 권한 설정

설치 후 sudo 없이 `docker ps` 명령어를 입력했을 때 권한 오류가 발생한다면 아래 명령어를 입력합니다.

`sudo chmod 666 /var/run/docker.sock`\
_Ref : [https://kyungyeon.dev/trouble-shooting/2](https://kyungyeon.dev/trouble-shooting/2)_

## 워크플로우 추가

드디어 마지막 워크플로우입니다. [3번에서 추가했던 워크플로우](/posts/docker-cicd/#워크플로우-추가-1)를 아래와 같이 수정해 줍니다.

```yaml
      # docker run in server
      - name: Deploy
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.AWS_HOST }} # EC2 인스턴스 퍼블릭 DNS
          username: ubuntu
          key: ${{ secrets.AWS_SSH_KEY }} # pem 키
          script: |
            docker pull ${{ steps.meta.outputs.tags }}-${{ steps.date.outputs.date }}
            docker stop demo
            docker rm demo
            docker run --restart always -d -p 80:8080 --name demo ${{ steps.meta.outputs.tags }}-${{ steps.date.outputs.date }}
```

- EC2에 접속하여 도커 이미지를 pull 하고 80번 포트로 demo라는 이름으로 실행한다.

이제 배포된 서버의 IP주소로 접속해보면 반가운(?) 404페이지가 뜨는 걸 볼 수 있습니다:)

![404 page](/assets/img/404page.png)
_404 page_


# 최종 워크플로우 파일

`.github/workflows/basic-ci.yml`

```yaml
name: Basic CI

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      # build
      - name: Setup Java JDK
        uses: actions/setup-java@v1
        with:
          java-version: 11

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Build with Gradle
        run: ./gradlew build

      # docker push
      - name: Get current date
        id: date
        run: echo "::set-output name=date::$(date +'%Y-%m-%d')"

      - name: Log in to Docker Hub
        uses: docker/login-action@f054a8b539a109f9f41c372932f1ae047eff08c9
        with:
          username: ${ secrets.DOCKERHUB_ID }
          password: ${ secrets.DOCKERHUB_TOKEN }

      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@98669ae865ea3cffbcbaa878cf57c20bbf1c6c38
        with:
          images: ${ secrets.DOCKERHUB_REPO }

      - name: Build and push Docker image
        uses: docker/build-push-action@ad44023a93711e3deb337508980b4b5e9bcdc5dc
        with:
          context: .
          push: true
          tags: ${ steps.meta.outputs.tags }-${ steps.date.outputs.date }
          labels: ${ steps.meta.outputs.labels }

      # docker run in server
      - name: Deploy
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.AWS_HOST }} # EC2 인스턴스 퍼블릭 DNS
          username: ubuntu
          key: ${{ secrets.AWS_SSH_KEY }} # pem 키
          script: |
            docker pull ${{ steps.meta.outputs.tags }}-${{ steps.date.outputs.date }}
            docker stop demo
            docker rm demo
            docker run --restart always -d -p 80:8080 --name demo ${{ steps.meta.outputs.tags }}-${{ steps.date.outputs.date }}
```