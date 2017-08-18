---
title: Serverless을 이용한 AWS Lambda의 배포 자동화
category: AWS
tags: AWS, Amazon, Web, Service, lambda, serverless, api, gateway, apigateway, deploy, 배포, 자동화
---

## Summary
---
AWS Lambda와 api gateway를 사용하여 작업하면 배포하는 부분에서 상당부분 시간을 사용한다.
또한 API Gateway와 lambda를 엮는 것은 별도의 설정 과정이 필요하며, Resource & Stage 개념이 있어서
변경사항이 생길 경우에 API배포를 매번 해주어야 한다.

Serverless framework는 이 모든 것을 자동화해주며, 내부적으로 CloudFormation을 사용하기 때문에
API gateway에서 api변동사항을 더욱 쉽게 반영해줄 수 있다.

---
### Serverless framework 란?
---

The open-source, application framework to easily build serverless architectures on AWS Lambda & more.
Startups and Fortune 500 companies are using it to build incredibly efficient applications.

오픈소스이며, 쉽게 서버리스 아키텍쳐를 AWS나 다른 클라우드 서비스에서 빌드할 수 있는 어플리케이션 프레임워크이다.
스타트업이나 포춘지 500여개의 회사가 효율적인 어플리케이션 빌딩에 사용하고 있다.

출처: [https://serverless.com/](https://serverless.com/)


---
### Serverless framework 특징
---
- Move Fast
  - Provision and deploy a REST API, data pipe-line, or one of many other use cases in minutes.
- Simplicity
  - Serverless CLI makes it simple to manage and build a serverless architecture by abstracting away provider-level complexity.
- 100% Utilization
  - Pay when your code runs, so you never have to worry about paying for idle server time.
- Collaboration
  - Serverless provide a flexible application structure for easy management of code, resources, and events across large projects & teams.
- Community
  - Serverless is an MIT Open-source project, actively maintained by a vibrant and engaged community of developers.
- Infinite Scaleability
  - Framework users are reacting to bilions of events per month on AWS Lambda infrastructure

---
### Serverless framework 가 지원하는 클라우드 서비스
---

- Amazon Web Services
- Microsoft Azure
- IBM OpenWhisk
- Google Cloud Platform

메이저급의 회사는 모두 지원하고 있다. 
이번에는 AWS Lambda를 통해 배포해볼 것이다.

---
### Serverless framework 사용해보기
---
배포하기에 앞서서 사전에 설치되어 있어야 할 것이 있다.

- [nodejs & npm](https://nodejs.org/ko/download/package-manager/)
- [AWS Cli](https://aws.amazon.com/ko/cli/)

링크된 곳에서 모두 설치하고 터미널을 열자.

serverless가 설치되지 않을 수도 있다.
이럴 땐 관리자 권한으로 터미널을 실행하야 설치할 수 있다.
npm을 global로 설치하기 때문에 권한 상승이 필요하다.

```bash
# Serverless framework 설치하기
$ npm install -g serverless

# 배포를 연습할 디렉터리 생성하고 해당 디렉터리로 들어가기
$ mkdir serverless-practice
$ cd serverless-practice

# Serverless framework template 생성하기
$ serverless create --template hello-world
```

---

이와같이 진행하면 아래와 같은 파일 트리가 생성된다.

```
serverless-practice
│  .gitignore
│  handler.js    
└  serverless.yml
```

---

serverless.yml을 보면 아래와 같다.
```yaml
# Welcome to serverless. Read the docs
# https://serverless.com/framework/docs/

# Serverless.yml is the configuration the CLI
# uses to deploy your code to your provider of choice

# The `service` block is the name of the service
service: serverless-hello-world

# The `provider` block defines where your service will be deployed
provider:
  name: aws
  runtime: nodejs6.10

# The `functions` block defines what code to deploy
functions:
  helloWorld:
    handler: handler.helloWorld
    # The `events` block defines how to trigger the handler.helloWorld code
    events:
      - http:
          path: hello-world
          method: get
          cors: true
```

위를 살펴보면 각 function에 대한 handler를 지정할 수 있으며, 서비스 명을 지정할 수 있는 것을 볼 수 있다.

---

이제 배포해보자.

```bash
$ serverless deploy
```

배포가 되지 않을 것이다. Credential 관련해서 따로 설정해준 적이 없기 때문이다.

AWS IAM console로 들어가서 Programmatic access유저를 생성하고 정신건강을 위해 policies는 AdministratorAccess권한을 주자.

최소한의 권한으로 주고 싶다면 [Minimal IAM policy for Serverless Issue](https://github.com/serverless/serverless/issues/588)를 참고해보자.

생성을 마친다면 AccessKeyId 와 SecretAccessKeyId 가 발급된다.

이젠 다시 터미널로 돌아와서 아래와 같이 진행하자.

```bash
$ aws configure
AWS Access Key ID[]: # AccessKeyId 입력.
AWS Secret Access Key ID[]: # SecretAccessKeyId 입력.
```

AWS계정과 연결이 완료되었다.

다시 배포해보자

```bash
$ serverless deploy
```

---

Serverless로 배포하게 된다면 내부적으로는 CloudFormation Template을 생성하고 
AWS 계정을 통해 CloudFormation으로 배포하게 된다.

궁금하신 분들은 deploy를 한 후에 CloudFormation을 살펴보면서
각 이벤트에 대해 S3, Lambda가 어떻게 반응하는지 확인해보면 될 것이다.

---

번외로 AWS 말고도 다른 클라우드 서비스에 대해서도 템플릿이 존재한다.

```bash
$ serverless create --template [아래의 목록 중 하나]
"aws-nodejs", "aws-python", "aws-python3", "aws-groovy-gradle", 
"aws-java-maven", "aws-java-gradle", "aws-scala-sbt", "aws-csharp", 
"aws-fsharp", "azure-nodejs", "openwhisk-nodejs", "openwhisk-python", 
"openwhisk-swift", "google-nodejs", "plugin", "hello-world"
```

---
### References
---

- [https://serverless.com/](https://serverless.com/)
- [https://serverless.com/framework/docs/](https://serverless.com/framework/docs/)