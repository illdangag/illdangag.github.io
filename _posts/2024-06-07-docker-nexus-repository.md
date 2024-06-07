---
title: "Docker를 이용한 Nexus repository 설치"
description: "사내에서만 사용할 비공개 프로젝트나 개인 프로젝트로 외부에 공개하고 싶지 않은 프로젝트인 경우에 Central maven repository 배포하지 않고 사설 repository를 사용한다"
author: illdangag

# permalink: docker
date: 2024-06-07T16:02:00+09:00
last_modified_at: 2024-06-07T16:02:00+09:00

categories: [Docker, Nexus repository]
tags: [Docker, Nexus repository]
---
사내에서만 사용할 비공개 프로젝트나 개인 프로젝트로 외부에 공개하고 싶지 않은 프로젝트인 경우에 Central maven repository 배포하지 않고 사설 repository를 사용한다

docker를 이용하여 nexus3 이미지로 사설 repository를 구축 해보려고 한다

- [Docker hub - Nexus repository](https://hub.docker.com/r/sonatype/nexus3/)

## Docker

### Docker image 다운로드 및 데이터 디렉토리 생성

```bash
docker pull sonatype/nexus3:3.54.1
```

docker 이미지를 받는다

```bash
mkdir /home/user/nexus-data
```

docker nexus container에서 사용 할 데이터를 저장할 디렉토리를 생성한다

### Docker container 실행

```bash
docker run --name nexus -d \
  -p 5000:5000 \
  -p 8081:8081 \
  -v /home/user/nexus-data:/nexus-data \
  -u root sonatype/nexus3:3.54.1
```

8081 port로 설정하여 실행하였기 때문에 [http://localhost:8081](http://localhost:8081/) 로 접속하면 nexus repository에 접속 할 수 있다

## Nexus repository 초기 설정

### 초기 관리자 admin 로그인

admin 계정으로 로그인 하여 초기 설정을 진행 하여야 한다

```bash
-rw-r--r--   1 root      root         36 May 27 13:52 admin.password
drwxr-xr-x   3 root      root       4096 May 27 13:52 blobs
drwxr-xr-x 324 root      root      12288 May 27 13:51 cache
drwxr-xr-x   6 root      root       4096 May 27 13:52 db
drwxr-xr-x   3 root      root       4096 May 27 13:52 elasticsearch
drwxr-xr-x   3 root      root       4096 May 27 13:51 etc
drwxr-xr-x   2 root      root       4096 May 27 13:51 generated-bundles
drwxr-xr-x   2 root      root       4096 May 27 13:51 instances
drwxr-xr-x   3 root      root       4096 May 27 13:51 javaprefs
-rw-r--r--   1 root      root          1 May 27 13:51 karaf.pid
drwxr-xr-x   3 root      root       4096 May 27 13:51 keystores
-rw-r--r--   1 root      root         14 May 27 13:51 lock
drwxr-xr-x   3 root      root       4096 May 27 13:52 log
drwxr-xr-x   2 root      root       4096 May 27 13:51 orient
-rw-r--r--   1 root      root          5 May 27 13:51 port
drwxr-xr-x   2 root      root       4096 May 27 13:51 restore-from-backup
drwxr-xr-x   7 root      root       4096 May 27 13:52 tmp
```

```bash
cat ./admin.password
045512e6-7e5d-47f4-84a9-6adc36403043
```

/nexus-data로 설정한 디렉토리에 `admin.password`에 admin 계정의 초기 비밀번호가 생성되어 있다

![docker-nexus-repository-00](/assets/img/post/2024-06-07/docker-nexus-repository-00.png)  
우상단의 Sign in을 선택하고 admin 계정으로 로그인 한다

![docker-nexus-repository-01](/assets/img/post/2024-06-07/docker-nexus-repository-01.png)  
위에서 확인한 admin 임시 비밀번호로 로그인 한다

![docker-nexus-repository-02](/assets/img/post/2024-06-07/docker-nexus-repository-02.png)  
admin 계정의 비밀번호를 재설정 한다

### 익명 사용자 접속 가능 여부

![docker-nexus-repository-03](/assets/img/post/2024-06-07/docker-nexus-repository-03.png)  
익명 사용자에 대해서 접속 가능 여부를 설정한다

![docker-nexus-repository-04](/assets/img/post/2024-06-07/docker-nexus-repository-04.png)  
Finish를 눌러 기본 설정을 마친다
