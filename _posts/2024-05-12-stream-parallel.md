---
title: "Stream 병렬 처리에 대한 성능"
description: '병렬 처리를 진행할 데이터의 양, Collection의 종류, 최종 연산의 종류에 따른 Stream 병렬 처리'
author: illdangag

# permalink: thread-pool
date: 2024-05-12T00:02:00+09:00
last_modified_at: 2024-05-12T00:02:00+09:00

categories: [Java, Thread]
tags: [Java, Thread, Stream]
---

Stream에서는 간단하게 병렬 처리가 가능하다

예제에 사용된 전체 소스 코드는 하단의 Repository 링크를 참고하면 된다

[illdangag/gitbook-code](https://github.com/illdangag/gitbook-code/tree/main/java-sample-code/src/main/java/com/illdangag/stream/performance)

## 병렬 처리 고려 사항

### 데이터의 양

병렬 처리는 즉 multi thread를 사용하여 처리하게 되므로 병렬 처리 하기 위해서 데이터를 나누는 작업과 thread를 관리하는 작업 그리고 작업 결과를 종합하는 작업이 필수적이다

데이터의 양이 많지 않으면 위 작업의 소요시간이 더 오래 걸리게 되므로 오히려 병렬 처리가 더 오래 걸리게 된다

```java
import lombok.extern.slf4j.Slf4j;

import java.util.stream.LongStream;

@Slf4j
public class ItemSizeMain {
    public static void main(String[] args) {
        long startTime;
        long endTime;

        long executeTime = 0;
        long parallelExecuteTime = 0;

        long range = 10_000L; // 10_000L, 100_000L, 1_000_000L, 10_000_000L, 100_000_000L
        int repeat = 5;

        for (int index = 0; index < repeat; index++) {
            startTime = System.nanoTime();
            LongStream.range(0, range).forEach(item -> {});
            endTime = System.nanoTime();
            executeTime += (endTime - startTime);

            startTime = System.nanoTime();
            LongStream.range(0, range).parallel().forEach(item -> {});
            endTime = System.nanoTime();
            parallelExecuteTime += (endTime - startTime);
        }

        executeTime /= repeat;
        parallelExecuteTime /= repeat;

        log.info("Stream execute time: {}ms", executeTime / 1000000D);
        log.info("Parallel stream execute time: {}ms", parallelExecuteTime / 1000000D);
    }
}
```
{: file='ItemSizeMain.java'}

range: 10_000L

```
Stream execute time: 0.306925ms
Parallel stream execute time: 0.626691ms
```
{: file='output'}

range: 100_000_000L

```
Stream execute time: 19.681308ms
Parallel stream execute time: 3.012125ms
```
{: file='output'}

요소의 개수가 많은 경우에는 병렬 처리가 효율적이지만 요소의 개수가 많지 않은 경우에는 오히려 병렬 처리가 더 오래 걸리게 된다

### Collection의 종류

병렬 처리 하려면 stream을 분할하여야 한다

분할 하려는 stream이 LinkedList라면 분할 위치을 찾으려면 처음 요소부터 탐색 해야 하지만 ArrayList인 경우에는 처음 요소부터 탐색하는 과정 없이 분할 위치를 찾을 수 있기 때문에 LinkedList보다 ArrayList가 성능이 좋다

```java
import lombok.extern.slf4j.Slf4j;

import java.util.ArrayList;
import java.util.LinkedList;

@Slf4j
public class CollectionMain {
    public static void main(String[] args) {
        long startTime = 0;
        long endTime = 0;
        long range = 10_000_000L; // 100_000L, 1_000_000L, 10_000_000L, 100_000_000L
        int repeat = 5;

        ArrayList<Long> arrayList = new ArrayList<>();
        LinkedList<Long> linkedList = new LinkedList<>();

        for (long index = 0; index < range; index++) {
            arrayList.add(index);
            linkedList.add(index);
        }

        int arrayListExecuteTime = 0;
        int parallelArrayListExecuteTime = 0;
        int linkedListExecuteTime = 0;
        int parallelLinkedListExecuteTime = 0;

        for (int index = 0; index < repeat; index++) {
            startTime = System.nanoTime();
            arrayList.stream().reduce(0L, Long::sum);
            endTime = System.nanoTime();

            arrayListExecuteTime += endTime - startTime;

            startTime = System.nanoTime();
            arrayList.stream().parallel().reduce(0L, Long::sum);
            endTime = System.nanoTime();

            parallelArrayListExecuteTime += endTime - startTime;

            startTime = System.nanoTime();
            linkedList.stream().reduce(0L, Long::sum);
            endTime = System.nanoTime();

            linkedListExecuteTime += endTime - startTime;

            startTime = System.nanoTime();
            linkedList.stream().parallel().reduce(0L, Long::sum);
            endTime = System.nanoTime();

            parallelLinkedListExecuteTime += endTime - startTime;
        }

        arrayListExecuteTime /= repeat;
        parallelArrayListExecuteTime /= repeat;
        linkedListExecuteTime /= repeat;
        parallelLinkedListExecuteTime /= repeat;

        log.info("Stream ArrayList: {}ms", arrayListExecuteTime / 1000000D);
        log.info("Paralle stream ArrayList: {}ms", parallelArrayListExecuteTime / 1000000D);
        log.info("Stream LinkedList: {}ms", linkedListExecuteTime / 1000000D);
        log.info("Paralle stream LinkedList: {}ms", parallelLinkedListExecuteTime / 1000000D);
    }
}
```
{: file='CollectionMain.java'}

```
Stream ArrayList: 46.298533ms
Paralle stream ArrayList: 18.005974ms
Stream LinkedList: 64.215308ms
Paralle stream LinkedList: 47.916683ms
```
{: file='output'}

데이터의 크기가 병렬 처리에서 성능을 낼 수 있는 정도로 많은 데이터이긴 하지만 List의 구현체의 차이로 LinkedList보다 ArrayList의 처리 성능이 좋다

### 최종 연산의 종류

최종 연산이 findAny라던가 reduce 같은 경우에 병렬 처리가 유리 할 수 있지만, collect와 같은 연산인 경우 병렬 처리가 불리 할 수 있다

```java
import lombok.extern.slf4j.Slf4j;

import java.util.stream.LongStream;

@Slf4j
public class TerminalOperationMain {
    public static void main(String[] args) {
        long startTime;
        long endTime;

        long forEachParallelTime = 0;
        long countParallelTime = 0;
        long findAnyParallelTime = 0;
        long findFirstParallelTime = 0;
        long sumParallelTime = 0;
        long reduceSumParallelTime = 0;

        long forEachTime = 0;
        long countTime = 0;
        long findAnyTime = 0;
        long findFirstTime = 0;
        long sumTime = 0;
        long reduceSumTime = 0;

        long range = 100_000_000L; // 10_000L, 100_000L, 1_000_000L, 10_000_000L, 100_000_000L
        int repeat = 5;

        final long targetValue = 123456L;

        for (int index = 0; index < repeat; index++) {
            startTime = System.nanoTime();
            LongStream.range(0, range).parallel()
                    .filter(item -> item > targetValue)
                    .forEach(item -> {});
            endTime = System.nanoTime();
            forEachParallelTime += (endTime - startTime);

            startTime = System.nanoTime();
            LongStream.range(0, range)
                    .filter(item -> item > targetValue)
                    .forEach(item -> {});
            endTime = System.nanoTime();
            forEachTime += (endTime - startTime);

            startTime = System.nanoTime();
            LongStream.range(0, range).parallel()
                    .filter(item -> item > targetValue)
                    .count();
            endTime = System.nanoTime();
            countParallelTime += (endTime - startTime);

            startTime = System.nanoTime();
            LongStream.range(0, range)
                    .filter(item -> item > targetValue)
                    .count();
            endTime = System.nanoTime();
            countTime += (endTime - startTime);

            startTime = System.nanoTime();
            LongStream.range(0, range).parallel()
                    .filter(item -> item > targetValue)
                    .findAny();
            endTime = System.nanoTime();
            findAnyParallelTime += (endTime - startTime);

            startTime = System.nanoTime();
            LongStream.range(0, range)
                    .filter(item -> item > targetValue)
                    .findAny();
            endTime = System.nanoTime();
            findAnyTime += (endTime - startTime);

            startTime = System.nanoTime();
            LongStream.range(0, range).parallel()
                    .filter(item -> item > targetValue)
                    .findFirst();
            endTime = System.nanoTime();
            findFirstParallelTime += (endTime - startTime);

            startTime = System.nanoTime();
            LongStream.range(0, range)
                    .filter(item -> item > targetValue)
                    .findFirst();
            endTime = System.nanoTime();
            findFirstTime += (endTime - startTime);

            startTime = System.nanoTime();
            LongStream.range(0, range).parallel()
                    .filter(item -> item > targetValue)
                    .sum();
            endTime = System.nanoTime();
            sumParallelTime += (endTime - startTime);

            startTime = System.nanoTime();
            LongStream.range(0, range)
                    .filter(item -> item > targetValue)
                    .sum();
            endTime = System.nanoTime();
            sumTime += (endTime - startTime);

            startTime = System.nanoTime();
            LongStream.range(0, range).parallel()
                    .filter(item -> item > targetValue)
                    .reduce(Long::sum);
            endTime = System.nanoTime();
            reduceSumParallelTime += (endTime - startTime);

            startTime = System.nanoTime();
            LongStream.range(0, range)
                    .filter(item -> item > targetValue)
                    .reduce(Long::sum);
            endTime = System.nanoTime();
            reduceSumTime += (endTime - startTime);
        }

        forEachParallelTime /= repeat;
        countParallelTime /= repeat;
        findAnyParallelTime /= repeat;
        findFirstParallelTime /= repeat;
        sumParallelTime /= repeat;
        reduceSumParallelTime /= repeat;

        forEachTime /= repeat;
        countTime /= repeat;
        findAnyTime /= repeat;
        findFirstTime /= repeat;
        sumTime /= repeat;
        reduceSumTime /= repeat;

        log.info("forEach execute time: {}ms", forEachTime / 1000000D);
        log.info("forEach parallel execute time: {}ms", forEachParallelTime / 1000000D);
        log.info("count execute time: {}ms", countTime / 1000000D);
        log.info("count parallel execute time: {}ms", countParallelTime / 1000000D);
        log.info("findAny execute time: {}ms", findAnyTime / 1000000D);
        log.info("findAny parallel execute time: {}ms", findAnyParallelTime / 1000000D);
        log.info("findFirst execute time: {}ms", findFirstTime / 1000000D);
        log.info("findFirst parallel execute time: {}ms", findFirstParallelTime / 1000000D);
        log.info("sum execute time: {}ms", sumTime / 1000000D);
        log.info("sum parallel execute time: {}ms", sumParallelTime / 1000000D);
        log.info("reduceSum execute time: {}ms", reduceSumTime / 1000000D);
        log.info("reduceSum parallel execute time: {}ms", reduceSumParallelTime / 1000000D);
    }
}
```
{: file='TerminalOperationMain.java'}

```
forEach execute time: 290.706441ms
forEach parallel execute time: 42.920983ms
count execute time: 512.688924ms
count parallel execute time: 71.366783ms
findAny execute time: 1.203433ms
findAny parallel execute time: 0.299958ms
findFirst execute time: 0.689358ms
findFirst parallel execute time: 0.836058ms
sum execute time: 553.864724ms
sum parallel execute time: 78.141624ms
reduceSum execute time: 597.052416ms
reduceSum parallel execute time: 75.754991ms
```
{: file='output'}

요소의 길이가 충분이 큰 경우에 여러 다양한 최종 연산에서는 대부분 병렬 연산이 성능에 이점이 있지만 findFirst 연산에 있어서는 병렬 연산이 성능이 낮게 측정 되었다

또한 병렬 연산을 사용하면서 최종 연산에서 thread safe하지 않는 연산으로 처리하게 된다면 예기치 않은 버그가 발생 할 수 있다는 것을 고려 해야 한다

## 결론

많은 양의 요소를 처리하는데 있어서 thread를 사용한 병렬 처리는 성능에 유리 할 수 있다

다만 모든 상황에서 병렬 처리가 단일 처리보다 좋은 성능을 보장 할 수 없으며, 요소의 수와 처리 하고자 하는 구현체의 종류, 그리고 실행할 작업에 대해서 thread safe 여부를 확인하여 단일와 병렬처리를 선택하여 사용 하여야 한다
