---
title: AWS CloudFormation을 활용한 Architecture
category: AWS
tags: AWS, Amazon, Web, Service, CloudFormation, Architecture, infrastructure, CloudFormation, 이란, 활용, 시작하기
---

## Summary
---
 AWS CloudFormation의 Master Class를 보고 Reference document를 통해 내용을 살펴보자.

---
### 특징
---

- Infra structure as a code를 실현하기에 간편한 도구
- 리소스를 Provisioning하고 update를 해줌
- Code로 관리하기 때문에 버전관리에 용이
- AWS cli 또는 AWS console을 통해 배포 및 업데이트가 가능
- 리소스에 대해서만 과금되기 때문에 별도의 비용지출이 없음
- Parameter를 통해 Project별로 Customizing이 용이
- 코드만 올리면 인프라가 형성되기 때문에 인프라 도입에 대한 리소스 투입이 적음

<!-- ---
### Agenda
---

- Creating Templates
- Using a Template to Create and Manage a Stack
- Working with the CloudFormation API
- Working with AWS Resources
- Bootstrapping Applications and Handing Updates -->

---
### Cloudformation template의 특징
---

- JSON, YAML 로 개발자 친화적인 포맷
- 코드로 관리하기 때문에 재사용에 용이
- Stack 생성시 message를 통해 feedback 제공
- Sample template 제공

아래는 yaml형식의 ec2를 생성하는 sample template이다. CloudFormation으로 ec2를 생성할 때 파라미터를 받는다.
instanceType, KeyName, SSHLocation을 설정할 수 있도록 되어 있다.
선택할 수 있는 인스턴스의 종류를 제한했기 때문에 t2계열의 인스턴스만 선택할 수 있다. 그리고 AMI는 Amazon linux를 사용하였다.

```yaml
# EC2 sample template
AWSTemplateFormatVersion: 2010-09-09
Description: EC2 Sample
Parameters: # Using before cloudformation is being created
  KeyName:
    Description: Name of an existing EC2 KeyPair to enable SSH access to the instance. Recommend use only office IP.
    Type: 'AWS::EC2::KeyPair::KeyName'
    ConstraintDescription: must be the name of an existing EC2 KeyPair.
  InstanceType:
    Description: WebServer EC2 instance type
    Type: String
    Default: t2.small
    AllowedValues:  # allow only restricted instance
      - t2.nano
      - t2.micro
      - t2.small
      - t2.medium
      - t2.large
    ConstraintDescription: must be a valid EC2 instance type.
  SSHLocation:
    Description: The IP address range that can be used to SSH to the EC2 instances
    Type: String
    MinLength: '9'
    MaxLength: '18'
    Default: 0.0.0.0/0
    AllowedPattern: '(\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})/(\d{1,2})'
    ConstraintDescription: must be a valid IP CIDR range of the form x.x.x.x/x.
Mappings: # Conditional value when cloudformation is being created
  AWSInstanceType2Arch:
    t2.nano:
      Arch: HVM64
    t2.micro:
      Arch: HVM64
    t2.small:
      Arch: HVM64
    t2.medium:
      Arch: HVM64
    t2.large:
      Arch: HVM64
  AWSInstanceType2NATArch:
    t2.nano:
      Arch: NATHVM64
    t2.micro:
      Arch: NATHVM64
    t2.small:
      Arch: NATHVM64
    t2.medium:
      Arch: NATHVM64
    t2.large:
      Arch: NATHVM64
  AWSRegionArch2AMI:
    ap-northeast-2:
      PV64: NOT_SUPPORTED
      HVM64: ami-2b408b45
      HVMG2: NOT_SUPPORTED
Resources:
  EC2Instance:
    Type: 'AWS::EC2::Instance'
    Properties:
      InstanceType: !Ref InstanceType
      SecurityGroups:
        - !Ref InstanceSecurityGroup
      KeyName: !Ref KeyName
      ImageId: !FindInMap 
        - AWSRegionArch2AMI
        - !Ref 'AWS::Region'
        - !FindInMap 
          - AWSInstanceType2Arch
          - !Ref InstanceType
          - Arch
  InstanceSecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: Enable SSH access via port 22
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: '22'
          ToPort: '22'
          CidrIp: !Ref SSHLocation
Outputs:
  InstanceId:
    Description: InstanceId of the newly created EC2 instance
    Value: !Ref EC2Instance
  AZ:
    Description: Availability Zone of the newly created EC2 instance
    Value: !GetAtt 
      - EC2Instance
      - AvailabilityZone
  PublicDNS:
    Description: Public DNSName of the newly created EC2 instance
    Value: !GetAtt 
      - EC2Instance
      - PublicDnsName
  PublicIP:
    Description: Public IP address of the newly created EC2 instance
    Value: !GetAtt 
      - EC2Instance
      - PublicIp

```

한국 유저는 [Asia/seoul region cloudformation sample template](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-services-ap-northeast-2.html#w2ab2c23c22c13c13)
을 통해 여러 샘플들을 볼 수 있다.

다른 지역의 템플릿을 사용하고 싶으면 left bar에서 다른 region을 선택하면 된다.

---
### AWS CLI for CloudFormation
---

CloudFormation은 다른 서비스와 마찬가지로 AWS Cli를 통해 사용할 수 있다.
명령어들은 아래와 같다.

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
### Intrinsic functions & pseudo parameters
---

아래와 같은 function과 parameter를 통해 yaml형식의 파일에서 별도의 로직을 추가할 수 있다.

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

위에서 EC2를 생성하는 경우에 Fn::FindInMap와 Ref과 같은 것을 사용하였는데 ref는 글자에서 느껴지듯이 reference로 특정 변수를 가리킬 때 사용한다.
Fn::FindInMap은 별도로 Mapping에 두엇던 파라미터에서 값을 불러오기 위해 사용한다.

더 자세한 내용을 알고 싶다면 [AWS Intrinsic function reference](docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html)를 참고하도록 하자.

<!-- ---
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
      
   -->

    

---
### References
---

- [aws.amazon.com/cloudformation](aws.amazon.com/cloudformation)
- [aws.amazon.com/cloudformation/getting-started](aws.amazon.com/cloudformation/getting-started)
- [aws.amazon.com/cloudformation/aws-cloudformation-templates](aws.amazon.com/cloudformation/aws-cloudformation-templates)
- [github.com/awslabs/cfncluster](github.com/awslabs/cfncluster)
- [docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks.html](docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks.html)