---
title: AWS ALB로 ElasticBeanstalk 배포하기
tags: [ALB, ELB, eb, elasticbeanstalk, elastic, beanstalk, application, load, balancer, 연동, docker, multicontainer, container, 배포]
date: "2017-09-25T11:30:03+00:00"
aliases: ["/aws/2017/09/25/ALB_to_elasticbeanstalk.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---
## Summary
---
ElasticBeanstalk를 Docker로 배포하기 위해 살펴보았는데 문제점이 있었다.
기존의 사용하던 ALB와 연동하여 사용하고 싶었지만 설정화면에서는 Classic Load Balancer만 지원되었기 때문이다.
eb-cli를 사용하여 AutoScaling Group을 생성하여 ALB의 Target Group에 설정하여 앱을 배포해보자.

---
### EB cli 설치하기
---

[Install eb cli](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html)를 참고하여 로컬에 eb-cli를 설치하자.

사전에 python이 2.7 또는 3.4이상의 버전이 설치되어 있어야 한다.

```sh
$ pip install awsebcli --upgrade --user
```

설치 후에 환경변수에 아래와 같은 path를 추가하자.

- Linux – ~/.local/bin
- macOS – ~/Library/Python/3.4/bin
- Windows – %USERPROFILE%\AppData\Roaming\Python\Scripts
  - Python 3.5 on Windows – %USERPROFILE%\AppData\Roaming\Python\Python3.5\Scripts
  - Python 3.6 on Windows – %USERPROFILE%\AppData\Local\Programs\Python\Python36\Scripts

올바르게 입력했다면 eb의 version을 확인할 수 있을 것이다.

```sh
$ eb --version
```

---
### EB Command Line Interface로 AutoScaling Group 생성하기
---

AutoScaling Group을 생성하기 위해선 아래와 같은 과정이 필요하다.

1. ElasticBeanstalk Application 생성하기
2. ElasticBeanstalk Environment 생성하기(여기서 AutoScaling Group을 설정한다)

그럼 위 과정에 대해서 eb-cli를 통해 생성해본다.

```sh
# 먼저 eb init을 통해 배포할 region을 지정해준다.
$ eb init

Select a default region
1) us-east-1 : US East (N. Virginia)
2) us-west-1 : US West (N. California)
3) us-west-2 : US West (Oregon)
4) eu-west-1 : EU (Ireland)
5) eu-central-1 : EU (Frankfurt)
6) ap-south-1 : Asia Pacific (Mumbai)
7) ap-southeast-1 : Asia Pacific (Singapore)
8) ap-southeast-2 : Asia Pacific (Sydney)
9) ap-northeast-1 : Asia Pacific (Tokyo)
10) ap-northeast-2 : Asia Pacific (Seoul)
11) sa-east-1 : South America (Sao Paulo)
12) cn-north-1 : China (Beijing)
13) us-east-2 : US East (Ohio)
14) ca-central-1 : Canada (Central)
15) eu-west-2 : EU (London)
(default is 3): 10

# region을 선택했으면 아래와 같은 내용이 나타난다. 여기서 사용할 Application을 생성해주자
Select an application to use
1) legacyEBApplication
2) [ Create new Application ]
(default is 2): 2

# 사용할 Application name을 입력하자.
Enter Application Name
(default is "novem"): TestApp
Application TestApp has been created.

# 사용할 Platform을 선택해주자. 여기서 Docker로 배포하기 때문에 Docker를 선택한다.
Select a platform.
1) Ruby
2) Go
3) Java
4) Tomcat
5) PHP
6) Python
7) Node.js
8) IIS
9) Docker
10) GlassFish
11) Packer
(default is 1): 9

# Platform version은 Community Edition을 선택해주도록 하자.
Select a platform version.
1) Docker 17.03.1-ce
2) Docker 1.12.6
3) Docker 1.11.2
4) Docker 1.9.1
(default is 1): 1

# SSH로 접근할 수 있도록 할지 선택할 수 있다.
Cannot setup CodeCommit because there is no Source Control setup, continuing with initialization
Do you want to set up SSH for your instances?
(Y/n): Y

# SSH 접근을 허용했다면 사용할 KeyPair를 선택해주자.
Select a keypair.
1) RootKeyPair
2) OtherUserKeyPair
3) [ Create new KeyPair ]
(default is 1): 1
```

ElasticBeanstalk Application을 생성하였다.

---

다음으로 ElasticBeanstalk environment를 생성하자.

이때 윈도우 환경이라면 CMD창을 Administrator권한으로 실행해야 올바르게 동작한다.

```sh
$ eb create
# Environment Name을 입력한다. Default로 [ApplicationName-dev] 로 잡힌다.
Enter Environment Name
(default is TestApp-dev):
# DNS Name을 입력한다. Default로 [ApplicationName-dev] 로 잡힌다.
Enter DNS CNAME prefix
(default is TestApp-dev):

# Load balancer type을 입력한다. Default로 [Classic Load Balancer] 로 잡힌다.
Select a load balancer type
1) classic
2) application
(default is 1): 2
Creating application version archive "app-170926_015406".
Uploading TestApp/app-170926_015406.zip to S3. This may take a while.
Upload Complete.
Environment details for: TestApp-dev
  Application name: TestApp
  Region: ap-northeast-2
  Deployed Version: app-170926_015406
  Environment ID: e-mdj25dyzuu
  Platform: arn:aws:elasticbeanstalk:ap-northeast-2::platform/Docker running on 64bit Amazon Linux/2.7.4
  Tier: WebServer-Standard
  CNAME: TestApp-dev.ap-northeast-2.elasticbeanstalk.com
  Updated: 2017-09-26 01:54:09.941000+00:00
Printing Status:
...
```

이제 모두 생성되었다.

소스코드를 배포해 보자. 로컬에 아래와 같은 예제 git repository를 가져와서 배포하자.

```
# 로컬에 소스코드를 가져온다.
$ git clone https://kyuhyun@bitbucket.org/kyuhyun/docker_node_server.git
```

1. 해당 폴더를 zip 파일로 압축한다.
2. AWS console에서 ElasticBeanstalk로 접속하여 해당 Environment로 이동한다.
3. 압축한 파일을 업로드하여 Deploy 한다.

이 과정을 eb cli로 진행하려 하였지만 [EB-cli 문서](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb3-deploy.html)에 
codecommit의 repository만 지정할 수 있게 나와있다. 내부적으로는 S3에 업로드하지만 S3 주소를 따로 가리키는 option이 없기 때문에
eb-cli로만 진행할 수는 없었다.

---
### Default로 생성된 ALB를 삭제하고 TargetGroup 재지정 후 상태 확인
---

이 글을 쓰게된 계기는 기존에 사용하던 ALB에서 TargetGroup만 지정하여 사용할 수 있는지가 궁금해서 시작한 것이다.

하지만 Default로 EB를 AutoScalingGroup과 TargetGroup만 생성해서 만들지는 못하기 때문에 ALB와 함께 생성하였다.

이제 궁금한 부분은 같이 생성된 ALB를 삭제하였을 때 지속적으로 다시 ALB가 살아나는지 확인하고 기존의 TargetGroup을 엮어줘도 괜찮은가이다.

아래와 같은 프로세스로 진행해보았다.

1. EB와 생성된 ALB를 삭제한다.
2. 새로 ALB를 생성하고 ALB의 Security Group을 인스턴스들의 SecurityGroup으로 열어준다.
3. 동작 확인!

문제없이 동작은 하였지만 살짝 마음에 걸리는 부분은 있다.

EB는 아직 ALB가 삭제되었다고 표시되는 부분이 없다. 또한 아래와 같이 url이 나타나있지만 실제로는 ALB가 사라졌기 때문에 동작하지 않는다.
나중에 전체 구조를 다시 파악할 때 큰 어려움이 있을 것으로 보인다.

![eb_url](/images/aws/eb/url.png)


---
### References
---
- [http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html)
- [http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb3-deploy.html](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb3-deploy.html)

