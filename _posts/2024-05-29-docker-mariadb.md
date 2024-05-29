---
title: "Docker를 이용한 MariaDB 설치"
description: "database라고 한다면 보편적으로 RDBMS를 말하며 그중 대표적인 database는 MariaDB가 있다 docker를 이용하여 mariadb를 설치 해보려고 한다"
author: illdangag

# permalink: docker
date: 2024-05-29T22:45:00+09:00
last_modified_at: 2024-05-29T22:45:00+09:00

categories: [Docker, MariaDB]
tags: [Docker, MariaDB]
---

database라고 한다면 보편적으로 RDBMS를 말하며 그중 대표적인 database는 MariaDB가 있다

docker를 이용하여 mariadb를 설치 해보려고 한다

- [Mariadb - Official Image](https://hub.docker.com/_/mariadb)

## Docker

### Docker image 다운로드

```bash
docker pull mariadb:11.1.2
```

docker mariadb image를 다운로드 한다

### Docker container 생성

```bash
docker run -p 3306:3306 --name mariadb-container-00 -e MARIADB_ROOT_PASSWORD=password -d mariadb:11.1.2
```

mariadb는 port를 3306을 사용하므로 내부의 port는 3306으로 설정한다

## MariaDB 접속

### Visual studio code에 plugin 설치

- [https://marketplace.visualstudio.com/items?itemName=formulahendry.vscode-mysql](https://marketplace.visualstudio.com/items?itemName=formulahendry.vscode-mysql)

mariadb를 접속하기 위해서 visual studio code에 위 plugin을 설치

![docker-mariadb-00](/assets/img/post/2024-05-29/docker-mariadb-00.png)  
plugin을 설치하면 vscode의 explorer 탭에 mysql 항목이 생성 되어 있다

mysql의 "+" 버튼을 누른다

host, username, password, port, ssl 인증서 등 접속 정보를 차례로 입력한다

![docker-mariadb-01](/assets/img/post/2024-05-29/docker-mariadb-01.png)  
올바르게 접속이 된다면 접속한 mariadb 인스턴스와 database 목록이 나타난다

![docker-mariadb-02](/assets/img/post/2024-05-29/docker-mariadb-02.png)  
mariadb 인스턴스를 오른쪽 클릭하고 New Query를 선택한다

![docker-mariadb-03](/assets/img/post/2024-05-29/docker-mariadb-03.png)  
```sql
SHOW databases;
```

![docker-mariadb-04](/assets/img/post/2024-05-29/docker-mariadb-04.png)
query를 입력하고 실행하고자 하는 query를 블록 설정한 후에 오른쪽 클릭하고 Run MySQL Query를 선택하면 해당 query를 실행 할 수 있다
