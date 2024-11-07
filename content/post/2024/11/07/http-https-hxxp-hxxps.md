---
title: "http와 https 대신 hxxp와 hxxps를 사용하는 이유"
tags: [http, https, hxxp, hxxps, URL, 보안, 클릭 방지, 피싱 공격, 보안 인식, 링크 차단 우회]
date: "2024-11-07T00:30:00+00:00"
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

### 질문이 생기게 된 이유

pm.me라는 도메인을 가진 이메일을 보았다. 이를 운영하는 회사는 [proton mail](https://proton.me/mail)이라는 회사인데, 여기에서 개발하는 방법에 대해서 찾아보니 모든 코드를 Github에서 공개하고 관리한다는 것을 보았다.

하필이면, [PR이 하나](https://github.com/ProtonMail/WebClients/pull/405) 열려있었고, 해당 PR의 제목은 "Add checks to prevent formatting hxxps"이었다.

이를 보고, hxxp와 hxxps가 무엇인지 궁금해졌다. http와 흡사한데 왜 이걸 활용하게 되는 것일까?

Perplexity에게 물어보니 가볍게 다음과 같이 답변이 나왔다. 이메일 보안을 위한 내용이니 알아두면 언젠가는 도움이 될 것 같다.

---

http와 https 대신 hxxp와 hxxps를 사용하는 주된 이유는 보안과 관련이 있습니다. 이 방식을 "URL 무해화(URL defanging)"라고 하며, 다음과 같은 목적으로 사용됩니다:

1. 우발적인 클릭 방지
hxxp나 hxxps로 시작하는 URL은 대부분의 이메일 클라이언트나 웹 브라우저에서 자동으로 클릭 가능한 링크로 변환되지 않습니다[1]. 이는 사용자가 실수로 링크를 클릭하는 것을 방지합니다.

2. 피싱 공격 예방
악의적인 이메일에서 종종 안전해 보이지만 실제로는 악성 웹사이트로 연결되는 URL을 사용합니다. hxxp를 사용하면 사용자가 URL을 직접 복사하여 브라우저에 붙여넣어야 하므로, URL의 실제 목적지를 확인할 수 있는 기회를 제공합니다[1].

3. 보안 인식 제고
URL을 수동으로 입력해야 하는 과정은 사용자의 보안 인식을 높이는 데 도움이 됩니다[1].

4. 자동 링크 생성 방지
일부 플랫폼이나 포럼에서는 http나 https로 시작하는 텍스트를 자동으로 클릭 가능한 링크로 변환합니다. hxxp를 사용하면 이를 방지할 수 있습니다[5].

5. 링크 차단 우회
일부 웹사이트나 광고 플랫폼에서는 댓글이나 게시물에 링크를 포함하는 것을 금지합니다. hxxp를 사용하면 이러한 제한을 우회할 수 있습니다[5].

6. 웹사이트 관리자를 위한 메트릭 제어
일부 웹사이트 관리자들은 자신의 사이트 링크가 어디에서 공유되는지, 얼마나 많은 트래픽을 생성하는지 등을 추적하기 위해 이 방법을 사용합니다[5].

이 방식은 보안 커뮤니티에서 널리 사용되며, 특히 대규모 내부 커뮤니케이션에서 보안 인식을 높이는 데 효과적입니다[1]. 그러나 이는 완벽한 해결책은 아니며, 사용자 교육과 함께 사용되어야 가장 효과적입니다.

Citations:
- [1] https://privacymatters.ubc.ca/node/223
- [2] https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/trusted-domain-hidden-danger-deceptive-url-redirections-in-email-phishing-attacks/
- [3] https://datatracker.ietf.org/doc/draft-salgado-hxxp/
- [4] https://malwaretips.com/threads/what-is-the-hxxp-prefix.31827/
- [5] https://www.reddit.com/r/TOR/comments/grd44r/why_do_people_write_hxxps_instead_of_https_when/