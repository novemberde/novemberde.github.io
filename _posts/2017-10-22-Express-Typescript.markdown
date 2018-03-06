---
title: TypeScript로 Node.js Express 서버 개발하기
category: node
tags: typescript, nodejs, node, js, express, server, development, 개발, 서버
---
## Summary
---
최근에 직방의 AWS Lambda 코드를 세미나에서 스쳐지나가는 것을 보았다. 
TypeScript로 짜여져 있었고 나중에 지속적인 프로젝트 관리적인 이점이 있을 것으로 생각이 되기 때문에 TypeScript로 프로젝트를 구성해보는 것을 정리해보기로 결심했다.

---
## node.js 프로젝트 생성하기
---

먼저 프로젝트를 생성한다. 기존에 npm과 nodejs가 설치되어 있다는 전제하에 시작한다.

만약 설치되어 있지 않다면 다음의 링크를 참고한다. [https://nodejs.org/ko/download/package-manager/](https://nodejs.org/ko/download/package-manager/)

```sh
$ mkdir node_typescript
$ npm init -y
```

그리고 편의를 위해 사전에 설치되어 있어야 하는 패키지들이 있다.

- npx: global로 패키지를 설치하지 않더라도 프로젝트 내에서 사용할 수 있게 해준다.
- nodemon: 파일이 변화될 때마다 재실행해준다.
- typescript: typescript로 구성한 코드를 javascript로 트랜스파일링 해준다.
- npm-run-all: 여러 npm 실행 명령을 병렬로 실행할 수 있게 해준다.
- webpack: 요즘 각광받는 모듈 번들러
- source-map-support: typescript로 개발시 source-map을 지원해준다.
- @types/express: express 모듈에 대한 type을 지원해준다.

```sh
$ npm install -g npx
$ npm install --save-dev typescript ts-loader npm-run-all webpack @types/express nodemon
$ npm install --save express source-map-support
```


---
## 기본 설정하기
---

다음과 같이 typescript에 대한 기본 설정을 한다.

```sh
$ npx tsc --init
```

tsconfig.json와 같은 파일이 생성되는데 안에 주석처리 되어 있는 "inlineSourceMap"을 true로 주석을 풀어준다.

#### [tsconfig.json]
```json
{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "sourceMap": true,
    "strict": true
  },
  "exclude": [
    "node_modules"
  ]
}
```

---

설정한 후에 아래와 같이 config 디렉터리를 생성하고 webpack.config.dev.js 파일을 생성하여 아래의 코드를 입력한다.

#### [config/webpack.config.dev.js]
```js
const path = require('path');
const webpack = require('webpack');

module.exports = {
  entry: path.resolve(__dirname, '../src/www.ts'),
  target: "node", // node js environment
  output: {
    path: path.resolve(__dirname, '../dist'),
    filename: 'dev.bundle.js'
  },
  resolve: {
    extensions: ['.ts', '.js']
  },
  devtool: 'source-map',
  module: {
    rules: [
      { test: /\.ts$/, loader: 'ts-loader' }
    ]
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin()
  ]
};
```

---

webpack을 실행하는 script를 작성한다.

#### [scripts/build-dev.ts]
```ts
process.env.NODE_ENV = "development";

const webpack = require('webpack');
const config = require('../config/webpack.config.dev');

const compiler = webpack(config, () => {
  console.log("Development build complete.");
});

const watching = compiler.watch({
  aggregateTimeout: 500,
  poll: 1200,
  ignored: /node_modules/
}, (err, stats) => {
  if(err) return console.error(err);

  // console.log(stats);
  console.log("Server is updated.");
});
```

---

설정을 완료했다면 src 디렉터리를 추가하고 App.ts파일을 생성하고 아래의 코드를 입력한다.

#### [src/App.ts]
```ts
import * as express from "express";

class App {
  public app: express.Application;

  /**
   * @ class App
   * @ method bootstrap
   * @ static
   * 
   */
  public static bootstrap (): App {
    return new App();
  }

  constructor () {
    this.app = express();
    this.app.get("/", (req: express.Request: res: express.Response, next: express.NextFunction) => {
      res.send("Hello world");
    });
  }
}

export default App;
```

---

express server를 실행하는 코드를 작성한다.

#### [src/www.ts]
```ts
import 'source-map-support/register'; // source-map을 사용하기 위해 추가함.
import App from './App';
import * as express from "express";

const port: number = Number(process.env.PORT) || 3000;
const app: express.Application = new App().app;

app.listen(port, () => console.log(`Express server listening at ${port}`))
.on('error', err => console.error(err));
```

---

아래와 같이 명령어를 입력하여 제대로 동작하는지 확인한다.

```sh
$ node scripts/build-dev.js
Server is updated.
Development build complete.
```

문제없이 webpack이 동작하여 dist/dev.bundle.js가 생기는 것을 확인했다면 package.json에 scripts에 다음 설정도 추가해준다.

webpack이 번들로 지속적으로 변환시켜주면 node express server를 재시작해준다.

#### [package.json]
```json
{
  "start": "npm-run-all -p start:server start:build",
  "start:server": "nodemon ./dist/dev.bundle.js --watch \"./dist\"",
  "start:build": "node ./scripts/build-dev"
}
```

마지막으로 npm start를 하면 코드가 바뀔 때마다 재시작 하는 것을 확인할 수 있다.

```sh
$ npm start
> practice_node_server@1.0.0 start 
> npm-run-all -p start:server start:build


> practice_node_server@1.0.0 start:server 
> nodemon ./dist/dev.bundle.js --watch "./dist"


> practice_node_server@1.0.0 start:build 
> node ./scripts/build-dev

[nodemon] 1.12.1
[nodemon] to restart at any time, enter `rs`
[nodemon] watching: 
[nodemon] starting `node ./dist/dev.bundle.js`
Express server listening at 3000
Server is updated.
[nodemon] restarting due to changes...
[nodemon] starting `node ./dist/dev.bundle.js`
[nodemon] restarting due to changes...
Development build complete.
[nodemon] starting `node ./dist/dev.bundle.js`
Express server listening at 3000
```

---
## 고찰
---

TypeScript를 따로 공부해두긴 했었지만 실제로 적용하려하니 개발환경을 설정하는 것이 제일 힘들었다. 
하나씩 webpack을 설정해야 되는 부분이 생각보다 디테일을 요구하였다. 또한 각각의 설정에 따른 결과를 명확히
알아야만 효율적인 bundling을 구현할 수 있다는 점이 힘들었다.

TypeScript의 문법 자체는 기존에 여러 웹사이트를 SPA 방식으로 React.js로 개발한 경험으로 어느정도 커버가 되었다.
디테일한 데코레이터를 지정하는 것은 아직 연습이 필요해 보이지만 개발하는데 지장될 정도는 아닌 것 같다.

만약 처음에 Node.js를 접하는 사람이 개발을 하는 경우에는 TypeScript를 사용하지 말라고 권하고 싶다.
다른 C++, Java, Go 와 같은 언어는 Type 지원이 Compiler에서 지원이 되기 때문에 따로 설정이
필요하지 않지만, TypeScript는 Webpack을 사용할 경우에는 ts-loader를 webpack에 module로 심어줘야 하는 등의
많은 번거로움이 있기 때문에 처음부터 진이 빠져 개발의 흥미를 잃을 가능성이 있기 때문이다.

하지만 장점으로는 처음에 설정만 제대로 한다면 Type지원이 되기 때문에 Javascript로 개발할 때 약점이었던
IDE상에서의 Hint가 정의한 Type대로 나타나는 점이다. 각 Object의 어떤 function이 있고 각 parameter의 Type을
알려주기 때문에 점차 큰 프로젝트로 변화할 때는 개발 속도의 지연이 없을 것이다.

이렇게 한 번 정리해놓았기 때문에 다음번에는 편한 개발이 가능할 것이다.

---
## References
---

- [https://webpack.js.org/](https://webpack.js.org/)
- [https://www.youtube.com/watch?v=ose1VIo213k](https://www.youtube.com/watch?v=ose1VIo213k)
- [https://github.com/novemberde/practice_node_server](https://github.com/novemberde/practice_node_server)