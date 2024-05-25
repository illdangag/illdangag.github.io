---
title: "Thread, Runnable"
description: 'java의 thread 기본 구현'
author: illdangag

date: 2024-05-11T00:00:00+09:00
last_modified_at: 2024-05-11T00:00:00+09:00

categories: [Java, Thread]
tags: [Java, Thread]
---

다중 thread를 사용하여 작업을 하기 위해선 특정 class를 구현 하여야 한다

예제에 사용된 전체 소스 코드는 하단의 Repository 링크를 참고하면 된다

- [gitbook-code/java-sample-code](https://github.com/illdangag/gitbook-code/tree/main/java-sample-code/src/main/java/com/illdangag/thread)


## Thread와 Runnable 구현
java에서 다중 thread를 사용하기 위해서는 Thread 또는 Runnable을 구현 해야 한다

### 예제 코드

```java
public class ThreadMain {
    public static void main(String[] args) {
        String threadName = Thread.currentThread().getName();
        System.out.println("Main thread name: " + threadName);

        Thread thread = new MyThread();
        thread.start();

        Runnable runnable = new MyRunnable();
        Thread runableThread = new Thread(runnable);
        runableThread.start();
    }
}

class MyThread extends Thread {
    @Override
    public void run() {
        String threadName = Thread.currentThread().getName();
        System.out.println("MyThread thread name: " + threadName);
    }
}

class MyRunnable implements Runnable {
    @Override
    public void run() {
        String threadName = Thread.currentThread().getName();
        System.out.println("MyRunnable thread name: " + threadName);
    }
}
```
{: file='ThreadMain.java'}

```
Main thread name: main
MyThread thread name: Thread-0
MyRunnable thread name: Thread-1
```
{: file='output'}

Thread class도 Runnable interface를 구현하고 있으므로 Thread와 Runnable 둘다 run method를 구현해야 한다

## 결론

특정 비즈니스 로직을 수행하기 위해서 다른 class를 상속하여 사용해야 하는 경우 단일 상속만 지원하는 java에서는 Thread class를 같이 상속하여 사용 할 수 없으므로 대부분 Runnable interface를 구현하여 thread를 사용한다
