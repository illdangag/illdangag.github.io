---
title: "Singleton 디자인 패턴"
description: "하나의 객체만 존재 하여야하는 상황에서 사용하는 디자인패턴"
author: illdangag

mermaid: true

# permalink: docker
date: 2024-08-11T16:18:00+09:00
last_modified_at: 2024-08-11T16:18:00+09:00

categories: [Design Pattern]
tags: [Singleton]
---

디자인 패턴중 singleton 패턴은 application에서 생성되는 객체를 특정 class에 대해서 하나의 객체만 생성 되어야 하는 경우에 사용한다

- singleton 패턴 - [https://ko.wikipedia.org/wiki/싱글턴_패턴](https://ko.wikipedia.org/wiki/%EC%8B%B1%EA%B8%80%ED%84%B4_%ED%8C%A8%ED%84%B4)

예를 들어 application에서 DB에 접속을 담당하는 connection pool과 같은 반복되는 작업에 대해서 무분별하게 실행되지 않도록 작업의 수행 횟수를 조절하는 경우에 주로 사용된다

## Singleton 특징

### 객체의 생성 방법

singleton 패턴으로 이루어진 객체는 개발자가 직접 객체를 생성할 수 없고 객체를 얻기 위한 별도의 method를 호출 해야 한다

### 다중 thread 환경에서 문제

singleton 패턴의 구현 방법에 따라 thread safe 하지 않을 수 있다

singleton 객체가 생성되는 과정에서 비즈니스 로직이 실행 되는 시간 보다 singleton 객체 요청 method가 빠르게 여러번 호출 된다면 singleton 패턴의 의도와 다르게 객체가 여러개 생성 된다

## 구현

### Thread safe하지 않은 singleton

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {
        System.out.println("Singleton 객체 생성");
        // 객체의 초기화 비즈니스 로직
        try {
            Thread.sleep(1000);
        } catch (Exception exception) {}
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
{:file='Singleton.java'}

singleton 객체의 생성 시점에 특정 비즈니스 로직에 의해서 어느정도 시간이 소요되는 것을 가정한다

```java
/**
 * single thread 환경에서의 singleton 테스트
 */
public class SingletonSingleThreadMain {
    public static void main(String[] args) {
        Singleton singleton00 = Singleton.getInstance();
        Singleton singleton01 = Singleton.getInstance();

        System.out.println("singleton00 hashCode: " + singleton00.hashCode());
        System.out.println("singleton01 hashCode: " + singleton01.hashCode());
    }
}
```
{:file='SingletonSingleThreadMain.java'}

```text
Singleton 객체 생성
singleton00 hashCode: 474675244
singleton01 hashCode: 474675244
```
{:file='output'}

단일 thread 환경에서는 `Singleton.getInstance` 메서드가 순차적으로 호출되기에 `Singleton` 객체는 의도한대로 하나의 객체만 생성 되고 나중에 호츨한 `Singleton.getInstance` 메서드에서는 직전에 생성한 객체가 반환된다

```java
/**
 * multi thread 환경에서의 singleton 테스트
 */
public class SingletonMultiThreadMain {
    public static void main(String[] args) {
        Thread thread00 = new Thread(() -> {
            Singleton singleton00 = Singleton.getInstance();
            System.out.println("singleton00 hashCode: " + singleton00.hashCode());
        });

        Thread thread01 = new Thread(() -> {
            Singleton singleton01 = Singleton.getInstance();
            System.out.println("singleton01 hashCode: " + singleton01.hashCode());
        });

        thread00.start();
        thread01.start();
    }
}
```
{:file='SingletonMultiThreadMain.java'}

```text
Singleton 객체 생성
Singleton 객체 생성
singleton00 hashCode: 1887756893
singleton01 hashCode: 1907500473
```

그러나 다중 thread 환경에서는 `Singleton.getInstance` 메서드가 호출후에 메서드의 종료가 `Singleton.getInstance`를 다시 호출하게 되고 `Singleton` 객체에 `static` 선언된 `instance` 변수에 할당 되기 전이므로 새로운 `Singleton` 객체를 생성하게 된다

### Thread safe한 singleton

#### Synchronized, Volatile

```java
public class SynchronizedSingleton {
    private static volatile SynchronizedSingleton instance;

    private SynchronizedSingleton() {
        System.out.println("Singleton 객체 생성");
        // 객체의 초기화 비즈니스 로직
        try {
            Thread.sleep(1000);
        } catch (Exception exception) {}
    }

    public static SynchronizedSingleton getInstance() {
        if (instance != null) {
            return instance;
        }

        synchronized (SynchronizedSingleton.class) {
            if (instance == null) {
                instance = new SynchronizedSingleton();
            }
            return instance;
        }
    }
}
```
{:file='SynchronizedSingleton.java'}

```text
Singleton 객체 생성
singleton01 hashCode: 640253301
singleton00 hashCode: 640253301
```
{:file='output'}

다중 thread 환경에서 thread safe하도록 수정한 `SynchronizedSingleton` 클래스이다

thread safe에 만족하기 위해서 두가지 키워드를 사용하였다

- volatile  
  변수에 `volatile` 키워드를 추가하면 변수를 항상 메인 메모리에 할당하여 사용하겠다는 의미이다  
  thread safe를 위해서 `volatile`를 사용하는 이유는 다중 thread 환경에서는 2개 이상의 CPU를 사용할 수 있는데 각 CPU의 cache 메모리가 존재하고 `volatile`를 사용하지 않으면 변수의 접근을 cache 메모리를 사용하게 되므로 `stiatc`으로 선언된 변수임에도 내용이 어긋날 가능성이 있기 때문에 `volatile`를 사용해서 항상 메인 메모리로 접근하게 하기 위함이다 

- synchronized  
  특정 비즈니스 로직을 동시에 실행하지 못하도록 동기화한다  
  클래스 단위로 동기화 하여 다중 thread 환경에서도 해당 비즈니스 로직이 동시에 실행되지 않게 하여 instance가 동시에 여러개가 생성되지 않도록 한다

#### Bill pugh 

```java
public class InnerClassSingleton {
    private InnerClassSingleton() {
        System.out.println("Singleton 객체 생성");
        // 객체의 초기화 비즈니스 로직
        try {
            Thread.sleep(1000);
        } catch (Exception exception) {}
    }

    private static class SingletonHolder {
        private static final InnerClassSingleton INSTANCE = new InnerClassSingleton();
    }

    public static InnerClassSingleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```
{:file='InnerClassSingleton.java'}

```text
Singleton 객체 생성
singleton00 hashCode: 1853204767
singleton01 hashCode: 1853204767
```
{:file='output'}

`InnerClassSingleton` 클래스는 `SynchronizedSingleton` 클래스의 단점을 보완한 thread safe한 singleton 객체이다

`SynchronizedSingleton` 클래스에서 사용한 `synchronized`는 다중 thread 환경에서 순차적인 처리를 강제하므로 병목 현상이 발생하므로 성능 하락의 문제가 발생한다

`static`으로 선언된 내부 클래스를 사용한 bill pugh 방법으로 구현한 singleton 객체는 첫번째 thread가 `InnerClassSingleton.getInstance`가 호출하면 `static` 내부 클래스가 생성될 때 까지 JVM이 두번째 `InnerClassSingleton.getInstance` 호출을 대기시키고 첫번째 thread에 호출이 종료되면 하나의 instance가 생성되고 두번째 thread에서 호출을 첫번째 thread에서 생성된 instance를 반환하게 된다

## 결론

application 전체에서 하나의 객체만을 생성하게 하는 디자인 패턴으로 객채 생성 비용을 절약하며 특정 비즈니스 로직에 대해서 유효성을 보장할 수 있지만, 해당 조건을 만족하기 위해서는 다중 thread 환경을 고려하여 구현 하여야 한다

