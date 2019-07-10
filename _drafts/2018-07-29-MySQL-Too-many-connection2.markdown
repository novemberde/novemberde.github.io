---
title: MySQL Too many connections error의 케이스
category: mysql
tags: aws, mysql, aurora, too, many, connections, error
---
## Summary
---
서비스를 운영하다보면 어느순간 DB에 접속하지 못하는 순간이 찾아온다.
급작스럽게 현상이 나타나 대응하기가 쉽지 않았다.
특히 DB가 접속이 되지 않는다면 서비스 자체가 불가능한 상황이기 때문에
빠르게 문제를 해결해야한다.

FLUSH HOSTS;