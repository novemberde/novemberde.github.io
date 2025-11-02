---
title: AWS Configure 여러 계정으로 스위칭하며 사용하기
tags: [aws, configure, profile, switching, switch, 바꾸기, assume-role, SSO]
date: "2018-06-20T11:30:03+00:00"
aliases: ["/aws/2018/06/20/AWS-config-switching.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

개인 프로젝트, 회사 업무, 클라이언트 작업 등 여러 AWS 계정을 동시에 관리하다 보면 계정 전환이 일상적인 작업이 됩니다. 실수로 프로덕션 계정에서 개발용 리소스를 생성하거나, 반대로 개발 계정에 운영 데이터를 배포하는 사고를 방지하려면 계정 전환을 체계적으로 관리해야 합니다.

이 글에서는 AWS CLI를 사용한 계정 전환의 다양한 방법을 실제 사용 사례와 함께 살펴보겠습니다.

## 기본적인 Profile 설정

AWS CLI를 처음 설정할 때는 `aws configure` 명령어를 사용합니다. IAM에서 사용자를 생성하고 Access Key를 발급받은 후, 다음과 같이 설정합니다:

```sh
$ aws configure
AWS Access Key ID [****************aaaa]:
AWS Secret Access Key [****************aaaa]:
Default region name [ap-northeast-2]:
Default output format [json]:
```

이렇게 설정하면 `~/.aws/credentials`와 `~/.aws/config` 파일에 default 프로필이 생성됩니다. 하지만 여러 계정을 사용한다면 default 프로필만으로는 부족합니다.

## 방법 1: Profile 옵션 사용하기

가장 기본적이면서도 안전한 방법은 `--profile` 옵션을 명시적으로 사용하는 것입니다.

```sh
# 회사 계정용 프로필 생성
$ aws configure --profile company
AWS Access Key ID [None]: AKIA...
AWS Secret Access Key [None]: ...
Default region name [None]: ap-northeast-2
Default output format [None]: json

# 개인 계정용 프로필 생성
$ aws configure --profile personal
AWS Access Key ID [None]: AKIA...
AWS Secret Access Key [None]: ...
Default region name [None]: us-east-1
Default output format [None]: json
```

프로필별로 명령어를 실행할 때는 다음과 같이 사용합니다:

```sh
# 회사 계정의 S3 버킷 목록 조회
$ aws s3 ls --profile company

# 개인 계정의 EC2 인스턴스 목록 조회
$ aws ec2 describe-instances --profile personal
```

이 방법의 장점은 명확합니다. 매번 어떤 계정을 사용하는지 명시하기 때문에 실수할 여지가 적습니다. 단점은 매번 `--profile` 옵션을 입력해야 한다는 번거로움이 있습니다.

### 현재 프로필 확인하기

지금 어떤 프로필을 사용하고 있는지 확인하려면:

```sh
$ aws configure list
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                  company           manual    --profile
access_key     ****************AKIA shared-credentials-file
secret_key     ****************XXXX shared-credentials-file
    region           ap-northeast-2      config-file    ~/.aws/config
```

## 방법 2: 환경변수로 전환하기

특정 작업을 하는 동안 하나의 프로필을 계속 사용한다면, 환경변수로 설정하는 것이 편리합니다.

```sh
# Linux/macOS
$ export AWS_PROFILE=company
$ aws s3 ls  # company 프로필 사용

# Windows (Command Prompt)
$ set AWS_PROFILE=company

# Windows (PowerShell)
$ $env:AWS_PROFILE="company"
```

이제 `--profile` 옵션 없이도 지정한 프로필이 사용됩니다. 하지만 주의할 점이 있습니다:

**주의사항:**
- 환경변수는 현재 터미널 세션에만 적용됩니다
- 새 터미널을 열면 다시 설정해야 합니다
- 실수로 다른 계정을 사용할 수 있으므로 작업 전 항상 확인하세요

```sh
# 현재 사용 중인 프로필 확인
$ echo $AWS_PROFILE
company

# 프로필 해제
$ unset AWS_PROFILE
```

### Shell 설정 파일에 추가하기 (권장하지 않음)

`~/.bashrc`나 `~/.zshrc`에 다음을 추가할 수도 있습니다:

```sh
export AWS_PROFILE=company
```

하지만 이 방법은 권장하지 않습니다. 모든 터미널에서 자동으로 해당 프로필이 활성화되어, 다른 계정을 사용하려고 할 때 혼란을 일으킬 수 있습니다. 명시적으로 설정하는 것이 더 안전합니다.

## 방법 3: AWS SSO (Single Sign-On) 사용하기

여러 AWS 계정을 관리하는 조직이라면 AWS SSO가 가장 현대적이고 안전한 방법입니다. Access Key를 직접 관리하지 않고, 임시 자격 증명을 자동으로 발급받습니다.

### SSO 프로필 설정

```sh
$ aws configure sso
SSO session name (Recommended): my-org
SSO start URL [None]: https://my-org.awsapps.com/start
SSO region [None]: ap-northeast-2
SSO registration scopes [None]: sso:account:access
```

브라우저가 열리면 로그인하고, 사용할 계정과 권한을 선택합니다:

```sh
There are 3 AWS accounts available to you.
> DevelopmentAccount, dev-account@example.com (123456789012)
  ProductionAccount, prod-account@example.com (210987654321)
  StagingAccount, staging-account@example.com (111222333444)

Using the account ID 123456789012
There are 2 roles available to you.
> AdministratorAccess
  ReadOnlyAccess

CLI default client Region [ap-northeast-2]:
CLI default output format [json]:
CLI profile name [AdministratorAccess-123456789012]: dev-admin
```

이렇게 설정하면 `~/.aws/config`에 다음과 같은 프로필이 생성됩니다:

```ini
[profile dev-admin]
sso_session = my-org
sso_account_id = 123456789012
sso_role_name = AdministratorAccess
region = ap-northeast-2
output = json

[sso-session my-org]
sso_start_url = https://my-org.awsapps.com/start
sso_region = ap-northeast-2
sso_registration_scopes = sso:account:access
```

### SSO 로그인 및 사용

```sh
# SSO 로그인 (브라우저가 열림)
$ aws sso login --profile dev-admin

# 로그인 후 명령어 실행
$ aws s3 ls --profile dev-admin

# 또는 환경변수 설정
$ export AWS_PROFILE=dev-admin
$ aws s3 ls
```

SSO의 장점:
- Access Key를 로컬에 저장하지 않아 보안성이 높음
- 임시 자격 증명이 자동으로 갱신됨
- 조직의 중앙 권한 관리 시스템과 연동 가능
- 다단계 인증(MFA) 통합

## 방법 4: AssumeRole을 통한 역할 전환

하나의 계정에서 다른 계정의 역할을 일시적으로 맡아야 할 때 사용합니다. 예를 들어, 개발 계정에서 프로덕션 계정의 리소스에 접근해야 하는 경우입니다.

### AssumeRole 프로필 설정

먼저 기본 프로필을 설정합니다:

```sh
$ aws configure --profile base-account
```

그 다음 `~/.aws/config` 파일을 편집하여 역할 전환 설정을 추가합니다:

```ini
[profile base-account]
region = ap-northeast-2
output = json

[profile production-role]
role_arn = arn:aws:iam::210987654321:role/ProductionAccessRole
source_profile = base-account
region = ap-northeast-2
```

이제 `production-role` 프로필을 사용하면 자동으로 AssumeRole이 실행됩니다:

```sh
$ aws s3 ls --profile production-role
```

### MFA를 요구하는 AssumeRole

보안을 강화하기 위해 역할 전환 시 MFA를 요구할 수 있습니다:

```ini
[profile production-role-mfa]
role_arn = arn:aws:iam::210987654321:role/ProductionAccessRole
source_profile = base-account
mfa_serial = arn:aws:iam::123456789012:mfa/myuser
region = ap-northeast-2
```

명령어 실행 시 MFA 토큰을 입력하라는 프롬프트가 나타납니다:

```sh
$ aws s3 ls --profile production-role-mfa
Enter MFA code for arn:aws:iam::123456789012:mfa/myuser:
```

## 방법 5: 프로필 전환 헬퍼 스크립트

자주 프로필을 전환한다면, 간단한 shell 함수를 만들어 사용할 수 있습니다.

### ~/.zshrc 또는 ~/.bashrc에 추가

```sh
# AWS 프로필 전환 함수
awsp() {
    if [ -z "$1" ]; then
        echo "현재 프로필: ${AWS_PROFILE:-default}"
        echo "사용 가능한 프로필:"
        grep '\[profile' ~/.aws/config | sed 's/\[profile \(.*\)\]/  - \1/'
        grep '^\[' ~/.aws/credentials | sed 's/\[\(.*\)\]/  - \1/' | grep -v 'default'
    else
        export AWS_PROFILE=$1
        echo "프로필이 '$1'(으)로 변경되었습니다"
        aws configure list
    fi
}

# AWS 프로필 해제 함수
awsp-clear() {
    unset AWS_PROFILE
    echo "AWS_PROFILE 환경변수가 해제되었습니다"
}
```

사용 예시:

```sh
# 현재 프로필 확인 및 사용 가능한 프로필 목록 보기
$ awsp
현재 프로필: company
사용 가능한 프로필:
  - company
  - personal
  - dev-admin
  - production-role

# 프로필 전환
$ awsp personal
프로필이 'personal'(으)로 변경되었습니다
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                 personal environment    AWS_PROFILE

# 프로필 해제
$ awsp-clear
AWS_PROFILE 환경변수가 해제되었습니다
```

## 방법 6: direnv를 활용한 프로젝트별 자동 전환

프로젝트마다 다른 AWS 계정을 사용한다면, `direnv`를 사용하여 디렉토리 진입 시 자동으로 프로필을 전환할 수 있습니다.

### direnv 설치 및 설정

```sh
# macOS
$ brew install direnv

# Linux
$ sudo apt-get install direnv  # Ubuntu/Debian
$ sudo yum install direnv      # CentOS/RHEL

# shell에 hook 추가 (~/.zshrc 또는 ~/.bashrc)
eval "$(direnv hook zsh)"  # zsh 사용자
eval "$(direnv hook bash)" # bash 사용자
```

### 프로젝트별 설정

각 프로젝트 디렉토리에 `.envrc` 파일을 생성합니다:

```sh
# ~/projects/company-project/.envrc
export AWS_PROFILE=company
export AWS_REGION=ap-northeast-2

# ~/projects/personal-project/.envrc
export AWS_PROFILE=personal
export AWS_REGION=us-east-1
```

디렉토리에 진입하면 자동으로 환경변수가 설정됩니다:

```sh
$ cd ~/projects/company-project
direnv: loading .envrc
direnv: export +AWS_PROFILE +AWS_REGION

$ echo $AWS_PROFILE
company

$ cd ~/projects/personal-project
direnv: loading .envrc
direnv: export ~AWS_PROFILE ~AWS_REGION

$ echo $AWS_PROFILE
personal
```

처음 사용할 때는 보안을 위해 명시적으로 허용해야 합니다:

```sh
$ direnv allow .
```

## 실전 활용 시나리오

### 시나리오 1: 프리랜서 개발자

개인 프로젝트, 클라이언트 A, 클라이언트 B의 계정을 관리하는 경우:

```sh
# 프로필 설정
aws configure --profile personal
aws configure --profile client-a
aws configure --profile client-b

# 프로젝트별로 direnv 설정
# ~/projects/my-app/.envrc
export AWS_PROFILE=personal

# ~/projects/client-a-project/.envrc
export AWS_PROFILE=client-a

# ~/projects/client-b-project/.envrc
export AWS_PROFILE=client-b
```

이렇게 하면 프로젝트 디렉토리로 이동만 하면 자동으로 올바른 계정이 활성화됩니다.

### 시나리오 2: 스타트업 개발자

개발, 스테이징, 프로덕션 환경을 별도 계정으로 분리한 경우:

```sh
# SSO 설정으로 모든 계정 연결
aws configure sso  # dev-account 프로필 생성
aws configure sso  # staging-account 프로필 생성
aws configure sso  # production-account 프로필 생성

# 하루 시작 시 로그인
aws sso login --profile dev-account

# 작업에 따라 환경변수 전환
export AWS_PROFILE=dev-account        # 개발 중
export AWS_PROFILE=staging-account    # 통합 테스트
export AWS_PROFILE=production-account # 긴급 패치
```

### 시나리오 3: 대기업 개발자

조직 계정과 역할 기반 접근 제어를 사용하는 경우:

```ini
# ~/.aws/config
[profile base]
region = ap-northeast-2

[profile dev-admin]
role_arn = arn:aws:iam::111111111111:role/DeveloperRole
source_profile = base

[profile prod-readonly]
role_arn = arn:aws:iam::222222222222:role/ReadOnlyRole
source_profile = base
mfa_serial = arn:aws:iam::123456789012:mfa/myuser

[profile prod-admin]
role_arn = arn:aws:iam::222222222222:role/AdminRole
source_profile = base
mfa_serial = arn:aws:iam::123456789012:mfa/myuser
```

개발은 자유롭게, 프로덕션은 조회만, 긴급 상황 시 MFA로 관리자 권한 획득하는 구조입니다.

## 보안 모범 사례

1. **Access Key 관리**
   - Access Key를 코드에 하드코딩하지 마세요
   - 주기적으로 키를 로테이션하세요
   - 가능하면 SSO나 AssumeRole을 사용하세요

2. **프로필 사용**
   - 프로덕션 계정은 절대 default 프로필로 설정하지 마세요
   - 명시적으로 `--profile` 옵션을 사용하거나 환경변수를 설정하세요

3. **권한 최소화**
   - 필요한 권한만 부여하세요
   - 읽기 전용 작업에는 읽기 전용 역할을 사용하세요

4. **MFA 활성화**
   - 중요한 계정이나 역할에는 MFA를 필수로 설정하세요
   - 특히 프로덕션 환경 접근 시 MFA를 요구하세요

5. **작업 전 확인**
   ```sh
   # 명령어 실행 전 항상 확인하는 습관
   $ aws configure list
   $ aws sts get-caller-identity
   ```

## 문제 해결

### 1. "InvalidAccessKeyId" 오류

```sh
An error occurred (InvalidAccessKeyId) when calling the ListBuckets operation
```

원인:
- 잘못된 Access Key
- 프로필이 올바르게 설정되지 않음
- 키가 비활성화되었거나 삭제됨

해결:
```sh
# 현재 사용 중인 자격 증명 확인
$ aws sts get-caller-identity --profile your-profile

# 프로필 재설정
$ aws configure --profile your-profile
```

### 2. SSO 세션 만료

```sh
Error when retrieving credentials from sso: Token has expired
```

해결:
```sh
# 다시 로그인
$ aws sso login --profile your-profile
```

### 3. AssumeRole 권한 오류

```sh
An error occurred (AccessDenied) when calling the AssumeRole operation
```

원인:
- 역할의 신뢰 정책에 사용자가 포함되지 않음
- MFA가 필요한데 제공하지 않음

해결: IAM 콘솔에서 역할의 신뢰 관계를 확인하고 수정하세요.

### 4. 프로필을 찾을 수 없음

```sh
The config profile (myprofile) could not be found
```

해결:
```sh
# 사용 가능한 프로필 확인
$ aws configure list-profiles

# 또는 직접 파일 확인
$ cat ~/.aws/config
$ cat ~/.aws/credentials
```

## 정리

AWS 계정을 전환하는 방법은 다양하며, 각 방법은 특정 상황에 적합합니다:

- **`--profile` 옵션**: 가장 안전하고 명시적, 일회성 작업에 적합
- **환경변수 (`AWS_PROFILE`)**: 한 세션 동안 여러 명령어 실행 시 편리
- **AWS SSO**: 조직 환경에서 가장 현대적이고 안전한 방법
- **AssumeRole**: 계정 간 역할 전환, 권한 분리에 유용
- **direnv**: 프로젝트별 자동 전환, 실수 방지에 효과적
- **Shell 함수**: 빠른 전환과 현재 상태 확인에 편리

실수를 방지하고 보안을 유지하려면:
1. 프로덕션 계정은 명시적으로만 사용
2. 작업 전 항상 현재 프로필 확인
3. 가능하면 SSO나 임시 자격 증명 사용
4. 중요한 작업에는 MFA 활성화

이러한 방법들을 조합하여 자신의 워크플로우에 맞는 계정 관리 전략을 구축하세요. 안전하고 효율적인 AWS 환경 관리의 첫 걸음은 올바른 계정 전환에서 시작됩니다.

## References

- [AWS CLI Configuration and Credential File Settings](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)
- [AWS CLI Named Profiles](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html)
- [AWS SSO Configuration](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html)
- [AssumeRole Documentation](https://docs.aws.amazon.com/cli/latest/reference/sts/assume-role.html)
- [direnv - unclutter your .profile](https://direnv.net/)
