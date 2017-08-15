---
title: 다른 서버로 Mongodb 이전하기
category: database
tags: mongodb, mongodump, scp, mongorestore
---

## Summary
---
 의뢰중에 호스팅 서버를 AWS로 옮겨달라는 요청이 있었다.
 MongoDB서버를 옮기기 위해서 DB를 백업하고 SCP를 사용하여 백업한 정보를 해당 인스턴스로 보내 백업을 진행하였다.
 몇몇 간단한 명령어를 통하면 DB의 backup정보를 통해 복구할 수 있다.
 과정을 살펴보자.

---
### 현재 db정보를 dump로 만들어 SCP로 파일 전송하기
---

 이전작업을 하기 전에 먼저 클라이언트에게 DB를 실제 서비스에서 분리하고 작업해야된다고 하였다.
 
 설정정보를 바꾼다면 DB를 재가동 해야하는 이유도 있었으며, mongodb dump파일을 생성한 시점부터는 추가되는 데이터가 없어야 하기 때문이다.

 먼저 외부와의 접속을 차단하고 mongodump파일을 생성해 보자.

```bash
# ~/dump 에 mongodumb파일 생성하기.
$ mongodump --port 27017

# scp로 파일 보내기
$ scp -r -i /home/<USER>/<AWS_USER>.pem /home/<USER>/dump <AWS_USER>@ec2-111-111-111-111.ap-northeast-2.compute.amazonaws.com:~/dump
```

---
### 이전될 서버에서 dbdump파일로 복구하기
---

 SCP를 통해 파일이 제대로 전송됐다면 mongorestore를 통해 DB를 restore해주자.

```bash
# localhost에서 동작중인 mongodb에 데이터를 복구하기.
$ mongorestore -h 127.0.0.1 ~/dump/
```
---
### References
---
- [https://docs.mongodb.com/v3.0/reference/program/mongodump/](https://docs.mongodb.com/v3.0/reference/program/mongodump/)