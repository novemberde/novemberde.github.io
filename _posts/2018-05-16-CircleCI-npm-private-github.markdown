---
title: CircleCI에서 Github private repository를 npm module로 사용하기
category: DevOps
tags: circleci, github, npm, private, repo, repository, module
---
## Summary
---
현재 공통적인 모듈을 Git으로 따로 관리하여 npm private module로 사용한다.
또한 Webpack을 사용하여 Typescript로 작성된 서버를 번들링한다.
로컬에서는 모두 정상적으로 동작하지만 CircleCI에서 빌드할 경우에는
CircleCI의 권한이 해당 repository에 대한 권한만 가지고 있기 때문에
npm private module을 가져올 수가 없다.

이와같은 문제를 해결하기 위해서 트러블 슈팅을 하였다.

### Problem
---

npm install을 할 때에 github에 있는 repository를 사용하는 방법은 두가지가 있다.

1. https
2. ssh

ssh로 사용할 경우 관련 명령어는 다음과 같다.

```sh
# ssh 키 생성하기
$ ssh-keygen -f id_rsa_kh -C ""

# 생성한 키를 ~/.ssh 에 추가하기
$ cp id_rsa_kh ~/.ssh/id_rsa_kh

# 생성한 키의 권한 수정하기
$ chmod 600 ~/.ssh/id_rsa_kh

# 생성한 키의 퍼블릭 키를 출력하기. 이때 받은 키를 github private repo에 ssh키를 추가하여야 한다.
$ cat id_rsa_kh.pub

# ssh의 개인키 개수 확인하기
$ ls -al ~/.ssh/

# ssh-agent 실행하기
$ eval "$(ssh-agent -s)"

# ~/.ssh 디렉터리에 있는 개인키를 모두 ssh agent에 등록하기
$ grep -slR "PRIVATE" ~/.ssh/ | xargs ssh-add

# 현재 등록되어 있는 ssh key의 fingerprint 확인하기
$ ssh-add -l -E md5
```

위와 같이 키를 입력하였으면 이제 npm install을 해본다.

```sh
# username, repository를 수정하여준다.
$ npm i -S git+ssh://git@github.com:<username>/<repository>.git

# https의 경우
$ npm i -S https://github.com/<username>/<repository>
$ npm i -S git+https://<token>:x-oauth-basic@github.com/<username>/<repository>.git
```

로컬에서 사용시 여기까지는 문제없이 진행되었다. 하지만 CircleCI에서 ssh키를 추가하여 
steps에 다음의 코드도 추가하였음에도 불구하고 npm install이 되지 않았다.

```yml
- add_ssh_keys:
          fingerprints:
            - "XX:XX:..."
```

그래서 이전에 사용했던 명령어들을 통해 디버깅을 시작하였다.
다음의 명령어를 통해 fingerprint를 확인해보니 정상적으로 추가가 되어 있었다.

```sh
$ ssh-add -l -E md5
2048 MD5:xx...  (RSA)
2048 MD5:xx:yy...  (RSA)
```

그렇다면 무엇이 문제일까? 해답은 다음과 같은 명령어를 통해 유추해낼 수 있었다.

```sh
$ ssh -Tv git@github.com
```

이것은 ssh로 github에 인증이 되었는지 확인할 수 있는데, 위의 명령어를 입력하니 다음과 같은 것을 확인할 수 있었다.

Hi \<username\>/\<another_repo\>! You've successfully authenticated, but GitHub does not provide shell access.

이를 보니 CircleCI에서 빌드하는 repository의 ssh키로 Github에 접근하니 추가한 ssh-key가 있다하더라도 default로 가지고 있는 키로
ssh를 시도하였다.

### Solution
---

해결방법은 고민이 무색할 정도로 간단하게 결정하였다. Github에 접근하는 키에 대해 switching 을 하는 
스크립트를 만들고 매번 실행하여 해결할까라는 고민도 하였지만 너무 불필요한 코드라고 여겨졌다.
그래서 Gmail 계정을 하나 더 생성하고, github에 가입한 다음 권한을 주었다.

그 다음은 https로 private repo에 접근하면 된다.

솔직히 제일 간단한 방법은 npm 에 private repository를 매달 비용을 지불하면서 사용하면 된다.
아니면 직접 npm repository를 운영하여 사용해도 좋다. 그런데 운영 리소스를 줄이고 싶은 마음이 제일
크기 때문에 직접 운영하는 배제하였고, npm private repo는 아직 크게 사용할 일이 없어서 하지 않았다.

나중에 private repository가 많아진다면 npm에 지불해야 된다고 생각한다.