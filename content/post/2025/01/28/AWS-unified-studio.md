---
title: "AWS Unified Studio 소개"
tags: ["aws", "unified-studio", "studio", "ai", "ml", "llm"]
date: "2025-01-28T00:30:00+00:00"
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

Amazon SageMaker Unified Studio는 데이터 분석과 AI 도구를 단일 플랫폼으로 통합한 포괄적인 개발 환경이다.

Unified Studio는 기존 SageMaker Studio의 단순한 진화가 아닌, AWS의 분석 및 AI/ML 서비스를 통합한 완전히 새로운 환경이다. Amazon Athena, Amazon EMR, AWS Glue, Amazon Redshift, Amazon MWAA 등 AWS의 친숙한 분석 도구들이 하나의 인터페이스로 통합되어 있다.

또한 Amazon Bedrock IDE가 통합되어 있어 생성형 AI 개발이 용이하며, Amazon Q Developer를 통해 개발 수명주기 전반의 작업을 가속화할 수 있다. 이를 통해 데이터 분석부터 AI/ML 모델 개발, 배포까지 전체 과정을 단일 환경에서 수행할 수 있다.

다음을 통해 Unified Studio의 주요 특징을 알아본다.

## 특징

Amazon SageMaker Unified Studio는 단일 URL을 통해 모든 지역과 계정의 프로젝트 및 도구에 접근할 수 있는 통합 개발 환경을 제공한다. Amazon Q와의 기본 통합으로 생산성을 높일 수 있으며, 프로젝트 기반 워크플로우를 통해 팀원들과 원활한 협업이 가능하다.

도구 측면에서는 다양한 소스의 데이터를 탐색할 수 있는 쿼리 에디터와 데이터 자산에 접근하기 위한 통합 데이터 탐색기를 제공한다. 또한 데이터 변환 워크플로우를 위한 시각적 ETL 도구와 크로스 컴퓨팅 서비스와 호환되는 통합 Jupyter 노트북 환경을 포함하고 있다.

Unified Studio는 도메인을 통해 스튜디오의 전반적인 설정을 관리한다. 도메인을 통해 사용자 접근, 권한, 네트워크 구성을 일관되게 관리할 수 있으며, 유연한 계정 및 데이터 공유 아키텍처를 지원한다.

또한, 비용 구조는 사용량 기반 과금 모델을 채택했다. SageMaker Unified Studio 플랫폼 자체의 사용은 무료이지만, SageMaker Catalog 사용과 개별 AWS 서비스 사용에 대한 요금이 부과된다. 또한 Git 제공자와 같은 서드파티 서비스를 사용할 경우 추가 비용이 발생할 수 있다.

보안 측면에서는 AWS IAM Identity Center를 통한 싱글 사인온을 지원하며, SAML과 IAM 사용자 인증 옵션을 제공한다. 사용자와 그룹을 위한 접근 제어 정책을 구현할 수 있으며, 프로젝트, 데이터, 모델에 대해 최소 권한 원칙을 적용하여 보안을 강화했다.

## 살펴보기

### 도메인 생성

