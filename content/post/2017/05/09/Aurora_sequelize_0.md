---
title: Amazon Aurora와 sequelize 연동해보기.
tags: [aws, aurora, sequelize, nodejs, orm]
date: "2017-05-09T11:30:03+00:00"
aliases: ["/2017/05/09/Aurora_sequelize_0.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

## Summary
---------------------
최근에 글 중에서  aurora와 sequelize가 과연 연동이 가능할까라는 글을 본적이 있다.
aurora는 MySQL과 호환이 가능한 DB라고 하기 때문에 나 또한 궁금증이 생겼다.
직접 aurora와 sequelize를 연동해보고, MySQL workbench로 aurora를 사용할 수 있는지까지 확인해보겠다.

---------------------

## Aurora instance 생성하기
---------------------

이전까지 사용할 때는 MySQL이나 MariaDB위주로 사용했다. 
스타트업은 라이센스 비용에 민감하기 때문에 Open Source를 주로 선택했기 때문이다.

또한 아래 리스트를 보면 RDS 이름에서 추측할 수 있듯이 RDS에서는 관계형 DB만 사용할 수 있는 것을 알 수 있다.
별도로 NoSQL을 사용하고 싶다면 AWS에서 지원하는 DynamoDB를 사용하거나 EC2에 DB를 올리는 방법이 있다.

이제 다시 본론으로 돌아와서 아래와 같은 화면에서 aurora를 선택한다.

특징을 살펴보면
- MySQL에 비해 5배의 처리 성능을 지님
- 15개의 레플리카를 만드는데 최고 10 ms 지연
- 64TB까지 오토스케일링 저장소가 여러 availability zone에 replicated

이와 같은 특징을 보면 별도로 MySQL에서 레플리카, 별도의 저장공간 확보나 클러스터링에 대해서 신경쓸 부분들이 확연히 줄어든다는 것을 알 수 있다.

![select_engine](/images/aws/rds/select_engine.png)
 
 ---

DB설정정보를 입력하자.

- DB Instance Identifier* 에는 DB를 구분할 값이다. 우리가 사용할 이름을 넣어주면된다.
- Multi_AZ Deployment는 여러 Zone에 Replica를 둠으로써 현재 DB가 올라가 있는 Zone에 이상이 발생하더라도 대처할 수 있게 해주는 고마운 도구다.
- Master Username* 에는 db의 마스터 계정 아이디라고 보면된다.
- Master Password* 에는 db의 마스터 계정 비밀번호를 입력하자.

![specify_db_details](/images/aws/rds/specify_db_details.png)

---

테스트 환경이므로 여기서는 큰 설정을 하지 않는다.

VPC Security Group에서 새로운 Security Group을 생성한다는 부분만 상기하고 넘어가자.

Security Group에서 별도로 Port를 열어주지 않더라도 RDS인스턴스를 생성할 때 접속한 로컬의 IP주소로 3306포트가 열려있다.

별도로 security group을 설정하지 않고 넘어가겠다.

![configure_advanced_setting](/images/aws/rds/configure_advanced_setting.png)

---

이제 Aurora DB가 생성되고 있음을 볼 수 있다.

![dashboard](/images/aws/rds/dashboard.png)

---

## MySQL Workbench로 접속하기
---

MySQL workbench를 실행하고 새로운 connection을 test해보자.

아래와 같은 화면으로 connection이 성공한 것을 볼 수 있다.

![mysql_workbench_test_connection](/images/aws/rds/mysql_workbench_test_connection.png)

---

이제 접속하여 SQL 쿼리를 사용하여 보자.
MySQL과 동일한 쿼리가 동작하는 것을 볼 수 있다.

```sql
-- 스키마 생성하기.
CREATE SCHEMA `test` ;

-- 테이블 생성하기. createdAt,updatedAt,deletedAt은 sequelize가 기본적으로 찾는 필드이다. 이를 제어하고 싶으면 별도의 설정이 필요하다.
CREATE TABLE `test`.`user` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(45) NOT NULL,
  `createdAt` DATETIME NULL,
  `updatedAt` DATETIME NULL,
  `deletedAt` DATETIME NULL,
  PRIMARY KEY (`id`));
```

아래와 같은 테이블이 생성된 것을 볼 수 있다.

![new_table](/images/aws/rds/new_table.png)

---

## Sequelize로 사용하기
---

sequelize-auto를 사용해서 models를 뽑아오자.

npm으로 사전에 sequelize-auto와 mysql을 global로 설치해 놓아야 한다.

```bash
# 기본 패키지 설치
$ npm install -g sequelize-auto mysql
# models 뽑기
$ sequelize-auto -o "./models" -d test -h HOST -u USERNAME -p 3306 -x PASSWORD -e mysql
```

나온 models는 아래와 같다.

```js
module.exports = function(sequelize, DataTypes) {
  return sequelize.define('user', {
    id: {
      type: DataTypes.INTEGER(11),
      allowNull: false,
      primaryKey: true,
      autoIncrement: true
    },
    name: {
      type: DataTypes.STRING(45),
      allowNull: false
    },
    createdAt: {
      type: DataTypes.DATE,
      allowNull: true
    },
    updatedAt: {
      type: DataTypes.DATE,
      allowNull: true
    },
    deletedAt: {
      type: DataTypes.DATE,
      allowNull: true
    }
  }, {
    tableName: 'user'
  });
};
```


마지막으로 Sequelize로 insert와 findAll을 해본다.

```js
const models = require('./models');

models.user.create({name: "KHBYUN"})
.then( result => {
    console.log(result.dataValues);
})
.catch( err => {
    console.error(err);
});

models.user.findAll()
.then( result => {
    console.log(result);
})
.catch( err => {
    console.error(err);
});
```

---

이상없이 동작하는 것을 확인할 수 있다. 

Aurora는 MySQL과 호환성이 있기 때문에 MySQL과 사용하듯 Sequelize를 적용하면 Aurora도 큰 무리없이 사용할 수 있을 것으로 예상된다.

무엇보다 관리적인 이점을 Aurora가 가지고 있기 때문에 Aurora를 사용하는 면이 개발자에게 많은 수고로움을 덜어줄 것이다.

<!-----

## References
----->