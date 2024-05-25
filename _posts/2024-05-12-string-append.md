---
title: "String 덧셈 연산의 성능 비교"
description: 'java 버전에 따른 덧셈 연산 처리 및 StringBuffer, StringBuilder 성능 차이 및 동기화 오류'
author: illdangag

# permalink: thread-pool
date: 2024-05-12T00:00:00+09:00
last_modified_at: 2024-05-12T00:00:00+09:00

categories: [Java, String]
tags: [Java, String]
---

java에서는 String 덧셈 연산에 대한 성능에 대한 이야기가 많다

> 단순 문자열 상수를 여러번 덧셈 하는 것은 성능상으로 불리하니 StringBuffer 또는 StringBuilder class를 사용하여 처리하는 것이 성능상에 이점이 있다

널리 알려진 String 덧셈 연산에 대한 이야기로는 위와 같은 이야기가 있다

그리고 위 이야기에 추가 의견으로 아래와 같은 이야기가 있다

> java에서 compile 하는 과정에서 '+' 연산자를 사용하여 문자열 덧셈 연산하여도 상수 또는 StringBuilder class를 사용한 코드로 변환된다

예제에 사용된 전체 소스 코드는 하단의 Repository 링크를 참고하면 된다

- [illdangag/gitbook-code](https://github.com/illdangag/gitbook-code/tree/main/java-sample-code/src/main/java/com/illdangag/string)

## Java 버전에 따른 덧셈 연산 처리

간단한 String 덧셈 관련 셈플 코드를 java 버전에 따라 compile 후에 byte code를 확인한다

java 버전은 LTS 기준으로 선정 하였다

```java
public class StringMain {
    public static void main(String[] args) {
        String text00 = "TEXT00";
        String text01 = "TEXT01";
        String text02 = "TEXT02";
        String text03 = text00 + text01 + text02 + "TEST03";
        String text04 = "TEST" + "0" + "4";
    }
}
```
{: file='StringMain.java'}

```shell
javac StringMain.java
javap -v StringMain
```

javac로 compile 하고 javap로 byte code를 확인한다

### **Java 1.8**

상세 버전: corretto-1.8.0_372 (ARM Build)

```
{
  public com.illdangag.string.process.StringMain();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=6, args_size=1
         0: ldc           #2                  // String TEXT00
         2: astore_1
         3: ldc           #3                  // String TEXT01
         5: astore_2
         6: ldc           #4                  // String TEXT02
         8: astore_3
         9: new           #5                  // class java/lang/StringBuilder
        12: dup
        13: invokespecial #6                  // Method java/lang/StringBuilder."<init>":()V
        16: aload_1
        17: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        20: aload_2
        21: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        24: aload_3
        25: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        28: ldc           #8                  // String TEST03
        30: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        33: invokevirtual #9                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        36: astore        4
        38: ldc           #10                 // String TEST04
        40: astore        5
        42: return
      LineNumberTable:
        line 5: 0
        line 6: 3
        line 7: 6
        line 8: 9
        line 9: 38
        line 10: 42
}
```
{: file='byte code'}

test00, test01, test02 변수와 "TEST03" 상수를 더하는 연산은 StringBuilder class의 append method로 변환된 것을 확인 할 수 있다

test04의 "TEST" + "0" + "4" 연산은 "TEST04" 상수 하나를 assign 하는 것으로 변환 된 것을 확인 할 수 있다

### Java 11

상세 버전: corretto-11.0.20.1 (ARM Build)

```
{
  public com.illdangag.string.process.StringMain();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=6, args_size=1
         0: ldc           #2                  // String TEXT00
         2: astore_1
         3: ldc           #3                  // String TEXT01
         5: astore_2
         6: ldc           #4                  // String TEXT02
         8: astore_3
         9: aload_1
        10: aload_2
        11: aload_3
        12: invokedynamic #5,  0              // InvokeDynamic #0:makeConcatWithConstants:(Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;
        17: astore        4
        19: ldc           #6                  // String TEST04
        21: astore        5
        23: return
      LineNumberTable:
        line 5: 0
        line 6: 3
        line 7: 6
        line 8: 9
        line 9: 19
        line 10: 23
}
```
{: file='byte code'}

test00, test01, test02 변수와 "TEST03" 상수를 더하는 연산은 makeConcatWithConstants을 이용하여 연산을 하도록 하고 있다

test04의 "TEST" + "0" + "4" 연산은 "TEST04" 상수 하나를 assign 하는 것으로 변환 된 것을 확인 할 수 있다

### Java 17

상세 버전: corretto-17.0.4 (ARM Build)

```
{
  public com.illdangag.string.process.StringMain();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=6, args_size=1
         0: ldc           #7                  // String TEXT00
         2: astore_1
         3: ldc           #9                  // String TEXT01
         5: astore_2
         6: ldc           #11                 // String TEXT02
         8: astore_3
         9: aload_1
        10: aload_2
        11: aload_3
        12: invokedynamic #13,  0             // InvokeDynamic #0:makeConcatWithConstants:(Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;
        17: astore        4
        19: ldc           #17                 // String TEST04
        21: astore        5
        23: return
      LineNumberTable:
        line 5: 0
        line 6: 3
        line 7: 6
        line 8: 9
        line 9: 19
        line 10: 23
}
```
{: file='byte code'}

test00, test01, test02 변수와 "TEST03" 상수를 더하는 연산은 makeConcatWithConstants을 이용하여 연산을 하도록 하고 있다

test04의 "TEST" + "0" + "4" 연산은 "TEST04" 상수 하나를 assign 하는 것으로 변환 된 것을 확인 할 수 있다

## 결론

> An implementation may choose to perform conversion and concatenation in one step to avoid creating and then discarding an intermediate `String` object. To increase the performance of repeated string concatenation, a Java compiler may use the `StringBuffer` class or a similar technique to reduce the number of intermediate `String` objects that are created by evaluation of an expression.

[Chapter 15. Expressions](https://docs.oracle.com/javase/specs/jls/se7/html/jls-15.html#jls-15.18.1)

Oracle의 java 문서에서 String 객체가 비효율적으로 생성되고 삭제하는 것을 방지하기 위해서 java compiler가 StringBuffer나 그와 비슷한 방법을 이용하여 String 간의 연산 과정에서 생성되는 String 객체 수를 줄일 수 있다 라고 되어 있다

간단한 String 덧셈 연산에 대해서는 java 버전에 따라 다를 수 있지만 java compiler가 StringBuffer 또는 그와 비슷한 방법으로 처리 하는 것을 확인 할 수 있다

java compiler가 명확하게 String 덧셈 연산을 개선 할 수 있는 경우에는 코드의 가독성을 생각하여 일반적은 덧셈 연산을 사용하고 반복문 또는 분기문을 사용하는 경우에는 명확하게 StringBuffer 또는 StringBuilder를 사용 하도록 한다
