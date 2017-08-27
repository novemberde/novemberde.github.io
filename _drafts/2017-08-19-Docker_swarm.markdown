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
  - For each ser

Decentralized design: Instead of handling differentiation between node roles at deployment time, the Docker Engine handles any specialization at runtime. You can deploy both kinds of nodes, managers and workers, using the Docker Engine. This means you can build an entire swarm from a single disk image.

Declarative service model: Docker Engine uses a declarative approach to let you define the desired state of the various services in your application stack. For example, you might describe an application comprised of a web front end service with message queueing services and a database backend.

Scaling: For each service, you can declare the number of tasks you want to run. When you scale up or down, the swarm manager automatically adapts by adding or removing tasks to maintain the desired state.

Desired state reconciliation: The swarm manager node constantly monitors the cluster state and reconciles any differences between the actual state and your expressed desired state. For example, if you set up a service to run 10 replicas of a container, and a worker machine hosting two of those replicas crashes, the manager will create two new replicas to replace the replicas that crashed. The swarm manager assigns the new replicas to workers that are running and available.

Multi-host networking: You can specify an overlay network for your services. The swarm manager automatically assigns addresses to the containers on the overlay network when it initializes or updates the application.

Service discovery: Swarm manager nodes assign each service in the swarm a unique DNS name and load balances running containers. You can query every container running in the swarm through a DNS server embedded in the swarm.

Load balancing: You can expose the ports for services to an external load balancer. Internally, the swarm lets you specify how to distribute service containers between nodes.

Secure by default: Each node in the swarm enforces TLS mutual authentication and encryption to secure communications between itself and all other nodes. You have the option to use self-signed root certificates or certificates from a custom root CA.

Rolling updates: At rollout time you can apply service updates to nodes incrementally. The swarm manager lets you control the delay between service deployment to different sets of nodes. If anything goes wrong, you can roll-back a task to a previous version of the service.


---
### References
---
- [https://docs.docker.com/engine/swarm/#feature-highlights](https://docs.docker.com/engine/swarm/#feature-highlights)