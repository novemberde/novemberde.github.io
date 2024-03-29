---
title: AWS에 Ubuntu OS를 사용하는 EC2 인스턴스 생성하기
tags: [AWS, ec2, instance, ubuntu]
date: "2017-03-31T11:30:03+00:00"
aliases: ["/2017/03/31/Docker_1.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

## Summary
---------------------
 요즘에는 직접 IDC를 통해 서버를 운영하는 경우가 사라지고 있다. 
 Amazon에서 지원하는 IaaS인 EC2를 사용하면 간단하게 서버 인프라를 구축할 수 있다.
 가입은 간단하니 생략하고, EC2를 생성하고 SSH로 접속하는 과정을 진행해보겠다.

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
## EC2 생성하기
---------------------
 처음에 접속하면 아래와 같은 콘솔화면이 나타난다.
 
 ![AWS Console Main](/images/aws/ec2/main.png)

---------------------
 여기서 왼쪽 위에 Services라는 것을 누르면 AWS에서 지원하는 서비스 목록을 확인할 수 있다.

 ![AWS Console Menu](/images/aws/ec2/menu.png)

---------------------
 Compute에서 EC2를 선택하면 아래와 같은 화면으로 넘어간다.

 메인화면으로 EC2 인스턴스의 목록을 확인해 볼 수 있는 창이다.

 Resources라는 가운데 카테고리는 아래와 같은 의미이다.


- Running Instances :현재 가동하고 있는 인스턴스의 수 
- Elastic IPs       : 고정적으로 사용하는 아이피의 수(Elastic IP를 할당하지 않으면 인스턴스가 stop/start를 할 때 아이피가 바뀌게 된다.) 
- Dedicated Hosts   : 전용 호스트. 예를 들어, 기존의 서버의 환경에서 라이센스가 있었을 경우 이전할 때 별도의 물리적 서버를 할당 받을 수 있다 
- Snapshot          : EBS 볼륨에 기록된 데이터를 캡쳐하는 기능 
- Volumes           : 현재 사용하고 있는 볼륨. 서버들의 하드디스크라고 보면 간단하다. 
- Load Balancer     : 로드밸런서. AWS를 사용하는 주된 이유중에 하나이다. 부하를 여러 서버에 분담하게 하는 기능 
- Key Pairs         : 공개키 암호화 기법을 사용하여 EC2 유저의 정보를 암호화 및 해독하는 키 쌍
- Security Groups   : 트래픽을 제어하는 가상 방화벽의 역할. 모든 인스턴스는 하나 이상의 Security Group에 속해있다.
- Placement Groups  : 배치그룹. 단일 가용 영역 내에 있는 인스턴스의 논리적 그룹.

 -----
 
 ![AWS EC2 Console Main](/images/aws/ec2/ec2_main.png)

 -----
  이제 각 항목에 대해 살펴봤으니 인스턴스를 생성하기 전에 필요한 것이 무엇인지 알아보자.
  - Key Pairs: 인스턴스에 접속할 유저의 비밀키를 가지고 있어야 하기 때문에 생성하여야 한다.
  - Security Groups: 인스턴스를 생성한 다음 SSH로 접속하기 위해 Port를 설정해주어야 한다.

 -----
  Key Pair는 왼쪽의 사이드 메뉴에서 Network & Security에 보면 Key Pairs에서 생성할 수 있다.

  Create Key Pair를 선택하면 키를 생성할 수 있다.

  ubuntu라는 키를 create하면 ubuntu.pem이라는 파일을 자동적으로 다운로드 한다.

  이 키는 내 인스턴스를 마음대로 접근할 수 있도록 하기 때문에 절대 유출되지 않도록 조심히 다루어야 한다.

  이 키가 유출되더라도 사용할 수 없도록 만들기 위해 비밀번호를 걸어주어야 한다.

  ![AWS EC2 Console Key Pairs](/images/aws/ec2/key_pairs.png)
  ![AWS EC2 Console Create Key Pair](/images/aws/ec2/create_key_pair.png)

-----
  ubuntu.pem 에 패스워드를 걸어 별도의 개인키로 관리하려고 한다.

  [PuTTY Key Generator](http://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) 는 이 링크에서 다운받으면 된다.

  다운 받고 실행하면 아래와 같은 화면을 볼 수 있다.

  Load버튼을 눌러 ubuntu.pem을 열어보자. 파일 목록이 보이지 않을 시에는 All Files로 모든 형식의 파일이 보이게 하자.

  Key passphrase에 개인키에 사용할 비밀번호를 쓰고 Confirm passphrase로 확인하도록 한다.

  마지막으로 Save private key를 통해 ubuntu.ppk파일을 생성하자


  ![PuTTY Key Generator](/images/aws/ec2/puttygen.png)

-----
  이제 Security Group을 생성하여 SSH포트를 열어주도록 하자.

  Network & Security에 Security Groups에서 생성할 수 있다.

  VPC는 default로 사용하도록 하자. 
  
  Network 개념을 확장시키면 VPC도 별도로 생성하여 프로젝트마다 네트워크 가상 공간을 분리할 수 있다는 것만 대충 알아두자.

  - group name을 지정하고
  - 간단한 description을 작성하고
  - Inbound rule에 SSH를 선택하고 Source는 My IP만 입력하여 다른 사람이 내 인스턴스에 접근하지 못하도록 하자.

  ![AWS EC2 Console Security Group](/images/aws/ec2/security_group.png)
  
-----
 마지막 단계만 남았다.

 생성한 Key Pair와 Security Group으로 인스턴스를 생성한다.

 - Instances를 선택한다.
 - Launch Instance로 인스턴스 생성화면으로 넘어간다.

 ![AWS EC2 Console Instance Main](/images/aws/ec2/instance_main.png)

-----
 
 여기서는 AWS Marketplace로 넘어가서 ubuntu를 검색하면 된다.

 1. Ubuntu 16.04 LTS - Xenial (HVM) 을 select 하면 다음 화면으로 넘어간다.

 2. 다음화면에서는 인스턴스 타입을 선택하는데 무료로 사용할 수 있는 t2.micro를 선택한다.

 3. 선택한 다음에 Configure Instance는 별도의 Subnet을 사용하지 않기 때문에 디폴트 설정으로 수정없이 넘어가자.

 4. Add Storage는 많은 용량을 사용하지 않기 때문에 Size는 8 GiB로 하고 Volume Type은 GP2로 선택하자.

 5. Add Tags는 인스턴스를 간단히 구분하기 위해 좋은 제도이다. 하지만 우린 하나만 운영하기 때문에 따로 설정하지 않겠다.

 6. Configure Security Group에서 우리가 방금 만든 Security Group에 속해야 한다. Select an existing security group을 선택하고, ubuntu로 사용하자.

 7. launch를 누르면 Key pair를 등록하는 화면이 나온다. Choose an existing key pair가 디폴트로 나올 것이다. 여기서 ubuntu를 key pair로 사용하도록 하자.

 ![AWS EC2 Console Instance choose_ami](/images/aws/ec2/choose_ami.png)
 ![AWS EC2 Console Instance choose_it](/images/aws/ec2/choose_it.png)

-----

 이제 접속하자.

 아래 이미지을 보면 오른쪽 밑에 Public DNS라는 란이 보일 것이다. 이부분을 복사한다.

 [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)를 열어 SSH로 접속을 시도한다.

 1. Host Name에는 public DNS값을 붙여넣고
 2. 왼쪽 카테고리에서 Connection > SSH > Auth 에 들어가면 아래쪽에 파일 첨부하는 란이 있다. 여기에 아까 만든 ubuntu.ppk를 넣는다.
 3. 이제 open을 눌러 접속을 시도해보자.
 4. login as: ubuntu 를 입력하고, 비밀번호는 ubuntu.ppk의 비밀번호를 사용한다.
 
 ![AWS EC2 Console Instance Created](/images/aws/ec2/instance_created.png)
 ![putty](/images/aws/ec2/putty.png)
 ![putty_open](/images/aws/ec2/putty_open.png)
 

## References
- [http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/concepts.html](http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/concepts.html)