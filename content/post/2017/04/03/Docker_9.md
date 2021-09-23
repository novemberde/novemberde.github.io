---
title: 만들어진 Docker image를 EC2 인스턴스에 배포하기
tags: [docker, ec2, deploy, image, ubuntu, 배포]
date: "2017-04-03"
aliases: ["/2017/04/03/Docker_9.html"]
---

## Summary
---------------------
 이제 모든 준비는 끝났다. Docker image를 EC2 인스턴스에 배포하자.

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

## Docker image를 받아오기
---------------------
 EC2 인스턴스에 SSH로 접속하자. 리눅스 계정은 이전에 만들었던 docker 로 로그인하자.

 로그인 후 docker에 이미지 리스트를 확인하자.

 ```bash
 # docker image의 목록 보기
 $ docker images
 ```

 아직 이미지를 pull한 적이 없으면 아무 리스트도 나오지 않을 것이다.

 이제 내 docker hub에서 image를 pull해보자

 image를 pull하기에 앞서서 docker hub에 내 이미지가 private인지 public인지 확인해볼 필요가 있다.
 만약 private으로 되어 있다면 pull을 해도 이미지를 찾을 수 없다고 나올것이다.
 이럴 때는 아래와 같이 로그인을 시도한다.

 ```bash
 # 이와 같이 커맨드를 실행하면 id와 password를 입력하는 커맨드를 만날 것이다.
 $ docker login
 ```

 로그인을 마쳤으면 이제 내 계정에 있는 image를 가져오자.

 ```bash
 # username에는 나의 docker hub의 name을 적자. 이는 오른쪽 위에서 내 설정에 들어가면 볼 수 있다.
 $ docker pull username:docker_node_server
 ```

 이미지를 성공적으로 pull하였으면 이제 run할 차례만 남았다.

 - 80포트로 동작하게 할 것. 우리 패키지는 3000번의 포트로 동작하니 포워딩시켜주어야 한다.
 - 컨테이너명은 my\_first\_server 로 해보자

 ```bash
 $ docker run --name my_first_server -p 80:3000 docker_node_server
 ```

 ---------------------
 잘 따라와 주었다. AWS에서 EC2 instance를 보면 public dns를 볼 수 있다.

 크롬브라우져를 켜고 복사/붙여넣기를 하면 우리의 사이트를 볼 수 있을 것이다.


---------------------