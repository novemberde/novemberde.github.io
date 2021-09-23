---
title: Serverless + AWS Lambda + AWS CloudWatch + Slack 를 활용한  Web crawler 만들기
tags: [node, crawler, lambda, cloudWatch, slack, crawler, crawling, 크롤링, got, cheerio, serverless]
date: "2017-12-31T11:30:03+00:00"
aliases: ["/node/2017/12/31/Node-crawler.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---
## Summary
---
우리는 매일 각 웹사이트를 확인하여 뉴스를 취합한다. 별다른 요구사항 없이 직접 업무를 진행하시지만 이건 자동화해야된다는 생각이 들었다.
하루에 한 번 이뤄지는 일이니 주기적으로 슬랙에 최신 뉴스를 보내도록 자동화한다면 업무의 작은 시간도 아낄 수 있을 것이다.
그래서 이번에 크롤링 자동화 방법에 대해서 정리하려고 한다.

Serverless framework를 활용하면 빠른 배포 및 관리가 가능하니 기술 스택을 다음과 같이 정했다.

- AWS Lambda: javascript로 작성. got, cheerio를 사용하여 crawler 설계
    - got: Request 모듈. 우리나라에는 많이 알려져있지 않지만 해외에서는 주로 got을 사용. npm에서 download 수가 압도적으로 많다. 사용법이 간단하다.
    - cheerio: jQuery형식으로 서버에서 사용하는 모듈
- AWS CloudWatch: 주기적으로 Lambda 함수를 실행하기 위함
- Serverless framework: 내부적으로 Serverless Application Model 파일을 생성하여 CloudFormation으로 배포하는 것을 자동화
- Slack bot: 정해진 시간이 지나고 다시 뉴스정보를 불러오고 싶을 때 사용한다. Lambda를 trigger하는 방법으로 사용

---
## 개발 환경 설정
---

이 과정은 npm version 5 이상을 사용하고 있다고 가정하고 시작한다.
만일 사용하고 있지 않다면 npm install -g npx 를 실행한다.
npx는 해당 패키지 내에서 node_modules/.bin에 있는 명령어를 바로 사용할 수 있게 해준다.

프로젝트 디렉터리를 생성하고 node project로 초기화한다.


```sh
$ mkdir lambda-crawler-demo
$ cd lambda-crawler-demo
$ npm init -y
```


해당 프로젝트에 필요한 패키지를 설치한다.


```sh
$ npm i -D serverless
```

위에서 serverless package를 Global로 설치하지 않고 dev-dependency에 넣는 이유는 나중에 다른 작업환경에서도
Serverless의 버전을 명확하게 하기 위함이다. 그렇지 않으면 1, 2년이 지난 후에는 환경이 달라져 Global로 설치했던
Serverless 버전을 찾아야한 하기 때문이다.

Serverless framework의 개발환경을 설정한다.


```sh
# app이라는 디렉터리에 aws-nodejs의 템플릿을 기본으로 생성한다.
$ npx serverless create --template aws-nodejs --path app
```

serverless.yml이라는 파일을 다음과 같이 편집한다.

- service 명: my-crawler
- stage: dev
- region: us-east-1

#### [app/serverless.yml]


```yml
service: my-crawler

provider:
  name: aws
  runtime: nodejs6.10
  stage: dev
  region: us-east-1
  profile: my-service
  memorySize: 128

functions:
  crawler:
    handler: handler.crawler
    events:
      - schedule: rate(1 day)
```

handler.js를 다음과 같이 편집한다.

#### [app/handler.js]


```js
'use strict';

module.exports.crawler = (event, context, callback) => {
  const response = {
    statusCode: 200,
    body: JSON.stringify({
      message: 'Go Serverless v1.0! Your function executed successfully!',
      input: event,
    }),
  };

  callback(null, response);
}
```

---
### 이제부터 모든 작업은 app 디렉터리에서 한다.
---

serverless로 deploy를 하기 위해선 사전에 AWS Command line interface가 설치되어 있고
IAM user를 생성하여 등록해야한다. 아직 하지 않았다면 [여기](/aws/2017/08/14/Serverless.html)를 참고하여 설정한다.

그리고 위에서 profile 설정을 해주었는데 기본적으로 aws configure로 설정하면
여러 service account을 매번 설정해주어야 하기 때문에 aws configure --profile [profile-name] 으로 설정하여 
여러 계정 정보를 가지고 있는 경우에도 문제없이 작업할 수 있도록 한다.

비지니스 로직을 작성하기 전에 배포가 되는지 확인한다.

```sh
$ npx serverless deploy
```

배포가 다 되었다면 [Lambda 콘솔](https://console.aws.amazon.com/lambda/)로 이동하여 다음과 같이 나타나는지 확인한다.

CloudWatch Events가 있다면 정상적으로 배포가 된 것이다. 주기적으로 실행하기 위해 CloudWatch Event를 사용한다.

![CloudWatch Events](/images/aws/lambda/crawler.png)

---
## Crawler 작성하기
---

크롤링 개요는 다음과 같다.

1. got을 사용하여 네이버 메인에 있는 뉴스 5개를 가져온다.
2. cheerio를 사용해서 해당 DOM을 찾아낸다.
3. 크롤링된 데이터를 Slack에 보낸다.

현재 디렉터리에 모듈을 설치한다

#### [app/]

```sh
$ npm init -y
$ npm i cheerio got
```

got의 사용법은 다음과 같다.

```js
const got = require('got');

got('https://www.naver.com').then(res => {
  console.log(res.body);
});
got("POST_URL", {
  method: "post",
  body: JSON.stringify({text: "hi"})
}, () => {

})
```

got과 cheerio를 모두 사용하면 다음과 같다.

네이버 메인 뉴스를 가져온다.

```js
const got = require('got');
const cheerio = require('cheerio');

got('https://www.naver.com').then(res => {
  $ = cheerio.load(res.body);

  const result = $('.ca_item .ca_a');

  console.log(result.text());
});
```

이제 크롤링을 했으니 Slack으로 내용을 보내본다.

Slack에 원하는 채널에 들어가서 Add an app을 클릭한다.

Incoming WebHooks의 설정으로 들어가서 Webhook URL을 복사하여 저장해둔다.

slack 설정은 모두가 다를 수 있으니 다음과 같이 파일을 작성한다.

#### [app/slack.config.json]

```json
{
  "URL": "Paste your slack url"
}
```

거의 다 왔다. 이제 handler.js를 다음과 같이 편집해준다.

#### [app/handler.js]

```js
'use strict';

const got = require('got');
const cheerio = require('cheerio');
const SLACK_URL = require('./slack.config.json').URL;

module.exports.crawler = (event, context, callback) => {
  let result;

  return got('https://www.naver.com').then(res => {
    const $ = cheerio.load(res.body);

    result = $('.ca_item .ca_a');

    return got(SLACK_URL, {
      method: "post",
      body: JSON.stringify({text: result.text()})
    });

    
  }).then(() => {
    const response = {
      statusCode: 200,
      body: JSON.stringify({
        message: result.text(),
      }),
    };

    callback(null, response);
  });

}
```

이제 배포하면 매일 주기적으로 슬랙 알람이 올 것이다.

```sh
$ npx serverless deploy
```

---
## 삭제하기
---

한 번 배포해봤으니 삭제하여 과금되지 않도록 한다.

```sh
$ serverless remove
```

---
## References
---

- [https://serverless.com/](https://serverless.com/)
- [https://ko.wikipedia.org/wiki/%ED%86%A0%EC%96%B4](https://ko.wikipedia.org/wiki/%ED%86%A0%EC%96%B4)
- [https://github.com/novemberde/lambda-crawler-demo](https://github.com/novemberde/lambda-crawler-demo)
- [https://www.npmjs.com/package/got](https://www.npmjs.com/package/got)
- [https://www.npmjs.com/package/cheerio](https://www.npmjs.com/package/cheerio)