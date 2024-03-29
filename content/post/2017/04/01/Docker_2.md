---
title: 접속 포트를 열어주고 별도의 Ubuntu 유저를 생성하기
tags: [ssh, ec2, ubuntu, create, user, 유저, 생성]
date: "2017-04-01T11:30:03+00:00"
aliases: ["/2017/04/01/Docker_2.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

## Summary
---------------------
 생성한 EC2 인스턴스에 docker라는 계정을 생성해 보려고 한다.

 또한 Security Group을 응용하여 접속포트는 80을 열어 사이트를 접근할 수 있도록 설정해주자.

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
## docker 라는 linux 계정을 생성해본다.
---------------------
먼저 docker 계정에서 사용할 Key pair를 생성해보자.

아래와 같은 EC2 > Network & Security > Key Pairs항목에서 Key Pair를 생성하여 준다. 이번엔 ubuntu가 아닌 docker로 생성하여주자

이전과 마찬가지로 자동으로 다운받은 파일을 간직하도록 하자.(Amazon web service에 Ubuntu OS를 사용하는 EC2 인스턴스 생성하기 참고)

![AWS EC2 Console Create Key Pair](/images/aws/ec2/key_pairs.png)


---------------------

생성된 Key pair를 [PuTTY Key Generator](http://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)를 사용하여 docker.ppk파일을 생성한다.

Save private key를 눌러 docker.ppk로 비밀번호를 설정하여 저장하도록 한다.(Amazon web service에 Ubuntu OS를 사용하는 EC2 인스턴스 생성하기 참고)

![PuTTY Key Generator](/images/aws/ec2/puttygen.png)

---------------------

기본준비가 완료되었다. ubuntu라는 계정으로 SSH접속을 먼저 시작한다.(Amazon web service에 Ubuntu OS를 사용하는 EC2 인스턴스 생성하기 참고)

![putty](/images/aws/ec2/putty.png)
![putty_open](/images/aws/ec2/putty_open.png)

---------------------

이제 linux 계정을 설치해보겠다. 여기서 --disabled-password를 사용하는 이유는 SSH로 비밀키의 비밀번호로 접속하기 때문이다. 키파일을 중요하게 관리해야 한다는 점을 잊지말자.

```bash
# 유저 추가하기
$ sudo adduser docker --disabled-password
```

```bash
# Docker 계정으로 전환하기
$ sudo su - docker
```

```bash
# /home/docker 에 .ssh 디렉터리를 생성하기
$ mkdir .ssh
```

```bash
# 자신을 제외한 계정이 .ssh폴더에 접근하지 못하도록 한다.
$ chmod 700 .ssh
```

```bash
# /home/docker/.ssh 디렉터리에 authorized_keys라는 파일을 생성과 동시에 편집하기.
$ vi .ssh/authorized_keys
```

1. vi 편집기는 linux에서 편리하게 문서를 관리할 수 있는 편집기이다. 'a'를 누르면 INSERT모드가 된다.
2. PuttyGen으로 docker.pem 파일을 열고 맨위에 ssh-rsa로 시작하는 부분을 전체 복사한다. 이것을 Public Key 공개키라고 한다.
3. 현재 켜져있는 ssh접속화면에 마우스 오른버튼을 눌러 붙여넣기를 해주면 된다.
4. 마지막으로 ESC를 누르고 ':wq'를 입력하면 docker계정은 SSH 접속정보가 담긴 유저가 된다.
5. .ssh/authorized_keys값은 아무나 변경할 수 없어야 하기 때문에 권한설정을 해준다.

```bash
# /home/docker/.ssh/authorized_keys값은 아무나 변경할 수 없어야 하기 때문에 권한설정을 해준다.
$ chmod 600 .ssh/authorized_keys
```

---------------------
 Docker 계정이 생성되었다. 이제 docker라는 계정으로 SSH접속을 해보자.(Amazon web service에 Ubuntu OS를 사용하는 EC2 인스턴스 생성하기 참고)

 Host Name 은 ubuntu 계정과 동일하고 SSH 키파일은 docker.ppk로 접속하면 된다.

---------------------

## Security Group에서 80 포트를 열어주자

 아래와 같은 화면에서 Add Rule을 눌러 Type은 HTTP를 고르고 Source는 Anywhere로 하던가 로컬에서 작성자만 접속하게 하고 싶다면 My IP로 진행한다.

 간단하게 80포트를 열어주었다. AWS가 나오기 이전에는 직접 ufw를 사용하여 방화벽 설정을 할 수 있었다. 하지만 Security Group을 사용하면 간단하게 방화벽을 구현할 수 있다.

![AWS EC2 Console Security Group](/images/aws/ec2/security_group.png)

## References
- [http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/managing-users.html](http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/managing-users.html)
