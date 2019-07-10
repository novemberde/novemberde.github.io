---
title: Firecracker 에 대한 고찰
category: aws
tags: aws, firecracker, troubleshooting, 고찰
---

이번 2018 AWS re:Invent를 통해서 평소에 가지고 있던 궁금증에 대해서 실마리를 찾을 수 있었다.
AWSKRUG Serverless 모임에서 자주 언급되는 내용으로 Lambda의 내부 구조가 어떤지 많이 토론했었다.
AWS 내부적으로 Lambda는 Docker로 이루어져있고 람다는 컨테이너를 CMD에서 바로 실행하고 정의해놓은 타임아웃
시간에 따라 종료되는 것으로 잠정적으로 생각했었다.

하지만 여기에서 풀리지 않는 의문점이 있었다.
