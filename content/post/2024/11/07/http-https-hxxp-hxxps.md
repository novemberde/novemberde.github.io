---
title: "hxxp와 hxxps: URL 무해화(Defanging)를 위한 http/https 대체 표기법 알아보기"
tags: [http, https, hxxp, hxxps, URL defanging, URL 무해화, 보안, 피싱 방지, 이메일 보안]
date: "2024-11-07T00:30:00+00:00"
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

## URL 무해화(Defanging)란?

보안 전문가들은 악성 URL을 공유할 때 특별한 표기법을 쓴다. http 대신 hxxp를, https 대신 hxxps를 쓰는 것이다. 이런 방식을 'URL 무해화(URL defanging)'라고 한다.

## hxxp/hxxps 표기법을 알게 된 계기

최근 ProtonMail(pm.me)의 이메일을 받아봤다. 이 회사의 개발 방식이 궁금해서 찾아보니 모든 코드를 GitHub에서 공개적으로 관리하고 있었다. 우연히 ["Add checks to prevent formatting hxxps"](https://github.com/ProtonMail/WebClients/pull/405)라는 제목의 PR을 발견했다.

이 PR을 보고 hxxp/hxxps 표기법이 무엇인지, 왜 쓰는지 궁금했다.

## URL 무해화(Defanging)가 필요한 이유

보안 전문가들이 hxxp/hxxps 표기법을 쓰는 주된 이유는 다음과 같다:

### 1. 의도하지 않은 클릭 방지
- 이메일이나 문서에서 http(s)로 시작하는 URL은 자동으로 클릭 가능한 링크가 된다
- hxxp(s)를 쓰면 자동 링크 생성을 막을 수 있다

### 2. 피싱 공격 예방
- 악성 URL을 분석하거나 공유할 때 실수로 클릭하는 것을 막는다
- URL을 수동으로 입력해야 해서 목적지를 한 번 더 확인하게 된다

### 3. 보안 인식 향상
- URL을 수동으로 처리하면서 보안 의식이 자연스럽게 높아진다
- 특히 기업 내부 소통에서 효과적이다

### 4. 보안 시스템 오탐 방지
- 정상적인 보안 문서가 악성으로 탐지되는 것을 막는다
- 보안 시스템의 불필요한 경고를 줄인다

## URL 무해화의 다양한 방법

URL 무해화는 단순히 http를 hxxp로 바꾸는 것에만 국한되지 않는다. 일반적으로 다음과 같은 방법들을 함께 사용한다:

1. 프로토콜 변경
   - http → hxxp
   - https → hxxps

2. 도메인 점(.) 처리
   - example.com → example[.]com
   - 또는 example(.)com

3. 특수문자 처리
   - @ → [at]
   - / → [/] 또는 [forward-slash]

4. IP 주소 무해화
   - 192.168.1.1 → 192[.]168[.]1[.]1

## 무해화가 필요한 대상들

URL 외에도 다음과 같은 정보들도 무해화가 필요하다:

- IP 주소
- 이메일 주소
- 도메인 이름
- 파일 확장자 (.exe, .bat 등)
- 레지스트리 키

## 활용 사례

이 표기법은 다음과 같은 상황에서 주로 쓴다:

1. 위협 인텔리전스 공유
   - 보안팀 간 악성 URL 정보 공유
   - 위협 보고서 작성

2. 이메일 보안 시스템
   - 스팸 필터 우회 방지
   - 보안 경고 메시지 작성

3. QR 코드 분석
   - 악성 QR 코드 분석 보고서
   - QR 코드 관련 보안 문서

4. 보안 교육
   - 보안 인식 교육 자료
   - 피싱 대응 훈련

5. 보안 문서 작성
   - 보안 보고서
   - 악성코드 분석 문서
   - 보안 포럼 게시물

## 결론

URL 무해화(defanging)는 완벽한 보안 해결책은 아니다. 하지만 보안 인식을 높이고 실수로 인한 보안 사고를 막는 데 효과적이다. 특히 보안 전문가들 사이에서는 이미 표준적인 관행이 됐다.

## 참고 자료
- [Blocking Email Links: Why we use HXXP in emails](https://privacymatters.ubc.ca/node/223)
- [URL Defanging Tool and Guide](https://trustifi.com/url-defang-tool/)
- [URL Defanger: Securing URLs for Safe Examination](https://blackheathpoint.com/tools/defang-url.html)
- [Threat Intelligence Sharing Best Practices](https://www.cyware.com/blog/dont-let-the-nuts-and-bolts-of-threat-intel-trip-you-up-defang-that-url-250e)
- [Malicious QR Codes Analysis](https://blog.talosintelligence.com/malicious_qr_codes/)