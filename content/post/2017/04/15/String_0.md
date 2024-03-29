---
title: Java에서 String, StringBuilder, StringBuffer의 차이
tags: [java, string, stringbuilder, stringbuffer]
date: "2017-04-15T11:30:03+00:00"
aliases: ["/2017/04/15/String_0.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

## Summary
---------------------
 Java를 개발하다보면 String에 대해서 별다른 고민없이 ("Some text" + " added text")와 같이 '+'기호를 통해 스트링을 더하곤 한다.
 하지만 Java 개발자라면 고민을 더 해보고 Class를 선택해야한다. String과 StringBuilder, 그리고 StringBuffer를 어떤 경우에 사용하는지 확인해보자.

---------------------

## 각 클래스의 특징
---------------------

설명에 들어가기에 앞서서 [java api](http://docs.oracle.com/javase/8/docs/api/)와 친숙해져야 된다고 생각한다.

[java api](http://docs.oracle.com/javase/8/docs/api/)에 접속해서 java.lang 패키지를 들어가보자. java.lang은 별도로 import를 해주지 않아도 사용할 수 있는 class들이 모여있다.
Classes항목을 보면 자주 기본적인 클래스들이 있는 것을 볼 수 있다. Boolean, Byte, Integer등과 같은 WrapperClass들도 찾아볼 수 있다.

그중에서 우리가 눈여겨 볼 것은 String, StringBuffer, StringBuilder이다.

![java.lang](/images/java/string/java.lang.png)

---------------------

자세히 알아보기 위해 클릭해보자.

Serializable, CharSequence, Comparable<String> 인터페이스가 상속되어 있고 public final class로 되어 있다.
해석해보면 serialize가 가능하며 문자열이고 비교가능한 값이라는 것을 알 수 있다. 또한 final class이기 때문에 String class를 상속받을 수는 없다. 여기까지는 널리 알려진 사실이다.

![String](/images/java/string/String.png)

---------------------

하지만 String타입에 공부하다보면 String을 직접 더하는 것보다는 StringBuffer나 StringBuilder를 사용하라는 말이 있다.
이유에 대해서 살펴보자.

```java
String stringValue1 = "TEST 1";
String stringValue2 = "TEST 2";

System.out.println("stringValue1: " + stringValue1.hashCode());
System.out.println("stringValue2: " + stringValue2.hashCode());

stringValue1 = stringValue1 + stringValue2;
System.out.println("stringValue1: " + stringValue1.hashCode());

StringBuffer sb = new StringBuffer();

System.out.println("sb: " + sb.hashCode());

sb.append("TEST StringBuffer");
System.out.println("sb: " + sb.hashCode());

결과:
    stringValue1: -1823841245
    stringValue2: -1823841244
    stringValue1: 833872391
    sb: 1956725890
    sb: 1956725890
```

 위에서 보는바와 같이 생성된 클래스의 주소값이 다른 것을 볼 수 있다. String은 새로운 값을 할당할 때마다 새로 생성되기 때문이다.
이와 달리 StringBuffer는 값은 memory에 append하는 방식으로 클래스를 직접생성하지 않는다. 논리적으로 따져보면 클래스가 생성될 때
method들과 variable도 같이 생성되는데, StringBuffer는 이런 시간을 사용하지 않는다. 

또한 지금은 한 번만 생성되었지만 수십번 String이 더해지는 경우에는 각 String의 주소값이 stack에 쌓이고 클래스들은 Garbage Collector가 호출되기 전까지 heap에 지속적으로 쌓이게 된다. 메모리 관리적인 측면에서는 치명적이라고 볼 수 있다.

---------------------
 
 그럼 String class의 내부는 어떤 구조로 되어 있기에 새로 생성될까.

 아래 이미지를 보면 value[]라는 char형의 배열이 보인다. 여기서 힌트를 찾을 수 있다. private final char형이라는 것을 눈여겨 보아야 한다.

 String에서 저장되는 문자열은 알고보면 char의 배열형태로 저장되며 이 값들은 외부에서 접근할 수 없도록 private으로 보호된다. 또한 final형이기 때문에
초기값으로 주어진 String의 값은 불변으로 바뀔 수가 없게 되는 것이다.

 ![String2](/images/java/string/String2.png)


---------------------

 String의 특징을 알아봤으니 memory에 값을 append하는 StringBuilder와 StringBuffer에 대해서 알아보자. api는 아래와 같다.
 
 해석해보면 StringBuilder는 변경가능한 문자열이지만 synchronization이 적용되지 않았다. 하지만 StringBuffer는 thread-safe라는 말에서처럼 변경가능하지만 multiple thread환경에서 안전한 클래스라고 한다. 이것이 StringBuilder와 StringBuffer의 가장 큰 차이점이다.

 ![StringBuilder](/images/java/string/StringBuilder.png)

 ![StringBuffer](/images/java/string/StringBuffer.png)

 ---------------------

 StringBuilder와 StringBuffer를 테스트 해보자. 아래의 결과를 보면 다른 값이 나온 것을 볼 수 있다. StringBuilder의 값이 더 작은 것을 볼 수 있는데 이는 쓰레드들이 동시에 StringBuilder클래스에 접근할 수 있기 때문에 일어난 결과다. 이와 달리 StringBuffer는 multi thread환경에서 다른 값을 변경하지 못하도록 하므로 web이나 소켓환경과 같이 비동기로 동작하는 경우가 많을 때는 StringBuffer를 사용하는 것이 안전할 것이다.

```java
StringBuffer stringBuffer = new StringBuffer();
StringBuilder stringBuilder = new StringBuilder();

new Thread(() -> {
    for(int i=0; i<10000; i++) {
        stringBuffer.append(i);
        stringBuilder.append(i);
    }
}).start();

new Thread(() -> {
    for(int i=0; i<10000; i++) {
        stringBuffer.append(i);
        stringBuilder.append(i);
    }
}).start();

new Thread(() -> {
    try {
        Thread.sleep(5000);

        System.out.println("StringBuffer.length: "+ stringBuffer.length());
        System.out.println("StringBuilder.length: "+ stringBuilder.length());
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}).start();

결과: 
    StringBuffer.length: 77780
    StringBuilder.length: 76412
```

---
### References
---
- [http://docs.oracle.com/javase/8/docs/api/](http://docs.oracle.com/javase/8/docs/api/)
- [http://javahungry.blogspot.com/2013/06/difference-between-string-stringbuilder.html](http://javahungry.blogspot.com/2013/06/difference-between-string-stringbuilder.html)
