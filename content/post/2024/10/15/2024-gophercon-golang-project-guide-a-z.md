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

## Slideshare

<iframe src="https://www.slideshare.net/slideshow/embed_code/key/lJMRAQqCn6t89P?startSlide=1" width="597" height="486" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px;max-width: 100%;" allowfullscreen></iframe><div style="margin-bottom:5px"><strong><a href="https://www.slideshare.net/slideshow/golang-project-guide-from-a-to-z-from-feature-development-to-enterprise-application-design/272422573" title="Golang Project Guide from A to Z: From Feature Development to Enterprise Application Design" target="_blank">Golang Project Guide from A to Z: From Feature Development to Enterprise Application Design</a></strong> from <strong><a href="https://www.slideshare.net/KyuhyunByun1" target="_blank">Kyuhyun Byun</a></strong></div>
