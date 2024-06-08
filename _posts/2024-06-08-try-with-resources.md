---
title: "try-with-resources 자원 해제 처리"
description: "리소스 처리와 try-catch-finally 구문, try-with-resources 구문의 차이, 그리고 리소스에 대한 예외 처리 "
author: illdangag

# permalink: docker
date: 2024-06-08T15:36:00+09:00
last_modified_at: 2024-06-08T15:36:00+09:00

categories: [Java, Try with resources]
tags: [Java, Try with resources]
---

예제에 사용된 소스 코드는 하단의 Repository 링크를 참고하면 된다

- [illdangag/gitbook-code - try catch resources](https://github.com/illdangag/gitbook-code/tree/main/try-catch-resources)

## 리소스

### 리소스 종류 및 파일 핸들러

java에서는 메모리, 파일, 네트워크 등 여러가지 리소스를 다룰 수 있다

리소스를 다루는데 가장 중요한 점은 사용하는 리소스에 대해서 맺고 끊음이 분명 해야 한다

파일 리소스에 접근하는 경우 OS에 설정에 따라서 하나의 프로세스에서 동시에 가질 수 있는 파일의 핸들, 즉 파일의 수가 정해져 있으며 그 이상의 파일을 열 수 없다

실행하는 java 어플리케이션이 실행 후 종료가 된다면 파일 핸들에 대한 오류가 발생하기 쉽지 않지만, 특정 주기로 종료 되지 않고 계속해서 실행되고 있다면 파일을 더 이상 열 수 없는 상황으로 서버의 가용성에 문제가 발생한다

#### 파일 핸들의 수에 대한 설정 확인

```shell
ulimit -n
256

ulimit -u
2666
```

`ulimit -n` 명령어로 하나의 프로세스에서 가질 수 있는 파일 핸들의 수, `ulimit -u` 명령어로 하나의 사용자가 가질 수 있는 프로세스의 수를 구하여 하나의 사용자가 가질 수 있는 파일 핸들의 수를 구할 수 있다

### 리소스 처리

java에서는 리소스 다루는 class에서는 대부분 `Closeable` interface를 구현하고 있다

```java
package java.io;

import java.io.IOException;

public interface Closeable extends AutoCloseable {
    public void close() throws IOException;
}
```
{:file='Closeable.java'}

`Closeable` interface에서는 `close` 메서드를 정의하고 있으며, 파일 관련 리소스에서는 파일 핸들러를 반납하고, 네트워크 관련 리소스에서는 원격 서버에 대한 요청의 종료 처리를 한다

`Closeable` interface의 `close` 메서드에서는 `IOException` 예외를 처리하도록 한다

개발자는 리소스 처리의 마무리는 `close` 메서드의 예외 처리이다

### 예외 처리

java 어플리케이션에서 기본적인 예외 처리 구문으로는 `try-catch-finally` 구문이 있다

`try` 블록에서 비즈니스 로직을 처리하는 도중에 예외가 발생하는 경우에 `catch` 블록과 `finally` 블록에서 예외에 대한 대응과 예외 발생과 무관하게 반드시 실행이 필요한 작업을 처리 할 수 있다

java 1.7부터 `try-with-resources`구문을 사용해서 `Closeable` interface를 구현하고 있는 class에 대한 close 처리를 `try-catch-finally` 구문 보다 간편하게 할 수 있다

## 예제

### Closeable 구현 class

```java
package com.illdangag;

import java.io.Closeable;
import java.io.IOException;

public class ThrowClose implements Closeable {
    public void process(boolean isThrowException) throws Exception {
        System.out.println("ThrowClose.process() called");
        if (isThrowException) {
            throw new IOException("ThrowClose.process() throw exception");
        }
    }

    @Override
    public void close() throws IOException {
        System.out.println("ThrowClose.close() called");
        throw new IOException("ThrowClose.close() throw exception");
    }
}
```
{:file='ThrowClose.java'}
테스트를 위해서 `Closeable` interface를 구현한 `ThrowClose` class를 작성하였다

### try-catch-finally 예제

```java
package com.illdangag;

import java.io.IOException;

public class TryCatchMain {
    public static void main(String[] args) {
        ThrowClose throwClose = new ThrowClose();

        try {
            throwClose.process(true);
        } catch (Exception exception) {
            System.out.println("process exception. message: " + exception.getMessage());
        } finally {
            try {
                throwClose.close();
            } catch (IOException exception){
                System.out.println("close exception. message: " + exception.getMessage());
            }
        }
    }
}
```
{:file='TryCatchMain.java'}

```text
ThrowClose.process() called
process exception. message: ThrowClose.process() throw exception
ThrowClose.close() called
close exception. message: ThrowClose.close() throw exception
```
{:file='output'}

`try-catch-finally` 구문으로 예외를 처리하는 class이다

`process` method도 예외를 전파하고 `close` method로 예외를 전파하기 때문에 위와 같이 `finally` 블록에서도 `try-catch-finally` 구문이 중첩 되었다

### try-with-resources 예제

```java
package com.illdangag;

import java.io.IOException;

public class TryResourceMain {
    public static void main(String[] args) {
        try (ThrowClose throwClose = new ThrowClose()) {
            System.out.println("try block start");

            throwClose.process(true);

            System.out.println("try block end");
        } catch (Exception exception) {
            System.out.println("Exception exception");

            exception.printStackTrace();
        } finally {
            System.out.println("finally block");
        }
    }
}
```
{:file='TryResourceMain.java'}

```text
try block start
ThrowClose.process() called
ThrowClose.close() called
Exception exception
finally block

java.io.IOException: ThrowClose.process() throw exception
	at com.illdangag.ThrowClose.process(ThrowClose.java:10)
	at com.illdangag.TryResourceMain.main(TryResourceMain.java:8)
	Suppressed: java.io.IOException: ThrowClose.close() throw exception
		at com.illdangag.ThrowClose.close(ThrowClose.java:17)
		at com.illdangag.TryResourceMain.main(TryResourceMain.java:5)
```
{:file='output'}

`try-with-resources` 구문으로 예외를 처리하는 class이다

`try-catch-finally` 구문과는 다르게 명시적으로 `close` method를 호출하지 않아도 `try` 블록의 구문이 모두 실행 되거나 예외가 발생하는 경우 `close` method를 호출 한다

`process` method도 예외를 전파하고 `close` method로 예외를 전파 하지만 하나의 `catch` 블록으로 예외를 처리 할 수 있다

`process` method와 `close` method에서 모두 예외를 발생 시켰으며 `process` method에서 발생한 예외에 `close` method에서 발생한 예외가 중첩되어 전파된다

## 결론

`Closeable` interface를 구현한 class를 사용하는 경우에는 `try-catch-finally` 구문 보다 `try-with-resources` 구문을 사용을 추천한다

`try-with-resources` 구문에서는 여러개의 리소스를 처리 할 수 있으며 그에 따라 여러개의 `close` method를 개발자가 직접 처리하지 않게 된다

그러므로 리소스를 사용한 후에 필수적인 처리에 대해서 누락하지 않게 되며, 코드에 대한 가독성에도 도움이 된다
