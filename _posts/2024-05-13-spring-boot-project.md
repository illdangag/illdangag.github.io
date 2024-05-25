---
title: "Spring boot 프로젝트 생성"
description: ''
author: illdangag

# permalink: thread-pool
date: 2024-05-13T00:00:00+09:00
last_modified_at: 2024-05-13T00:00:00+09:00

categories: [Java, Spring boot]
tags: [Java, Spring boot]
---

Spring boot 프로젝트을 생성 방법 intellij과 같은 IDE에서 생성 할 수 있지만 spring boot initializr 페이지에서 프로젝트를 생성 하는 것을 선호한다

- [start.spring.io](https://start.spring.io/)

## 기본 설정

![spring-boot-project-00](/assets/img/post/2024-05-13/spring-boot-project-00.png)

프로젝트의 빌드 관리 도구와 사용할 언어, spring boot의 버전과 프로젝트의 metadata를 설정한다

![spring-boot-project-01](/assets/img/post/2024-05-13/spring-boot-project-01.png)

우측 상단에 Add dependencies 버튼을 선택하여 프로젝트의 목적에 따라 사용할 dependency를 추가한다

### 자주 사용하는 종속성 라이브러리

#### spring-boot-devtools

개발 환경에서 주로 사용 되는 기능으로 개발 환경에서 서버가 실행되어 있는 상태에서 코드가 변경되면 자동으로 서버를 재시작 해주는 기능을 포함한다

#### spring-boot-starter-web

spring boot의 기본 내장 웹 컨테이너로 tomcat을 사용하도록 한다

#### lombok

annotation 기반으로 자주 사용하는 코드를 자동으로 생성해준다

- getter, sertter
- builder pattern
- logger

![spring-boot-project-02](/assets/img/post/2024-05-13/spring-boot-project-02.png)

spring web(spring-boot-stater-web) 종속성 라이브러리를 추가한 후 explore 버튼을 선택하여 생성 될 파일의 구조를 미리 볼 수 있다

![spring-boot-project-03](/assets/img/post/2024-05-13/spring-boot-project-03.png)

미리보기 창을 닫은 후 generate 버튼을 선택하여 프로젝트를 다운로드 받는다

![spring-boot-project-04](/assets/img/post/2024-05-13/spring-boot-project-04.png)
![spring-boot-project-05](/assets/img/post/2024-05-13/spring-boot-project-05.png)

<figure class="half">
  <a href="link"><img src="/assets/img/post/2024-05-13/spring-boot-project-04.png"></a>
  <a href="link"><img src="/assets/img/post/2024-05-13/spring-boot-project-04.png"></a>
<figcaption>2개이미지.</figcaption></figure>

프로젝트의 java 버전을 spring boot 프로젝트 생성시 설정한 java 버전과 동일한 버전으로 설정한다

![spring-boot-project-06](/assets/img/post/2024-05-13/spring-boot-project-06.png)

main 메서드를 실행하여 spring boot 프로젝트를 실행 할 수 있다
