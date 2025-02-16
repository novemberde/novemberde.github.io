---
title: "Golang Project Guide from A to Z: From Feature Development to Enterprise Application Design"
tags: [golang, go, gophercon, engineering, chatting]
date: "2024-10-14T11:30:03+00:00"
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

## Summary

"고언어 프로젝트 가이드 A-Z"라는 주제로, 작은 프로젝트부터 엔터프라이즈 애플리케이션까지 Go 언어를 사용한 개발 방법론에 대해서 공유해보았습니다.

작은 프로젝트와 큰 프로젝트의 차이점을 설명하고, 작은 프로젝트에 적합한 간단한 코드 패턴을 소개하고, 기능 단위의 프로젝트를 위한 코드 패턴으로 Handler와 HandlerFunc 방식을 비교하고 각각의 장단점을 설명합니다.
엔터프라이즈 애플리케이션을 위한 코드 패턴으로 Domain Driven Design (DDD) 접근 방식을 소개하고, 각 레이어(Presenter, Handler, Usecase, Service, Repository, Recorder)의 역할과 장점을 설명합니다.
NoSQL 데이터베이스 (DynamoDB) 사용 시의 모델링 방법을 간략히 소개합니다. 그리고 각 레이어에서의 테스트 코드 작성 방법과 모킹 도구인 counterfeiter의 사용법을 설명합니다.
애플리케이션 릴리즈를 위해 필요한 도구들(APM, 에러 모니터링, 메트릭 수집, 로그 서비스)을 소개하고 그 중요성을 강조합니다.

Go 언어를 사용하여 다양한 규모의 프로젝트를 개발할 때 고려해야 할 아키텍처, 코드 패턴, 도구 등에 대한 포괄적인 가이드가 되길 바랍니다.

## Youtube

<iframe width="560" height="315" src="https://www.youtube.com/embed/zdMuLvK0pNg?si=N56-ArTLerBi7rne&amp;start=2326" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### Summarized by Gemini

**Go 언어 프로젝트 가이드: 기능 단위 개발부터 엔터프라이즈 애플리케이션 설계까지**

**소개**

Go 언어는 간결하고 효율적인 특징을 지니고 있어 다양한 규모의 프로젝트에 활용되고 있습니다. 이번 발표에서는 Go 언어를 이용하여 프로젝트를 개발하는 방법에 대한 전반적인 내용을 다룹니다. 작은 프로젝트부터 엔터프라이즈급 애플리케이션까지, 각 규모에 맞는 코드 패턴과 아키텍처 설계, 그리고 운영 환경에 배포하기 위한 도구들을 소개합니다. [cite: 1]

**작은 프로젝트 vs 큰 프로젝트**

작은 프로젝트는 트래픽이 적고 기능이 단순하며 빠른 개발이 필요한 경우가 많습니다. [cite: 6, 7] 초기 단계의 서비스, 신규 기능 실험, 또는 개인 프로젝트 등이 이에 속합니다. [cite: 8] 이러한 프로젝트에서는 복잡한 아키텍처나 디자인 패턴보다는, 문제 해결에 집중하여 빠르게 개발하는 것이 중요합니다. [cite: 11] 예를 들어, 간단한 CLI 도구나 API 서버를 개발하는 경우, 외부 라이브러리 의존성을 최소화하고 Go의 기본 라이브러리를 활용하여 개발하는 것이 효과적입니다. [cite: 13, 14, 16, 17]

반면, 엔터프라이즈급 프로젝트는 트래픽이 많고 복잡한 기능을 요구하며, 안정성과 유지보수성이 중요합니다. [cite: 43] 이러한 프로젝트에서는 Clean Architecture, DDD, Microservices Architecture 등의 아키텍처 및 디자인 패턴을 적용하여 코드를 체계적으로 구조화하고, 각 레이어의 역할을 명확하게 분리하는 것이 좋습니다. [cite: 9, 45] 또한, APM, Error Monitoring, Metric Collection, Log Service 등의 도구를 활용하여 애플리케이션의 성능을 모니터링하고 문제 발생 시 신속하게 대응할 수 있도록 준비해야 합니다. [cite: 83]

**작은 프로젝트를 위한 코드 패턴**

작은 프로젝트에서는 Go의 기본 라이브러리를 활용하여 최대한 간결하게 코드를 작성하는 것이 좋습니다. [cite: 16] 예를 들어, CLI 도구를 개발할 때는 `os` 패키지를 이용하여 명령줄 인수를 처리하고, 각 명령어에 해당하는 함수를 호출하는 방식으로 구현할 수 있습니다. [cite: 13] API 서버를 개발할 때는 `net/http` 패키지를 이용하여 HTTP 서버를 구축하고, 각 API 엔드포인트에 대한 핸들러 함수를 작성하는 방식으로 구현할 수 있습니다. [cite: 18]

**엔터프라이즈 애플리케이션을 위한 코드 패턴**

