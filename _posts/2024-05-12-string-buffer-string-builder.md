---
title: "StringBuffer, StringBuilder의 차이"
description: ''
author: illdangag

# permalink: thread-pool
date: 2024-05-12T00:01:00+09:00
last_modified_at: 2024-05-12T00:01:00+09:00

categories: [Java, String]
tags: [Java, String]
---

StringBuffer와 StringBuilder는 String 관련 연산시 String 객체가 많이 생성 되는 것을 방지하기 위해서 사용한다

StringBuilder와 StringBuffer의 차이는 다중 thread에 대해서 안전한지 아닌지의 차이가 있다

예제에 사용된 전체 소스 코드는 하단의 Repository 링크를 참고하면 된다

[illdangag/gitbook-code](https://github.com/illdangag/gitbook-code/tree/main/java-sample-code/src/main/java/com/illdangag/string)

## 단일 또는 다중 Thread 간의 차이

### 단일 thread에서의 성능

```java
public class BuilderBufferMain {
    public static void main(String[] args) {
        int range = 1_000_000; // 100_000, 1_000_000, 10_000_000, 100_000_000
        int repeat = 5;

        long startTime = 0;
        long endTime = 0;
        long stringExecuteTime = 0;
        long bufferExecuteTime = 0;
        long builderExecuteTime = 0;

        for (int repeatIndex = 0; repeatIndex < repeat; repeatIndex++) {
            startTime = System.nanoTime();
            String string = "";
            for (int index = 0; index < range; index++) {
                string += "O";
            }
            endTime = System.nanoTime();
            stringExecuteTime += (endTime - startTime);
            System.out.println("string length: " + string.length());

            startTime = System.nanoTime();
            StringBuffer stringBuffer = new StringBuffer();
            for (int index = 0; index < range; index++) {
                stringBuffer.append("O");
            }
            String bufferResult = stringBuffer.toString();
            endTime = System.nanoTime();
            bufferExecuteTime += (endTime - startTime);
            System.out.println("StringBuffer length: " + bufferResult.length());

            startTime = System.nanoTime();
            StringBuilder stringBuilder = new StringBuilder();
            for (int index = 0; index < range; index++) {
                stringBuilder.append("O");
            }
            String builderResult = stringBuilder.toString();
            endTime = System.nanoTime();
            builderExecuteTime += (endTime - startTime);
            System.out.println("StringBuilder length: " + builderResult.length());
        }

        stringExecuteTime /= repeat;
        bufferExecuteTime /= repeat;
        builderExecuteTime /= repeat;

        System.out.printf("String: %f ms\n", (stringExecuteTime / 1000000D));
        System.out.printf("StringBuffer: %f ms\n", (bufferExecuteTime / 1000000D));
        System.out.printf("StringBuilder: %f ms\n", (builderExecuteTime / 1000000D));
    }
}
```
{: file='BuilderBufferMain.java'}

```
string length: 1000000
StringBuffer length: 1000000
StringBuilder length: 1000000
string length: 1000000
StringBuffer length: 1000000
StringBuilder length: 1000000
string length: 1000000
StringBuffer length: 1000000
StringBuilder length: 1000000
string length: 1000000
StringBuffer length: 1000000
StringBuilder length: 1000000
string length: 1000000
StringBuffer length: 1000000
StringBuilder length: 1000000
String: 23408.821983 ms
StringBuffer: 2.765758 ms
StringBuilder: 2.229525 ms
```
{: file='output'}

StringBuffer, StringBuilder 모두 String 덧셈 연산에 비해서는 성능이 아주 좋게 측정 되었다

단일 thread 환경에서는 StringBuffer와 StringBuilder 모두 오류가 발생하지 않고 문자열이 유실 없이 처리 되었다

다만 동기화 기능에 의해서 StringBuffer보다 StringBuilder가 조금 더 빠르게 측정 되었다

### 다중 thread에서의 안정성

#### StringBuffer

```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class StringBufferMultiThread {
    public static void main(String[] args) throws Exception {
        int range = 10_000_000; // 100_000, 1_000_000, 10_000_000, 100_000_000
        int threadSize = 20;
        int threadJobRange = range / threadSize;

        StringBuffer stringBuffer = new StringBuffer();

        ExecutorService executorService = Executors.newFixedThreadPool(threadSize);
        List<Future> futureList = new ArrayList<>(threadSize);

        for (int threadIndex = 0; threadIndex < threadSize; threadIndex++) {
            Future future = executorService.submit(() -> {
                for (int index = 0; index < threadJobRange; index++) {
                    stringBuffer.append("O");
                }
            });
            futureList.add(future);
        }

        for (int threadIndex = 0; threadIndex < threadSize; threadIndex++) {
            futureList.get(threadIndex).get();
        }
        executorService.shutdown();
        String bufferResult = stringBuffer.toString();
        System.out.println("StringBuffer length: " + bufferResult.length());
    }
}
```
{: file='StringBufferMultiThread.java'}

```
StringBuffer length: 10000000
```
{: file='output'}

StringBuffer는 다중 thread 환경에서도 오류 없이 실행이 된다

#### StringBuilder

```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class StringBuilderMultiThread {
    public static void main(String[] args) throws Exception {
        int range = 10_000_000; // 100_000, 1_000_000, 10_000_000, 100_000_000
        int threadSize = 20;
        int threadJobRange = range / threadSize;

        StringBuilder stringBuilder = new StringBuilder();

        ExecutorService executorService = Executors.newFixedThreadPool(threadSize);
        List<Future> futureList = new ArrayList<>(threadSize);

        for (int threadIndex = 0; threadIndex < threadSize; threadIndex++) {
            Future future = executorService.submit(() -> {
                for (int index = 0; index < threadJobRange; index++) {
                    stringBuilder.append("O");
                }
            });
            futureList.add(future);
        }

        for (int threadIndex = 0; threadIndex < threadSize; threadIndex++) {
            futureList.get(threadIndex).get();
        }
        executorService.shutdown();
        String bufferResult = stringBuilder.toString();
        System.out.println("StringBuilder length: " + bufferResult.length());
    }
}
```
{: file='StringBuilderMultiThread.java'}

```
Exception in thread "main" java.util.concurrent.ExecutionException: java.lang.ArrayIndexOutOfBoundsException: arraycopy: last destination index 4607 out of bounds for byte[4606]
	at java.base/java.util.concurrent.FutureTask.report(FutureTask.java:122)
	at java.base/java.util.concurrent.FutureTask.get(FutureTask.java:191)
	at com.illdangag.string.StringBuilderMultiThread.main(StringBuilderMultiThread.java:30)
Caused by: java.lang.ArrayIndexOutOfBoundsException: arraycopy: last destination index 4607 out of bounds for byte[4606]
	at java.base/java.lang.System.arraycopy(Native Method)
	at java.base/java.lang.String.getBytes(String.java:3192)
	at java.base/java.lang.AbstractStringBuilder.putStringAt(AbstractStringBuilder.java:1667)
	at java.base/java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:539)
	at java.base/java.lang.StringBuilder.append(StringBuilder.java:178)
	at com.illdangag.string.StringBuilderMultiThread.lambda$main$0(StringBuilderMultiThread.java:23)
	at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
	at java.base/java.lang.Thread.run(Thread.java:829)
Caused by: java.lang.ArrayIndexOutOfBoundsException: arraycopy: last destination index 4607 out of bounds for byte[4606]
```
{: file='output'}

StringBuilder는 StringBuffer와 다르게 append method에 synchronized 키워드를 사용하지 않았기에 다중 thread에서 위와 같은 오류가 발생 할 수 있다

## 결론

> A mutable sequence of characters. This class provides an API compatible with StringBuffer, but with no guarantee of synchronization. This class is designed for use as a drop-in replacement for StringBuffer in places where the string buffer was being used by a single thread (as is generally the case). Where possible, it is recommended that this class be used in preference to StringBuffer as it will be faster under most implementations.

java 11에서 StringBuilder javadoc의 일부이다

StringBuilder는 단일 thread 환경에서 StringBuffer보다 대부분의 상황에서 성능이 더 좋으므로 단일 thread 환경에서는 StringBuilder를 우선적으로 사용하는 것을 추천하고 있다

> String buffers are safe for use by multiple threads.
> 
> As of release JDK 5, this class has been supplemented with an equivalent class designed for use by a single thread, StringBuilder. The StringBuilder class should generally be used in preference to this one, as it supports all of the same operations but it is faster, as it performs no synchronization.

java 11에서 StringBuffer javadoc의 일부이다

StringBuffer는 다중 thread에서 사용 할 수 있도록 설계 되었으며, 단일 thread 환경에서 StringBuffer를 사용하기 보다는 단일 thread 환경으로 고려하여 설계된 StringBuilder를 사용하도록 추천하고 있다

직접 성능을 측정해 보았을 때 StringBuilder와 StringBuffer 사이의 성능 차이는 String의 덧셈 연산과의 성능 차이에 비해서는 많은 차이는 발생하지 않았지만, 실행 환경이 단일 thread인지 다중 thread인지 판단 하여 그에 맞는 기능을 사용하도록 한다
