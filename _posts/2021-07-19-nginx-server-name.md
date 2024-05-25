---
title: "[emerg] could not build server_names_hash"
description: '도메인의 길이가 nginx에 기본 설정보다 긴 경우 발생하는 오류'
author: illdangag

date: 2021-07-19T00:00:00+09:00
last_modified_at: 2021-07-19T00:00:00+09:00

categories: [Nginx]
tags: [Nginx]
---

## nginx 시작시 오류 메시지
```shell
Job for nginx.service failed because the control process exited with error code.
See "systemctl status nginx.service" and "journalctl -xe" for details.
nginx status
nginx: [emerg] could not build server_names_hash, you should increase server_names_hash_bucket_size: 32
```
리버스 프록시 설정시 사용한 도메인의 길이가 nginx에 기본 설정보다 긴 경우 발생하는 오류

```
http {
	...
	server_names_hash_bucket_size 64;
	server_names_hash_max_size 8192;
	...
}
```
`nginx.conf` 파일에 위와 같이 설정
필요에 따라 `server_names_hash_bucket_size`값을 `64` 이상으로 설정

## 참고 페이지
- [stackoverflow.com](https://stackoverflow.com/questions/13895933/nginx-emerg-could-not-build-the-server-names-hash-you-should-increase-server)
- [nginx document](https://nginx.org/en/docs/http/server_names.html#optimization)
