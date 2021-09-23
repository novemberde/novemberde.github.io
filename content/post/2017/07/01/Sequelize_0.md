---
title: Sequelize에서 parallel execution과 serial execution
tags: [nodejs, sequelize, Promise, parallel, execution]
date: "2017-07-01T13:30:03+00:00"
aliases: ["/nodejs/2017/07/01/Sequelize_0.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

## Summary
---
 node js로 서버를 구성할 때 ORM framework로 sequelize를 사용한다. 
 하지만 비동기로 모든 CRUD가 진행되다보니 동시에 여러 쿼리문의 결과가 요구될 때도 있다.
 serial execution과 parallel execution을 살표보자.

---
### parallel execution과 serial execution
---

 Promise pattern을 활용하여 절차적으로 함수를 실행하면 여러 트랜잭션 과정을 파악하기 쉽다.
 callback 패턴의 늪에서 벗어날 수 있는 Promise는 현재 서버사이드에서 사용되는 라이브러리들에서
 적극 활용되어지고 있다.

 먼저 serial execution을 살펴보고 이에 대한 단점도 파악해 보자.

 아래의 예제는 User를 검색하고 관계가 형성되어 있지 않은 다른 테이블을 Select하는 경우이다.

```javascript
// Serial execution
let users;
let menus;

models.User.findAll({
    attributes: ['id', 'name', 'phone', 'address']
})
.then( result => {
    if (!result) throw new Error("Not Found");

    users = result;

    return models.Menu.findAll()
})
.then( result => {
    if (!result) throw new Error("Not Found");

    menus = result;

    console.log(users);
    console.log(menus);
});
```

이를 살펴보면 조금 이상한 점이 있다. 하나의 결과를 받고 그 다음에 또 다시 쿼리를 실행하는 느낌이다.
서로 관계없는 데이터이기 때문에 두개의 쿼리를 동시에 보내고 따로 값을 받아서 처리하는게 나아보인다.
왜냐하면 절차적으로 반드시 이렇게 이루어져야만 하는 쿼리가 아니기 때문이다.

만약 하나의 값을 select해서 insert하는 경우라면 사용할 수 있겠지만 위의 경우는 상관관계가 없기 때문이다.

그렇다면 어떻게 사용할 수 있을까?

Promise.all() 함수를 사용하면 간단히 해결할 수 있다.
```javascript
// Serial execution
let users;
let menus;

Promise.all([
    models.User.findAll({
        attributes: ['id', 'name', 'phone', 'address']
    }),
    models.Menu.findAll()
])
.then( result => {
    if (!result[0]) throw new Error("Not Found");
    if (!result[1]) throw new Error("Not Found");
    
    users = result;
    menus = result;

    console.log(users);
    console.log(menus);
});
```

위 뿐만아니라 하나의 transaction이 일어나는 과정에서 여러 데이터를 CRUD하는 경우에도 유용하게 사용될 수 있다.

이처럼 Promise는 비동기적으로 일어나는 과정을 간단히 표현할 수 있다.
모든 이벤트가 종료되었을 때 then절이 동작하기 때문에 코드가 난잡해지는 것을 방지하기 때문이다.

Promise 패턴이 javascript에서 많은 변화를 가져왔다. 
최근에는 Async/Await가 사용되는데 nodejs 8 version이후부터 사용 가능하다고 한다.

비동기 패턴에 대해 더욱 공부하고 싶으신 분들은 Async/Await에 대해 찾아보길 바란다.

---
### References
---
- [http://docs.sequelizejs.com/manual/tutorial/transactions.html](http://docs.sequelizejs.com/manual/tutorial/transactions.html)
- [https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise)