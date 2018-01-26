---
title: Serverless Framework와 CircleCI를 통한 NoOps 맛보기
category: aws
tags: serverless, circleci, noops, demo, 맛보기, aws, framework, 서버리스, 데모
---
## Summary
---
AWSKRUG의 Architecture모임에서 Serverless + CircleCI로 NoOps맛보기라는 주제로 발표를 하였다.
간단하게 데모시연을 하였지만 좋은 질문을 많이 받았기 때문에 
따로 정리할 필요를 느껴 데모 튜토리얼과 질문에 대해 답변했던 내용을 정리한다.

---
## 기초 내용
---

Serverless Framework와 CircleCI에 대한 소개는 발표 자료 및 공식 사이트를 참고한다.

- [Serverless framework](https://serverless.com/)
- [CircleCI](https://circleci.com/)

<iframe src="//www.slideshare.net/slideshow/embed_code/key/5BFDCV65TLUncA" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/KyuhyunByun1/serverless-framework-circleci-noops" title="Serverless framework와 CircleCI를 통한 NoOps 맛보기" target="_blank">Serverless framework와 CircleCI를 통한 NoOps 맛보기</a> </strong> from <strong><a href="https://www.slideshare.net/KyuhyunByun1" target="_blank">Kyuhyun Byun</a></strong> </div>

---
## 개발 및 배포 환경

- Github repo.
- CircleCI 계정
- Typescript로 구성한 node.js 서버
- Github repository


---
### 준비하기

1. Github repository를 생성한다. Public으로 두던 Private으로 두던 상관없다. CircleCI에서 Github 계정 권한을 통해 repository에 접근하기 때문이다.
2. CircleCI 계정을 생성한다. Github계정을 연동하면 손쉽게 처리할 수 있다.

---
### Typescript로 node.js 서버 만들기

간단한 데모이지만 Typescript로 구성하는 이유가 있다. Typescript로 구성하면 javascript로 compile하는 작업이 필요한데,
이러한 build과정을 CircleCI에서 진행하기 위함이다.

CircleCI에서 CD/CI의 각 단계를 가볍게 표현해보기 위함이다.

코드를 직접 작성하는데 문제가 생길 수 있으니 다음의 repository를 fork하여 구현한다.

- [https://github.com/novemberde/serverless-webapp-demo](https://github.com/novemberde/serverless-webapp-demo)

fork한 repository를 로컬에 clone하여 수정해본다.

그 전에 앞서서 디렉터리 구조에 대한 설명은 다음과 같다

```txt
serverless-webapp-demo
|
├── .circleci
|   └── config.yml  // CircleCI에서 진행할 Pipeline을 정의한다.
├── src
|   ├── spec
|   |   └── index.spec.ts // 서버가 동작하는지 확인하는 Test 코드
|   ├── App.ts      // express 서버를 라우팅 하기 위한 코드
|   ├── handler.ts  // express 를 AWS Lambda에서 실행 가능하도록 변환해주는 코드
|   └── www.ts      // 로컬에서 실행해보기 위한 코드
├── .gitignore      // .serverless, bin 디렉터리를 제외함
├── package.json
├── serverless.yml  // serverless framework 를 사용하기 위한 configuration
└── tsconfig.json   // 사용하는 typescript의 configuration
```

---
## 코드의 구성
---

#### [./serverless.yml]

```yml
service: aws-nodejs

provider:
  name: aws
  runtime: nodejs6.10
  memorySize: 128
  stage: dev
  region: us-east-1
  environment:
    NODE_ENV: production

plugins:
 - serverless-apigw-binary
custom:
  apigwBinary:
    types:
      - '*/*'

functions:
  hello:
    handler: bin/handler.app
    events:
      - http: ANY {proxy+}
```

---
#### [./.circleci/config.yml]

```yml
version: 2
jobs:
  build:
    docker:
      - image: novemberde/node-awscli # aws-cli가 사전에 설치되어 있는 Docker image.
                                      # 기본으로 제공되는 이미지는 serverless deploy를 했을 때 aws로 배포가 되지 않는다.
    working_directory: ~/repo
    steps:
      - checkout
      - restore_cache:
          keys:
            - v1-dependencies-{{ checksum "package.json" }}
      - run: npm install              # 빌드를 하기 전에 필요한 패키지를 install 한다.
      - save_cache:
          paths:
            - node_modules
          key: v1-dependencies-{{ checksum "package.json" }}
      # Build
      - run: npx tsc                  # typescript로 구성된 코드를 ./bin/ 디렉터리에 javascript코드로 compile한다.

      # Test
      - run: npm test                 # 정해진 spec에 대해 test를 진행한다.

      # Deploy App
      - run: npx serverless deploy    # serverless framework를 통해 deploy한다.
```

---
## 먼저 Admin권한으로 CloudFormation 설정하기
---

CircleCI에 AWS AccessKey를 입력하기 전에 염두해두어야 할 것이 있다. CircleCI에 AdminisratorAccess 권한의 키를 주어 사용하면 편하지만 보안을 고려한 관점으로 봤을 땐
문제가 생길 여지가 있다. 그래서 처음에 한 번 배포를 할 때만 AdminisratorAccess의 권한을 주고 다음부턴 최소의 권한으로만 접근한다.

