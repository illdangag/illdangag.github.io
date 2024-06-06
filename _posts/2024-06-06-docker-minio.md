---
title: "Docker를 이용한 Minio 설치"
description: "Minio는 고성능의 object storage로 AWS S3와 호환되는 API로 구성되어 있어서 개발 또는 테스트 목적으로 사용 하기 적합하다"
author: illdangag

# permalink: docker
date: 2024-06-06T13:23:00+09:00
last_modified_at: 2024-06-06T13:23:00+09:00

categories: [Docker, Minio]
tags: [Docker, Minio]
---

고성능의 Object storage

AWS S3와 호환되는 API로 구성되어 있어서 개발 또는 테스트 목적으로 사용 하기 적합하다

- [MinIO](https://min.io/) 
- [docker hub - minio](https://hub.docker.com/r/minio/minio)

## Docker

### Docker image 다운로드

```bash
docker pull minio/minio:RELEASE.2023-08-04T17-40-21Z.hotfix.bfb2c8508
```

### Docker container 실행

```bash
docker run \
  --name minio \
  -p 9000:9000 \
  -p 9001:9001 \
  -e "MINIO_ROOT_USER=admin" \
  -e "MINIO_ROOT_PASSWORD=password" \
  minio/minio server /data --console-address ":9001" &
```

minio가 API를 위한 docker port는 9000이고 minio의 web console에서 사용할 port는 임의로 지정 되지만 `console-address` 옵션으로 지정 할 수 있으므로 9001로 지정 한다

내부의 9000, 9001 port를 외부에서 접근 할 수 있도록 설정 한다

## Minio 시작하기

![docker-minio-00](/assets/img/post/2024-06-06/docker-minio-00.png)  
docker container 실행시 옵션으로 설정 하였던 `MINIO_ROOT_USER`을 Username, `MINIO_ROOT_PASSWORD`을 Password로 로그인 한다

![docker-minio-01](/assets/img/post/2024-06-06/docker-minio-01.png)  
![docker-minio-02](/assets/img/post/2024-06-06/docker-minio-02.png)  
처음 접속하는 경우에는 이전에 생성한 bucket이 존재하지 않기 때문에 bucket을 생성하여야 한다

bucket의 이름을 입력하고 bucket을 생성 한다

![docker-minio-03](/assets/img/post/2024-06-06/docker-minio-03.png)  
![docker-minio-04](/assets/img/post/2024-06-06/docker-minio-04.png)  
/buckets 페이지에서 생성된 bucket 목록을 확인 할 수 있으며 그 중 하나를 선택하면 bucket의 정보를 조회 할 수 있다

![docker-minio-05](/assets/img/post/2024-06-06/docker-minio-05.png)  
![docker-minio-06](/assets/img/post/2024-06-06/docker-minio-06.png)  
메뉴의 Object Browser를 선택하면 파일 업로드 및 다운로드를 할 수 있다
