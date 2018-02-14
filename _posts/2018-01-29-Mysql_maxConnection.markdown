---
title: AWS Lambda와 MySQL의 max connection 문제
category: aws
tags: lambda, concurrency, mysql, database, max, connection, max_connections, 병렬
---
## Summary
---
MySQL에서 Max Connection에 대해서 알아보려고 한다. AWS Lambda container가 지속적으로 생성될 때
Database가 감당할 수 있는 Max connection이 얼마인지 알아야 대응할 수 있기 때문이다.
만약 Lambda container가 MySQL이 감당할 수 없을 정도로 계속 생성된다면 Database에 connection이
일어나지 않게 되고, too many connections error가 발생하여 웹서버 역할을 해야하는 Lambda가 동작하지 않을 수 있다.
물론 MySQL에 접속하는 다른 Worker들도 동작하지 않는다.

RDS를 사용하면 스케일 업이 될 때마다 Max connection 설정을 따로 하지 않더라도 알아서 늘어난다.
별다른 고민할 것 없이 Max connection이 생길 때마다 RDS를 scale up 해주어도 되겠지만 개발자이기 때문에
더 Graceful하게 문제를 해결해야한다.


## MySQL에서 max connection 확인하기
---

MySQL에서 max connection을 확인하는 쿼리는 다음과 같다.

```sql
show variables like 'max_connections';
```

그리고 RDS instance type에 따른 max_connections는 다음과 같다.

- 2.micro: 66
- t2.small: 150
- m3.medium: 296
- t2.medium: 312
- M3.large: 609
- t2.large: 648
- M4.large: 648
- M3.xlarge: 1237
- R3.large: 1258
- M4.xlarge: 1320
- M2.xlarge: 1412
- M3.2xlarge: 2492
- R3.xlarge: 2540

Amazon Aurora의 instance type에 따른 max_connections는 다음과 같다.

- db.t2.small: 45
- db.t2.medium: 90
- db.r3.large: 1000
- db.r3.xlarge: 2000
- db.r3.2xlarge: 3000
- db.r3.4xlarge: 4000
- db.r3.8xlarge: 5000
- db.r4.large: 1000
- db.r4.xlarge: 2000
- db.r4.2xlarge: 3000
- db.r4.4xlarge: 4000
- db.r4.8xlarge: 5000
- db.r4.16xlarge: 6000


## 사용하는 Persistance framework의 Connection pool size알기
---

WebApp 또는 Worker에서 동작하는 connection pool 옵션을 확인하고
RDS가 사용되는 max_connection의 수를 나누면 대략적으로 Concurrency하게 동작하는 개수를 알아낼 수 있다.

그 다음 Lambda의 옵션 중 concurrency 옵션을 수정하면 된다.


## 고찰

현재 업무에서 사용하는 모든 웹앱이 AWS Lambda에서 동작하고 있다. 점점 사용자가 늘어감에 따라
가끔 설계해놓은 스펙 이상으로 부하가 몰릴 경우가 있다. Lambda에서 Timeout이 빈번하게 발생할 경우
timeout되기 전 상태의 lambda가 계속 생성되어 순간적으로 RDS의 connection에 문제가 생겼었다.
물론 Lambda timeout을 큰 고찰없이 10초 이상으로 설정해 놓을 수도 있다.
예를 들어, 10초 사이에 많은 요청이 생긴다고 가정해보자.
rds는 (컨테이너의 수*connection pool수)만큼 connection을 맺는데,
timeout으로 끝나는 컨테이너가 많아질수록 RDS의 max_connections옵션을 초과하여 
too many connections error를 내뿜게 된다.
결과적으로 클라이언트들은 원하는 상태를 처리하기 위해 지속적인 요청을 보낼 것이고 RDS는 CPU 100%로 클라이언트가 진정될 때까지
계속 힘들어할 것이다.

이런 문제점 때문에 Timeout도 길게 설정하지 말고 4~8초 내외로 잡는 것이 낫다. 물론 배치잡같은 경우를 제외하고 말이다.
이렇게 되면 어느정도 too many connections error를 잡는데 도움을 준다.

그렇지만 무엇보다 비지니스 로직에서 최적화를 잘 시켜서 timeout상태를 만들지 않는게 중요할 것이다.