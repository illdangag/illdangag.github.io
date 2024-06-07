---
title: "Logback 간단 사용 예제"
description: "java 어플리케이션에서 로그를 위한 프레임워크는 logback, log4j 등이 존재 한다 로그 프레임워크를 유연하게 사용할 수 있게 하는 인터페이스인 slf4j가 있다"
author: illdangag

# permalink: docker
date: 2024-06-07T14:11:00+09:00
last_modified_at: 2024-06-07T14:11:00+09:00

categories: [Java, Logback]
tags: [Java, Logback]
---

java 어플리케이션에서 로그를 위한 프레임워크는 logback, log4j 등이 존재 한다

로그 프레임워크를 유연하게 사용할 수 있게 하는 인터페이스인 slf4j가 있다

로그 프레임워크를 사용하기 위해서는 slf4j와 다양한 로그 프레임워크중 하나를 선택하여 java 어플리케이션에 적용 한다

- [SLF4J](https://www.slf4j.org/)
- [Logback Home](https://logback.qos.ch/)
- [Log4j](https://logging.apache.org/log4j)

예제에 사용된 소스 코드는 하단의 Repository 링크를 참고하면 된다

- [illdangag/gitbook-code - logback sample code](https://github.com/illdangag/gitbook-code/tree/main/java-logback-sample-code)

## Java Application

### dependency

```groovy
plugins {
    id 'java'
}

group = 'com.illdangag'
version = '1.0'

repositories {
    mavenCentral()
}

dependencies {
    // https://mvnrepository.com/artifact/org.slf4j/slf4j-api
    implementation 'org.slf4j:slf4j-api:2.0.13'
    // https://mvnrepository.com/artifact/ch.qos.logback/logback-classic
    implementation 'ch.qos.logback:logback-classic:1.5.6'

    testImplementation platform('org.junit:junit-bom:5.10.0')
    testImplementation 'org.junit.jupiter:junit-jupiter'
}

test {
    useJUnitPlatform()
}
```
{:file='build.gradle'}

`build.gradle` 에 slf4j와 logback에 대한 의존성을 추가한다

#### resources

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="30 seconds" debug="false">
    <property name="LOGS_ABSOLUTE_PATH" value="./log" />
    <property name="LOG_PATTERN_0" value="[%d{yyyy-MM-dd HH:mm:ss.SSS, GMT+9}][%thread][%-5level][%marker]\(%F:%L\): %m%n" />

    <!-- 콘솔 출력 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${LOG_PATTERN_0}</pattern> <!-- 로그 출력 패턴 -->
        </encoder>
    </appender>

    <!-- 파일 출력 -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender"> <!-- RollingFileAppender 사용 -->
        <file>${LOGS_ABSOLUTE_PATH}/application.log</file> <!-- 파일 경로 및 파일 이름 -->
        <encoder>
            <pattern>${LOG_PATTERN_0}</pattern> <!-- 로그 출력 패턴 -->
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy"> <!-- 시간으로 파일 분할 정책 -->
            <fileNamePattern>${LOGS_ABSOLUTE_PATH}/application.%d{yyyy-MM-dd_HH-mm-ss}.%i.log.gz</fileNamePattern> <!-- 분할 파일 이름 패턴 -->
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>500KB</maxFileSize> <!-- 분할 파일 크기 기준 -->
            </timeBasedFileNamingAndTriggeringPolicy>
            <maxHistory>10</maxHistory> <!-- 분할된 파일의 최대 개수 -->
        </rollingPolicy>
    </appender>

    <root level="DEBUG"> <!-- 로그 출력 기준 -->
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="FILE" />
    </root>
</configuration>
```
{:file='logback.xml'}

logback을 사용하므로 logback.xml 파일을 resources 경로에 생성하여 log 설정을 한다

하나의 log 형식으로 console과 file로 출력하기 위해서 `LOG_PATTERN_0` property로 선언하여 사용한다

console 출력을 하기 위해서 `ch.qos.logback.core.ConsoleAppender` 을 등록한다

file 출력을 하기 위해서 `ch.qos.logback.core.rolling.RollingFileAppender` 을 등록한다

#### RollingFileAppender

- `file`

  파일 경로 및 파일 이름

- `file.encoder.pattern`

  로그 출력 패턴

- `rollingPolicy.fileNamePattern`

  조건에 의해서 분할된 파일의 이름

  파일 이름에 시간 패턴이 들어가는 경우 시간을 표현한 단위 수준에서 시간에 따라 파일 분할

- `rollingPolicy.timeBasedFileNamingAndTriggeringPolicy` 의 `class` 요소

  파일이름이 시간 패턴에 따른 파일 분할의 구현체

- `rollingPolicy.timeBasedFileNamingAndTriggeringPolicy.maxFileSize`

  분할 파일의 최대 크기

- `rollingPolicy.maxHistory`

  분할 파일의 최대 개수

  분할 파일의 개수가 설정 값보다 커지는 경우 가장 먼저 생성된 파일이 삭제되고 새로운 분할 파일이 생성


#### Java application

```java
package com.illdangag;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.Marker;
import org.slf4j.MarkerFactory;

public class AppMain {
    private static final Marker FATAL_MARKER = MarkerFactory.getMarker("FATAL");
    private static final Logger logger = LoggerFactory.getLogger(AppMain.class);

    public static void main(String[] args) throws Exception {
        for (int index = 0; index < 1000; index++) {
            logger.debug(FATAL_MARKER, "RollingFileAppender TEST: {}", index);
            Thread.sleep(1L);
        }
    }
}
```
{:file='AppMain.java'}

```
[2024-05-15 14:00:37.288][main][DEBUG][FATAL](AppMain.java:14): RollingFileAppender TEST: 0
[2024-05-15 14:00:37.294][main][DEBUG][FATAL](AppMain.java:14): RollingFileAppender TEST: 1
[2024-05-15 14:00:37.295][main][DEBUG][FATAL](AppMain.java:14): RollingFileAppender TEST: 2
[2024-05-15 14:00:37.297][main][DEBUG][FATAL](AppMain.java:14): RollingFileAppender TEST: 3
...
...
...
[2024-05-15 14:00:38.678][main][DEBUG][FATAL](AppMain.java:14): RollingFileAppender TEST: 996
[2024-05-15 14:00:38.679][main][DEBUG][FATAL](AppMain.java:14): RollingFileAppender TEST: 997
[2024-05-15 14:00:38.681][main][DEBUG][FATAL](AppMain.java:14): RollingFileAppender TEST: 998
[2024-05-15 14:00:38.682][main][DEBUG][FATAL](AppMain.java:14): RollingFileAppender TEST: 999
```
{:file='output'}

```
├── application.2024-05-15_14-00-29.0.log.gz
├── application.2024-05-15_14-00-30.0.log.gz
├── application.2024-05-15_14-00-31.0.log.gz
├── application.2024-05-15_14-00-32.0.log.gz
├── application.2024-05-15_14-00-33.0.log.gz
├── application.2024-05-15_14-00-34.0.log.gz
├── application.2024-05-15_14-00-35.0.log.gz
├── application.2024-05-15_14-00-37.0.log.gz
└── application.log
```
{:file='file list'}

logback.xml 파일의 설정에 맞게 로그 형식에 맞게 출력되고 파일 분할이 되는 것을 확인 할 수 있다
