---
title: AWS Configure 여러 계정으로 스위칭하며 사용하기
category: aws
tags: aws, configure, profile, switching, switch, 바꾸기
---

## Summary

개인용 개발계정, 회사계정, 워크샵 전용 계정 등등 여러 계정들을 사용하다보니 Default로 Access Key ID 와 Secret Access Key를
관리하고 싶어졌다. Default로 두고 사용하다가 잘못하면 회사계정에 잘못된 인프라를 생성 및 변경할 수도 있기 때문이다.


## AWS Configure --profile

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

## AWS Console의 다중 세션 지원 (2025년 1월 업데이트)

로컬에서는 위와 같이 프로파일을 스위칭하며 사용하지만, AWS Console에서도 계정을 자주 전환해야 하는 경우가 많다.
이런 불편함을 해소하기 위해 2025년 1월부터 AWS Management Console에서 여러 계정을 동시에 로그인할 수 있게 되었다.
단일 브라우저에서 최대 5개의 세션까지 로그인이 가능하며, 서로 다른 계정이나 같은 계정의 
root, IAM, 페더레이션 역할들을 조합해서 사용할 수 있다.

AWS에서는 모범 사례로 여러 계정을 사용하여 애플리케이션을 확장하는 것을 권장한다.
개발, 테스트, 프로덕션처럼 환경별로 계정을 나누고, 문제 해결이나 다른 작업을 할 때 
여러 계정의 리소스 설정과 상태를 비교하곤 한다.
이제 AWS Console의 다중 세션 기능을 사용하면 여러 계정에 동시에 로그인해서 한 브라우저에서 리소스를 관리할 수 있다.

다중 세션은 다음과 같이 활성화할 수 있다:
1. AWS Console 로그인
2. 계정 메뉴 선택
3. "Turn on multi-session" 선택

이 기능은 모든 상업용 리전에서 사용할 수 있으며, 계정 메뉴에서 언제든 끌 수 있다.

## References

- [https://docs.aws.amazon.com/cli/latest/reference/configure/list.html](https://docs.aws.amazon.com/cli/latest/reference/configure/list.html)
- [https://aws.amazon.com/ko/about-aws/whats-new/2025/01/aws-management-console-simultaneous-sign-in-multiple-accounts/](https://aws.amazon.com/ko/about-aws/whats-new/2025/01/aws-management-console-simultaneous-sign-in-multiple-accounts/)