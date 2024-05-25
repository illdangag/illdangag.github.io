---
title: "Thread pool"
description: 'java의 thread pool 예제'
author: illdangag

# permalink: thread-pool
date: 2024-05-11T00:01:00+09:00
last_modified_at: 2024-05-11T00:01:00+09:00

categories: [Java, Thread]
tags: [Java, Thread]
---

생성 소멸하는 과정 등에 컴퓨터 자원이 많이 들어가는 리소스가 반복적으로 사용되는 경우 전체적인 성능 저하로 이어질 수 있다

일정 갯수의 리소스를 미리 만들어 놓고 리소스를 사용 해야 하는 작업이 필요한 경우 유휴 상태의 리소스로 작업을 처리하는 방법을 pool이라고 한다

예제에 사용된 전체 소스 코드는 하단의 Repository 링크를 참고하면 된다

- [gitbook-code/java-sample-code](https://github.com/illdangag/gitbook-code/tree/main/java-sample-code/src/main/java/com/illdangag/thread)

## Executors

보편적으로 많이 사용하는 thread pool 종류를 쉽게 사용 할 수 있도록 ExecutorService를 반환하여 준다

### **Thread pool의 종류**

#### newSingleThreadExecutor

하나의 thread를 가진 pool

thread 작업 실행중에 오류가 발생 하여 thread가 비정상 종료 된다면 다음 작업 실행시 새로운 thread가 생성 되어 작업을 처리한다

2개 이상의 thread를 가지지 않는 것이 보장된다

#### newFixedThreadPool

특정 개수의 thread를 가진 pool

pool이 종료될 때 까지 정해진 수의 thread를 유지한다

#### newCachedThreadPool

작업이 필요할 때 마다 새로운 thread를 생성 하여 작업한다

일정 시간 유휴 상태가 유지된 thread는 종료된다

thread pool이 생성된 직후에는 thread의 갯수가 0이며 작업이 실행 될 때 마다 thread의 갯수가 늘어나고 thread pool을 오랫동안 사용하지 않으면 thread의 갯수가 0이 된다

### ExecutorService

Executors를 통해 생성한 thread pool이다

#### 작업 실행 및 관리 method

#### execute

Runnable interface를 구현한 객체를 매개 변수로 전달 받으며 작업 실행에 대한 결과를 반환 받을 수 없다

#### submit

Runnable과 Callable interface를 구현한 객체를 매개 변수로 전달 받으며 작업 실행에 대한 결과를 반환 받을 수 있다

#### shutdown

현재까지 등록된 작업이 종료되면 thread pool을 종료하게 한다

shutdown method가 호출된 이후에는 thread pool에 submit을 할 수 없게 된다

#### awaitTermination

shutdown method는 non-blocking이므로 thread pool에 등록된 모든 작업이 종료 여부를 shutdown method로는 알기 쉽지 않다

awaitTermination method를 호출하여 해당 thread pool에 등록된 모든 작업이 종료 될 때까지 blocking 할 수 있다

#### shutdownNow

thread pool의 모든 thread에 interrupt를 발생시켜 thread를 종료 시도 한다

## 예제 코드

```java
public class Job implements Runnable {
    private String name;
    private boolean throwException;

    public Job(String name, boolean throwException) {
        this.name = name;
        this.throwException = throwException;
    }

    @Override
    public void run() {
        long wait = (long) (Math.random() * 50);
        String threadName = Thread.currentThread().getName();

        System.out.printf("START: [%s, %s, %2dms]\n", this.name, threadName, wait);

        try {
            Thread.sleep(wait);
        } catch (Exception exception) {}

        if (this.throwException) {
            System.out.printf("THROW: [%s, %s]\n", this.name, threadName);
            throw new RuntimeException("Throw exception");
        }

        System.out.printf("END  : [%s, %s]\n", this.name, threadName);
    }
}
```
{: file='Job.java'}

thread를 테스트 하기 위한 Runnable 구현체이다

### newSingleThreadExecutor

```java
import java.util.Arrays;
import java.util.List;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class ThreadPoolMain {
    public static void main(String[] args) throws Exception {
        List<Job> jobList = Arrays.asList(
                new Job("JOB 0", false),
                new Job("JOB 1", false),
                new Job("JOB 2", true),
                new Job("JOB 3", false),
                new Job("JOB 4", false)
        );

        // 하나의 thread를 가진 pool
        ExecutorService executorService = Executors.newSingleThreadExecutor();

        for (Job job : jobList) {
            executorService.submit(job);
        }

        executorService.shutdown();
        executorService.awaitTermination(Long.MAX_VALUE, TimeUnit.MILLISECONDS);
    }
}
```
{: file='ThreadPoolMain.java'}

```
START: [JOB 0, pool-1-thread-1, 49ms]
END  : [JOB 0, pool-1-thread-1]
START: [JOB 1, pool-1-thread-1,  5ms]
END  : [JOB 1, pool-1-thread-1]
START: [JOB 2, pool-1-thread-1, 21ms]
THROW: [JOB 2, pool-1-thread-1]
START: [JOB 3, pool-1-thread-1, 40ms]
END  : [JOB 3, pool-1-thread-1]
START: [JOB 4, pool-1-thread-1,  2ms]
END  : [JOB 4, pool-1-thread-1]
```
{: file='output'}

newSingleThreadExecutor를 이용하여 1개의 thread를 가진 pool을 생성하고 5개의 작업을 등록하여 실행한다

1개의 thread로 작업을 처리 하므로 작업이 순차적으로 실행 된다

### newFixedThreadPool

```java
import java.util.Arrays;
import java.util.List;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class ThreadPoolMain {
    public static void main(String[] args) throws Exception {
        List<Job> jobList = Arrays.asList(
                new Job("JOB 0", false),
                new Job("JOB 1", false),
                new Job("JOB 2", true),
                new Job("JOB 3", false),
                new Job("JOB 4", false)
        );

        // 3개의 thread를 가진 pool
        ExecutorService executorService = Executors.newFixedThreadPool(3);

        for (Job job : jobList) {
            executorService.submit(job);
        }

        executorService.shutdown();
        executorService.awaitTermination(Long.MAX_VALUE, TimeUnit.MILLISECONDS);
    }
}
```
{: file='ThreadPoolMain.java'}

```
START: [JOB 2, pool-1-thread-3, 13ms]
START: [JOB 0, pool-1-thread-1, 49ms]
START: [JOB 1, pool-1-thread-2, 12ms]
END  : [JOB 1, pool-1-thread-2]
THROW: [JOB 2, pool-1-thread-3]
START: [JOB 3, pool-1-thread-2, 47ms]
START: [JOB 4, pool-1-thread-3,  8ms]
END  : [JOB 4, pool-1-thread-3]
END  : [JOB 0, pool-1-thread-1]
END  : [JOB 3, pool-1-thread-2]
```
{: file='output'}

newFixedThreadPool으로 3개의 thread를 가진 pool을 생성하고 5개의 작업을 등록하여 실행한다

우선 3개의 작업이 먼저 실행되며, 먼저 종료된 2번 thread가 3번 작업을 실행하고 그 다음 종료된 3번 thread가 4번 작업을 실행한다

### newCachedThreadPool

```java
import java.util.Arrays;
import java.util.List;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class ThreadPoolMain {
    public static void main(String[] args) throws Exception {
        List<Job> jobList = Arrays.asList(
                new Job("JOB 0", false),
                new Job("JOB 1", false),
                new Job("JOB 2", true),
                new Job("JOB 3", false),
                new Job("JOB 4", false)
        );

        // 작업이 등록될 때 새로운 thread를 생성하고 일정시간 사용하지 않으면 thread를 제거하는 pool
        ExecutorService executorService = Executors.newCachedThreadPool();

        for (Job job : jobList) {
            executorService.submit(job);
        }

        executorService.shutdown();
        executorService.awaitTermination(Long.MAX_VALUE, TimeUnit.MILLISECONDS);
    }
}
```
{: file='ThreadPoolMain.java'}

```
START: [JOB 2, pool-1-thread-3, 46ms]
START: [JOB 3, pool-1-thread-4, 11ms]
START: [JOB 4, pool-1-thread-5, 11ms]
START: [JOB 1, pool-1-thread-2, 38ms]
START: [JOB 0, pool-1-thread-1, 24ms]
END  : [JOB 3, pool-1-thread-4]
END  : [JOB 4, pool-1-thread-5]
END  : [JOB 0, pool-1-thread-1]
END  : [JOB 1, pool-1-thread-2]
THROW: [JOB 2, pool-1-thread-3]
```
{: file='output'}

newCachedThreadPool으로 최초에는 thread가 존재하지 않으나, 작업이 등록 되면 thread를 새로 생성 하여 작업을 실행한다

최대 thread 갯수는 Integer.MAX_VALUE 크기 만큼 생성 될 수 있으며, thread 별로 60초 이상 사용되지 않으면 해당 thread는 제거된다

## 결론

thread를 사용하는 경우에는 thread에서 작업 해야 할 비즈니스 로직에 대해서  thread safe에 관한 영향도를 고려 해야 하며 thread를 사용하야 할 만큼 작업의 양이 많은지 또는 운용중인 서버의 사양에 대해서도 고려 하여 적절한 전략을 세워야 한다
