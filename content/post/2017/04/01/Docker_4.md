---
title: Bitbucket을 사용하여 git repository 생성하기
tags: [bitbucket, git, repository]
date: "2017-04-01"
aliases: ["/2017/04/01/Docker_4.html"]
---

## Summary
---------------------
 Bitbucket을 사용하여 git respository를 생성한다. 보통은 github로 관리하지만 github는 private한 repository를 사용하기 위해서는 사용료를 내야한다. 하지만 Bitbucket은 무제한적인 repository를 생성할 수 있으며 repository의 크기만 제한되어 있다. 2GB까지 사용이 가능한데 보통 코드로만 200MB도 사용하기 어려운 관계로 사실상 무제한이라고 볼 수 있다.

---------------------

## 순서
---------------------
1. Amazon web service에 Ubuntu OS를 사용하는 EC2 인스턴스 생성하기
1. 접속 포트를 열어주고 별도의 Ubuntu 유저를 생성하기
1. EC2에 Docker를 설치하고 Ubuntu 유저에게 권한주기
1. Bitbucket을 사용하여 git repository 생성하기
1. Express JS를 사용하여 Node 서버 구축하기
1. 테스트로 PM2를 사용하여 EC2에 Node 서버 배포하기
1. Node 서버를 바탕으로 Dockerfile로 만들기
1. Docker Hub의 automated build를 사용하여 Docker image를 만들기
1. 만들어진 Docker image를 EC2 인스턴스에 배포하기

---------------------

## Bitbucket Repository 생성하기
---------------------
먼저 [bitbucket.org/](https://bitbucket.org/)에 접속하여 회원가입을 진행하고 로그인한다. 

로그인을 하면 아래와 같은 대시보드 화면이 나타난다.

왼쪽위에서 Repositories > Create repository 를 클릭하면 생성화면으로 넘어갈 수 있다.

생성은 Default로 private repository로 되어 있으며 Repository name만 입력하면 손쉽게 생성할 수 있다.

최종 목적은 docker_node_server 이므로 이렇게 Repository name을 선정하겠다.

![bitbucket_main](/images/aws/ec2/bitbucket_main.png)

---------------------

## Source Tree 설치하기
---------------------
만들어진 Respository를 효과적으로 관리하기 위해서는 GUI로 버전정보를 확인할 수 있는 [Source Tree](https://www.sourcetreeapp.com/)가 사용하기 적합하다.

Git repository를 Github로 사용하면 [Github desktop](https://desktop.github.com/)을 사용하는 것도 좋다.

[Source Tree](https://www.sourcetreeapp.com/)를 설치하고 계정정보를 입력하여 로그인을 하면 기본 준비는 끝난다.


---------------------
메인화면에서 왼쪽 상단의 복제/생성 버튼을 누르면 아래와 같은 화면을 볼 수 있다. 소스 경로 / URL 은 Bitbucket에서 docker_node_server의 상세페이지로 넘어가면 볼 수 있다. 오른쪽 맨 위에 https를 선택하면 해당 url을 복사할 수 있다. 이값을 붙여넣도록 하자.

목적지 경로는 로컬 PC에 저장할 위치를 선택하면 된다.

![source_tree](/images/aws/ec2/source_tree.png)

