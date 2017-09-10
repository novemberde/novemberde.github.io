---
title: Docker swarm으로 Multi container application 배포
category: docker
tags: docker, swarm, multi, container, application, deploy
---
## Summary
---
docker container를 배포할 때 AWS의 ElasticBeanstalk나 ECS를 사용하지 않고,
여러 인스턴스 상에서 docker container를 사용하면 어떨지 궁금하였다.

Swarm으로 관리하면 어떤 형식으로 나타나는지 Demo project 개념으로 진행해보자.

---
### 주요 특징
---

- Docker engine과 통합된 클러스터 관리
  - Docker engine CLI로 배포할 App의 swarm을 생성할 수 있다.
  - 별도의 Docker ochestration이 필요한 software가 필요 없다.
- Decentralized design
  - 배포시에 각 노드 role을 핸들링하는 대신에 Docker engine이 런타임동안 모든 것을 다룹니다.
  - Docker engine을 통해 Managers와 Workers를 배포할 수 있다.
  - 모든 Swarm을 Single disk image에서 관리할 수 있다.
- Declarative service model
  - Docker Engine은 Application stack에서 원하는 다양한 서비스를 정의할 수 있도록 선언적 접근 방식을 사용한다.
  - 예를 들면, Web Frontend 와 Backend를 포함한 서비스를 설명할 수 있다.
- Scaling
  - 각 서비스에 대해 실행하고 싶은 Task의 갯수를 조절 할 수 있다.
  - Scale up 또는 down에 대해서 swarm manager가 자동적으로 설정한 상태에 따라 컨테이너를 추가하거나 삭제한다.
- Desired state reconciliation
  - Swarm manager는 지속적으로 클러스터 상태를 모니터링하고,  실제 상태를 비교 조정한다.
  - 예를 들어 2개의 컨테이너 레플리카가 문제가 생기면 매니저가 2개의 컨테이너를 새로 생성하여 대체한다.
- Multi-host networking
  - 서비스 위에 네트워크망을 구성할 수 있다. 어플리케이션의 생성이나 업데이트를 할 때 Swarm manager가 동적으로 주소를 할당한다.
- Service discovery
  - Swarm manager node는 각 서비스에 유일한 DNS명과 로드밸런서를 할당한다.
  - Swarm에 임베드된 DNS 서버를 통해 실행된 모든 컨테이너에 질의할 수 있다.
- Load balancing
  - External load balancer에 포트를 할당할 수 있다.
  - 내부적으로는 Swarm이 각 노드간에 서비스 컨테이너를 분배할지 명시해야한다.
- Secure by default
  - Swarm의 각 노드는 보안을 위해 TLS(Transport Layer Security) 상호 인증과 암호화가 강제된다.
  - Self-signed certificates 를 사용하는 옵션도 있다.
- Rolling updates
  - Swarm manager가 각 서비스를 배포할 때 나타나는 딜레이에 대해서 조정하고 점진적으로 업데이트를 적용한다.
  - 만약 문제가 생긴다면 이전 버전의 서비스로 롤백한다.

<!-- 
- Decentralized design: Instead of handling differentiation between node roles at deployment time, the Docker Engine handles any specialization at runtime. You can deploy both kinds of nodes, managers and workers, using the Docker Engine. This means you can build an entire swarm from a single disk image.
- Declarative service model: Docker Engine uses a declarative approach to let you define the desired state of the various services in your application stack. For example, you might describe an application comprised of a web front end service with message queueing services and a database backend.
- Scaling: For each service, you can declare the number of tasks you want to run. When you scale up or down, the swarm manager automatically adapts by adding or removing tasks to maintain the desired state.
- Desired state reconciliation: The swarm manager node constantly monitors the cluster state and reconciles any differences between the actual state and your expressed desired state. For example, if you set up a service to run 10 replicas of a container, and a worker machine hosting two of those replicas crashes, the manager will create two new replicas to replace the replicas that crashed. The swarm manager assigns the new replicas to workers that are running and available.
- Multi-host networking: You can specify an overlay network for your services. The swarm manager automatically assigns addresses to the containers on the overlay network when it initializes or updates the application.
- Service discovery: Swarm manager nodes assign each service in the swarm a unique DNS name and load balances running containers. You can query every container running in the swarm through a DNS server embedded in the swarm.
- Load balancing: You can expose the ports for services to an external load balancer. Internally, the swarm lets you specify how to distribute service containers between nodes.
- Secure by default: Each node in the swarm enforces TLS mutual authentication and encryption to secure communications between itself and all other nodes. You have the option to use self-signed root certificates or certificates from a custom root CA.
- Rolling updates: At rollout time you can apply service updates to nodes incrementally. The swarm manager lets you control the delay between service deployment to different sets of nodes. If anything goes wrong, you can roll-back a task to a previous version of the service.
 -->

---
### 시작하기
---

특징을 알아봤으니 Tutorial을 따라 Swarm을 돌려보자

```bash
# 초기화 하기
$ docker swarm init --advertise-addr 192.168.99.121
Swarm initialized: current node (4qm9tb5mpm7fm5gmy6r4qppae) is now a manager.

To add a worker to this swarm, run the following command:
    docker swarm join \
    --token SWMTKN-1-2ykngegvud4tmefd4qulxhhb1f3ebrgkrk8k2b76mhu2eggats-04aezv6ubsi1vwbduz6gba36z \
    172.31.33.203:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```




---
### References
---
- [https://docs.docker.com/engine/swarm/#feature-highlights](https://docs.docker.com/engine/swarm/#feature-highlights)