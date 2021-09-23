---
title: Docker를 활용하여 AWS의 Elastic Cache 에 접근하기.
tags: [vpc, aws, elasticcache, elasticsearch, access, docker, redis, internal, 접근, 접속]
date: "2017-05-09T11:30:03+00:00"
aliases: ["/2017/04/30/VPC_Redis_0.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

## Summary
---------------------
 Localhost에서 elastic cache에는 원칙적으로 접근할 수 없다.
별도로 VPC내에 Virtual Private Gateways나 Customer Gateways를 사용하여 VPC내에 존재하는 Elasic Cache에 접근할 수 있다.
하지만 이는 별도의 설정과정이 들어가므로 귀찮았다.
그래서 바로 VPC내의 EC2 인스턴스에서 Elastic Cache에 접근하였다.
Docker 이미지 중에서 Redis-cli가 있기 때문에 손쉽게 별도의 설치과정 없이 접근할 수 있다.

 이 과정은 이미 elastic cache를 사용하고 있고 인스턴스에 Docker가 설치되어 있다는 전제 하에 진행한다.

---------------------

## Docker hub에서 redis-cli 이미지를 받아 접속하자.
---------------------

 아래와 같은 Dockerfile을 기반으로 redis client를 사용할 수 있다.

 이 이미지를 [도커허브](https://hub.docker.com/r/novemberde/redis-cli/)에 등록해 놓았다.

```dockerfile
FROM redis

ENTRYPOINT [ "redis-cli" ]
```

---

이미지는 찾았으니 같은 VPC내에 Elastic Cache를 사용하는 EC2 인스턴스에 SSH로 접속한다.

```bash
# docker image를 가져온다.
$ docker pull novemberde/redis-cli

# redis client를 실행하자.
$ docker run --name redis-cli -it novemberde/redis-cli
not connected>
```

---

여기까지 진행했으면 거의 끝났다.

not connected라고 나와 있다. 이제 VPC내의 redis에 접속하자. redis의 url을 복사한 다음 접속하자.

```bash
# redis 에 접속하기
not connected> connect $URL $PORT
```

---
접속이 완료되었다. 이제 하고싶은 것을 할 수 있다.

개발 환경중일 경우에는 Redis를 가끔 비울 경우가 있다.

다음의 명령어들을 참고하자.
```bash
# 현재 접속한 DB의 모든 정보를 지우기
192.168.99.100:6379> flushdb

# 모든 데이터를 지우기
192.168.99.100:6379> flushall
```