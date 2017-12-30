---
title: Node + AWS Lambda + AWS CloudWatch + Tor + Slack 를 활용한  Web crawler 만들기
category: node
tags: node, crawler, lambda, cloudWatch, slack, crawler, crawling, 크롤링, got, cheerio, serverless
---
## Summary
---
각 웹사이트를 확인하여 업무를 진행하는 우리 팀을 위해 무엇을 할 수 있을지 생각했다.
하루에 한 번 이뤄지는 일이니 주기적으로 슬랙에 정리하여 보내면 업무의 작은 시간도 아낄 수 있게 하려고 한다.

Serverless framework를 활용하면 빠른 배포 및 관리가 가능하니 기술 스택을 다음과 같이 정했다.

- AWS Lambda: javascript로 작성. got, cheerio를 사용하여 crawler 설계
    - got: 우리나라에는 많이 알려져있지 않지만 해외에서는 주로 got을 사용. npm에서 download 수가 압도적으로 많다.
    - cheerio
- AWS CloudWatch: 주기적으로 Lambda 함수를 실행하기 위함
- Serverless framework: 내부적으로 Serverless Application Model 파일을 생성하여 CloudFormation으로 배포하는 것을 자동화
- Tor: Tor는 온라인 상에서의 익명을 보장하고 검열을 피할 수 있게 해주는 자유 소프트웨어

---
## References
---
- [https://ko.wikipedia.org/wiki/%ED%86%A0%EC%96%B4](https://ko.wikipedia.org/wiki/%ED%86%A0%EC%96%B4)