---
title: "2024 AWS Serverless Computing 종류 정리"
tags: [AWS, serverless, computing, Lambda, Fargate, Eventbridge, Step Functions, SQS, SNS, API Gateway, AppSync, S3, EFS, DynamoDB, Aurora, Redshift, Neptune, OpenSearch, ElastiCache]
date: "2024-10-23T10:30:00+00:00"
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

**1. Computing & Application**

**AWS Lambda**
- 소개: 이벤트 기반의 서버리스 컴퓨팅 서비스
- 해결문제: 짧은 실행 시간의 작업, 이벤트 처리, API 요청 처리
- 대체서비스: EC2, ECS, EKS
- 규모: 
  - 최소: 128MB 메모리, 실행시간 제한 15분
  - 최대: 10GB 메모리, 동시실행 1000개 (증설 가능)

**AWS Fargate**
- 소개: 컨테이너화된 애플리케이션을 위한 서버리스 컴퓨팅 엔진
- 해결문제: 컨테이너 운영에 따른 서버 관리 부담 제거
- 대체서비스: ECS/EKS with EC2
- 규모:
  - 최소: vCPU 0.25개, 메모리 0.5GB
  - 최대: vCPU 16개, 메모리 120GB

**2. Integration & Orchestration**

**Amazon Eventbridge**
- 소개: 서버리스 이벤트 버스 서비스
- 해결문제: 분산 시스템 간 이벤트 기반 통합
- 대체서비스: Apache Kafka, RabbitMQ
- 규모:
  - 최소: 초당 1개 이벤트
  - 최대: 초당 10,000개 이벤트 (증설 가능)

**AWS Step Functions**
- 소개: 서버리스 워크플로우 오케스트레이션 서비스
- 해결문제: 복잡한 비즈니스 프로세스 자동화
- 대체서비스: Airflow, Jenkins
- 규모:
  - 최소: 상태 전환 1개
  - 최대: 실행시간 1년, 25,000개 상태전환

**3. Messaging**

**Amazon SQS**
- 소개: 완전관리형 메시지 큐 서비스
- 해결문제: 시스템 간 비동기 통신, 부하 분산
- 대체서비스: RabbitMQ, Apache ActiveMQ
- 규모:
  - 최소: 메시지 1개
  - 최대: 무제한 메시지, 메시지 크기 256KB

**Amazon SNS**
- 소개: 완전관리형 pub/sub 메시징 서비스
- 해결문제: 다중 구독자에게 메시지 발행
- 대체서비스: Apache Kafka
- 규모:
  - 최소: 토픽 1개
  - 최대: 토픽당 1천만개 구독, 메시지 크기 256KB

**4. API Services**

**Amazon API Gateway**
- 소개: API 생성, 관리, 모니터링 서비스
- 해결문제: REST/WebSocket API 관리
- 대체서비스: Nginx, Kong
- 규모:
  - 최소: API 1개
  - 최대: 초당 10,000개 요청 (증설 가능)

**AWS AppSync**
- 소개: 완전관리형 GraphQL 서비스
- 해결문제: 실시간 데이터 동기화, GraphQL API 구현
- 대체서비스: 직접 GraphQL 서버 구축
- 규모:
  - 최소: Schema 1개
  - 최대: 초당 2,500개 실시간 클라이언트

**5. Storage**

**Amazon S3**
- 소개: 확장 가능한 객체 스토리지 서비스
- 해결문제: 대용량 파일 저장, 정적 웹사이트 호스팅
- 대체서비스: 직접 관리하는 파일 서버
- 규모:
  - 최소: 객체 1개 (최소 0바이트)
  - 최대: 객체 크기 5TB, 버킷당 무제한 객체

**Amazon EFS**
- 소개: 완전관리형 파일 시스템
- 해결문제: 여러 컴퓨팅 리소스간 파일 공유
- 대체서비스: NFS 서버
- 규모:
  - 최소: 1KB
  - 최대: 페타바이트 규모

**6. Database**

**Amazon DynamoDB**
- 소개: 완전관리형 NoSQL 데이터베이스
- 해결문제: 대규모 확장이 필요한 키-값 저장소
- 대체서비스: MongoDB, Cassandra
- 규모:
  - 최소: 1RCU/1WCU
  - 최대: 무제한 스토리지, 테이블당 초당 수만 건의 요청

**Amazon Aurora Serverless**
- 소개: 자동 확장/축소되는 관계형 데이터베이스
- 해결문제: 가변적인 워크로드 처리
- 대체서비스: Aurora Provisioned
- 규모:
  - 최소: 0.5 ACU
  - 최대: 256 ACU

**Amazon Redshift Serverless**
- 소개: 서버리스 데이터 웨어하우스
- 해결문제: 대규모 데이터 분석
- 대체서비스: Redshift Provisioned
- 규모:
  - 최소: 8 RPU (Redshift Processing Units)
  - 최대: 512 RPU

**Amazon Neptune Serverless**
- 소개: 서버리스 그래프 데이터베이스
- 해결문제: 복잡한 관계 데이터 처리
- 대체서비스: Neptune Provisioned
- 규모:
  - 최소: 1 NCU (Neptune Capacity Unit)
  - 최대: 128 NCU

**Amazon OpenSearch Serverless**
- 소개: 서버리스 검색 및 분석 엔진
- 해결문제: 전문 검색, 로그 분석
- 대체서비스: OpenSearch Provisioned
- 규모:
  - 최소: 2 OCU (OpenSearch Compute Units)
  - 최대: 100 OCU

**Amazon ElastiCache Serverless**
- 소개: 서버리스 인메모리 캐싱
- 해결문제: 데이터베이스 부하 감소, 응답 속도 향상
- 대체서비스: ElastiCache Provisioned(Redis, Memcached)
