---
title: Docker image를 EB(Elastic Beanstalk)를 통해 배포하기
category: docker
tags: docker, eb, aws, elastic beanstalk
---

## Summary
---
 AWS에서 Elastic Beanstalk를 통해 docker image를 배포할 수 있다.
 이번에는 기존에 만들었던 Dockerfile을 Elastic beanstalk에 배포해 보겠다.

---
### Dockerfile 준비하기
---

이전에 node js server를 하나의 Dockerfile로 만들어 놓았다. 
테스트하고 싶으신 분들은 [this repository](https://bitbucket.org/kyuhyun/docker_node_server)를 참고하길 바란다.

```Dockerfile
FROM novemberde/node-pm2

MAINTAINER KH BYUN "novemberde.github.io"

ENV NODE_ENV production

EXPOSE 3000

COPY ./ /src

RUN npm install --prefix /src

CMD ["pm2-docker", "/src/app.js"]
```

---
### Elastic Beanstalk 설정하기
---

Elastic Beanstalk로 배포하는 경우에 아래와 같이 2가지 방법이 있다. 
- Single instance
- Load balancing, auto scaling

Single instance는 하나에 인스턴스에 해당 dockerfile로 구성된 인스턴스가 올라가는 경우를 발한다.

만약 실무적으로 하나의 웹어플리케이션을 배포한다고 봤을 때는 Load balancing, auto scaling은
필수적으로 이루어져야 하기 때문에 load balancing, auto scaling을 택하는 것이 좋을 것이다.

이번에는 테스트이기 때문에 Single instance로 배포하고 메뉴를 살펴볼 것이다.

![eb_conf]({{ site.baseurl }}/images/aws/eb/configuration.png)


---
### Elastic Beanstalk에 Docker패키지 배포하기
---

Environment를 생성하면 다음과 같은 화면을 볼 수 있다.

여기서 Confiuration을 통해 docker 설정 정보를 변경할 수 있다.

우리는 Dockerfile과 함께 웹어플리케이션을 배포해야하므로 Upload and Deploy를 선택하자.

![eb_dashboard]({{ site.baseurl }}/images/aws/eb/dashboard.png)

직접 업로드하기를 선택하고, git repository에서 배포에 필요한 파일들인 
app.js, Dockerfile, package.json을 zip으로 압축하여 업로드하면 배포가 완료된다.

upload가 완료되었으면 Dashboard화면에서 위쪽의 url의 3000번 포트로 접속하여 배포가 완료되었는지 확인해 보자.

---
### Addition
---

 Seoul region 사용자들은 ECS(EC2 Container Service)를 사용하지 못해 안타까워하고 있다.
 하지만 Elastic Beanstalk에서 Multicontainer Docker Platform을 사용할 때
 내부적으로 ECS cluster를 사용하고 있다.

 이말은 ECS는 공식적으로 지원하지 않지만 Elastic Beanstalk를 docker로 활용하면
 ECS를 사용하는 것과 같은 형태라고 볼 수 있는 것이다.

 이에 대해서 좀 더 자세히 알고 싶으신 분은 다음의 링크를 참고하길 바란다.
 [Working with multicontainer docker platform](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_ecs.html)

---
### References
---

- [https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/Welcome.html?icmpid=docs_elasticbeanstalk_console](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/Welcome.html?icmpid=docs_elasticbeanstalk_console)
- [https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker.html](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker.html)
- [https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_ecs.html](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_ecs.html)