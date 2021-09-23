---
title: AWS Configure 여러 계정으로 스위칭하며 사용하기
tags: [aws, configure, profile, switching, switch, 바꾸기]
date: "2018-06-20T11:30:03+00:00"
aliases: ["/aws/2018/06/20/AWS-config-switching.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

## Summary
---
개인용 개발계정, 회사계정, 워크샵 전용 계정 등등 여러 계정들을 사용하다보니 Default로 Access Key ID 와 Secret Access Key를
관리하고 싶어졌다. Default로 두고 사용하다가 잘못하면 회사계정에 잘못된 인프라를 생성 및 변경할 수도 있기 때문이다.


## AWS Configure --profile
---

기본적인 aws cli를 설정하는 것은 어렵지 않다.
AWS Console의 IAM에서 유저를 생성하고 Access Key를 생성하면 된다.
생성된 키를 통해 로컬이 AWS의 권한을 사용하도록 설정하는 것은 다음과 같다.

```sh
# 생성된 키와 리전을 입력하면 된다.
$ aws configure
AWS Access Key ID [****************aaaa]:
AWS Secret Access Key [****************aaaa]:
Default region name [ap-northeast-2]:
Default output format [json]:
```

그렇다면 여러 계정을 관리하려면 어떻게 해야할까?

--profile 옵션을 사용하면 어렵지 않다.

```sh
$ aws configure --profile testUser
AWS Access Key ID [****************aaaa]:
AWS Secret Access Key [****************aaaa]:
Default region name [ap-northeast-2]:
Default output format [json]:
```

새로운 유저의 Profile을 입력하였으니 명령어를 통해 리소스가 정말 다르게 나오는지 확인해본다.

```sh
# 생성한 버킷의 리스트가 출력된다.
$ aws s3 ls --profile testUser
```

3-party도구를 사용하다보면 Default User를 피치못할 사정으로 사용하게 된다.
그런 상황에서는 Default User를 스위칭해가며 사용한다.

```sh
# 환경변수로 default profile을 등록하여 준다.
$ export AWS_DEFAULT_PROFILE=testUser

# 만일 윈도우 사용자라면 set을 사용한다.
$ set AWS_DEFAULT_PROFILE=testUser

# 방금 전에 --profile 옵션과 함께 출력됐던 버킷의 리스트가 출력된다.
$ aws s3 ls

# 현재 사용하고 있는 default profile user가 출력된다.
$ aws configure list
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                 testUser           manual    --profile
access_key     ****************aaaa shared-credentials-file
secret_key     ****************aaaa shared-credentials-file
    region           ap-northeast-2      config-file    ~/.aws/config
```

이렇게 하면 당장은 되는 것처럼 보이지만 다른 Terminal을 열어서 해보면 되지 않는다.

```sh
$ aws s3 ls
An error occurred (InvalidAccessKeyId) when calling the ListBuckets operation: The AWS Access Key Id you provided does not exist in our records.
```

당황하지 말고 ~/.bashrc 또는 ~/.zshrc파일의 마지막 라인에
"export AWS_DEFAULT_PROFILE=testUser"를 추가한다.

#### ~/.zshrc 또는 ~/.bashrc

```sh
...
...
export AWS_DEFAULT_PROFILE=testUser
```

```sh
# 재설정한다.
$ source ~/.zshrc # 또는 source ~/.bashrc

# Default로 설정이 완료되었다.
$ aws s3 ls
```

하지만 변동 가능한 환경변수에 대한 설정정보를 bashrc에 넣는 것은 바람직해보이지 않는다.
귀찮더라도 이정도는 매번 손으로 설정하는 것이 위험 부담을 줄이는 길이라고 생각한다.

## References
---

- [https://docs.aws.amazon.com/cli/latest/reference/configure/list.html](https://docs.aws.amazon.com/cli/latest/reference/configure/list.html)