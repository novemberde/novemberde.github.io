---
title: Node 서버를 바탕으로 Dockerfile로 만들기
tags: [dockerfile, docker, nodejs, node]
date: "2017-04-02T11:30:03+00:00"
aliases: ["/2017/04/02/Docker_7.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

## Summary
---------------------
 Dockerfile로 이미지로 관리하면 배포 및 관리가 간편하게 가능하다.
 여기서는 node를 베이스 이미지로하여 노드 서버를 배포할 수 있도록 준비한다.

---------------------

## 순서
---------------------
1. Amazon web service에 Ubuntu OS를 사용하는 EC2 인스턴스 생성하기
1. 접속 포트를 열어주고 별도의 Ubuntu 유저를 생성하기
1. EC2에 Docker를 설치하고 Ubuntu 유저에게 권한주기
1. Bitbucket을 사용하여 git repository 생성하기
1. Express JS를 사용하여 Node 서버 구축하기
1. PM2를 사용하여 EC2에 Node 서버 배포하기
1. Node 서버를 바탕으로 Dockerfile로 만들기
1. Docker Hub의 automated build를 사용하여 Docker image를 만들기
1. 만들어진 Docker image를 EC2 인스턴스에 배포하기

---------------------

## Dockerfile 이란?
---------------------

 Docker image의 설정 정보를 담고 있는 파일이다. 
 실제 운영 소프트웨어를 배포할 경우 node 베이스 이미지를 올린 다음 패키지를 설치하고 volume을 할당할 수도 있다.
 현재 이미지를 주기적으로 commit하여 백업하고 직접 docker exec를 하여 컨테이너 내에 실행명령어를 보내야하는 단점이 있었다.
 하지만 Dockerfile은 docker image를 생성할 때 source file을 가져와서 컨테이너 구동과 동시에 서버를 가동시킬 수 있다.
 이러한 docker container를 관리하는 방법으로는 docker-compose가 있는데 추후에 살펴볼 것이다.

---------------------
## Dockerfile 작성하기
---------------------

[https://hub.docker.com/](https://hub.docker.com/)에서 여러 베이스 이미지를 검색해볼 수 있다.
예를 들어 node 를 검색한다고 하자. 아래와 같은 화면을 볼 수 있을 것이다.

![docker_hub](/images/aws/ec2/docker_hub.png)

---------------------
맨 위의 node를 들어가면 Full Description에서 "7.8.0, 7.8, 7, latest (7.8/Dockerfile)"과 같은 글이 보일 것이다.

여기에 들어가면 Dockerfile이 어떻게 구성되어 있는지 확인할 수 있다.

- FROM : 베이스 이미지
- RUN : 실행 커맨드 라인
- ENV : 설정할 환경변수
- CMD : 데몬으로 실행할 명령어
- VOLUME : 호스트와 컨테이너의 디렉터리 공유
- COPY : 호스트에서 가져올 디렉터리 또는 파일
- EXPOSE : 노출할 포트 설정

![docker_hub_detail](/images/aws/ec2/docker_hub_detail.png)
![dockerfile_node](/images/aws/ec2/dockerfile_node.png)

---------------------

간단히 Dockerfile의 구성에 대해서 살펴봤으니 Dockerfile을 만들어보자.

우리가 만들 이미지의 구성은 아래와 같다
- 베이스이미지 : node
- 실행할 커맨드라인 : npm install -g pm2
- 환경변수 : NODE_ENV=production
- 데몬으로 실행할 명령어 : node app.js
- 호스트에서 가져올 디렉터리 또는 파일 : /home/docker/docker_node_server
- 노출할 포트: 80

만든 Dockerfile을 배포할 node js 서버 패키지에 만들자.

Dockerfile은 별도의 확장자가 없다.

파일 트리 구조는 아래와 같다.

```

├ docker_node_server
    ├ app.js
    ├ Dockerfile
    └ package.json

```
Dockerfile은 아래와 같다.

{% gist novemberde/ff49bcb8cbe9f1bab15220ae68b92a69 %}

---------------------

만들어진 Dockerfile을 빌드해보자


```bash
# 현재 폴더를 중심으로 한다.
docker build -t my-node ./
```

---------------------

build가 성공하면 git 에 push해주자.
---------------------
## References
- [https://hub.docker.com/](https://hub.docker.com/)