---
title: EC2 meta data에 대해 알아보기
tags: [ec2, meta, data, metadata, instance, role, 알아보기, 사용법]
date: "2017-12-06T11:30:03+00:00"
aliases: ["/aws/2017/11/06/Instance_metadata.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---
## Summary
---
Amazon EC2의 설정을 자동으로 하기 위해선 인스턴스의 정보를 받아와서 설정할 수 있어야 한다.

예를 들어 특정 태그로 묶인 그룹에게 재가동시 소스코드 갱신과 서버 재가동의 명령어를 init.d에 등록했을 때
인스턴스 정보를 얻어온다면 개별적으로 인스턴스의 역할에 맞는 work load를 할당할 수 있을 것이다.

---
## meta-data 확인하기
---

Metadata에 대해서 찾아보니 위키백과에 아래와 같이 쓰여 있었다.

- 메타데이터(metadata)는 데이터(data)에 대한 데이터이다. 이렇게 흔히들 간단히 정의하지만 엄격하게는, Karen Coyle에 의하면 "어떤 목적을 가지고 만들어진 데이터 (Constructed data with a purpose)"라고도 정의한다. 가령 도서관에서 사용하는 서지기술용으로 만든 것이 그 대표적인 예이다. 지금은 온톨로지의 등장과 함께 기계가 읽고 이해할 수 있는 (Machine Actionable)한 형태의 메타데이터가 많이 사용되고 있다.

이 뜻을 인스턴스에 대입해보면, 인스턴스에 대한 데이터라고 생각해볼 수 있다.

여기에는 인스턴스가 가지고 있는 intance-id값과 같이 인스턴스의 특성을 알 수 있는 값들이 있을 것이다.

아래와 같이 예제를 통해 인스턴스의 Meta-data를 받아 본다.

169.254.169.254 라는 IP는 인스턴스의 정보를 받아 볼 수 있는 IP이다. 이 IP를 통해 정보를 받는다.

```sh
$ curl http://169.254.169.254/latest/meta-data/
ami-id
ami-launch-index
ami-manifest-path
block-device-mapping/
hostname
instance-action
instance-id
instance-type
local-hostname
local-ipv4
mac
metrics/
network/
placement/
profile
public-hostname
public-ipv4
public-keys/
reservation-id
security-groups
services/

# 인스턴스의 ami id
$ curl http://169.254.169.254/latest/meta-data/ami-id
ami-ccccccccc

# 인스턴스의 Security group
$ curl http://169.254.169.254/latest/meta-data/security-groups
my-security-group1
my-security-group2

# 인스턴스의 Role 정보
$ curl http://169.254.169.254/latest/meta-data/iam/info
{
  "Code" : "Success",
  "LastUpdated" : "2017-11-05T13:34:37Z",
  "InstanceProfileArn" : "arn:aws:iam::111111111:instance-profile/my-instance-role",
  "InstanceProfileId" : "AAAAAAAAAAAAAA"
}
```

---
## 고찰
---

지금까지 인스턴스의 정보를 받아보았다. 169.254.169.254에 get 요청을 하면 받는 인스턴스 정보를 특정하여 받아볼 수 있는데,
이는 활용하는 사람에 따라 다양하게 적용할 수 있을 것이다.

먼저 Security Group의 정보를 받을 경우에는 인스턴스가 가동할 때 원하는 Security-Group이 할당여부를 확인할 수 있을 것이다.
또한 Role 을 받아오면 인스턴스의 역할을 명확히 알 수 있기 때문에, 인스턴스의 소스코드를 갱신하여 인스턴스가 재시작 될 때마다
동작해야 할 명령어들을 구분지을 수 있다.

다른 활용 방법을 간단하게 구상해보자. 특정 S3에는 스크립트 파일들을 모아두고 role과 파일명을 일치해둔다.
그리고 AMI를 만드는데 내용은 다음과 같다. 가동될 때 meta-data로 role정보를 받아오면
role과 일치하는 S3버킷의 파일을 받아와 shell 또는 python 등 자신이 자신있는
script로 실행한다. script가 실행되면 각 인스턴스의 역할에 맞는 서버가 가동된다.

이런 식으로 설정을 한다면 Scaling Group이 Scale out이 될 때 큰 걱정없이 배포용 코드로 서버들이 가동될 수 있을 것이다.


---
## References
---

- [https://ko.wikipedia.org/wiki/%EB%A9%94%ED%83%80%EB%8D%B0%EC%9D%B4%ED%84%B0](https://ko.wikipedia.org/wiki/%EB%A9%94%ED%83%80%EB%8D%B0%EC%9D%B4%ED%84%B0)
- [http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/ec2-instance-metadata.html](http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)