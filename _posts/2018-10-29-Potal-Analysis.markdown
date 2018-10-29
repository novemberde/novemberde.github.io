---
title: 0원으로 시작하는 데이터 수집 및 분석
category: aws
tags: aws, serverless, 서버리스, lambda, glue, quicksight, athena, analysis
---
## Summary
---
[부산 스마트 앱 개발자 포럼](https://www.onoffmix.com/event/155396)에서 서버리스를 활용하여 데이터를 수집 및 분석 후기를 공유하였다.

- [발표자료](https://www.slideshare.net/KyuhyunByun1/0-121022533)
- [Github repo.](https://github.com/novemberde/serverless-crawler-demo)

## 발표 슬라이드
---

<iframe src="//www.slideshare.net/slideshow/embed_code/key/xrAl5GcoWdJ9pd" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/KyuhyunByun1/0-121022533" title="0원으로 시작하는 서버리스 데이터 수집 및 분석" target="_blank">0원으로 시작하는 서버리스 데이터 수집 및 분석</a> </strong> from <strong><a href="//www.slideshare.net/KyuhyunByun1" target="_blank">Kyuhyun Byun</a></strong> </div>

## 고찰
---

지난 여름부터 모아온 데이터를 분석하고 이 데이터를 기반으로 AWS의 데이터 처리 서비스들에 대해서 공부하는 계기였다.
실제로 현업에서 데이터를 분석할 일은 많다. 데이터를 쌓고 보기 쉽게 변환하는 과정 그리고 시각화까지 다양한 기술을 요구한다.
데이터를 다루는 역량 뿐만 아니라 전체적인 웹서비스 구축 역량을 요구한다.

기본적인 시각화는 프론트엔드에 대한 역량, 데이터를 수집하고 서비스를 운영하는 것은 백엔드의 역량을 필요로 한다.
이러한 역량을 데이터사이언티스트가 모두 지니는 것은 쉽지 않다.
직접 플랫폼을 구축하고 각 데이터 분석 도구의 Configuration을 설정하는 것은 상당한 지식을 필요로 한다.
또한, 데이터 분석을 위한 클러스터를 직접 운영한다는 것은 고급 인력을 요구한다.

그럼에도 불구하고 비지니스의 성장을 위해 데이터 분석을 해야한다면 관리형서비스를 적극적으로 사용해야한다.
하둡 클러스터를 띄우고 스키마를 정의하고 직접 운영하는 것보다는 AWS Glue에 카탈로그를 추가하고, 주기적인 작업을 
설정만 해두고 필요할 때 호출만 하는 것이 가장 좋을 것이다. 그리고 직접적으로 Serdes를 지정하고 데이터 스키마를 관리하는 것을
클라우드에 맡겨두고 비용은 Glue가 동작한 시간에 대해서만 지불하는 것이 낫다.

QuickSight를 쓰면 직접 비주얼라이징 도구를 서버에 호스팅하지 않아도 되며,
Athena를 통해 사용하는 데이터 스캔 비용만 지불하면 상당히 적은 비용으로 데이터 분석 도구를 운영할 수 있다.