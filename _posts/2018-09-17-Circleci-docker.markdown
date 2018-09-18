---
title: Circle CI에서 Docker Build 하기
category: docker
tags: circleci, docker, build, devops, 빌드, 도커
---
## Summary
---
현재 Terraform으로 ECS 인프라를 생성하고, CircleCI에서 docker image를 빌드하고 ECR에 푸시하는 과정을 자동화하고 있다.
이 과정에 있어서 docker image를 빌드하는 것에서 문제가 발생했다.

CircleCI에서 script를 실행하는 환경은 Container 기반으로 되어 있기 때문이다.
Docker image를 빌드하기 위해서는 docker가 설치되어 있는 환경이어야 했다.

docker-in-docker라는 개념으로 docker hub에 "docker"라는 docker image가 있다.
하지만 이 이미지로는 프로덕션 이미지를 빌드하고 배포할 때 사용하는 기본 패키지가 없는데, 이러한 기본패키지를 추가하여
CircleCI에서 docker를 빌드할 수 있는 이미지 생성하는 방법 및 CircleCI 설정을 정리해보았다.

## Dockerfile 작성하기
---

Dockerfile 작성은 어렵지 않다.

베이스 이미지로는 alpine linux 기반의 이미지를 사용한다. 또한 nodejs, npm, awscli, docker가 설치되어 있어야 한다.

작성한 Dockerfile은 다음과 같다.

```Dockerfile
FROM docker:stable

# Install node
RUN apk update && apk add --update nodejs nodejs-npm

# Install build-essentials
RUN apk add --virtual build-dependencies build-base gcc wget git

# Install aws-cli
RUN apk -v --update add \
        python \
        py-pip \
        groff \
        less \
        mailcap \
        && \
    pip install --upgrade awscli==1.14.5 s3cmd==2.0.1 python-magic && \
    apk -v --purge del py-pip && \
    rm /var/cache/apk/*

ENTRYPOINT []
CMD ["sh"]
```
[https://github.com/novemberde/node-awscli/blob/master/docker/Dockerfile](https://github.com/novemberde/node-awscli/blob/master/docker/Dockerfile)

여기서 docker:stable은 alpine linux 기반으로 docker가 사전 설치된 이미지인데, docker 재단에서 공식적으로 제공한다.
"docker" image는 docker in docker를 사용하기 위한 환경을 제공한다.
그 다음, 차례대로 nodejs, npm, build-essentials, awscli 등을 설치한다.

ENTRYPOINT는 docker:stable에서 Default로 docker-entrypoint.sh로 지정되어 있어 오류가 발생하기 때문에, 다시 재정의해주었다.

이렇게 생성된 이미지는 현재 [docker hub에 public](https://hub.docker.com/r/novemberde/node-awscli/)으로 올렸다.
태그는 docker로 되어 있으며, 이미지는 다음과 같은 명령어로 받아올 수 있다.

```sh
$ docker pull novemberde/node-awscli:docker
```

## CircleCI에서 설정하기
---

Docker가 설치되어 있는 docker image를 생성했으니 CircleCI에서 설정을 시작해보자.

평소와 달라지는 부분은 'setup_remote_docker' 부분이다.
이 설정을 추가하면 docker command에 대해서 격리된 새로운 환경에서 사용할 수 있게 해준다.


```yml
version: 2
jobs:
  build:
    docker:
      - image: novemberde/node-awscli:docker
    working_directory: ~/repo
    steps:
      - checkout

      ### This configuration is enable to build a docker image.
      - setup_remote_docker:
          docker_layer_caching: true
      ###
      - run: docker build -t test ./
```

## 확인해보기
---

CircleCI configuration을 마치고 git push를 하면 빌드가 시작된다.

샘플 Dockerfile에 대해서 빌드한 결과는 다음과 같다.

![docker-build](/images/circleci/docker-build.png)


## 고찰
---

alpine linux를 사용하는 이유는 기타 리눅스 배포판과 기본 사이즈의 차이가 많이 나기 때문이다.
참고할 표는 다음과 같다. 적은 용량은 도커 이미지를 가져올 때의 네트워크 부하를 줄여주고,
이미지 생성 및 사용에 대해서도 속도를 향상시키는 효과를 가져올 수 있다.

|   DISTRIBUTION |	VERSION |	SIZE    |
|---|---|---|
|   Debian |	Jessie	|   123MB    |
|   CentOS	|   7	|   193MB    |
|   Fedora	|   25	|   231MB    |
|   Ubuntu	|   16.04	|   118MB    |
|   Alpine	|   3.6	|   3.98MB    |


## References
---

- 참고자료: [https://nickjanetakis.com/blog/the-3-biggest-wins-when-using-alpine-as-a-base-docker-image](https://nickjanetakis.com/blog/the-3-biggest-wins-when-using-alpine-as-a-base-docker-image)
- 참고자료: [https://circleci.com/docs/2.0/building-docker-images/](https://circleci.com/docs/2.0/building-docker-images/)