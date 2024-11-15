---
title: "AWS EBS, FSx, EFS: Storage Comparison"
tags: ["AWS", "EBS", "FSx", "EFS", "Storage"]
date: "2024-11-07T00:30:00+00:00"
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

ML Data platform engineering 에서 Feature Store의 장기적인 성능 개선 과제를 고민하다가 cache 뿐만 아니라, disk storage를 활용하는 것 또한 검토해보고 있다. 왜냐하면 network latency를 줄이고, disk storage를 활용하면 remote cache보다 더 빠르게 데이터를 읽고 쓸 수 있을거라 기대하기 때문이다.
그래서 AWS에서 Disk storage처럼 활용할 수 있는 EBS, FSx와 EFS에 대해 알아보고자 한다.

---

# AWS EBS

AWS EBS(Elastic Block Store)는 다양한 종류의 볼륨 타입을 제공하며, 크게 SSD 기반과 HDD 기반으로 나눌 수 있다. 각 볼륨 타입은 성능 특성과 가격이 다르므로 애플리케이션의 요구사항에 맞게 선택할 수 있다.

## SSD 기반 볼륨

SSD 기반 볼륨은 잦은 읽기/쓰기 작업이 필요한 트랜잭션 워크로드에 최적화되어 있으며, IOPS(초당 입출력 작업)가 주요 성능 지표이다.

### 범용 SSD (gp2/gp3)

- **gp3**: 최신 버전으로, 기본 3,000 IOPS와 125MB/s의 처리량을 제공한다. 최대 16,000 IOPS와 1,000MB/s 처리량까지 확장 가능하다[2].
- **gp2**: 볼륨 크기와 IOPS가 연결되어 있으며, 최대 16,000 IOPS를 제공한다[2].
- 용도: 개발 환경, 테스트 환경, 부팅 볼륨 등 다양한 용도로 사용할 수 있다[2].

### 프로비저닝된 IOPS SSD (io1/io2/io2 Block Express)

- **io1/io2**: 4GiB에서 16TiB 용량을 제공하며, 최대 64,000 IOPS (Nitro 기반 EC2 인스턴스)를 지원한다[2].
- **io2 Block Express**: 최고 성능을 제공하며, 최대 256,000 IOPS와 밀리초 미만의 지연 시간을 제공한다[2][5].
- 용도: 고성능 데이터베이스, I/O 집약적인 애플리케이션에 적합하다[4].

## HDD 기반 볼륨

HDD 기반 볼륨은 대용량 스트리밍 워크로드에 최적화되어 있으며, 처리량(MB/s)이 주요 성능 지표이다.

### 처리량 최적화 HDD (st1)

- 최대 500MB/s의 처리량과 500 IOPS를 제공한다[2].
- 용도: 빅데이터, 데이터 웨어하우징, 로그 처리 등에 적합하다[2][6].

### Cold HDD (sc1)

- 최대 250MB/s의 처리량과 250 IOPS를 제공한다[2].
- 용도: 자주 접근하지 않는 데이터의 아카이브에 적합하다[2][6].

## 주요 특징

- **데이터 저장**: EC2 인스턴스에 연결하여 데이터를 저장할 수 있다[4].
- **스냅샷**: 볼륨의 특정 시점 스냅샷을 생성하고 복원할 수 있다[4].
- **볼륨 크기 조정**: 필요에 따라 동적으로 볼륨 크기를 조정할 수 있다[4].
- **복제**: 다른 EC2 인스턴스나 리전으로 볼륨을 복제할 수 있다[4].
- **높은 가용성**: 시스템 장애에 대비한 높은 가용성을 제공한다[4].

EBS 볼륨 선택 시 애플리케이션의 성능 요구사항, 비용, 데이터 접근 패턴 등을 고려하여 최적의 볼륨 타입을 선택해야 한다.

## Citations

- [1] https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html
- [2] https://ssunw.tistory.com/entry/EC2-%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4-%EC%8A%A4%ED%86%A0%EB%A6%AC%EC%A7%80-EBS-%EB%B3%BC%EB%A5%A8-%EC%9C%A0%ED%98%95
- [3] https://jayendrapatil.com/aws-ebs-volume-types/
- [4] https://lazismlee.com/44
- [5] https://bluexp.netapp.com/blog/ebs-volumes-5-lesser-known-functions
- [6] https://aws.amazon.com/ebs/volume-types/

# AWS FSx

AWS FSx는 클라우드에서 고성능 파일 시스템을 쉽게 구축, 실행 및 확장할 수 있는 완전 관리형 서비스이다. 주요 특징과 이점은 다음과 같다:

