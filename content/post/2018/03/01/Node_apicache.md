---
title: NODE response를 apicache로 처리하기
tags: [nodejs, apicache, cache, 캐시, caching]
date: "2018-03-01T11:30:03+00:00"
aliases: ["/node/2018/03/01/Node_apicache.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---
## 문제인식
---
서비스가 성장해가면서 기존에는 보이지 않던 문제점들이 나타났다.
처음에는 Slow query의 최적화를 필요로하지도 않았고, DB에 부하가 몰려서
Bottle neck point가 되는 것을 상상만 했지 직접 경험하지는 못했었다.
하지만 점차 Query의 optimization이 필요한 경우가 생겼고,
DB로 몰리는 부하를 분산하기 위해 어떻게 처리해야하는지 고민하기 시작했다.


---
## Cache me if you can

다행히도 AWSKRUG활동을 하면서 AWS re:Invent행사에 참석한 경험이 도움이 되었다.
들었던 세션중에서 가장 감명깊은 것중에 하나는 "Cache me if you can"이었다.
주된 내용은 점차 서비스가 커져감에 따라서 우린 Database로 몰리는 부하를 분산시켜야만 하고,
그것을 해결하기 위해서는 각 요청에 따라 Cache로 처리할 수 있는 부분은 모두 Cache로 
해결해야 된다는 내용이었다.

[https://www.youtube.com/watch?v=WFRIivS2mpo](https://www.youtube.com/watch?v=WFRIivS2mpo)


---
## Apicache를 사용하여 문제 해결하기

물론 현재도 AWS ElasticCache Service를 쓰고 있다.
그렇지만 최대한 활용하여 쓰고 있다고 생각하지는 않는다.
Client에 응답을 빠르게하기 위해 Redis를 사용하는데,
DB 스키마와 형식이 다르기 때문에 막상 적용하려하면 처음에는 당혹스럽다.
또한 여러 전처리 과정이 들어가기 때문에 뭔가 아름답게 정리되지도 않는다.
그럼에도 불구하고 Redis를 사용해야만 했다. 회원마다 빠르게 처리해야하는
개인화 정보도 다르며, 실시간으로 많은 요청을 처리해야되기 때문에 최대한
데이터 유실이 없는 범위 내에서는 Redis를 사용했다.

그래도 이것만으로는 부족했다. 하나하나 이렇게 변환해서 사용하기도 귀찮고
페이지 단위로 각 웹서버에서 처리하는 것도 괜찮다고 생각했다.
굳이 Redis에서 데이터를 가져오지 않고 Local memory에서 처리하는게
더 빠를 것이라 생각했고, 그 데이터의 양은 서비스 구조상 크지 않기 때문이었다.

또한 AWS Lambda를 활용하여 서비스를 하고 있기 때문에 일정 시간(30분 미만) 후에 
Container가 내려가고 새로운 Container가 올라가는 형식으로 메모리 누수에
비교적 자유로운 형태의 아키텍쳐였다.

그래서 Redis를 연동하지 않고 local cache를 이용하는 것을 선택했다.
만일 Redis를 더 연동하면 점점 Redis에 의존성이 증가하기 때문에 이를
피하기 위한 점도 있었다.

node.js로 서버를 구성하고 있었기 때문에 library를 찾아보았고
apicache라는 사용법이 간단한 라이브러리를 찾을 수 있었다.
express.js에서 사용법은 다음과 같다.

```js
let cache = require("apicache").middleware
 
app.use(cache('5 minutes'))
app.get('/will-be-cached', (req, res) => {
  res.json({ success: true })
})
```

적용 방법은 매우 간단하지만 결과는 차원이 다르게 나타났다.
이전 평균 응답시간은 120~250ms 내외였다면
현재 평균 응답시간은 20~ 50ms로 성능 향상을 할 수 있었다.

실제로 어플리케이션의 응답이 빨라져 사용자들이 더욱 쾌적하게
사용하게 되었다.

이뿐만 아니라 근본적으로 Cache를 도입하게 된 배경인 Database의 관점에서도
눈에 띄는 성능 변화가 나타났다.
자주 Database의 CPU사용율이 70%를 쉽게 초과하고 그것에 대응하기 위해 
Database의 Instance type을 두 배의 성능으로 증가시켰던 것에 비해
현재는 기존의 Instance type만으로도 버틸 수 있다.


---
## 고찰

서버개발자 또는 인프라엔지니어로만 일을 할 수도 있었다. 하지만 스타트업에서 일하다보니
웹개발, 어플리케이션 개발, DB tuninng, 인프라 관리 등등 할일이 너무 많았다.
처음에는 이런점이 아쉽기도 했다. '하나만 파도 모자를 판에 어떻게 여러 문제를 해결할까'라는
의문점이 들면서도 '모든게 다 경험이다'라고 생각하며 계속 문제를 해결하기 위한 방안들을
생각하고 잘 해결되지 않는 문제점에 대해서 방법을 바꿔가며 길을 찾았다.

이것이 점점 쌓이다보니 다른 시각이 생겼다. Application 및 웹서버의 최적화를 진행하며
인프라에서만 해결하려는 것이 아닌 전방위적인 해결방법을 모색하게 되었다.
웹서버에 요청이 계속 몰리는 것을 보며 웹서버를 확장할 고민을 하기 전에 어떤
요청이 클라이언트에게 오는지 보고 Client에서 어지간한 요청에 대해서 캐싱한다던지,
아니면 정규식이 없는 곳이 정규식을 통해 적절한 요청만 서버에 올 수 있도록 수정하던지,
아니면 request queue를 통해 실시간성이 필요하지 않는 요청이나 반드시 필요한 데이터가
아닌 경우에는 시간을 두어 데이터를 한번에 보내던지,
다양한 방법을 통해 서버의 부하를 줄일 수 있었다.

무작정 서버의 비용을 증가시키기 보다는 클라이언트에서 처리를 하고, 서버에서는
기본을 잘 지켜 Redis 및 ReadReplica를 활용하여 부하를 분산시키고,
Monitoring의 습관화를 통해 이상 징후를 계속 확인하였다.

이제는 문제가 발생하더라도 빠른 복구가 가능하며,
성능이 점점 최적화되어서 큰 문제없이 서비스가 운영되고 있다.

지금처럼 엔터프라이즈급이 되는 상황을 상상하여 설계하여
큰부하가 몰리거나, 갑작스런 서비스의 성장에도 대비할 수 있도록 해야겠다.


---
## References

- [https://www.npmjs.com/package/apicache](https://www.npmjs.com/package/apicache)
- [https://www.youtube.com/watch?v=WFRIivS2mpo](https://www.youtube.com/watch?v=WFRIivS2mpo)