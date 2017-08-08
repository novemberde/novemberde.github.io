---
title: AWS CloudFormation을 활용한 Architecture
category: AWS
tags: AWS, Amazon Web Service, CloudFormation, AWS Architecture, AWS infrastructure, CloudFormation 이란
---

## Summary
---
 AWS CloudFormation의 Master Class를 통해 내용을 살펴보자.

---
### 목표
---

- 기본을 넘어 기술적으로 더 자세히 살펴보기. A technical deep dive that goes beyond the basics
- AWS 서비스에서 가장 잘 사용하기 위한 방법. Intended to educate you on how to get the best from AWS services
- 어떻게 진행되는지 살펴본다. Show you how things work and how to get things done

---
### 특징
---

- AWS의 리소스들을 생성하고 관리하는 쉬운 방법. An easy way to create & manage a collection of AWS resources.
- 리소스를 예상가능하게 프로비져닝하고 업데이트하도록 해줌. Allows orderly and predictable provisioning and updating of resources
- AWS 인프라에 대해 버전 관리가 가능하도록 함. Allows you to version control your AWS infrastructure
- 콘솔 또는 명령어를 통해 스택을 배포 및 업데이트 할 수 있음. Deploy and update stacks using console, command line or API
- 생성한 리소스에 대해서만 과금. You only pay for the resources you create

---

- Transparent and Open
- Don't reinvent the whell
- Declarative & Flexible
- Noe Extra Charge
- Customized via Parameters
- Integration Ready

---

- CloudFormation - Components & Thechnology IMAGE

---
### Agenda
---

- Creating Templates
- Using a Template to Create and Manage a Stack
- Working with the CloudFormation API
- Working with AWS Resources
- Bootstrapping Applications and Handing Updates

---
### Creating Templates
---

- Familiar JSON Format
- Manage Relationships
- Reusable
- Provide Feedback
- Automate Generation
- Look Up Resources
- Write & Go
- Avoid Collisions

---
### AWS CLI actions for CloudFormation
---

```bash
# example
$ aws cloudformation update-stack help
```
- get-stack-policy
- set-stack-policy
- create-stack
- update-stack
- cancel-update-stack
- delete-stack
- list-stack-resources
- list-stacks
- describe-stack-events
- describe-stack-resouce
- describe-stack-resouces
- describe-stacks
- get-template
- validate-template

---
### Working with aws resources
---

- Designed to use your existing experience with AWS
- Each resource has a set of parameters with names that ar identical to the names used to create the resources through their native API

---
### Referencing the properties of another resource
---

- Reference function
- Literal references
- Referencing input parameters - cloundformation을 생성할 때 입력한다. key pairs와 같은 경우.
  - Validate parameters 기능.



---
### Conditional Value
---

- Mappings

---
### Intrinsic functions and pseudo parameters
---

- Intrinsic function
  - Fn::Base64
  - Fn::FindInMap
  - Fn::GetAtt
  - Fn::GetAZs
  - Fn::Join
  - Fn::Select
  - Ref
- Pseudo parameters
  - AWS::NotificationARNs
  - AWS::Region
  - AWS::StackId
  - AWS::StackName

---
### Working with non-AWS Resources
---

- Defining custom resources allows you to include non-AWS resources in a CloudFormation stack

---
### Bootstrapping Applications and handling updates
---

- Continue to use EC2 UserData, which is available as a property of AWS::EC2::Instance resources
- AWS CloudFormation provides helper scripts for deployment within your EC2 instances
- Creating files on Instance Filesystems
- Controlling Services
  - The services key allows you ensures that the services are not only running when cfn-init finishes (ensureRunning is set to true)
  - but that they are also restarted upon reboot (enabled is set to true)

All this functionality is available for Windows instances too.

Also, Chef and Puppet are available.

---
### 
---
AWS CloudFormation Stacks Updates

When you need to make changes to a stack's settings or change its resources, you update the stack instead of deleting it and creating a new stack. For example, if you have a stack with an EC2 instance, you can update the stack to change the instance's AMI ID.

When you update a stack, you submit changes, such as new input parameter values or an updated template. AWS CloudFormation compares the changes you submit with the current state of your stack and updates only the changed resources. For a summary of the update workflow, see How Does AWS CloudFormation Work?.

Note
When updating a stack, AWS CloudFormation might interrupt resources or replace updated resources, depending on which properties you update. For more information about resource update behaviors, see Update Behaviors of Stack Resources.
Update Methods

AWS CloudFormation provides two methods for updating stacks: direct update or creating and executing change sets. When you directly update a stack, you submit changes and AWS CloudFormation immediately deploys them. Use direct updates when you want to quickly deploy your updates.

With change sets, you can preview the changes AWS CloudFormation will make to your stack, and then decide whether to apply those changes. Change sets are JSON-formatted documents that summarize the changes AWS CloudFormation will make to a stack. Use change sets when you want to ensure that AWS CloudFormation doesn't make unintentional changes or when you want to consider several options. For example, you can use a change set to verify that AWS CloudFormation won't replace your stack's database instances during an update.

http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks.html

