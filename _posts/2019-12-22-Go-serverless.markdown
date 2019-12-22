---
title: Go언어로 서버리스 서비스 시작하기
category: serverless
tags: serverless, aws, golang, go, 서버리스
---

AWS에서 서버리스로 구현하는 앱은 보통 Javascript 또는 Python으로 작성된다.
그렇지만 AWS Lambda에서는 거의 모든 언어를 지원하고 있다.
더욱이 인프라 및 서버사이드에서 이뤄지는 프로젝트는 대부분 고언어로 작성되고 있다.
생산성 뿐만 아니라 배포시에도 이점을 가져가고 있기 때문이다.
Serverless의 장단점에 대해서 이야기하고,
Go 언어를 통해 서버리스 Todo 앱을 작성하고 배포하는 예제를 Golang Korea Meetup에서 발표하였다.
다음은 발표 때 사용했던 슬라이드 및 참고한 자료들이다.

## Presentation

<iframe src="//www.slideshare.net/slideshow/embed_code/key/ca6Q7hn0PKDAlA" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/KyuhyunByun1/start-serverless-with-golang" title="Start Serverless with Golang!" target="_blank">Start Serverless with Golang!</a> </strong> from <strong><a href="https://www.slideshare.net/KyuhyunByun1" target="_blank">Kyuhyun Byun</a></strong> </div>

## References

- [https://github.com/awslabs/aws-lambda-go-api-proxy](https://github.com/awslabs/aws-lambda-go-api-proxy)
- [https://echo.labstack.com/guide](https://echo.labstack.com/guide)
- [https://serverless.com/](https://serverless.com/)
- [https://github.com/novemberde/go-serverless-demo](https://github.com/novemberde/go-serverless-demo)
- [https://novemberde.github.io/ppts/svelte/](https://novemberde.github.io/ppts/svelte/)
- [https://github.com/spf13/cobra](https://github.com/spf13/cobra)