엔터프라이즈 애플리케이션에서는 코드의 복잡성을 관리하고 유지보수성을 높이기 위해 적절한 코드 패턴과 아키텍처를 적용해야 합니다. [cite: 43] DDD (Domain-Driven Design)는 복잡한 비즈니스 로직을 도메인 모델 중심으로 구조화하여 코드의 가독성과 재사용성을 높이는 데 효과적인 방법입니다. [cite: 45] 또한, Clean Architecture는 애플리케이션을 여러 개의 레이어로 분리하여 각 레이어의 역할을 명확하게 정의하고, 의존성을 관리함으로써 코드의 유연성과 테스트 용이성을 향상시킵니다. [cite: 9]

**운영 환경에 배포하기 위한 도구**

애플리케이션을 운영 환경에 배포할 때는 APM (Application Performance Monitoring), Error Monitoring, Metric Collection, Log Service 등의 도구를 활용하여 애플리케이션의 성능을 모니터링하고 문제 발생 시 신속하게 대응할 수 있도록 준비해야 합니다. [cite: 83] APM 도구는 애플리케이션의 성능을 실시간으로 추적하고 분석하여 성능 저하의 원인을 파악하고 개선하는 데 도움을 줍니다. [cite: 84] Error Monitoring 도구는 애플리케이션에서 발생하는 에러를 수집하고 분석하여 에러 발생 원인을 파악하고 수정하는 데 도움을 줍니다. [cite: 87] Metric Collection 도구는 애플리케이션의 다양한 지표를 수집하고 시각화하여 애플리케이션의 동작 상태를 파악하는 데 도움을 줍니다. [cite: 89] Log Service는 애플리케이션의 로그를 수집하고 저장하여 문제 발생 시 디버깅 및 분석에 활용할 수 있도록 합니다. [cite: 89]

**결론**

Go 언어는 작은 프로젝트부터 엔터프라이즈급 애플리케이션까지 다양한 규모의 프로젝트를 개발하는 데 적합한 언어입니다. [cite: 91] 적절한 코드 패턴과 아키텍처, 그리고 운영 도구를 활용하면 Go 언어로 안정적이고 유지보수하기 쉬운 애플리케이션을 구축할 수 있습니다. [cite: 91]

## Slideshare

<iframe src="https://www.slideshare.net/slideshow/embed_code/key/lJMRAQqCn6t89P?startSlide=1" width="597" height="486" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px;max-width: 100%;" allowfullscreen></iframe><div style="margin-bottom:5px"><strong><a href="https://www.slideshare.net/slideshow/golang-project-guide-from-a-to-z-from-feature-development-to-enterprise-application-design/272422573" title="Golang Project Guide from A to Z: From Feature Development to Enterprise Application Design" target="_blank">Golang Project Guide from A to Z: From Feature Development to Enterprise Application Design</a></strong> from <strong><a href="https://www.slideshare.net/KyuhyunByun1" target="_blank">Kyuhyun Byun</a></strong></div>

## FAQ

- 다른 서비스 클라이언트에서 받은 데이터는 어느 레이어에서 처리해야 할까요?
    - 다른 서비스 클라이언트는 pkg 디렉터리 하위에 구현하고 있어요. `xxxclient` 처럼요. 이렇게 하고, usecase나 service에서 이 클라이언트를 사용해요. repository와 같은 레이어라고 인지하고 사용하고 있어요.
- 한 번에 이 구조를 적용하기에는 너무 복잡하다고 느껴집니다. 어떻게 시작해야 할까요?
    - 처음부터 이 구조로 시작한 것은 아니에요. 비즈니스 로직이 복잡해짐에 따라 service를 먼저 도입하고, data modeling을 도메인 로직에서 알면 점점 로직처리에서 인지부하가 발생하여, repository와 domain model을 도입하게 되었어요. 다양하게 외부 요청을 처리하다보니 handler에서 presenter라는 레이어가 필요해졌어요.
    - 우리가 가장 해결하기 어려운 문제상황에 필요한 레이어를 도입하면서 하나씩 문제를 해결해나가길 바래요. 작은 구조에선 큰 복잡도가 필요없고, 서비스가 커지고 불편함이 있다는 말이 2~3번 이상 나오면 아키텍처를 개선해야되는 신호라고 생각해요.
- Public/Internal/Protocol별 handler를 어떻게 구분해야 할까요?
    - publichandler, internalhandler, grpchandler와 같이 handler하위에 디렉터리를 만들어서 구분해요.
    - 처음에는 handler하위에 go 파일을 관리하고 있었지만 점차 각 handler마다 다른 요구사항이 생겨서 구분하여 개발하고 있어요.
- 저희는 applicationcontext, service와 같은 네이밍패턴을 쓰고 있어요. 이렇게 해도 괜찮을까요?
    - 팀원끼리 합의한 네이밍 룰이 중요하다고 생각해요. 팀에서의 인지부하가 없다면 팀내에서 정한 네이밍룰이 현 시점에선 최적의 룰이라고 생각해요.
    - 만약 현재의 구조에서 불편함을 느끼거나, 새로운 팀원이 들어왔을 때 이해하기 어렵다고 느낀다면, 그때 네이밍 룰을 변경하는 것도 좋은 방법이라고 생각해요.
