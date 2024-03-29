---
title: Jenkins의 Blue ocean을 활용하여 배포 관리하기
tags: [jenkins, deployment, 배포, 관리, Blueocean, pipeline, cd, ci, cicd]
date: "2017-10-21T11:30:03+00:00"
aliases: ["/devops/2017/10/21/Jenkins.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---
## Summary
---
Jenkins는 CD/CI로 가장 알려진 도구이다. 이번에는 Jenkins 설정부터 시작하여 Pipeline으로 배포관리하는 과정을 정리해본다.

---
## Jenkins로 기본 설정하기
---

Jenkins를 [Install Jenkins](https://jenkins.io/doc/book/installing/)를 참고하여 설치한다.

설치하여 맨처음 접속하면 아래와 같은 화면이 나타난다.
다음의 명령어로 나온 텍스트를 입력한다.
```
$ cat /var/jenkins_home/secrets/initialAdminPassword
```

![gettingStarted](/images/jenkins/gettingStarted.png)


---

Customize Jenkins화면에서 여기서는 크게 설정할 필요없이 Install suggested plugins을 선택한다.

이후에 해당 플러그인을 설치하고 관리자 계정을 설정할 수 있다.

테스트를 위해 계정 정보는 다음과 같이 설정한다.

- 계정명: admin
- 암호: 1234
- 이름: admin
- 이메일 주소: 본인 이메일 주소

![main](/images/jenkins/main.png)


---

Jenkins 관리 > 플러그인 관리 > 설치 가능 으로 들어가서 Blue ocean을 선택하여 재시작없이 설치하기 버튼을 클릭한다.

설치 후에 메인으로 들어가면 아래와 같이 Open blue ocean이 생긴 것을 볼 수 있다.

![mainBlueocean](/images/jenkins/mainBlueocean.png)


---
## Blue Ocean 시작하기
---

Open blue ocean 을 클릭하고 create pipeline을 하면 아래와 같은 화면을 볼 수 있다.

Github를 선택하여 repository를 통한 Pipeline을 생성한다.

![createPipeline](/images/jenkins/createPipeline.png)


---

클릭후에 Github에서 Token을 Default로 선택된 옵션을 선택하여 발급한다.

그 다음 pipeline으로 생성하고 싶은 repository를 선택한 후 pipeline을 생성한다.

![createAccessToken](/images/jenkins/createAccessToken.png)


---
## Pipeline setting하기
---

Pipeline을 생성하면 아래와 같은 화면을 확인할 수 있다.

\+ 버튼을 누르면 stage를 설정할 수 있는 화면이 나타난다.

\+ add steps를 통해 작업을 실행할 수 있다.

Shell script를 선택하여 ls 를 입력한다.

![createdPipeline](/images/jenkins/createdPipeline.png)


---

설정 후에 상단의 save 버튼을 누르면 아래와 같이 pipeline에 대한 commit을 할 수 있다.

Jenkinsfile을 repository에 반영하게 된다.

![save](/images/jenkins/save.png)


---

저장 완료 후에 Pipeline의 Stage에서 동작 로그를 확인해볼 수 있다.

![activity](/images/jenkins/activity.png)


---
## 고찰
---

요즘엔 AWS의 managed service를 쓰는 것을 좋아해서 AWS Code Pipeline을 사용하는 연습을 했었다.

하지만 AWS의 종속적이지 않을 경우에 CD/CI를 구축한다면 어떻게 Pipeline을 구성할지 생각해보았다.

기존의 서비스에서 Jenkins를 통한 배포 방식은 하나의 Project를 생성하여 직접 배포를 하였다. 현재처럼 각 단계를 두어 Pipeline으로 분리하지는 않았었다.
그래서 Pipeline으로 구성할 필요성을 느꼈었다.
Pipeline으로 구성하면 어떤 곳에서 문제가 생겼는지 시각화되어 명확하게 디버깅을 할 수 있기 때문이다.

Blue Ocean은 그런점에 있어서 적절한 대안이 된다. 또한, Blue Ocean이 기존의 Jenkins 대시보드 화면보다 훨씬 세련되기 때문에 사용하기 편리하기 때문이다.

<!-- ---

각 DOM Component 구성을 React.js로 했기 때문인지 몰라도 조작하는게 굉장히 깔끔한 느낌이 든다. 역시 요즘 대세는 SPA방식인 듯하다.

--- -->

마지막으로, 직접 서비스를 구성한다면 AWS의 Code Pipeline을 사용하겠지만, AWS의 가치는 이번 포스팅처럼 꽤나 긴 여정을 지나야만 느낄 수 있기 때문에 Blue Ocean을 한 번 구성해보는 것도 좋을 것이다.

---
## References
---

- [https://jenkins.io/download/](https://jenkins.io/download/)
- [https://jenkins.io/projects/blueocean/](https://jenkins.io/projects/blueocean/)