## 파일 시스템 옵션

AWS FSx는 네 가지 널리 사용되는 파일 시스템을 제공한다:

1. NetApp ONTAP
2. OpenZFS
3. Windows File Server
4. Lustre

이 중에서 워크로드 요구사항에 가장 적합한 파일 시스템을 선택할 수 있다[1].

## 주요 특징

### 높은 성능과 확장성

- 수백만 IOPS와 밀리초 미만의 지연 시간을 제공한다[1].
- 처리량 수준과 스토리지 유형을 선택할 수 있어 요구사항에 맞게 조정 가능하다[3].
- 필요에 따라 스토리지 용량을 증가시킬 수 있다[3].

### 고가용성 및 내구성

- 단일 AZ 또는 다중 AZ 배포 옵션을 제공한다[3].
- AWS 가용 영역 내 또는 간에 데이터를 자동으로 복제하여 구성 요소 장애로부터 보호한다[1].

### 보안 및 규정 준수

- 전송 중 및 저장 시 데이터를 자동으로 암호화한다[3].
- ISO, PCI-DSS, SOC 인증을 받았으며 HIPAA를 충족한다[3].
- VPC 내에서 실행하여 네트워크 액세스를 제어할 수 있다[3].

### 관리 용이성

- 하드웨어 프로비저닝, 패치 적용, 백업 등을 자동으로 처리한다[1].
- AWS Backup과 통합되어 중앙 집중식 백업 관리가 가능하다[3].

## 사용 사례

1. 엔터프라이즈 애플리케이션을 클라우드로 마이그레이션[1]
2. 머신 러닝, 분석 및 HPC 애플리케이션 구축[1]
3. 비즈니스 연속성 간소화 (백업, 아카이브, 복제)[1]
4. 개발 및 테스트 환경의 민첩성 향상[1]
5. Windows 애플리케이션 및 워크로드 지원 (FSx for Windows File Server의 경우)[2]

AWS FSx는 다양한 워크로드에 대해 신뢰성, 보안, 확장성 및 광범위한 기능을 제공하며, 최신 AWS 컴퓨팅, 네트워킹 및 디스크 기술을 기반으로 구축되어 높은 성능과 낮은 TCO(Total Cost of Ownership)를 제공한다[1].

## Citations:

- [1] https://aws.amazon.com/id/fsx/
- [2] https://docs.aws.amazon.com/fsx/latest/WindowsGuide/what-is.html
- [3] https://bluexp.netapp.com/blog/aws-fsxo-blg-aws-fsx-6-reasons-to-use-it-in-your-next-project
- [4] https://www.3ritechnologies.com/what-is-amazon-fsx/
- [5] https://aws.amazon.com/fsx/windows/features/


# AWS FSx의 최대 대역폭?
AWS FSx for Windows File Server의 최대 대역폭은 12 GB/s입니다. 이에 대한 자세한 내용은 다음과 같습니다:

1. Amazon FSx는 특정 AWS 리전에서 최대 12 GB/s의 처리량과 400,000 IOPS를 제공합니다[1]. 이 최대 성능을 제공하는 리전들:
    - 미국 동부 (버지니아 북부)
    - 미국 서부 (오레곤)
    - 미국 동부 (오하이오)
    - 유럽 (아일랜드)
    - 아시아 태평양 (도쿄)
    - 아시아 태평양 (싱가포르)

2. 2023년 8월 9일에 발표된 업데이트에 따르면, Amazon FSx for Windows File Server의 최대 처리량이 2 GB/s에서 12 GB/s로 증가되었음을 공지[3]. 이는 성능 집약적인 워크로드를 더 광범위하게 지원할 수 있음을 의미합니다.

3. 이 업데이트와 함께 최대 IOPS도 80,000에서 350,000으로 증가[3].

4. 새로운 처리량 용량 수준 선택 시, 디스크 IOPS도 함께 증가합니다. 예시로, 12 GB/s의 처리량 선택 시 최대 350,000 IOPS 제공[3].

5. 이러한 높은 처리량 용량 수준에 적합한 워크로드[3]:
    - 미디어 처리
    - 클라우드 비디오 편집
    - 데이터베이스 애플리케이션
    - 데이터 분석
    - 가상 데스크톱 인프라 (VDI)

6. 처리량 용량은 파일 시스템의 성능을 결정하는 주요 요소입니다. 네트워크 I/O, 파일 서버 CPU 및 메모리, 디스크 I/O 등의 성능에 영향을 줍니다[1].

## Citations:

