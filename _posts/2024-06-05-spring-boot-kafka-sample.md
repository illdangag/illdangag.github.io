---
title: "Spring boot, Kafka 간단한 예제"
description: "Kafka를 단순한 Message Queue로서 사용하는 간단한 예제이다"
author: illdangag

# permalink: docker
date: 2024-06-05T16:13:00+09:00
last_modified_at: 2024-06-05T16:13:00+09:00

categories: [Java, Spring boot]
tags: [Java, Spring boot, Kafka]
---

Kafka를 단순한 Message Queue로서 사용하는 간단한 예제이다

예제에 사용된 소스 코드는 하단의 Repository 링크를 참고하면 된다

- [illdangag/gitbook-code - spring boot kafka sample 00](https://github.com/illdangag/gitbook-code/tree/main/spring-kafka-sample-00)

## Spring boot

### dependency

#### spring-boot-starter-web

org.springframework.boot:spring-boot-starter-web

Spring boot의 MVC 구조를 지원하고 logback logger와 JSON 관련 라이브러리인 jackson를 포함한다
추가로 기본 내장 웹서버인 tomcat을 포함하고 있다

#### spring-kafka

org.springframework.kafka:spring-kafka

[Spring for Apache Kafka](https://spring.io/projects/spring-kafka)

Spring boot와 Kafka를 연동하기 위한 라이브러리

#### lombok

org.projectlombok:lombok

annotation 기반의 코드 자동 생성 라이브러리

### application.yaml

```yaml
server:
  port: 8081 # kafka ui가 8080으로 실행되어 있어서 spring boot는 8081로 설정
spring:
  kafka:
    bootstrap-servers: # kafka brokers
      - localhost:9092
      - localhost:9093
      - localhost:9094
    consumer:
      auto-offset-reset: earliest # consumer가 topic을 가져오는 순서, latest, earliest, none
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.ErrorHandlingDeserializer
      properties:
        spring:
          deserializer:
            value:
              delegate:
                class: org.springframework.kafka.support.serializer.JsonDeserializer
          json:
            trusted:
              packages: com.illdangag.kafka.data
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
```
{: file='application.yaml'}

Kafka의 메시지 종류로는 Java bean을 사용하기 위해서 Spring boot와 Kafka는 객체를 JSON 형태로 직렬화/역직렬화 하여 메시지를 송수신 하여야 한다

Consumer는 `org.springframework.kafka.support.serializer.JsonDeserializer`를 사용하고 Producer는 `org.springframework.kafka.support.serializer.JsonSerializer`를 사용한다

위 application.yaml 설정을 보면 `ErrorHandlingDeserializer`를 사용하고 `JsonDeserializer`를 간접적으로 사용하고 있는데 이유는 역직렬화 과정에서 오류로 인하여 예외가 발생한다면 `JsonDeserializer`를 바로 사용 하는 경우에는 동일한 오류 메시지가 멈추지 않고 매우 빠른 속도로 많이 발생하는데 `ErrorHandlingDeserializer`를 사용하고 `JsonDeserializer`를 간접적으로 사용 한다면 오류 메시지는 한번 출력된다

consumer.properties.spring.json.trusted.packages는 역직렬화 하는 Class에 대한 whitelist이다
whitelist에 등록되지 않은 Class인 경우에는 역직렬화 과정에서 예외가 발생하여 Kafka로부터 메시지를 올바르게 소비 할 수 없다

```text
Caused by: java.lang.IllegalArgumentException: The class 'com.illdangag.kafka.data.User' is not in the trusted packages: [java.util, java.lang, com.illdangag.kafka].
If you believe this class is safe to deserialize, please provide its name. If the serialization is only done by a trusted source, you can also enable trust all (*).
```
{: file='output'}

### Message

```java
package com.illdangag.kafka.data;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.ToString;

@AllArgsConstructor
@NoArgsConstructor
@Getter
@ToString
public class User {
    private String name;
    private int age;
}
```
{: file='User.java'}

Kafka를 통하여 송수신 할 Java bean 클래스이다

lombok 라이브러리를 사용하여 몇가지 메서드를 생성했다

### Producer

```java
package com.illdangag.kafka.service;

import com.illdangag.kafka.data.User;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;

@Slf4j
@Service
public class KafkaProducer {
    private final KafkaTemplate<String, User> kafkaTemplate;

    @Autowired
    public KafkaProducer(KafkaTemplate kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void sendMessage(User user) {
        log.info("Produce: {}", user);
        this.kafkaTemplate.send("User", user); // topic, message
    }
}
```
{:file='KafkaProducer.java'}

Spring boot에서 Kafka로 메시지를 송신한다

`KafkaTemplate.send` 메서드의 첫번째 인자는 Kafka의 topic, 두번째 인자는 Kafka에 전송할 메시지이다

```java
package com.illdangag.kafka.service;

import com.illdangag.kafka.data.User;
import lombok.extern.slf4j.Slf4j;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

import java.io.IOException;

@Slf4j
@Service
public class KafkaConsumer {
    @KafkaListener(topics = "User", groupId = "user-group")
    public void consume(User user) throws IOException {
        log.info("Consume: {}", user.toString());
    }
}
```
{:file='KafkaConsumer.java'}

`KafkaListener` annotation으로 Kafka Broker로부터 Consumer가 구독 할 topic과 Consumer의 그룹을 설정한다

Producer로 부터 Kafka로 구독중인 topic에 메세지가 등록되면 `KafkaListener` annotation이 등록된 메서드가 호출 되고 메시지가 해당 메서드의 인자로 전달된다

### Controller

```java
package com.illdangag.kafka.controller;

import com.illdangag.kafka.data.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import com.illdangag.kafka.service.KafkaProducer;

@RestController
@RequestMapping(value = "/send")
public class UserController {
    private final KafkaProducer producer;

    @Autowired
    public UserController(KafkaProducer producer) {
        this.producer = producer;
    }

    @RequestMapping(method = RequestMethod.GET, value = "/user")
    public ResponseEntity<String> send00(@RequestParam(name = "name", defaultValue = "", required = false) String name,
                                         @RequestParam(name = "age", defaultValue = "", required = false) int age) {
        User user = new User(name, age);
        producer.sendMessage(user);

        return ResponseEntity.status(HttpStatus.OK).body("OK");
    }
}
```
{:file='UserController.java'}

Kafka로 메시지를 전송하기 위해서 간단히 구현한 REST API이다

간편하게 웹 브라우저를 사용하기 위해서 http method는 GET을 사용하고 User 객체에 설정할 이름과 나이는 query parameter로 전달 하도록 한다

## 서버 실행 및 테스트

KafkaApplication.main 메서드를 실행하여 Spring boot 서버를 실행한다

http://localhost:8081/send/user?name=kim&age=27

로컬 환경에 spring boot의 port가 상단 application.yaml 설정을 따른다면 위 URL로 Kafka에 메시지를 생성 할 수 있다

```text
2023-05-20T15:00:37.209+09:00  INFO 49060 --- [nio-8081-exec-1] c.illdangag.kafka.service.KafkaProducer  : Produce: User(name=kim, age=27)
2023-05-20T15:00:37.306+09:00  INFO 49060 --- [ntainer#0-0-C-1] c.illdangag.kafka.service.KafkaConsumer  : Consume: User(name=kim, age=27)
```
{:file='output'}

### Kafka-ui

![spring-boot-kafka-sample-00](/assets/img/post/2024-06-05/spring-boot-kafka-sample-00.png)  
![spring-boot-kafka-sample-01](/assets/img/post/2024-06-05/spring-boot-kafka-sample-01.png)  
kafka에 등록된 Consumer와 Kafka가 수신 한 메시지를 확인 할 수 있다