현재(2025-01-28) 미리보기 버전이 제공되고 있다. [AWS SageMaker 콘솔](https://us-east-1.console.aws.amazon.com/datazone/home?region=us-east-1#/)에서 시작할 수 있다. 다음과 같은 화면에서 `Create a Unified Studio domain` 버튼을 클릭하면 도메인을 생성할 수 있다.

![unified-studio-create-domain](/images/unified_studio/SageMaker_Unified_Studio_Create_Button.png)

### SSO 설정하기

회사에서 AWS상에서 데이터 관련 서비스를 연동할 때 여러 불편한 점이 있었을 것이다. AWS 계정을 개인에게 부여하는 것은 부담이 되는 상황이었고, 이로 인해 보안 문제가 발생할까봐 오히려 보수적으로 접근하여 권한을 부여하기도 했다.

이런 불편함은 Unified Studio에서 어느정도 피해갈 수 있다.

OKTA, Google Workspace, Microsoft 365, Auth0 등의 서비스를 통해 사용자 인증을 할 수 있다.

![unified-studio-sso](/images/unified_studio/SageMaker_Unified_Studio_SSO.png)

인증 설정이 끝났으면, `Amazon SageMaker Unified Studio URL`을 클릭하여 나만의 Unified Studio에 접근할 수 있다.

### 구조 살펴보기

### Discover

다음과 같이 접근하면 Discover에서 내가 가진 데이터셋이나, 아니면 Unified Studio에서 기본적으로 활용할 수 있는 것을 살펴볼 수 있다.

![unified-studio-discover](/images/unified_studio/SageMaker_Unified_Studio_Discover.png)

이는 Data Catalog인데, 우리가 가진 Business Glossary, Metadata, Asset type 등을 확인할 수 있다.

![unified-studio-discover-data-catalog](/images/unified_studio/SageMaker_Unified_Studio_Discover_Data_Catalog.png)

이 뿐만 아니라, Bedrock models에 접근하면 Bedrock IDE가 내장되어있는데, 다음과 같이 채팅으로 기존 활성화된 모델을 활용할 수 있다.

![unified-studio-discover-bedrock-ide](/images/unified_studio/SageMaker_Unified_Studio_Discover_Bedrock_IDE.png)

### Build

다음과 같이 서비스에 접근하면 어떠한 기능이 있는지 확인할 수 있다.

![unified-studio-service](/images/unified_studio/SageMaker_Unified_Studio_Services.png)

아직 Preview라 그런지 Project 생성이나, 각 기능을 사용할 때 막히는 경우에 대해서 뚜렸한 원인과 해결방법이 부족하다. 이는 추후에 서비스가 General Availability로 전환되면 다시금 확인해보도록 한다.

아래는 각 기능이 어떤 것을 제공하는지 정리해보았다.

#### IDE & APPLICATIONS
JupyterLab은 데이터 과학자와 개발자를 위한 웹 기반 통합 개발 환경이다. PyTorch, TensorFlow, Keras, NumPy, Pandas, Scikit-learn 등의 패키지가 기본적으로 포함되어 있다. Partner AI Apps는 다양한 파트너사의 AI 애플리케이션을 통합 제공하며, 모든 데이터는 사용자의 보안 구성 내에서 유지되어 제3자와 공유되지 않는다.

#### DATA ANALYSIS & INTEGRATION
Query Editor는 데이터베이스 쿼리 작성 및 실행이 가능하며, Amazon Q를 통해 자연어로 질문하여 SQL 쿼리를 생성할 수 있는 기능을 제공한다. Visual ETL flows는 AWS Glue interactive sessions Version 5.0을 활용하여 데이터의 추출, 변환, 적재 과정을 시각적으로 설계하고 관리할 수 있다.

#### ORCHESTRATION
Workflows는 Apache Airflow를 기반으로 데이터 처리 절차를 모델링하고 코드 아티팩트를 조율하는 자동화 도구이다. ML Pipelines는 시각적 파이프라인 디자이너를 통해 머신러닝 모델의 학습, 평가, 배포 과정을 자동화하는 파이프라인을 구축하고 관리할 수 있다.

#### MACHINE LEARNING & GENERATIVE AI
**App Development**
Chat agent는 Amazon Bedrock 모델을 활용하여 대화형 AI 시스템을 개발하고 배포할 수 있다. Flow는 AI 애플리케이션의 로직을 시각적으로 구성하며, Prompt는 AI 모델의 프롬프트를 체계적으로 관리한다.

**Model Development**
Jumpstart models는 사전 훈련된 다양한 AI 모델을 제공하여 빠른 개발을 지원한다. Training jobs는 모델 학습 작업을 관리하고, Inference endpoints는 학습된 모델을 API 형태로 서비스할 수 있게 한다.

**AI OPS**
Model registry는 학습된 모델의 버전을 관리하고 승인 워크플로우를 제공하며, Model evaluations는 모델의 성능을 다양한 메트릭으로 평가하고 결과를 시각화한다.

## Govern

중요한 항목이다. 거버넌스 관련 기능이고, 유저를 관리한다던지, 블루프린트를 통해 프로젝트 멤버들이 사용할 수 있는 도구와 서비스를 정의할 수 있다.

SSO Group을 통해 유저를 관리할 수 있어서 데이터 플랫폼이 일정 규모가 되더라도 권한 관리가 용이하다.

![unified-studio-govern](/images/unified_studio/SageMaker_Unified_Studio_Govern.png)

## 마무리

전부 살펴보고 싶어서 정리를 시작했지만, 아직 Preview라 그런지 활성화된 기능이 완벽하게 동작한다는 느낌은 아직 부족하다.

다만 AWS의 방향성에서 ML + Data 관련한 서비스를 통합하는 것이 느껴지고 있고, AWS 계정에 국한되는 것이 아닌, 별도의 서비스로 느껴지게 디자인 했다는 것을 느낄 수 있었다.

AWS에서 많은 회사들이 인프라를 운영하고 있는데, 이를 통해 쉽게 ChatOps를 구축할 수 있다는 점이 매력적이다. 최근들어서 보안으로 인해 각 회사에서 내부적으로 Chatting interface를 구현하는 것을 보았는데, 권한을 Chat만 열어둔 Project, MLOps를 위한 Project 등 목적에 맞게 프로젝트를 생성한다면 운영하는 것이 좀 더 쉬워질 것으로 기대된다.

AWS에 많은 서비스가 있어서 시작하는 사람들에게 부담스러웠는데, 이번 SageMaker Unified Studio가 출시되면서 데이터 관련 플랫폼 구축하는 것이 좀 더 쉬워질 것으로 기대된다.

만약 관심이 있다면 다음의 두 링크에서 자세히 살펴보길 바란다. 이걸 미리 봤으면 글을 쓰지 않았겠지만, 혹시나해서 다시 찾아보니 위에서 테스트해보지 못한 화면들을 확인할 수 있다.

- 살펴보기: https://aws.amazon.com/blogs/aws/introducing-the-next-generation-of-amazon-sagemaker-the-center-for-all-your-data-analytics-and-ai/
- Unified Studio 각 기능이 설명된 페이지 모음: https://github.com/aws/Unified-Studio-for-Amazon-Sagemaker?tab=readme-ov-file

## References

- https://aws.amazon.com/ko/about-aws/whats-new/2024/12/preview-amazon-sagemaker-unified-studio/?nc1=h_ls
- https://aws.amazon.com/ko/blogs/korea/introducing-the-next-generation-of-amazon-sagemaker-the-center-for-all-your-data-analytics-and-ai/
- https://aws.amazon.com/ko/sagemaker/
- https://aws.amazon.com/ko/sagemaker-ai/faqs/
- https://aws.amazon.com/ko/sagemaker/pricing/?nc1=h_ls