- [1] https://docs.aws.amazon.com/fsx/latest/WindowsGuide/performance.html
- [2] https://docs.aws.amazon.com/ko_kr/fsx/latest/WindowsGuide/performance.html
- [3] https://aws.amazon.com/about-aws/whats-new/2023/08/amazon-fsx-windows-file-server-increases-throughput-12-gbs/

# AWS FSx최대 저장 용량?

AWS FSx의 최대 저장 용량은 파일 시스템 유형에 따라 다음과 같이 다름:

1. FSx for Windows File Server:
    - SSD 및 HDD 스토리지 모두 최대 64 TiB (65,536 GiB)까지 지원함[3].

2. FSx for Lustre:
    - 최대 100 PiB까지 확장 가능함[7].

3. FSx for ONTAP:
    - 최대 1 PiB (1,048,576 GiB)까지 지원함[6].

4. FSx for OpenZFS:
    - 최대 512 TiB (524,288 GiB)까지 지원함[2].

이러한 최대 용량은 AWS 리전과 특정 구성에 따라 다를 수 있음. 또한, 일부 파일 시스템의 경우 초기 생성 시 더 낮은 최대값이 적용될 수 있으며, 필요에 따라 스토리지 용량을 증가시킬 수 있다[1][2].

FSx의 스토리지 용량을 늘릴 때는 최소 10% 이상 증가시켜야 하며, 일부 파일 시스템의 경우 특정 증분 단위로만 증가가 가능하다[5][7].

## Citations:

- [1] https://docs.aws.amazon.com/fsx/latest/WindowsGuide/managing-storage-configuration.html
- [2] https://docs.aws.amazon.com/fsx/latest/OpenZFSGuide/managing-storage-capacity.html
- [3] https://docs.aws.amazon.com/fsx/latest/WindowsGuide/limits.html
- [4] https://docs.aws.amazon.com/fsx/latest/LustreGuide/performance.html
- [5] https://aws.amazon.com/blogs/aws/amazon-fsx-for-windows-file-server-storage-size-and-throughput-capacity-scaling/
- [6] https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/aws-resource-fsx-filesystem.html
- [7] https://aws.amazon.com/fsx/lustre/faqs/

# 같은 트래픽이라고 가정했을 때, FSx와 EBS 중에 무엇이 비용 효율적인지?

FSx와 EBS의 비용 효율성은 사용 사례와 워크로드 특성에 따라 다르다. 같은 트래픽을 가정했을 때, 일반적으로 다음과 같은 상황에서 비용 효율성이 달라진다:

## EBS가 비용 효율적인 경우

1. **단일 인스턴스 접근:** 하나의 EC2 인스턴스에서만 스토리지에 접근하는 경우, EBS가 더 비용 효율적이다.

2. **소규모 스토리지:** 작은 용량의 스토리지가 필요한 경우, EBS의 GB당 가격이 더 저렴하다.

3. **고성능 요구사항:** 높은 IOPS가 필요한 경우, EBS io1 또는 io2 볼륨이 비용 대비 성능이 좋다.

## FSx가 비용 효율적인 경우

1. **다중 인스턴스 접근:** 여러 EC2 인스턴스에서 동시에 파일 시스템에 접근해야 하는 경우, FSx가 더 효율적이다.

2. **대규모 스토리지:** 대용량 스토리지가 필요한 경우, FSx의 확장성과 가격 정책이 유리하다.

3. **Windows 환경:** Windows 기반 애플리케이션의 경우, FSx for Windows File Server가 더 적합하고 비용 효율적이다.

4. **데이터 압축:** FSx는 데이터 압축 기능을 제공하여 실제 저장 공간을 줄일 수 있다. 예를 들어, 50% 압축률을 가정하면 10TB 데이터에 대해 5TB만 프로비저닝하면 된다[3].

5. **백업 비용:** FSx는 증분 백업을 제공하여 백업 스토리지 비용을 절감할 수 있다[2].

## 비용 비교 예시

FSx for OpenZFS의 경우:
- SSD 스토리지: GB당 월 $0.18
- 처리량: MB/s당 월 $0.87
- 백업: GB당 월 $0.05 [2]

EBS의 경우:
- gp3 (범용 SSD): GB당 월 $0.08
- io1 (프로비저닝된 IOPS SSD): GB당 월 $0.125 [1]

이러한 가격을 바탕으로, 특정 워크로드에 대한 상세한 계산이 필요하다. 예를 들어, 다중 인스턴스 접근이 필요하고 데이터 압축이 가능한 10TB 워크로드의 경우 FSx가 더 비용 효율적일 수 있다.

