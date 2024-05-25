---
title: "Fork join pool"
description: ''
author: illdangag

# permalink: thread-pool
date: 2024-05-11T00:02:00+09:00
last_modified_at: 2024-05-11T00:02:00+09:00

categories: [Java, Thread]
tags: [Java, Thread]
---

java 1.7 부터 지원한다

많은 양의 데이터를 작업하기 용이한 크기로 나누어 여러 thread에서 데이터를 처리한다

데이터를 처리함에 유휴 thread가 최대한 존재하지 않도록 하기 위함이다

예제에 사용된 전체 소스 코드는 하단의 Repository 링크를 참고하면 된다

- [gitbook-code/java-sample-code](https://github.com/illdangag/gitbook-code/tree/main/java-sample-code/src/main/java/com/illdangag/thread)

## ForkJoinPool

작업을 실행할 thread pool

### 작업 실행 및 관리 method

#### execute

ForkJoinTask interface를 구현한 객체를 매개 변수로 전달 받으며 작업 실행에 대한 결과를 반환 받을 수 없다

#### submit

ForkJoinTask interface를 구현한 객체를 매개 변수로 전달 받으며 작업 실행에 대한 결과를 반환 받을 수 있다

### ForkJoinTask

fork join pool에 작업을 추가하기 위해서는 ForkJoinTask을 상속한 class를 사용 하여야 한다

#### RecursiveAction

ForkJoinTask 상송한 class이다

작업의 결과를 반환하지 않는다

#### RecursiveTask

ForkJoinTask 상속한 class이다

작업의 결과를 반환한다

## 예제 코드

```java
class MaxIntegerTask extends RecursiveTask<Integer> {
    private List<Integer> list;

    public MaxIntegerTask(List<Integer> list) {
        this.list = list;
    }

    @Override
    protected Integer compute() {
        String threadName = Thread.currentThread().getName();
        System.out.printf("START: [%s]\n", threadName);

        int listSize = this.list.size();

        if (listSize > 2) { // 분할
            int subSize0 = (int) Math.floor(listSize / 2f);
            int subSize1 = (int) Math.ceil(listSize / 2f);

            List<Integer> subList0 = this.list.subList(0, subSize0);
            List<Integer> subList1 = this.list.subList(listSize - subSize1, listSize);
            MaxIntegerTask subTask0 = new MaxIntegerTask(subList0);
            MaxIntegerTask subTask1 = new MaxIntegerTask(subList1);

            subTask0.fork();
            subTask1.fork();

            Integer result0 = subTask0.join();
            Integer result1 = subTask1.join();

            return Math.max(result0, result1);
        } else { // 분할하지 않음
            if (this.list.size() == 1) {
                return this.list.get(0);
            } else {
                return Math.max(this.list.get(0), this.list.get(1));
            }
        }
    }
}
```
{: file='MaxIntegerTask.java'}

Integer list중에 가장 큰 수를 구하는 작업이다

list의 길이가 2개 이상인 경우에 분할하고, 1개인 경우에는 그 Integer를 반환하고 2개인 경우에는 둘중에 큰 수를 반환한다

```java
public class ForkJoinPoolMain {
    public static void main(String[] args) throws Exception {
        ForkJoinPool forkJoinPool = new ForkJoinPool(2);
        List<Integer> list = Arrays.asList(0, 1, 2, 3, 4, 12, 6, 7, 8, 9, 10);
        MaxIntegerTask task = new MaxIntegerTask(list);

        ForkJoinTask<Integer> forkJoinTask = forkJoinPool.submit(task);
        Integer result = forkJoinTask.get();
        System.out.println(result);
    }
}
```
{: file='ForkJoinPoolMain.java'}

```
START: [ForkJoinPool-1-worker-3]
START: [ForkJoinPool-1-worker-1]
START: [ForkJoinPool-1-worker-1]
START: [ForkJoinPool-1-worker-1]
START: [ForkJoinPool-1-worker-1]
START: [ForkJoinPool-1-worker-1]
START: [ForkJoinPool-1-worker-1]
START: [ForkJoinPool-1-worker-1]
START: [ForkJoinPool-1-worker-3]
START: [ForkJoinPool-1-worker-1]
START: [ForkJoinPool-1-worker-3]
START: [ForkJoinPool-1-worker-3]
START: [ForkJoinPool-1-worker-1]
12
```
{: file='output'}

ForkJoinPool에 thread를 2개로 설정한 후에 작업을 할당 하면 2개의 thread로 분할 작업 하게 된다

## 결론

thread를 이용한 병렬 처리의 목적은 많은 요소를 짧은 시간에 효율적으로 처리하고자 사용한다

특히  fork join pool인 경우에는 유휴 thread가 최소한으로 발생 하도록 하는 방법이므로 분할한 작업들이 최대한 균일한 작업 시간이 소요 할 수 있는 경우에 fork join pool에서 효율이 증가한다
