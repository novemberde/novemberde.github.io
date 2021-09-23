---
title: EC2에 Docker를 설치하고 Ubuntu 유저에게 권한주기
tags: [ec2, docker, 도커, 설치, 유저, 권한]
date: "2017-04-01"
aliases: ["/2017/04/01/Docker_3.html"]
---

## Summary
---------------------
 Docker의 개념을 간단히 살펴보고 EC2 ubuntu instance에 Docker를 설치해보자.

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

## Docker 란?
---------------------
 공식사이트의 소개를 보면 이렇게 말한다. 
##### Package software into standardized units for development, shipment and deployment
 패키지 소프트웨어를 개발 및 구축, 배포를 위한 표준화된 단위로.

 이말은 도커는 하나의 패키징된 소프트웨어로 만들어 배포를 하는 도구로 활용할 수 있고 도커는 이러한 단위라고 말하고 있다. 이전까지는  VMware, Microsoft Hyper-V(Virtual PC), Virtual Box와 같은 가상머신(Virtual Machine)에 리눅스를 올려 작업하였다.

 이미지를 백업하고 관리상의 이점은 분명히 있었지만 성능적인 이슈가 문제였다. 가상머신에는 OS이미지가 포함되어 있기 때문에 무겁고 네트워크적으로 가상화이미지에 데이터를 주고 받는 것도 부담된다. 또한, CPU가상화를 위한 기능이 아직까지는 실제 머신의 성능까지 이끌지는 못한다.

 하지만 이러한 문제를 해결하기 위해 나타난 것이 반가상화(Paravirtualization) 방식이다. OS자원은 호스트와 공유하여 이미지 용량의 감소된다. 가상화 레이어가 없기 때문에 커널과 직접적으로 사용하기 때문에 빠른 네트워크 응답과 하드웨어 성능을 이끌어낼 수 있다. 아래 이미지는 Docker의 성능분석 자료이다. Native와 동일한 성능을 내고 있음을 알 수 있다.

 ![docker_performance](/images/aws/ec2/docker_performance.png)
 * 출처:  [https://blogs.vmware.com/performance/2014/10/docker-containers-performance-vmware-vsphere.html](https://blogs.vmware.com/performance/2014/10/docker-containers-performance-vmware-vsphere.html)

 아래의 사진을 보면 각 컨테이너는 운영체제의 Kernel 즉 핵심을 공유하고 있고 그 안에서 경량화되고 독립적인, 실행가능한 패키지의 한 조각으로 존재한다.

 이러한 컨테이너는 이것들을 실행하기 위한 모든 것을 포함하고 있으며 code, system tools, system libraries와 같은 기능과 정보를 갖고 있다.

 이 개념을 사용하면 리눅스, 윈도우 베이스의 어플리케이션까지 환경에 상관없이 사용가능한 앱을 구축할 수 있다고 한다. (하지만 경험해본 결과 안되는 경우도 있다. 라즈베리파이와 같은 ARM 기반의 하드웨어에는 docker container가 정상적으로 동작하지 않을 때도 있고, MongoDB의 wiredtiger 같은 DB엔진은 windows에서 filesystem이 호환되지 않아 동작하지 않는다. 하지만 보편적으로는 문제없이 가동된다.)
 
 ![Docker](/images/aws/ec2/docker.png)

---------------------

## Docker 설치하기
---------------------
 도커에 대해 기본적인 개념을 알아봤다. 상세한 내용은 아래 References를 참고하면 좋은 내용을 얻을 수 있을 것이다.

 먼저 이전에 만든 EC2 인스턴스에 ubuntu 계정으로 SSH로 접속한다.

```bash
# 먼저 Package Database를 update한다.
$ sudo apt-get update
```

```bash
# Docker 공식페이지의 keyserver에서 GPG key를 추가한다.
$ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
```

```bash
# Docker repository를 APT sources에 추가한다.
$ sudo apt-add-repository 'deb https://apt.dockerproject.org/repo ubuntu-xenial main'
```

```bash
# 새로 추가된 Docker package를 업데이트한다.
$ sudo apt-get update
```

```bash
# Default Ubuntu 16.04 repository 대신에 Docker repository로 부터 install하는 것을 확인한다.
$ apt-cache policy docker-engine
```

```bash
# docker-engine을 설치한다.
$ sudo apt-get install -y docker-engine
```

```bash
# Daemon에서 docker가 실행되고 있는지 확인한다. CentOS의 경우에는 별도로 Daemon으로 실행해주어야 한다.
$ sudo systemctl status docker
```

```bash
# 현재 접속한 ubuntu 계정을 docker 실행 그룹에 포함시켜준다. 이전에 생성한 docker 계정도 포함시켜주자.
$ sudo usermod -aG docker $(whoami)
$ sudo usermod -aG docker docker
```

---------------------

## Docker 실행해보기
---------------------

```bash
# Docker의 images를 pull해서 확인해보자.
$ docker pull hello-world
$ docker images
```

```bash
# Docker container를 생성해보자.
$ docker run hello-world
```

```bash
# Docker container의 상태를 확인해보자.
$ docker ps
```

여기서 docker ps를 할 경우에 아무것도 나타나지 않는 것을 볼 수 있다. container가 Daemon에서 동작하는 것이 아닌 한번 실행하고 종료되었기 때문에 실행되고 있는 컨테이너 목록에 나타나지 않는 것이다. 모든 컨테이너의 상태를 보려면 아래와 같이 확인한다.

```bash
# 모든 Docker container의 상태를 확인해보자.
$ docker ps -a
```

컨테이너의 STATUS를 보면 Exited (0) ~~~ ago 라는 문구를 볼 수 있다. 이미 종료되었다는 의미이다.

컨테이너를 삭제해보자. CONTAINER_ID 부분에 삭제하고자 하는 docker container의 아이디를 입력한다. 전부 입력할 필요 없이 첫 글자 또는 두글자만 입력해도 삭제된다. 만약에 여러 컨테이너일 경우에는 구분되는 값까지만 입력하면 삭제된다.

```bash
# Docker container를 삭제하기
$ docker rm CONTAINER_ID
```


## References
- [https://www.docker.com/what-docker](https://www.docker.com/what-docker)
- [http://pyrasis.com/Docker/Docker-HOWTO](http://pyrasis.com/Docker/Docker-HOWTO)
- [https://www.slideshare.net/pyrasis/docker-docker-38286477](https://www.slideshare.net/pyrasis/docker-docker-38286477)
- [https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html](https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html)
- [https://docs.docker.com/engine/installation/linux/ubuntu/](https://docs.docker.com/engine/installation/linux/ubuntu/)
- [https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04)
