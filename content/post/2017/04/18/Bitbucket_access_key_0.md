---
title: Bitbucket access key 발급하기
tags: [bitbucket, accesskey, ubuntu, git, ssh, deploy-key, app-password, personal-access-token]
date: "2017-05-09T11:30:03+00:00"
aliases: ["/2017/04/18/Bitbucket_access_key_0.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

서버에 애플리케이션을 배포할 때 FTP로 파일을 일일이 옮기는 방식은 비효율적이다. Git을 활용하면 코드 변경사항을 추적하면서도 간단한 명령어로 서버 코드를 업데이트할 수 있다.

하지만 서버에서 매번 사용자 이름과 비밀번호를 입력하는 것도 번거롭고 보안상 좋지 않다. 이 글에서는 Bitbucket에서 안전하게 인증하는 여러 방법을 실제 사용 사례와 함께 살펴본다.

## Bitbucket 인증 방법의 종류

Bitbucket은 다양한 인증 방법을 제공하며, 각각의 용도가 다르다:

1. **SSH Keys (개인 계정용)**: 개발자 개인이 사용하는 일반적인 방법
2. **Deploy Keys (배포 전용)**: 서버에서 저장소를 읽기 전용으로 접근
3. **Access Keys (구 방식)**: Deploy Keys의 이전 명칭, 현재는 Deploy Keys로 통합
4. **App Passwords**: HTTPS 접근 시 비밀번호 대신 사용
5. **Personal Access Tokens**: API 접근 및 자동화 스크립트용

각 방법을 언제, 어떻게 사용하는지 자세히 알아보자.

## 방법 1: SSH Keys - 개발자용 인증

개발자가 자신의 머신에서 Bitbucket에 접근할 때 사용하는 가장 일반적인 방법이다.

### SSH Key 생성하기

```bash
# SSH key pair 생성
$ ssh-keygen -t ed25519 -C "your_email@example.com"

# 또는 RSA 방식 (더 널리 호환됨)
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/username/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/username/.ssh/id_ed25519
Your public key has been saved in /home/username/.ssh/id_ed25519.pub
```

**참고**:
- `ed25519`는 더 짧고 안전한 최신 알고리즘이다
- 구형 시스템과의 호환성이 필요하다면 `rsa`를 사용하자
- passphrase를 설정하면 보안이 강화되지만, 매번 입력해야 한다

### SSH 파일 권한 설정

보안을 위해 올바른 권한을 설정해야 한다:

```bash
# .ssh 디렉토리는 700 (소유자만 읽기/쓰기/실행)
$ chmod 700 ~/.ssh

# 개인키는 600 (소유자만 읽기/쓰기)
$ chmod 600 ~/.ssh/id_ed25519

# 공개키는 644 (모두 읽기 가능, 소유자만 쓰기)
$ chmod 644 ~/.ssh/id_ed25519.pub
```

잘못된 권한 설정은 SSH 접속 실패의 주요 원인이다.

### SSH Agent에 키 등록하기

매번 passphrase를 입력하지 않으려면 SSH agent에 키를 등록한다:

```bash
# SSH agent 실행 여부 확인
$ eval "$(ssh-agent -s)"
Agent pid 59566

# SSH 키 추가
$ ssh-add ~/.ssh/id_ed25519
Enter passphrase for /home/username/.ssh/id_ed25519:
Identity added: /home/username/.ssh/id_ed25519 (your_email@example.com)

# 등록된 키 확인
$ ssh-add -l
256 SHA256:abcd1234... your_email@example.com (ED25519)
```

**macOS 사용자를 위한 팁**:
```bash
# macOS에서 키체인에 자동 저장
$ ssh-add --apple-use-keychain ~/.ssh/id_ed25519

# ~/.ssh/config 파일에 추가하면 재부팅 후에도 자동 로드
$ cat >> ~/.ssh/config << EOF
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
EOF
```

### Bitbucket에 SSH Key 등록하기

1. 공개키 내용 복사:
```bash
$ cat ~/.ssh/id_ed25519.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJqfXXXXXXXXXXXX your_email@example.com
```

2. Bitbucket 웹사이트에서:
   - 우측 상단 프로필 아이콘 클릭
   - **Personal settings** 선택
   - 좌측 메뉴에서 **SSH keys** 선택
   - **Add key** 버튼 클릭
   - Label에 "My Laptop" 같은 구분 가능한 이름 입력
   - Key 필드에 공개키 전체 내용 붙여넣기
   - **Add key** 클릭

### 연결 테스트

```bash
# Bitbucket SSH 연결 테스트
$ ssh -T git@bitbucket.org
authenticated via ssh key.

You can use git to connect to Bitbucket. Shell access is disabled.
```

이제 SSH URL로 저장소를 클론하고 사용할 수 있다:

```bash
$ git clone git@bitbucket.org:your-workspace/your-repo.git
```

## 방법 2: Deploy Keys - 서버 배포용 인증

Deploy Keys는 특정 저장소에만 접근 권한을 부여하는 SSH 키다. 주로 CI/CD 서버나 프로덕션 서버에서 사용한다.

### Deploy Keys의 특징

- **저장소별 권한**: 하나의 저장소에만 접근 가능
- **읽기 전용 또는 쓰기 가능**: 필요에 따라 권한 선택
- **계정 연동 불필요**: Bitbucket 계정 없이도 사용 가능
- **감사 추적**: 어떤 키가 언제 사용되었는지 추적 가능

### Deploy Key 생성 및 등록

서버에서 전용 키를 생성한다:

```bash
# 서버에 SSH 접속
$ ssh user@your-server.com

# 배포 전용 SSH 키 생성 (passphrase 없이)
$ ssh-keygen -t ed25519 -C "deploy-key-production" -f ~/.ssh/deploy_key_production
Generating public/private ed25519 key pair.
Enter passphrase (empty for no passphrase): [Enter]
Enter same passphrase again: [Enter]

# 공개키 확인
$ cat ~/.ssh/deploy_key_production.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJqfXXXXXXXXXXXX deploy-key-production
```

**보안 주의사항**:
- 프로덕션 서버의 키는 절대 개발 머신과 공유하지 말자
- 서버마다 별도의 키를 생성하자
- 키가 노출되면 즉시 Bitbucket에서 삭제하자

### Bitbucket에 Deploy Key 등록하기

1. Bitbucket 저장소 페이지 접속
2. 좌측 메뉴에서 **Repository settings** 클릭
3. **Access keys** (또는 **Deploy keys**) 메뉴 선택
4. **Add key** 버튼 클릭
5. Label에 "Production Server" 같은 설명 입력
6. Key 필드에 공개키 붙여넣기
7. 쓰기 권한이 필요하면 **Allow write access** 체크
8. **Add key** 클릭

### SSH Config 설정으로 키 지정하기

여러 키를 관리할 때는 `~/.ssh/config`에 설정을 추가하면 편리하다:

```bash
$ cat >> ~/.ssh/config << EOF
# Production deployment key
Host bitbucket.org-production
  HostName bitbucket.org
  User git
  IdentityFile ~/.ssh/deploy_key_production
  IdentitiesOnly yes

# Staging deployment key
Host bitbucket.org-staging
  HostName bitbucket.org
  User git
  IdentityFile ~/.ssh/deploy_key_staging
  IdentitiesOnly yes
EOF
```

이제 각 환경에 맞는 호스트명으로 클론할 수 있다:

```bash
# 프로덕션 서버에서
$ git clone git@bitbucket.org-production:your-workspace/your-repo.git

# 스테이징 서버에서
$ git clone git@bitbucket.org-staging:your-workspace/your-repo.git
```

### 배포 자동화 스크립트 예제

```bash
#!/bin/bash
# deploy.sh - 간단한 배포 스크립트

REPO_DIR="/var/www/myapp"
BRANCH="main"

cd $REPO_DIR || exit 1

echo "Fetching latest changes..."
git fetch origin $BRANCH

echo "Checking out $BRANCH..."
git checkout $BRANCH

echo "Pulling latest code..."
git pull origin $BRANCH

echo "Installing dependencies..."
npm install --production

echo "Restarting application..."
pm2 restart myapp

echo "Deployment completed!"
```

## 방법 3: App Passwords - HTTPS 접근용

2단계 인증(2FA)을 활성화한 경우 HTTPS로 Git을 사용하려면 App Password가 필요하다.

### App Password 생성하기

1. Bitbucket 웹사이트에서:
   - 우측 상단 프로필 아이콘 클릭
   - **Personal settings** 선택
   - 좌측 메뉴에서 **App passwords** 선택
   - **Create app password** 버튼 클릭

2. Label에 "Laptop HTTPS Access" 같은 이름 입력

3. 필요한 권한 선택:
   - **Repositories**: Read, Write
   - **Pull requests**: Read, Write (필요시)
   - 최소 권한 원칙을 따라 필요한 것만 선택

4. **Create** 클릭 후 생성된 비밀번호 복사 (한 번만 표시됨!)

### App Password 사용하기

```bash
# HTTPS로 클론할 때
$ git clone https://bitbucket.org/your-workspace/your-repo.git
Username: your-username
Password: [생성한 App Password 입력]

# 자격증명 저장 (Linux/macOS)
$ git config --global credential.helper store
# 또는 macOS keychain 사용
$ git config --global credential.helper osxkeychain

# 다음 push/pull 시 입력한 자격증명이 저장됨
```

### 자격증명 직접 저장하기 (주의!)

URL에 자격증명을 포함할 수 있지만 보안상 권장하지 않는다:

```bash
# 보안에 주의! 스크립트나 CI/CD 환경에서만 사용
$ git clone https://username:app-password@bitbucket.org/workspace/repo.git
```

더 안전한 방법은 환경변수를 사용하는 것이다:

```bash
# 환경변수로 설정
export BITBUCKET_USERNAME="your-username"
export BITBUCKET_APP_PASSWORD="your-app-password"

# Git 명령어에서 사용
git clone https://${BITBUCKET_USERNAME}:${BITBUCKET_APP_PASSWORD}@bitbucket.org/workspace/repo.git
```

## 방법 4: Personal Access Tokens - API 및 자동화용

Bitbucket API를 사용하거나 자동화 스크립트를 작성할 때 사용한다.

### Access Token 생성하기

1. **Workspace settings** (전체 workspace 권한 필요)
2. 또는 **Repository settings** (특정 저장소만)
3. **Access Tokens** 메뉴 선택
4. **Create Repository Access Token** 또는 **Create Access Token**
5. Token name과 권한 설정
6. 만료 기간 설정 (보안을 위해 짧게 설정 권장)
7. 생성된 토큰 복사 (다시 볼 수 없음!)

### Access Token 사용 예제

```bash
# API 호출 예제
curl -X GET \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  https://api.bitbucket.org/2.0/repositories/workspace/repo

# Git 명령어에서도 사용 가능
git clone https://x-token-auth:YOUR_ACCESS_TOKEN@bitbucket.org/workspace/repo.git
```

### Python 스크립트 예제

```python
import requests
import os

# 환경변수에서 토큰 가져오기
token = os.environ.get('BITBUCKET_ACCESS_TOKEN')
workspace = 'your-workspace'
repo_slug = 'your-repo'

headers = {
    'Authorization': f'Bearer {token}',
    'Content-Type': 'application/json'
}

# Pull requests 목록 가져오기
url = f'https://api.bitbucket.org/2.0/repositories/{workspace}/{repo_slug}/pullrequests'
response = requests.get(url, headers=headers)

if response.status_code == 200:
    prs = response.json()['values']
    for pr in prs:
        print(f"PR #{pr['id']}: {pr['title']} - {pr['state']}")
else:
    print(f"Error: {response.status_code}")
```

## 실전 활용 시나리오

### 시나리오 1: 프리랜서 개발자

여러 클라이언트의 프로젝트를 관리하는 경우:

```bash
# 클라이언트별로 다른 SSH 키 사용
$ cat >> ~/.ssh/config << EOF
Host bitbucket.org-client-a
  HostName bitbucket.org
  User git
  IdentityFile ~/.ssh/id_ed25519_client_a

Host bitbucket.org-client-b
  HostName bitbucket.org
  User git
  IdentityFile ~/.ssh/id_ed25519_client_b
EOF

# 각 클라이언트의 저장소 클론
$ git clone git@bitbucket.org-client-a:client-a/project.git
$ git clone git@bitbucket.org-client-b:client-b/project.git
```

### 시나리오 2: CI/CD 파이프라인 구축

GitHub Actions, Jenkins, GitLab CI 등에서 Bitbucket 저장소 접근:

```yaml
# GitHub Actions 예제
name: Deploy from Bitbucket

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Setup SSH Key
        env:
          SSH_PRIVATE_KEY: ${{ secrets.BITBUCKET_DEPLOY_KEY }}
        run: |
          mkdir -p ~/.ssh
          echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_ed25519
          chmod 600 ~/.ssh/id_ed25519
          ssh-keyscan bitbucket.org >> ~/.ssh/known_hosts

      - name: Clone Bitbucket Repository
        run: |
          git clone git@bitbucket.org:workspace/repo.git

      - name: Deploy
        run: |
          cd repo
          ./deploy.sh
```

### 시나리오 3: 멀티 환경 배포

개발, 스테이징, 프로덕션 서버에 각각 다른 권한 부여:

```bash
# 개발 서버: 읽기/쓰기 가능
# ~/.ssh/config
Host bitbucket.org-dev
  HostName bitbucket.org
  User git
  IdentityFile ~/.ssh/deploy_key_dev

# 스테이징 서버: 읽기 전용
Host bitbucket.org-staging
  HostName bitbucket.org
  User git
  IdentityFile ~/.ssh/deploy_key_staging

# 프로덕션 서버: 읽기 전용
Host bitbucket.org-prod
  HostName bitbucket.org
  User git
  IdentityFile ~/.ssh/deploy_key_prod
```

각 서버의 Deploy Key는 Bitbucket에서 적절한 권한으로 등록한다.

## 보안 모범 사례

### 1. 키 관리

```bash
# 주기적으로 사용하지 않는 키 제거
$ ssh-add -l  # 현재 등록된 키 확인
$ ssh-add -D  # 모든 키 제거
$ ssh-add ~/.ssh/id_ed25519  # 필요한 키만 다시 추가

# 키 파일 백업 (안전한 장소에)
$ tar -czf ssh-keys-backup-$(date +%Y%m%d).tar.gz ~/.ssh/
$ gpg -c ssh-keys-backup-20250102.tar.gz  # 암호화
```

### 2. 권한 최소화

- Deploy Keys는 가능한 읽기 전용으로 설정
- App Passwords와 Access Tokens는 필요한 권한만 부여
- 만료 기간을 설정하여 정기적으로 갱신

### 3. 키 로테이션

```bash
# 새 키 생성
$ ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/id_ed25519_new

# Bitbucket에 새 키 추가
# (웹 UI에서 수행)

# 새 키로 테스트
$ ssh -T -i ~/.ssh/id_ed25519_new git@bitbucket.org

# 문제없으면 기존 키를 새 키로 교체
$ mv ~/.ssh/id_ed25519 ~/.ssh/id_ed25519.old
$ mv ~/.ssh/id_ed25519_new ~/.ssh/id_ed25519

# Bitbucket에서 구 키 삭제
```

### 4. 환경변수 활용

비밀정보를 코드에 직접 넣지 말고 환경변수나 secrets 관리 도구를 사용하자:

```bash
# .env 파일 (절대 git에 커밋하지 말 것!)
BITBUCKET_APP_PASSWORD=your-app-password
BITBUCKET_ACCESS_TOKEN=your-access-token

# .gitignore에 추가
echo ".env" >> .gitignore

# 스크립트에서 사용
source .env
git clone https://${BITBUCKET_USERNAME}:${BITBUCKET_APP_PASSWORD}@bitbucket.org/workspace/repo.git
```

### 5. 감사 및 모니터링

Bitbucket에서 주기적으로 확인하자:
- **SSH Keys**: 사용하지 않는 키 삭제
- **Deploy Keys**: 각 저장소의 접근 키 점검
- **Access Tokens**: 만료되거나 사용하지 않는 토큰 제거
- **Access logs**: 이상한 활동 모니터링

## 문제 해결

### 1. Permission denied (publickey)

```bash
$ git clone git@bitbucket.org:workspace/repo.git
Permission denied (publickey).
fatal: Could not read from remote repository.
```

**원인**:
- SSH 키가 Bitbucket에 등록되지 않음
- 잘못된 키를 사용하고 있음
- SSH agent에 키가 로드되지 않음

**해결**:
```bash
# 어떤 키를 시도하는지 확인
$ ssh -vT git@bitbucket.org

# SSH agent에 키 추가
$ ssh-add ~/.ssh/id_ed25519

# 올바른 키가 로드되었는지 확인
$ ssh-add -l
```

### 2. Repository not found

```bash
$ git clone git@bitbucket.org:workspace/repo.git
repository not found
```

**원인**:
- 저장소 경로가 잘못됨
- 접근 권한이 없음
- Deploy Key에 해당 저장소 권한이 없음

**해결**:
```bash
# 정확한 저장소 URL 확인
# Bitbucket 웹사이트에서 Clone 버튼 클릭하여 URL 복사

# Deploy Key가 올바른 저장소에 등록되었는지 확인
```

### 3. SSH 연결이 느림

```bash
# SSH 연결 디버그
$ ssh -v git@bitbucket.org

# DNS 확인
$ nslookup bitbucket.org

# ~/.ssh/config에 추가하여 연결 속도 개선
Host bitbucket.org
  ControlMaster auto
  ControlPath ~/.ssh/sockets/%r@%h-%p
  ControlPersist 600
```

### 4. 여러 키를 사용할 때 혼란

```bash
# 특정 키로 명령어 실행
$ GIT_SSH_COMMAND="ssh -i ~/.ssh/specific_key" git clone git@bitbucket.org:workspace/repo.git

# 또는 ~/.ssh/config에 명확히 지정
Host bitbucket-work
  HostName bitbucket.org
  User git
  IdentityFile ~/.ssh/work_key
  IdentitiesOnly yes

Host bitbucket-personal
  HostName bitbucket.org
  User git
  IdentityFile ~/.ssh/personal_key
  IdentitiesOnly yes
```

### 5. App Password가 작동하지 않음

**원인**:
- 2FA가 활성화되지 않았거나 App Password 기능이 비활성화됨
- 잘못된 권한 설정

**해결**:
```bash
# Git credential helper 재설정
$ git config --global --unset credential.helper
$ git config --global credential.helper store

# 다시 클론하여 자격증명 재입력
$ git clone https://bitbucket.org/workspace/repo.git
```

## SSH vs HTTPS: 언제 무엇을 사용할까?

### SSH를 사용하는 경우:

**장점**:
- 비밀번호 입력 불필요 (키 기반 인증)
- 더 안전한 인증 방식
- 키 하나로 모든 저장소 접근 가능

**단점**:
- 방화벽에서 SSH 포트(22)를 막을 수 있음
- 초기 설정이 복잡할 수 있음

**추천 상황**:
- 개발자 개인 머신
- 서버 배포 (Deploy Keys 사용)
- 장기적으로 사용하는 환경

### HTTPS를 사용하는 경우:

**장점**:
- 방화벽 친화적 (포트 443 사용)
- 설정이 간단함
- 임시 접근에 적합

**단점**:
- App Password 또는 Access Token 관리 필요
- 매번 자격증명 입력 (credential helper 없이)

**추천 상황**:
- 방화벽이 엄격한 회사 네트워크
- CI/CD 파이프라인 (토큰 사용)
- 임시 또는 테스트 목적

## 정리

Bitbucket 접근 방법은 사용 목적에 따라 선택해야 한다:

- **개발 작업**: SSH Keys (개인 계정에 등록)
- **서버 배포**: Deploy Keys (저장소별 등록, 읽기 전용 권장)
- **HTTPS 접근**: App Passwords (2FA 사용 시)
- **API 자동화**: Personal Access Tokens (만료 기간 설정)
- **CI/CD**: Deploy Keys 또는 Access Tokens

보안을 위한 체크리스트:
- ✅ 개인키는 절대 공유하지 않기
- ✅ 각 환경마다 별도의 키 사용
- ✅ 사용하지 않는 키는 Bitbucket에서 제거
- ✅ 최소 권한 원칙 적용
- ✅ 주기적인 키 로테이션
- ✅ passphrase 설정 (개인 키)
- ✅ 올바른 파일 권한 설정 (600 for private keys)

이러한 방법들을 조합하여 안전하고 효율적인 코드 관리 및 배포 환경을 구축하자.

## References

- [Bitbucket SSH Keys Documentation](https://support.atlassian.com/bitbucket-cloud/docs/set-up-an-ssh-key/)
- [Bitbucket Deploy Keys](https://support.atlassian.com/bitbucket-cloud/docs/use-deployment-keys/)
- [Bitbucket App Passwords](https://support.atlassian.com/bitbucket-cloud/docs/app-passwords/)
- [Bitbucket Access Tokens](https://support.atlassian.com/bitbucket-cloud/docs/repository-access-tokens/)
- [SSH Key Types and Security](https://www.ssh.com/academy/ssh/keygen)
