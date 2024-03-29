---
title: Electron으로 Cloud 9 (c9.io) desktop app 만들기
tags: [c9, desktopapp, cloud9, cloud9, electron]
date: "2017-08-18T11:30:03+00:00"
aliases: ["/etc/2017/08/18/make_c9_app.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---
## Summary
---
[c9.io](https://c9.io/)를 살펴보니 상당히 괜찮은 IDE로 느껴졌다.
Docker container를 할당해서 독립적인 작업 공간을 부여하는 점과
리눅스환경이기 때문에 window를 사용하는 사람에게 여러 환경설정의
늪에서 벗어나게 해줄 수 있을 것 같았다.

하지만 불편한 점도 있다. 브라우저 안에 갇혀 있기 때문에 답답한 느낌이 드는게 가장 큰 부분이었다.
거기에다가 installer도 지원해주지 않기 때문에 별도로 app형식으로 사용하지 못했다.

이럴 때 Electron이 떠올랐는데, Chromium 기반으로 Google chrome에서 돌아가는 웹페이지는
모두 어플리케이션으로 만들 수 있기 때문이다.

혼자 Toy project의 개념으로 Electron을 활용해서 c9.io의 installer를 만들고 로컬에서
c9.io를 어플리케이션으로 사용해보자.

---
### Electron 시작하기
---

Electron quick start 템플릿을 받고 폴더명만 c9-app으로 고쳐주자.

```bash
$ git clone https://github.com/electron/electron-quick-start
```

그 다음 'hello world'를 보기 전에 필수 패키지를 설치하자.
사전에 nodejs와 npm이 설치되어 있어야 한다.([nodejs & npm](https://nodejs.org/ko/download/package-manager/) 참고)

```bash
# install electron as global
$ npm install -g electron

~/c9-app $ npm start
```

새 창이 열리면서 'hello world'가 나타날 것이다.

거의 다 왔다.

별도로 앱 커스텀은 따로하면 되고 설정하기 귀찮으니 'index.html'만 변경해주자.

아래쪽의 스크립트영역에 아래와 같이 써주자.

```javascript
location.href = 'https://c9.io'
```

다시 시작해주면 아래와 같은 화면이 나타날 것이다.

```bash
~/c9-app $ npm start
```

![c9.io main page](/images/c9/main.png)

---
### Build Desktop app
---

앱으로 만들기 위해선 electron-packagr가 필요하다.
Windows환경에서 개발하고 있으니 win32로 platform을 지정해주자.

```bash
$ npm install electron-packager -g
$ electron-packager . c9-app --platform=win32 --arch=x64
```

모든 과정을 마치면 루트 디렉터리에 'c9-app-win32-x64' 폴더가 생길 것이다.

실행하면 아까와 같은 c9.io 페이지가 열릴 것이다.

---

이제 편하게 c9으로 작업하자.

관련 작업한 repository는 public으로 github에 올려놓았다.
[https://github.com/novemberde/c9-app](https://github.com/novemberde/c9-app)

---
### References
---

- [https://electron.atom.io/docs/](https://electron.atom.io/docs/)
- [https://electron.atom.io/docs/tutorial/quick-start/](https://electron.atom.io/docs/tutorial/quick-start/)
- [https://github.com/electron-userland/electron-packager](https://github.com/electron-userland/electron-packager)