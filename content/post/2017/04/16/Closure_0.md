---
title: Javascript에서 Closure와 Constructor
tags: [javascript, closure, constructor, performance]
date: "2017-04-16T11:30:03+00:00"
aliases: ["/2017/04/16/Closure_0.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

## Summary
---------------------
 node js나 web ui를 다루기 위해 javascript를 사용하면 closure를 사용할 것이다. constructor로 객체를 생성하지 않고 왜 closure를 사용하는지 살펴보자.

---------------------

## Constructor와 Closure의 성능차이
---------------------

샘플코드는 아래와 같다. 성능 결과값을 보면 거의 2배의 차이가 나는 것을 볼 수 있다. 

이러한 차이는 node 서버를 사용할 경우 더욱 두드러지게 체감할 것이다.

constructor를 사용하는 경우 javascript 객체가 반드시 가져야할 함수들을 상속받기 때문에 속도차이가 나타나는 것이다. 하지만 front에서는 클라이언트가 메모리를 가지기 때문에 크게 우려하지 않아도 된다. 

react와 같이 사용하는 프레임워크는 ES6문법으로 class를 사용하는 것을 권장하고 있다.

```javascript
var iterations = 1000000;

var closureObj = function( a, b ) {
    var c = a + b;

    return {
        getA: function() {
            return a;
        },
        getB: function() {
            return b;
        },
        getC: function() {
            return c;
        },
        sayHello: function() {
            console.log("say");
        }
    };
}

var ConstructorObj = function(a, b) {
    this.a = a;
    this.b = b;
    this.c = a + b;

    this.sayHello = function() {
        console.log("say");
    }
}

console.time('Closure');
for(var i = 0; i < iterations; i++ ){
    var a = closureObj("1", "2");
};
console.timeEnd('Closure')

console.time('Constructor');
for(var i = 0; i < iterations; i++ ){
    var a = new ConstructorObj("1", "2");
};
console.timeEnd('Constructor');

결과: 
    Closure: 103.231ms
    Constructor: 206.072ms
```


---
### References
---
- [https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Closures](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Closures)