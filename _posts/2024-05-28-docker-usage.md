---
title: "Docker 기본 사용"
description: 'docker의 기본 사용법 및 명령어'
author: illdangag

# permalink: docker
date: 2024-05-28T00:20:39+09:00
last_modified_at: 2024-05-28T00:20:39+09:00

categories: [Docker]
tags: [Docker]
---

# Image

```bash
# 이미지 목록
docker images

# 이미지 삭제
docker rmi {이미지 이름}:{태그}

# 컨테이너를 이미지:태그로 저장
docker commit {컨테이너 ID} {이미지 이름}:{태그}
```

# Container

```bash
# 컨테이너 실행
docker run
# * 옵션
#   -d
#      detached mode 백그라운드 실행
#
#   --restart {옵션}
#       옵션에 따라 재시작 여부 설정
#       no, on-failure, always, unless-stopped
#
#   -p {외부 접속 포트}:{연결될 내부 접속 포트}
#       포트 바인딩
#
#   --name {컨테이너 이름}
#       컨테이너의 이름 설정
#
#   -e "{환경변수 키=환경변수 값}"
#       환경 변수

# 컨테이너 목록
docker ps -a

# 컨테이너 종료
docker stop {컨테이너 아이디}

# 컨테이너 삭제
docker rm {컨테이너 아이디}

# 컨테이너 쉘 접속
docker exec -it {컨테이너 아이디} /bin/bash
```
