---
title: Docker Hub의 automated build를 사용하여 Docker image를 만들기
tags: [docker, hub, dockerhub, automated, build, image, 만들기]
date: "2017-04-02"
aliases: ["/2017/04/02/Docker_8.html"]
---

## Summary
---------------------
 이전에는 Dockerfile을 만들었다. 하지만 매번 build하는 것은 소모적인 시간이라고 생각한다. 그럼 만들어진 Dockerfile을 git repository에 push하면 Dockerfile이 빌드되는지 확인하는 것은 어떨까. docker hub의 automated_build기능을 사용하면 간단히 구현된다.

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

## Automated_build
---------------------

 Docker hub에서 지원해주는 이미지 빌딩시스템이다. docker hub와 github나 bitbucket 계정은 단 하나만 연동이 가능하며 다른 계정과는 연동할 수 없다.
 또한 만능으로 항상 build를 해주는 것이 아닌 역시 limit 이 존재한다.
 이 링크를 참고하길 바란다.
 [https://forums.docker.com/t/automated-build-resource-restrictions/1413](https://forums.docker.com/t/automated-build-resource-restrictions/1413)

---------------------
## Create Automated_build
---------------------

 docker hub의 메인페이지에서 로그인을 하면 오른쪽 위에 Create > Create Automated build 가 있다.

 ![docker_hub](/images/aws/ec2/docker_hub.png)

 ---------------------

 이것을 누르면 github나 bitbucket을 선택하라는 창이 나온다.

 우린 bitbucket respository를 사용하니 bitbucket을 선택하도록 하자.

 계정을 연동하면 내가 생성한 git respository list를 볼 수 있다.

 docker_node_server를 선택하면 아래와 같은 창을 볼 수 있을 것이다.

 내 이미지를 공유하고 싶다면 Visibility > public 으로 하면 되고 default는 private이다. 하지만 private의 갯수도 제한이 있다.

 ![docker_hub_create_automated_build](/images/aws/ec2/docker_hub_create_automated_build.png)

---------------------
 여기까지 왔으면 거의 끝났다.

 Build Settings 에 들어가면 내 Branch목록을 볼 수 있다. 우린 master로 작업하고 있으니 별다른 설정은 필요없다.

 Trigger 버튼을 누르고 Build Details를 들어가면 우리의 이미지가 생성되고 있는 것을 볼 수 있다.

![docker_hub_automated_build_detail](/images/aws/ec2/docker_hub_automated_build_detail.png)
![docker_hub_automated_build_detail_setting](/images/aws/ec2/docker_hub_automated_build_detail_setting.png)

---------------------

 빌드가 성공하면 Status가 Success로 바뀌고, Build 내용은 상세화면에서 볼 수 있다.