---
### 인프라 자동 배포를 위한 AWS CloudFormation 고급 활용법 - 박철수 솔루션즈 아키텍트 AWS Korea
---
- [https://www.youtube.com/watch?v=DpkB38n7Yv4](https://www.youtube.com/watch?v=DpkB38n7Yv4)

- 특징
  - 사용자 인프라용 템플릿 생성
  - 코드처럼 버전 관리 / 코드 검토 / 템플릿 업데이트
  - 종속성 요구에 기반하여 AWS 리소스 제공
  - 개발, CI/CD 및 관리 도구와 통합
  - 추가적인 비용 없음
- 주요 기능
  - JSON 또는 YAML로 템플릿 작성
  - 변경 집합(change set)를 사용하여 변경 내용 미리 보기
  - 내보내기(export)로 크로스-스택 참조 가능
  - 스택을 위한 CD workflow
- 구문 개선

- CloudFormation 디자이너 YAML 지원
  - 템플릿 리소스 시각화
  - 드래그-앤-드롭 제스처로 템플릿 수정
  - 샘플 템플릿 사용자 정의
  - YAML 템플릿 지원

- CloudFormation 변경 집합(change set)
  - 스택을 생성하거나 업데이트하기 전에 CloudFormation이 사용자를 대신하여 수행할 액션 집합을 미리 보여줌
  - Change set은 어떤 리소스가 생성, 업데이트 또는 대체될지 보여주며, 이를 통해서 기대하는 작업만 실행할 수 있음
  - 순서
    - 원본스택
      1. 변경 집합(change set) 생성
    - 변경 집합 (change set)
      2. 변경 집합 보기
    - 변경 집합(change set)
      3. 변경 집합 실행
    - AWS CloudFormation이 스택 업데이트

- 크로스-스택 참조(Cross-Stack Reference)
  - 독립적인 스택간에 정보 공유 가능
  - 스택 출력값(Output)을 내보내기(Export)하면, 동일 계정 및 리전내의 다른 스택에서 내보낸 값을 가져올 수 있음

- 내포된 스택 (Nested stack)
  - 모든 아키텍쳐를 하나의 템플릿에 넣어서 관리하지 않고 별도로 분리해서 관리하는 스택
  - 자주 사용되는 리소스가 있는 템플릿들을 작성하고 재사용
  - 구조
    - Parent Template(Application)
      - Child nested template (Network Resources)
      - Child nested template (ECS Service)


- 파이프라인에 대한 변경을 모델링, 프로비저닝 및 관리하기 위해 CloudFormation 사용
- AWS CodePipeline 
  - 빠르고 신뢰성 있는 애플리케이션 및 인프라 업데이트를 위한 지속적 전달 서비스
  - 코드 변경이 있을 때마다 빌드, 테스트 및 배포
  - CloudFormation이 CodePipeline에 built in 되어 있다.(Action mode > Create or update a stack)
  - 단계
    - 소스
      - CloudFormation 템플릿을 위한 소스 단계
      - AWS CodeCommit, S3 및 Github와 같은 저장소와 연동
    - 테스트
      - 실행하기 앞서 CloudFormation의 Change Set을 사용하여 배포 검증
    - 배포
      - 스택 또는 변경 집합을 생성, 업데이트 및 삭제
  - 파이프라인 모델링
    - 네트워크 리소스 파이프라인
      - 자신의 영역마다 네트워크 리소스를 별도로 관리
      - 샌드 박스와 프로덕션 네트워크 환경을 반영하고 따로 관리
      - 순서
        - Source repository
        - 샌드박스/ 개발 환경을 위한 네트워킹 리소스
        - 개별 스택, 종속성을 고려한 순서
        - 프로덕션 환경의 변경을 미리보기 위한 변경집합
        - 변경이 프로덕션에 적용되기 전에 수동 승인
        - 프로덕션에 변경 적용
    - 어플리케이션 파이프라인
      - 애플리케이션과 인프라 코드에서 더 자주 반복
      - 개발환경에서 새 버전을 시작해보고 프로덕션에서 실행
      - 순서
        - 새 버전이 게시되는 즉시 실행되는 파이프라인
        - 테스트를 실행하고, 완료되면 개발 환경을 제거. 사용하지 않는 인스턴스에 대한 요금이 부과되지 않음
        - 리소스 수정 또는 교체가 예상대로 이루어지는지 검토
        - 프로덕션에 변경사항을 지속적으로 전달
  - CloudFormation을 이용하여 파이프라인 생성 및 관리
    - 파이프라인을 성정하기 위한 CloudFormation 템플릿
      - 파이프라인 artifacts s3 bucket
      - 파이프라인 알림 SNS 이메일 알림
      - 파이프라인
      - IAM role 지정
        - 별도로 크로스-스택 참조를 사용해서 별도의 스택 안에서 IAM 리소스 프로비저닝 가능.
      
  

    

---
### References
---

- [aws.amazon.com/cloudformation](aws.amazon.com/cloudformation)
- [aws.amazon.com/cloudformation/getting-started](aws.amazon.com/cloudformation/getting-started)
- [aws.amazon.com/cloudformation/aws-cloudformation-templates](aws.amazon.com/cloudformation/aws-cloudformation-templates)
- [github.com/awslabs/cfncluster](github.com/awslabs/cfncluster)