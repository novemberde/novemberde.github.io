---
title: PM2를 사용하여 EC2에 Node 서버 배포하기
tags: [pm2, ec2, nodejs, 서버, 배포]
date: "2017-04-02T11:30:03+00:00"
aliases: ["/2017/04/02/Docker_6.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

## Summary
---------------------
 node js 패키지 중에서 PM2를 사용하여 웹서버를 배포하자.

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

## 웹서버에 node js를 설치하고 PM2를 설치하기
---------------------

이전에 생성한 EC2 인스턴스에 ubuntu계정으로 SSH 접속한 후 Node js를 설치하자.

apt-get을 통해 설치하면 간단하게 node 서버 및 npm 패키지를 설치할 수 있다.

```bash
# node 7.x대 버전으로 받자.
$ curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
$ sudo apt-get install -y nodejs

# node-gyp 패키지와 같이 native addon을 컴파일하고 실행하려면 build-essential이 필요하다.
$ sudo apt-get install -y build-essential

# PM2를 전역적으로 사용하기 위해 -g 옵션을 붙여 설치하자.
$ sudo npm install -g pm2
```


---------------------


 docker 계정으로 File zilla를 사용하여 접속하자

/home/docker/ 디렉터리에 현재까지 작업한 node 서버를 node_modules를 제외하고 전송한다.

![file_zilla](/images/aws/ec2/file_zilla.png)

---------------------

 docker 계정으로 SSH접속을 한 후에 pm2로 서버를 실행시키자. PM2에 대한 자세한 사용법은 [PM2 공식사이트](http://pm2.keymetrics.io/) 를 참고하면 된다.

```bash
# 천천히 명령어 한줄씩 실행하는 방법.
$ cd docker_node_server
$ npm install
$ pm2 start app.js

# 디렉터리 이동없이 실행.
$ npm install --prefix /home/docker/docker_node_server && pm2 start /home/docker/docker_node_server
```

![pm2](/images/aws/ec2/pm2.png)

---------------------




## References
- [https://nodejs.org/ko/download/package-manager/](https://nodejs.org/ko/download/package-manager/)
- [http://pm2.keymetrics.io/](http://pm2.keymetrics.io/)