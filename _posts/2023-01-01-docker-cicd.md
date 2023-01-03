---
title: 가장 간단한 CI/CD 구축 방법 with docker, github actions, aws ec2
author: Huey-J
date: 2022-12-31 12:00:00 +0800
categories: [데브옵스, 네트워크]
tags: [DevOps, Network]
---

# 준비물

- 도커허브 계정
- AWS 계정
- Github 계정
- AWS EC2


# 1. 기본 github action CI 설정

프로젝트 루트 폴더에서 .github/workflows/basic-ci.yml 파일을 추가하고 아래 내용을 입력해 준다.

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

- main 브랜치에 push 되었을 때 해당 CI가 시작된다.
- 우분투 환경에서 실행된다.
- JDK 버전은 17
- gradlew의 권한을 바꿔준다.
- gradlew를 통해 build 파일을 생성한다.

해당 파일을 main브랜치에 푸시하면 github action탭에서 성공하는 것을 볼 수 있다.

![succeed basic ci](/assets/img/docker_cicd_succeed_basic_ci.png)


# 2. 자동으로 도커 허브에 푸시

## 도커파일 생성

루트 폴더에 Dockerfile을 생성한다. 확장자 없이 그냥 만들면 된다.

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


## 도커허브 토큰 생성

Account Settings - Security - New Access Token 을 눌러준다.

생성된 토큰을 따로 저장해 놓자

## github secret 추가

public repo라면 비밀번호를 숨길 필요가 있으므로 github secret에 아이디와 repository, 발급받은 토큰을 저장해 주자.


## 워크플로우 추가

기존에 작성되어 있는 부분 아래에 추가해 준다.

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


![succeed docker push ci](/assets/img/docker_cicd_succeed_docker_push_ci.png)

자 이제 빌드파일을 생성하고 도커에 푸시함으로써 CI는 끝입니다. CD를 위한 준비가 끝났으니 이제 배포를 해봅시다.

# 3. AWS 접속

## github secret 추가

AWS EC2의 퍼블릭 IPv4 DNS와 pem키를 에디터로 열면 나오는 내용을 추가해줍니다.

## 워크플로우 추가

기존에 작성되어 있는 부분 아래에 추가해 준다.

```yaml
      # docker pull in server
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

ec2에 접속해서 hello.txt가 생겼다면 성공이다.


# 4. 도커 pull & run

이제 접속한 ec2에서 도커 허브에 푸시한 이미지를 가져와 실행만 시켜주면 끝이다.

## AWS 도커 설치

[우분투 도커 설치 공식 문서](https://docs.docker.com/engine/install/ubuntu/)를 통해 도커를 설치한다.

## 도커 권한 설정

설치 후 sudo 없이 `docker ps` 명령어를 입력했을 때 권한 오류가 발생한다면 아래 명령어를 입력하자

`sudo chmod 666 /var/run/docker.sock`

https://kyungyeon.dev/trouble-shooting/2

## 워크플로우 추가

마지막에 추가했던 Deploy를 수정해준다.

```yaml
      # docker pull in server
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




dckr_pat_Qtf6t9_ltIbXddQ7REmwCsJ9Wp4