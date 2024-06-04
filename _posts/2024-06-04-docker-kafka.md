---
title: "Docker를 이용한 Kafka 설치"
description: "docker를 사용하여 로컬 및 테스트용 kafka cluster와 zookeeper를 구성"
author: illdangag

# permalink: docker
date: 2024-06-04T22:45:00+09:00
last_modified_at: 2024-06-04T22:45:00+09:00

categories: [Docker, Kafka]
tags: [Docker, Kafka]
---

zookeeper 1대에 kafka 3대 연동, 그리고 kafka 모니터링을 위해서 kafka-ui를 구성하려고 한다

kafka 모니터링 툴은 kafka-ui와 kafdrop 두가지를 많이 사용 하나 docker에의 kafka-ui는 apple sillicon을 지원하고 kafdrop은 apple silicon을 지원하지 않아 kafka-ui를 사용하기로 결정하였다

- [docker hub - cp zookeeper](https://hub.docker.com/r/confluentinc/cp-zookeeper)
- [docker hub - cp kafka](https://hub.docker.com/r/confluentinc/cp-kafka)
- [docker hub - kafka ui](https://hub.docker.com/r/provectuslabs/kafka-ui)
- [docker hub - kafdrop](https://hub.docker.com/r/obsidiandynamics/kafdrop)

## Docker

### docker-compose.yaml

```yaml
version: '2.1'

services:
  zoo1:
    image: confluentinc/cp-zookeeper:7.4.0
    hostname: zoo1
    container_name: zoo1
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_SERVER_ID: 1
      ZOOKEEPER_SERVERS: zoo1:2888:3888

  kafka1:
    image: confluentinc/cp-kafka:7.4.0
    hostname: kafka1
    container_name: kafka1
    ports:
      - "9092:9092"
      - "29092:29092"
    environment:
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka1:19092,EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:9092,DOCKER://host.docker.internal:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT,DOCKER:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181"
      KAFKA_BROKER_ID: 1
      KAFKA_LOG4J_LOGGERS: "kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO"
      KAFKA_AUTHORIZER_CLASS_NAME: kafka.security.authorizer.AclAuthorizer
      KAFKA_ALLOW_EVERYONE_IF_NO_ACL_FOUND: "true"
    depends_on:
      - zoo1

  kafka2:
    image: confluentinc/cp-kafka:7.4.0
    hostname: kafka2
    container_name: kafka2
    ports:
      - "9093:9093"
      - "29093:29093"
    environment:
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka2:19093,EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:9093,DOCKER://host.docker.internal:29093
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT,DOCKER:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181"
      KAFKA_BROKER_ID: 2
      KAFKA_LOG4J_LOGGERS: "kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO"
      KAFKA_AUTHORIZER_CLASS_NAME: kafka.security.authorizer.AclAuthorizer
      KAFKA_ALLOW_EVERYONE_IF_NO_ACL_FOUND: "true"
    depends_on:
      - zoo1

  kafka3:
    image: confluentinc/cp-kafka:7.4.0
    hostname: kafka3
    container_name: kafka3
    ports:
      - "9094:9094"
      - "29094:29094"
    environment:
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka3:19094,EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:9094,DOCKER://host.docker.internal:29094
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT,DOCKER:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181"
      KAFKA_BROKER_ID: 3
      KAFKA_LOG4J_LOGGERS: "kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO"
      KAFKA_AUTHORIZER_CLASS_NAME: kafka.security.authorizer.AclAuthorizer
      KAFKA_ALLOW_EVERYONE_IF_NO_ACL_FOUND: "true"
    depends_on:
      - zoo1

  kafka-ui:
    image: provectuslabs/kafka-ui:v0.7.0
    container_name: 'kafka-ui'
    ports:
      - "8080:8080"
    environment:
      KAFKA_CLUSTERS_0_NAME: 'local-kafka'
      KAFKA_CLUSTERS_0_ZOOKEEPER: 'zookeeper:2181'
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: 'kafka1:19092,kafka2:19093,kafka3:19094'
    depends_on:
      - kafka1
      - kafka2
      - kafka3
```
{: file='docker-compose.yaml'}

docker-compose.yaml 내용은 [https://github.com/conduktor/kafka-stack-docker-compose](https://github.com/conduktor/kafka-stack-docker-compose) 참고하여 작성 하였다

### docker container 실행

```bash
docker-compose -f ./docker-compose.yaml up -d
```

![docker-kafka-00](/assets/img/post/2024-06-04/docker-kafka-00.png)
docker image를 다운로드 한 후에 container가 실행 된다

![docker-kafka-01](/assets/img/post/2024-06-04/docker-kafka-01.png)  
kafka-ui의 port를 8080으로 설정 하였기에 localhost:8080으로 접속하면 실행된 kafka를 확인 할 수 있다
