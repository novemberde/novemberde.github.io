---
title: AWS Lambda에서의 Cold Start 문제 해결하기(Golang + serverless framework)
tags: [apigateway, lambda, serverless, framework, go, golang, go언어, coldstart, cold, start, 문제]
date: "2018-02-02T11:30:03+00:00"
aliases: ["/aws/2018/02/02/Lambda_coldStart.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---
## Summary
---
AWS Lambda와 API Gateway를 연동하여 웹서버를 Serverless architecture로 운영하고 있다.
관리할 서버가 없고 비지니스 로직에만 집중할 수 있어서 좋기도 하지만 항상 Lambda container가 
대기중인 상태가 아니다 보니 Cold start관련하여 요청사항이 발생했다.

일정시간이 경과된 후에 해당 페이지를 접속할 경우에 로딩이 3~4초이상 걸리기 때문에 사용이 불편하였다.
이를 해결하기 위해 5분마다 각 api를 health check하는 scheduler를 두기로 결정하였다.
그렇게되면 지속적인 lambda 호출로 인해 항상 하나이상의 lambda container가 대기하는 형태가 되어
lambda의 cold start의 가장 큰 시간을 소비하는 DB connection의 시간을 느낄 수 없다.

먼저 현재 사용하는 웹서버가 5개라고 하고, heathCheck는 비지니스 로직을 통과하지 않고 
빠른 응답을 하기 때문에 처음에는 300ms에 응답한다고 가정하려하였다.
그런데 5분에 한 번씩 모든 api를 호출하고, 돌아오는 응답의 시간이 대략적으로 2초정도 걸렸다.

무엇인가 이상하다고 생각하였다.

단순한 request인데 몇번하였다고 2초가 걸리는 건 코드상의 오류라고 생각되었다. 
이때 Go lang을 활용하였는데, 익숙치 않다보니 실수가 있었다.

기존의 node로 개발했을 때는 Promise.all 패턴으로 비동기 작업을 한번에 보냈었지만, 
go로 생각없이 짜다보니 request하고 응답하고 request하고 응답하고
이런식으로 여러번하다보니 전체적인 시간이 늘어났다. 얼마되진 않지만 이 때문에 
비용이 4배 이상으로 올라가서 Gorutine를 사용하여 처리하였다.

동시에 요청을 보내고 전부 response를 받으면 lambda가 종료하는 방식이다.
기존에는 2초가 걸렸지만 현재는 1.5초로 실행시간을 감소시켰다. 빠른 때는 1초도 걸리지 않는다.


---
## 비용 계산하기
---

위 테스트를 기반으로
[https://s3.amazonaws.com/lambda-tools/pricing-calculator.html](https://s3.amazonaws.com/lambda-tools/pricing-calculator.html)에서 
참고하여 비용을 측정하였다.

- Number of Executions: 5 * 12(시간당 호출수) * 24 * 30 = 43200
- Memory: 단순 로직이기 때문에 128MB로 선정
- Estimated Execution Time (ms): 1500ms
- Include Free Tier: no
- TOTAL COSTS
  - Request Costs: $0.01
  - Execution Costs: $0.14
  - $0.14/month

계산에 따르면 한달에 200원 정도선에서 모든 api를 Healthy상태로 둘 수 있다.

현재 가격은 최대로 계산한 것이고 Lambda의 메트릭을 살펴보면 비용은 좀 더 낮게 책정될 것을 예상할 수 있다.
보통은 1초 내외로 끝나기 때문이다.

![lambda metric](/images/aws/lambda/lambda_metric.png)


---
## 고찰
---

본격적으로 go언어를 사용하기로 마음을 먹고 이번에 실제로 적용해본 것은 처음이다.
기존에 스크립트 언어에 익숙했기 때문에 적응은 쉽게 되지 않았다.
기본적으로 스크립트 언어는 async한 로직을 쉽게 넣었지만
compile언어를 오랜만에 써보니 비동기 작업을 다르게 구현하는 점이 새로웠다.
물론 JAVA에서 Thread를 통해 비동기 작업을 처리하기도 했지만 gorutine을 통해
로직을 구현해보니 상당히 golang이 매력적인 녀석이라는 것을 깨달았다.

Java처럼 각 request 마다 thread를 생성하지 않고 gorutine의 channel을 통한
방식이 아직은 낯설기도 하지만 점차 적응해나갈 것으로 보인다.

그리고 node를 사용한 serverless deploy는 바로 소스코드를 업로드하는 방식이었지만
go는 해당 application을 Makefile 을 통해 빌드하고 bin 디렉터리만 serverless deploy를 하는 형식이었다.
이것은 go가 컴파일 언어이기 때문에 컴파일된 binany형태의 파일만으로 동작하기 때문이다.

왜 기존에 잘 쓰던 node를 두고 golang으로 작성하였는지 물어본다면 performance에 대한
욕구가 나날이 증가하기 때문이다. 현재 서비스가 성장해감에 따라서 web-application의 성능의
차이가 느껴지기 시작했다. node는 결국 script언어이기 때문에 compile 언어와 차이가 날 수 밖에 없기 때문이다.
그렇다고 java를 사용하기엔 lambda에 jvm이 얹혀가는 구조이기 때문에 무겁게 동작해서 사용할 수가 없었다.

go언어를 점차 사용해서 모든 프로덕션에 적용하는게 목표이지만 언제 이룰 수 있을진 잘 모르겠다.


---
## References
---

- [https://s3.amazonaws.com/lambda-tools/pricing-calculator.html](https://s3.amazonaws.com/lambda-tools/pricing-calculator.html)