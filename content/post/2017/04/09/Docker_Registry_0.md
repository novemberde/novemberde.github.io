---
title: 나만의 private docker registry 구성하기.
tags: [docker, registry, private, ec2, s3]
date: "2017-04-09T11:30:03+00:00"
aliases: ["/2017/04/09/Docker_Registry_0.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

## Summary
---
 Docker hub에 private image를 올리는 것은 제한이 있다. 
 개인 사용자의 경우 하나의 이미지만 private이 가능하고 organization의 경우에는 비용을 지불해야만 사용이 가능하다.
 이런점에 비추어 볼 때 우리는 private registry환경을 구축하고 싶다는 생각이 들 것이다.
 
 EC2에 개인 registry를 구축하고 local 또는 다른 서버에서 접근하는 방법에 대해서 진행해보겠다. 그리고 Amazon S3 를 이미지 저장소로 사용하겠다.

## Docker registry 구축하기
---

docker가 설치되어 있는 EC2에 접근하여 registry  이미지를 pull 해보자. 

- docker registry의 기본포트는 5000번이다.

```bash
# registry 이미지를 가져오기
$ docker pull registry
```
 
```bash
# registry를 실행하기
$ docker run -dit --name docker-registry -p 5000:5000 registry
```

## Docker image를 push하기
---

[도커허브](https://hub.docker.com)를 사용할 때는 <계정아이디>/registry:latest 처럼 tag명에 내 아이디가 들어가는 모양이었다.
하지만 private registry를 사용할 때는 <계정아이디>부분에 내 registry의 url주소를 사용하여야 한다.

localhost에서 테스트를 진행할테니 localhost:5000/hello-world:latest 이미지를 만들어보자.

```bash
# hello-world 이미지가 없으니 docker hub에서 pull하자.
$ docker pull hello-world

# localhost/hello-world 이미지를 만들어보자.
$ docker tag hello-world localhost:5000/hello-world
```

이미지를 만들었으니 내 registry에 push하자.

```bash
# 이미지 push하기
$ docker push localhost:5000/hello-world

# 이미지 확인하기
$ curl -X GET http://localhost:5000/v2/_catalog
# 출력 {"repositories":["hello-world"]}

# 태그 정보 확인하기
$ curl -X GET http://localhost:5000/v2/hello-world/tags/list
# 출력 {"name":"hello-world","tags":["latest"]}
```

## 원격지에서 Docker image를 push하기
---

지금 이미지의 태그명을 보면 localhost/~ 로 되어 있는 것을 볼 수 있다.
하지만 원격지에서는 특정도메인 또는 IP로 접근하기 때문에 localhost, 127.0.0.1을 사용할 수 없다.
gabia, godaddy 또는 AWS 53을 사용하여 DNS설정을 하는 방법과 직접 아이피로 접근해서 등록하는 방법 2가지가 있다.

이번에는 테스트용 도메인 docker-registry.kh-developer.info 로 DNS를 설정하여 docker registry를 사용하겠다.

gabia > 네임플러스 > 호스트(IP) 추가/관리 페이지에서 docker-registry를 추가하고 내 EC2 아이피를 할당한다.
![gabia](/images/aws/ec2/gabia.png)

- 아이피를 할당한 후에 현재 pc에서 docker에 push를 해보자.(로컬에도 docker가 설치되어 있어야 한다.)
- 포트를 5000번으로 registry를 생성했으니 5000번으로 접속하자
- EC2에 security group에서 inbound rule에서 5000번으로 설정해주자. my ip를 선택하여 다른 사람이 접근하지 못하도록 하자.


```bash
# 현재 이미지 목록 보기.
$ docker images

# 아직 hello-world가 없으므로 docker pull하기
$ docker pull hello-world

# docker-registry.kh-developer.info:5000/hello-world 이미지를 만들어보자.
$ docker tag hello-world docker-registry.kh-developer.info:5000/hello-world

# 이미지가 생성되었는지 확인해보자.
$ docker images

# push 해보자. 실패할 것이다.
$ docker push docker-registry.kh-developer.info:5000/hello-world
```

아래와 같은 메시지가 나오면서 실패할 것이다.

```bash
Get https://docker-registry.kh-developer.info:5000/v1/_ping: http: server gave HTTP response to HTTPS client
```

실패한 이유는 docker registry는 로컬머신에서 사용하는 것이 아니라면 https만 지원을 하기 때문이다.
그럼 원격지에서 접속하기 위해서는 docker registry 설정을 해주어야 한다.

현재의 docker registry 컨테이너를 내리고 다시 registry를 올려보자.

```bash
# docker registry 컨테이너 내리기
$ docker stop docker-registry && docker rm docker-registry
```

SSL 사설 인증서를 발급하자. 종잣돈이 많다면 인증서를 구입해도 괜찮다.

이번에는 개인 서명 SSL 인증서를 생성하겠다. openssl이 EC2 인스턴스에 설치되어 있을 것이다.

```bash
# openssl 버전 확인하기
$ openssl version

# cert.d 폴더에 개인키 생성하기. 비밀번호를 입력하자. 테스트를 위해 개인키 비밀번호는 test로 하겠다.
$ mkdir certs && cd certs && openssl genrsa -des3 -out server.key 2048

# 인증 요청서 생성
$ openssl req -new -key server.key -out server.csr

Country Name (2 letter code) [XX]:KR
State or Province Name (full name) []:Seoul
Locality Name (eg, city) [Default City]:Seongdonggu
Organization Name (eg, company) [Default Company Ltd]:NOVEMBERDE
Organizational Unit Name (eg, section) []:TEST
Common Name (eg, your name or your server\'s hostname) []:docker-registry.kh-developer.info
Email Address []:

# 생성된 파일 확인하기
$ ll

# 개인키에서 패스워드 제거하기
$ cp server.key server.key.origin && openssl rsa -in server.key.origin -out server.key && rm server.key.origin

# 인증서 생성하기. 1년으로 사용하겠다. 2년 3년할 수도 있다. server.crt파일이 생길 것이다.
$ openssl x509 -req -days 730 -in server.csr -signkey server.key -out server.crt
``` 

인증서를 발급했으니 registry를 다시 가동해보자.

```bash
$ docker run -d -p 5000:5000 --restart=always --name docker-registry \
  -v /home/<username>/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/server.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/server.key \
  registry
```

가동을 성공적으로 마쳤으면 다시 로컬에서 push를 해보자

```bash
# 다시 로컬환경으로 돌아와서 push하기
$ docker push docker-registry.kh-developer.info:5000/hello-world

The push refers to a repository [docker-registry.kh-developer.info:5000/hello-world]
Get https://docker-registry.kh-developer.info:5000/v1/_ping: x509: certificate signed by unknown authority
```

위와 같은 메시지가 나오면서 push가 되지 않을 것이다.

사설인증서이기 때문에 현재 사용하는 PC의 docker가 push하지 못하는 것이다.

windows 환경이라면 하단의 상태표시창에서 docker > setting > insecure-registry에서 docker-registry.kh-developer.info 를 설정해주어야 한다.

kitematic을 사용하고 있다면 아래와 같은 virtual box를 더블클릭하면 콘솔화면이 나타나는데 여기서 /var/lib/boot2docker/profile 파일을 수정해주어야 한다.

EXTRA_ARGS 에 --insecure-registry를 아래와 같이 추가한다.

![virtual_box](/images/aws/ec2/virtual_box.png)

추가 후에 docker를 restart하자.

---

이제 다시 docker push를 해보자. 성공적으로 push가 될 것이다.

```bash
# 다시 로컬환경으로 돌아와서 push하기
$ docker push docker-registry.kh-developer.info:5000/hello-world
```


### S3를 저장소로 사용하기
---

사용하기에 앞서서 AWS에 user를 생성하도록 하자.

AWS Menu > Security, Identity & Compliance > IAM 에서 user를 생성한다.
username은 docker-registry로 하고, Access type은 Programmatic access로 하자.

![add_user](/images/aws/ec2/add_user.png)

---

Permission은 Attach existing policies directly로 하여 S3 FullAccess를 선택하여 주자.(FullAccess가 불안하다면 [여기](https://docs.docker.com/registry/storage-drivers/s3/#s3-permission-scopes)를 참고하여 Policy를 생성하길 바란다.)

Create를 하면 Access Key와 Secret access key를 부여받는다. 잘 보관하도록 하자.

---

docker registry에서 S3에 접근할 수 있도록 설정하자.

```bash
# 기존의 registry를 내려주고, 새로 올리자.
$ docker stop docker-registry && docker rm docker-registry

# 새로 올리기
$ docker run -d -p 5000:5000 --restart=always --name docker-registry \
  -v /home/docker/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/server.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/server.key \
  -e REGISTRY_STORAGE=s3 \
  -e REGISTRY_STORAGE_S3_BUCKET=docker-registry.kh-developer \
  -e REGISTRY_STORAGE_S3_ACCESSKEY=ASEFWAF1232REWE \
  -e REGISTRY_STORAGE_S3_SECRETKEY=ASERWER1234WERFASER354SFDSDF1234 \
  -e REGISTRY_STORAGE_S3_REGION=ap-northeast-1 \
  registry

# 다시 로컬환경으로 돌아와서 push 해보기
$ docker push docker-registry.kh-developer.info:5000/hello-world
```

S3 bucket을 가면 storage가 형성될 것이다.


### Authentification 추가하기
---

여기까지 S3를 이미지 저장소로 사용하는 docker registry를 구성하였다면, 지금부터는 docker registry 접근에 대한 인증절차를 두려고 한다.

```bash
# ~/auth라는 디렉터리에 testuser를 아이디로 갖고 testpassword를 비밀번호로 갖게 해보자.
$ mkdir auth && docker run --entrypoint htpasswd registry:2 -Bbn testuser testpassword > auth/htpasswd


# docker registry container를 다시 실행해보자.
$ docker run -d -p 5000:5000 --restart=always --name docker-registry \
  -v /home/docker/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/server.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/server.key \
  -e REGISTRY_STORAGE=s3 \
  -e REGISTRY_STORAGE_S3_BUCKET=docker-registry.kh-developer \
  -e REGISTRY_STORAGE_S3_ACCESSKEY=ASEFWAF1232REWE \
  -e REGISTRY_STORAGE_S3_SECRETKEY=ASERWER1234WERFASER354SFDSDF1234 \
  -e REGISTRY_STORAGE_S3_REGION=ap-northeast-1 \
  -v /home/docker/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  registry

# 다시 로컬환경으로 돌아와서 docker push를 해보자
$ docker push docker-registry.kh-developer.info:5000/hello-world
98c944e98de8: Preparing
no basic auth credentials
```

위처럼 auth credentials이 없다고 나온다. docker login을 해주자.

```bash
$ docker login docker-registry.kh-developer.info:5000
Username: testuser
Password:
Login Succeeded

# 로그인이 됐다면 다시 push를 해주자
$ docker push docker-registry.kh-developer.info:5000/hello-world
```



### 2018-05-29 추가사항
---

위에 글을 보면 어렵사리 Docker registry 를 구축하였다. 하지만 살펴보면 곳곳에 문제점이 보일 것이다.
이글을 다시 살펴보면서 정리한 의문점은 다음과 같다.

1. 동시에 수십 수백대의 서버가 업데이트를 하는 경우에는 단일 registry 서버로 감당할 수 있을 것인가? 만약 그럴 수 없다면 어떻게 설계해야할까?
2. 이걸 구축하지 않고 편하게 사용할 수 있는 다른 Managed Service는 없을까?

먼저 첫번째 질문에 답을 하자면, docker registry 관련 문서에 다음과 같이 잘 나와 있다.

> Load balancing considerations
>
> One may want to use a load balancer to distribute load, terminate TLS or provide high availability. While a full load balancing setup is outside the scope of this document, there are a few considerations that can make the process smoother.
>
> The most important aspect is that a load balanced cluster of registries must share the same resources. For the current version of the registry, this means the following must be the same:
>
> Storage Driver
>
> HTTP Secret
>
> Redis Cache (if configured)
>
> Differences in any of the above cause problems serving requests. As an example, if you’re using the filesystem driver, all registry instances must have access to the same filesystem root, on the same machine. For other drivers, such as S3 or Azure, they should be accessing the same resource and share an identical configuration. The HTTP Secret coordinates uploads, so also must be the same across instances. Configuring different redis instances works (at the time of writing), but is not optimal if the instances are not shared, because more requests are directed to the backend.

[출처: https://docs.docker.com/registry/deploying/#load-balancing-considerations](https://docs.docker.com/registry/deploying/#load-balancing-considerations)

이 내용을 간단하게 요약하면 다음과 같다.
Load balancing을 고려한 설계를 한 경우이다. 웹어플리케이션 설계와 비슷한 방법으로 공통 스토리지는 Storage Driver를 통해 동일 Storage에 접근하도록 하고 Caching을 위해 Redis를 올려놓늗다. 또한 HTTP Secret을 통해 업로드하므로 모든 인스턴스는 동일한 HTTP Secret을 가져야한다.

Cluster를 인스턴스 Auto-scaling하듯이 여러 인스턴스를 배포하고 공통된 Storage에 접근하도록 설정한 다음 배포하면 되는 것이다. 그렇다면 동시에 많은 요청을 감당할 수 있다.

두번째로 Managed Service를 찾아보았다. 
docker hub에 Billing plan을 변경하여 private repository를 생성하는 방법이 있다.
또한 AWS ECR을 사용하여 사용하는 저장공간과 네트워크 비용만 지출할 수 있다.
만약 이렇게 비용을 지불하기 싫다면, 배포할 Artifact를 tar로 압축하여
사용할 서버에 던진 다음에 image를 압축해제해서 사용하면 된다.
이러한 방식은 docker save / load 명령어를 통해 사용해볼 수 있다.

요즘에는 AWS를 기본으로 사용하다보니 docker registry를 올려본지 오래되었다.
그렇지만 처음 내용을 참고하시는 분들이 계신 것 같아, 처음에 글을 쓴 목적과 달리 다른 방법을 사용하는 것을 추천하고 싶다.

tar로 git hash를 이용하여 versioning하고, 그 다음에 이 tar에 대해서 artifact를 관리하던지,
아니면 Managed service를 활용하여 운영리소스를 줄이는 방법이다.
피치 못할 사정으로 자체 IDC에 docker registry를 올려야 한다면 docker registry를 단일 컨테이너로 올리는 것이 아닌,
k8s나 swarm으로 해당 docker registry를 autoscaling group으로 묶어서 배포하면 될 것이다.

### References
---

- [http://zetawiki.com/wiki/%EB%A6%AC%EB%88%85%EC%8A%A4_%EA%B0%9C%EC%9D%B8%EC%84%9C%EB%AA%85_SSL_%EC%9D%B8%EC%A6%9D%EC%84%9C_%EC%83%9D%EC%84%B1](http://zetawiki.com/wiki/%EB%A6%AC%EB%88%85%EC%8A%A4_%EA%B0%9C%EC%9D%B8%EC%84%9C%EB%AA%85_SSL_%EC%9D%B8%EC%A6%9D%EC%84%9C_%EC%83%9D%EC%84%B1)
- [https://docs.docker.com/registry/deploying/#lets-encrypt](https://docs.docker.com/registry/deploying/#lets-encrypt)
- [https://docs.docker.com/registry/deploying/#native-basic-auth]()
- [http://www.notrudebuthonest.com/2016/02/kitematic-enable-insecure-registry/](http://www.notrudebuthonest.com/2016/02/kitematic-enable-insecure-registry/)
- 별도로 arn에 따라 policy를 주고 싶은 경우는 아래와 같은 policy를 넣어준다.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:GetBucketLocation",
                "s3:ListBucketMultipartUploads"
            ],
            "Resource": "arn:aws:s3:::*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:DeleteObject",
                "s3:ListMultipartUploadParts",
                "s3:AbortMultipartUpload"
            ],
            "Resource": "arn:aws:s3:::S3_BUCKET_NAME/*"
        }
    ]
}
```