## Citations:

- [1] https://aws.amazon.com/ebs/pricing/?nc2=type_a
- [2] https://aws.amazon.com/fsx/openzfs/pricing/
- [3] https://bluexp.netapp.com/blog/aws-fsxo-blg-aws-fsx-pricing-explained-with-real-life-examples
- [4] https://aws.amazon.com/fsx/windows/pricing/?nc1=h_ls

# AWS FSx에선 같은 파일에 쓰기 작업 이 되는 경우에 어떻게 처리되는지?

AWS FSx에서 같은 파일에 대한 동시 쓰기 작업이 발생할 경우, 다음과 같은 방식으로 처리된다:

1. **로킹 메커니즘 사용:**
    FSx는 Windows 파일 시스템의 표준 로킹 메커니즘을 사용한다. 이 메커니즘은 동일한 파일에 대한 동시 쓰기 작업을 조정하여 데이터 일관성을 유지한다. 파일에 대한 쓰기 작업이 진행 중일 때, 다른 클라이언트의 쓰기 요청은 일시적으로 차단된다.

2. **SMB 프로토콜 활용:**
    FSx for Windows File Server는 SMB(Server Message Block) 프로토콜을 사용한다. SMB 프로토콜은 파일 수준의 잠금을 지원하여 동시 접근을 관리한다.

3. **마지막 쓰기 우선:**
    여러 클라이언트가 동시에 같은 파일을 수정하려고 할 경우, 일반적으로 "마지막 쓰기 우선" 정책이 적용된다. 즉, 가장 마지막에 저장된 변경사항이 최종적으로 파일에 반영된다.

4. **버전 관리 지원:**
    FSx는 Shadow Copies 기능을 통해 파일의 이전 버전을 유지할 수 있다. 이를 통해 동시 쓰기로 인한 데이터 손실을 방지하고 필요시 이전 버전으로 복원할 수 있다.

5. **분산 파일 시스템 설계:**
    FSx는 분산 파일 시스템으로 설계되어 있어, 여러 클라이언트의 동시 쓰기 요청을 효율적으로 처리할 수 있다.

6. **성능 최적화:**
    FSx는 내부적으로 쓰기 작업을 버퍼링하고 캐싱하여 I/O 성능을 최적화하고 경합을 줄인다.

주의사항:

- FSx File Gateway를 사용할 경우, 글로벌 파일 잠금을 지원하지 않는다. 따라서 여러 게이트웨이 인스턴스에서 동시에 쓰기 접근을 허용하면 데이터 일관성 문제가 발생할 수 있다[4].

- 다중 클라이언트 환경에서 동시 쓰기 작업을 관리하기 위해서는 애플리케이션 레벨에서의 추가적인 동기화 메커니즘이 필요할 수 있다.

- 높은 동시성이 요구되는 워크로드의 경우, 충분한 처리량 용량을 프로비저닝하고 스토리지 최적화를 통해 성능을 향상시킬 수 있다[2].

FSx의 이러한 메커니즘들은 파일 시스템의 일관성을 유지하면서도 동시 접근을 효율적으로 관리할 수 있게 해준다. 그러나 특정 워크로드에 따라 추가적인 애플리케이션 레벨의 동기화가 필요할 수 있음을 고려해야 한다.

## Citations:
- [1] https://docs.aws.amazon.com/fsx/latest/WindowsGuide/self-managed-AD.html
- [2] https://docs.aws.amazon.com/fsx/latest/WindowsGuide/performance.html
- [3] https://docs.aws.amazon.com/fsx/latest/WindowsGuide/managing-storage-configuration.html
- [4] https://www.youtube.com/watch?v=OsziLgOPeH8


# Conclusion

고성능의 로컬 스토리지가 필요한 경우, AWS EBS io2 Block Express가 가장 적합할 것으로 보인다. 그 이유는:

1. **높은 IOPS**: 최대 256,000 IOPS를 제공하여 remote cache에 버금가는 성능을 제공
2. **낮은 지연시간**: 밀리초 미만의 지연 시간으로 빠른 응답 가능
3. **단일 서버 최적화**: 단일 EC2 인스턴스에 직접 연결되어 네트워크 지연을 최소화
4. **비용 효율성**: 다중 접근이 필요없는 단일 서버 환경에서는 FSx보다 비용 효율적

다만 고려사항:
- 단일 AZ에서만 사용 가능
- 고가용성이 필요한 경우 추가 설계 필요
- Nitro 기반 EC2 인스턴스에서만 최대 성능 발휘