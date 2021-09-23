---
title: Docker로 EC2에 Node 배포하기
tags: [docker, ec2, ubuntu, node, nodejs, 배포, deploy]
date: "2017-03-31"
aliases: ["/2017/03/31/Docker_0.html"]
---

## 소개
 Node JS 서버를 배포하려고 한다.
 매일 같이 하던 방식이지만 방법을 잊을 수도 있다는 생각이 들었다.
 이것을 보고 사람들이 노드 서버를 간결하게 배포하였으면 한다.

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

