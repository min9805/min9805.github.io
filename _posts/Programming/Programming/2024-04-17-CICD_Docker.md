---
title: "[CI/CD] Docker, Github Action, AWS 를 통한 자동 배포!"
excerpt: "[CI/CD] Docker, Github Action, AWS 를 통한 자동 배포!"
categories:
  - programming
tags:
  - [cicd]

toc: true
toc_sticky: true

date: 2024-04-17
last_modified_at: 2024-04-17
---

# 개요

백엔드 개발 시 main 브랜치 push 에 따라 자동 배포를 통해 개발 서버를 유지하고자 한다. 이를 위해 이전 포스팅에서는 Github action 와 AWS 의 S3, CodeDeploy, EC2 를 사용해서 배포하였다.

이번에는 Docker 를 사용해 DockerHub 와 EC2 만으로 간단하게 배포해보자.

![image](https://github.com/min9805/min9805.github.io/assets/56664567/b45d4e5a-69e6-4d43-bd8c-8a787ac950df)

전체적인 구조는 위와 같다.

1. Github 에 push 시에 Github Action 이 동작한다.
2. Github Action 을 통해 Docker 이미지가 생성되고 Docker Hub 에 저장된다.
3. Github Action 에서 ssh 를 통해 EC2 에 직접 접근해 Docker Hub 의 이미지를 가져와 실행시킨다. 

해당 프로세스를 통해 Github push 를 통해 개발 서버를 배포할 수 있다!

# 코드 작성

## Dockerfile

```
FROM amazoncorretto:17

WORKDIR /usr/app

COPY ./build/libs/demo-0.0.1-SNAPSHOT.jar /usr/app/app.jar

CMD ["java", "-jar", "app.jar"]
```

build 된 jar 파일을 통해 Docker 이미지를 만드는 Dockerfile 이다.

## workflows

```
name: Java CI with Gradle

on:
  push:
    branches: [ "master" ]
  pull_request:
    branches: [ "master" ]

jobs:
  build:

    runs-on: ubuntu-latest
    permissions:
      contents: read

    steps:
    - uses: actions/checkout@v4
    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'corretto'

    - name: gradlew permmision
      run: chmod +x ./gradlew
    # Configure Gradle for optimal use in GiHub Actions, including caching of downloaded dependencies.
    # See: https://github.com/gradle/actions/blob/main/setup-gradle/README.md
    - name: Setup Gradle
      uses: gradle/actions/setup-gradle@417ae3ccd767c252f5661f1ace9f835f9654f2b5 # v3.1.0

    - name: Build with Gradle Wrapper
      run: ./gradlew clean build -x test

    - name: Docker build
      run: |
        docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_PASSWORD }}
        docker build -t app .
        docker tag app ${{ secrets.DOCKER_USERNAME }}/cicd_docker
        docker push ${{ secrets.DOCKER_USERNAME }}/cicd_docker

    - name: Deploy
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.SSH_HOST }}
        username: ${{ secrets.SSH_USERNAME }}
        key: ${{ secrets.SSH_KEY }}
        script: |
          docker pull ${{ secrets.DOCKER_USERNAME }}/cicd_docker
          docker stop $(docker ps -a -q)
          docker rm $(docker ps -a -q)
          docker run -d --log-driver=syslog -p 8080:8080 ${{ secrets.DOCKER_USERNAME }}/cicd_docker
          docker rm $(docker ps --filter 'status=exited' -a -q)
          docker image prune -a -f
```

Github Action 에서 dockerfile 을 빌드하고 ssh로 EC2 에서 해당 이미지를 실행시켜야한다.

이를 위해 secrets 에 아래 요소를 넣어야한다.

- DOCKER_USERNAME : Docker Hub 의 아이디
- DOCKER_PASSWORD : Docker Hub 에 로그인 할 수 있는 비밀번호
- SSH_HOST : EC2 의 Host
- SSH_USERNAME : EC2 의 Username
- SSH_KEY : EC2 의 .pem key

# 주의 사항

SSH 를 통해 접속하기 때문에 port 를 열어놓아야한다.
Github Action 의 IP 주소를 알아내 배포 시 해당 IP 를 ssh 루트에 추가해 배포하고 다시 삭제하는 방법도 사용 가능하지만 여기에서는 모두 열어두었다. 

![image](https://github.com/min9805/min9805.github.io/assets/56664567/03fcfbd4-f1a7-4af4-b25e-c4d7ac1af213)

# 결과


![image](https://github.com/min9805/min9805.github.io/assets/56664567/d506b2ba-5ae8-47cc-9cf7-6a5d03a0be2f)
![image](https://github.com/min9805/min9805.github.io/assets/56664567/a469e60b-8e44-4c1a-a137-571d3430b6b8)

결론적으로 더 간단한 프로세스로 CI/CD 를 구성할 수 있다. 


[Github Code](https://github.com/min9805/CICD-AWS-GithubActions)


# 참고 

[AWS + Github Action + Docker로 spring 서버 자동배포하기](https://velog.io/@leedool3003/AWS-Github-Action-Docker%EB%A1%9C-spring-%EC%84%9C%EB%B2%84-%EC%9E%90%EB%8F%99%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0)
