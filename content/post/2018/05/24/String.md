---
title: The difference among String, StringBuilder, and StringBuffer in JAVA
tags: [java, string, stringbuilder, stringbuffer, difference]
date: "2018-05-24T11:30:03+00:00"
aliases: ["/2018/05/24/String.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---
## Summary
---
 If you're a lazy JAVA developer, you concatenate Strings by using the plus sign ("Some text" + " added text")
 But if you want to level-up your skills as a JAVA developer, you should be more careful about the Class that you choose.
 Let's take a glance at the use cases of String, StringBuilder and StringBuffer.


## Feature of Classes
---

It's necessary to be familiar with the [Java API](http://docs.oracle.com/javase/8/docs/api/) documents before getting your hands dirty.

Let's take a look at the java.lang package on [Java API](http://docs.oracle.com/javase/8/docs/api/). Java.lang package has a bundle of classes that does not require you to import them manually. It contains basic classes ('WrapperClass') like Boolean, Byte, and Integer.

Among them, we are going to look into String, StringBuffer, and StringBuilder.

![java.lang](/images/java/string/java.lang.png)

---

String class is inherited from Serializable, CharSequence, Comparable<String> interfaces, and configured as 'public final class'.
In other words, String class is a sequence of characters, a comparable value, and is able to be serialized. Also, it cannot be used for the inherited class.

![String](/images/java/string/String.png)

---

When you look up the use cases of String, you can easily find articles that suggest using StringBuffer or StringBuilder instead of String.

Let's find out why.

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

Results:
    stringValue1: -1823841245
    stringValue2: -1823841244
    stringValue1: 833872391
    sb: 1956725890
    sb: 1956725890
```

On the snippet above, a different reference value is created for each of the instances as a instance is created everytime a new value is assigned to the String. However, StringBuffer appends a string value on memory to concatenate and reuse the instance that was already created.
Logically, Class creates methods and variables on creating an instance, but StringBuffer does not allow this time delay on instance creation.

On the above example, String instance is only created once, but if you concatenate several Strings, its instances stack on heap area of memory and it takes memories until JVM calls the Garbage Collector. This is a critical issue in managing memory of your application.

---

Hmm, why is String instance created every time calling 'new'? Let's look at the structure of String class.

On the following image, you can see an array 'value[]' consist of character type. You should look carefully 'value[]' is 'private final char' type.

A string value is saved as an array of characters, and this private array cannot be accessed from other classes. Also, 'value[]' is a final type, cannot be changed since initialized.


 ![String2](/images/java/string/String2.png)

---

Let's take a look at StringBuilder and StringBuffer that uses 'append' method for concatenation.

On the following image, StringBuilder class is a mutable sequence of characters, but has no guarantee of syncronization.
StringBuffer can also by safely used in multi-thread environments. This is the main difference between StringBuilder and StringBuffer classes.

 ![StringBuilder](/images/java/string/StringBuilder.png)

 ![StringBuffer](/images/java/string/StringBuffer.png)

 ---

On the following sample snippet, results show the difference between the two classes.
The result value of StringBuilder has a smaller value because multiple threads accessed a StringBuilder instance simultaneously.
However, StringBuffer is thread-safe unlike StringBuilder. The StringBuffer instance has a bigger and precise value
on multiple threads environment because StringBuffer supports synchronization.

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

Results: 
    StringBuffer.length: 77780
    StringBuilder.length: 76412
```

---
### References
---
- [http://docs.oracle.com/javase/8/docs/api/](http://docs.oracle.com/javase/8/docs/api/)
- [http://javahungry.blogspot.com/2013/06/difference-between-string-stringbuilder.html](http://javahungry.blogspot.com/2013/06/difference-between-string-stringbuilder.html)
