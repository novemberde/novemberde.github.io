---
title: Express JS를 사용하여 Node 서버 구축하기
tags: [expressjs, nodejs]
date: "2017-04-01T11:30:03+00:00"
aliases: ["/2017/04/01/Docker_5.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

## Summary
---------------------
 Node JS의 Express framework를 사용하여 Node JS 서버를 구축해본다. 
 기본적으로는 http모듈을 사용하여 서버 리스너를 구동하는 것이 있다. 
 http 모듈을 사용하여 서버를 가동하면 exception이 발생하는 동시에 서버가 정지하게 된다.
 하지만 Express framework를 사용하면 error처리가 가능하다.

---------------------

## 순서
---------------------
1. Amazon web service에 Ubuntu OS를 사용하는 EC2 인스턴스 생성하기
1. 접속 포트를 열어주고 별도의 Ubuntu 유저를 생성하기
1. EC2에 Docker를 설치하고 Ubuntu 유저에게 권한주기
1. Bitbucket을 사용하여 git repository 생성하기
1. Express JS를 사용하여 Node 서버 구축하기
1. 테스트로 PM2를 사용하여 EC2에 Node 서버 배포하기
1. Node 서버를 바탕으로 Dockerfile로 만들기
1. Docker Hub의 automated build를 사용하여 Docker image를 만들기
1. 만들어진 Docker image를 EC2 인스턴스에 배포하기

---------------------

## Express JS 시작하기
---------------------

편집기는 아무거나 사용해도 상관없다. 손에 익은 VS code를 사용하여 편집하겠다.

MS에서 만든 걸작중에 하나라고 생각된다. 보통은 Sublime text, Atom 과 같은 에디터를 많이 사용한다.

VS code를 사용하는 이유는 아래 사진에서 보이는 것과 같이 터미널을 폴더를 기준으로 사용하기 때문이다.

Ctrl + `(~) 단축키를 누르면 터미널이 열린다.

![vs_code](/images/aws/ec2/vs_code.png)

---------------------

혹시나 아직 [Node JS](https://nodejs.org)를 설치하지 않으신 분이 있다면 공식 사이트에서 설치하고 진행하길 바란다.

노드가 설치되어 있다면 아래 커맨드를 따라주면 된다.

```bash
# 패키지 초기화 하기. 모두 enter를 치고 default로 사용하자.
$ npm init
```

---------------------
이제 작업 디렉터리에 package.json이 생성되었을 것이다.

```json
{
  "name": "docker_node_server",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "git+https://kyuhyun@bitbucket.org/kyuhyun/docker_node_server.git"
  },
  "author": "",
  "license": "ISC",
  "homepage": "https://bitbucket.org/kyuhyun/docker_node_server#readme"
}
```

---------------------
Express JS를 사용하기 위해 npm 명령어를 실행하자.

```bash
# express 라이브러리 추가하기.
$ npm install --save express
```

--------------------
이제 서버를 만들어보자.

app.js라는 파일을 만들고 아래의 코드를 삽입한다.

```javascript
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res, next) => {
    res.send('hello world!');
});

app.listen(port, () => {
    console.log(`Server is running at ${port}`);
});
```

Express JS의 프로젝트를 생성하는 방법은 [http://webframeworks.kr/](http://webframeworks.kr/)에 잘 설명되어 있다.

Node JS의 Express Framework에 대한 깊은 이해가 필요한 분들은 참고하길 바란다.

--------------------
서버가 만들어졌다.

브라우저를 키고 [http://localhost:3000](http://localhost:3000)에 접속해보자.

Hello world! 라는 문구를 봤으면 순조롭게 진행된 것이다.

--------------------
다음은 지금 만든 프로젝트의 버전관리를 하기 위해 git repository에 push할 예정이다.

Source Tree를 보면 스테이지에 올라가지 않은 파일 목록이 많은 것을 볼 수 있다.

npm으로 설치한 모듈 버전 정보는 package.json에 담겨있기 때문에 실제로 모듈이 git으로 관리될 필요가 없다.

루트 디렉토리에 .gitignore파일을 만들고 아래 코드를 넣은 다음에 다시 확인하면 node_modules가 전부 사라진 것을 확인할 수 있다. Source Tree는 동기화가 조금 걸리니 나중에 확인하면 사라질 것이다.

```bash
# node_modules 제외하기
node_modules
```

--------------------
최소화된 정보만 갖고 있으니 repository에 push하자

SourceTree에서 Stage All 버튼을 눌러 모든 파일을 저장소에 올릴 준비를 하고 아래에 커밋 메시지를 작성한다.

Commit 메시지는 코드를 추후에 추적하기 편하게 하기 위해 상세히 내용을 담도록하자.

커밋한 후에 상단의 푸시 버튼을 눌러 repository에 현재 버전 정보를 올리자.

![source_tree_commit](/images/aws/ec2/source_tree_commit.png)

## References
- [http://webframeworks.kr/](http://webframeworks.kr/)