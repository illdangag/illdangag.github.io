---
title: "HLS(HTTP Live Streaming)와 FFmpeg를 이용한 영상 생성"
description: "HLS 개념과 FFmpeg로 HLS 영상 생성"
author: illdangag

# permalink: docker
date: 2024-06-23T13:37:00+09:00
last_modified_at: 2024-06-23T13:37:00+09:00

categories: [FFmpeg, HLS]
tags: [FFmpeg, HLS]
---

HLS는 인터넷을 이용하여 비디오 컨텐츠를 제공하는데 사용되는 스트리밍 프로토콜이다  
비디오 파일 전체를 한번에 다운로드 하지 않고, 비디오 파일을 일정 기준으로 분할 하여 HTTP 프로토콜로 전송한다  
HSL는 최대한 범용적으로 사용하기 위해서 비디오의 포맷은 H.264 또는 H.265를 사용한다  
비디오는 특정 시간 길이의 세그먼트로 나눈다  
비디오를 480p, 720p, 1080p 등의 다양한 품질로 여러 재생목록의 세그먼트를 복제한다

- [apple - HLS Document](https://developer.apple.com/streaming)
- [cloudflare - HLS Document](https://www.cloudflare.com/ko-kr/learning/video/what-is-http-live-streaming)
- [cdnetworks - HLS Document](https://www.cdnetworks.com/ko/media-delivery-blog/http-live-streaming)

## FFmpeg로 HLS 영상 변환

ffmpeg로 일반 비디오 파일을 HLS로 서비스 할 수 있도록 비디오 파일을 세그먼트로 분리 할 수 있다

기본적인 ffmpeg 사용법은 아래 링크에서 참고한다
- [FFmpeg 기본 사용법](/posts/ffmpeg-sample)
- [FFmpeg - ffmpeg-formats](https://ffmpeg.org/ffmpeg-formats.html)

```shell
./ffmpeg \
  -i sample.mp4 \
  -f hls \
  -hls_time 5 \
  -hls_list_size 0 \
  -hls_segment_filename ./output/%v/video_%04d.ts \
  -master_pl_name master.m3u8 \
  -c:v libx264 \
  -c:a aac \
  -ar 48000 \
  -map 0:v:0 -map 0:a:0 -map 0:v:0 -map 0:a:0 -map 0:v:0 -map 0:a:0 \
  -var_stream_map "v:0,a:0,name:1080p v:1,a:1,name:720p v:2,a:2,name:480p" \
  -b:v:0 5000k -maxrate:v:0 5000k -bufsize:v:0 10000k -s:v:0 1920x1080 -crf:v:0 15 -b:a:0 128k \
  -b:v:1 2500k -maxrate:v:1 2500k -bufsize:v:1 5000k -s:v:1 404x720 -crf:v:1 22 -b:a:1 96k \
  -b:v:2 1000k -maxrate:v:2 1000k -bufsize:v:2 2000k -s:v:2 270x480 -crf:v:2 28 -b:a:2 64k \
  ./output/%v/playlist.m3u8
```
{:file='hls'}

```shell
./output
├── 1080p
│   ├── playlist.m3u8
│   ├── video_0000.ts
│   ├── video_0001.ts
│   ├── video_0002.ts
│   └── video_0003.ts
├── 480p
│   ├── playlist.m3u8
│   ├── video_0000.ts
│   ├── video_0001.ts
│   └── video_0002.ts
├── 720p
│   ├── playlist.m3u8
│   ├── video_0000.ts
│   ├── video_0001.ts
│   └── video_0002.ts
└── master.m3u8
```
{:file='output'}

-f
: 옵션에 hls로 설정하여 hls을 사용할 수 있는 세그먼트로 분리하도록 설정

-hls_time
: 옵션은 세그먼트의 기준  
설정 값 만큼이 지난 다음에 가장 가까운 비디오 키프레임에서 영상을 분할  
비디오의 키프레임은 비디오가 모든 프레임에 대하여 프레임에 전체 정보를 저장하지 않고 특정 프레임에 프레임 전체 정보를 저장하고 그 뒤로 일정 프레임 까지 프레임에 대한 변화만 기록하여 비디오의 절대적인 데이터양을 줄인다

-hls_list_size
: 재생 스트림의 길이를 설정  
0으로 설정하면 제한 없음

-hls_segment_filename
: 세그먼트 파일의 이름  
`%v`는 재생목록의 이름, `%d`는 하나의 재생목록에서 분할된 세그먼트의 순서

-master_pl_name
: 전체 재생 스트림의 정보를 가지고 있는 마스터 재생 스트림의 이름

-map
: 원본 비디오 파일에는 영상 및 음성이 1개 이상의 스트림을 가질 수 있으며 출력 파일에 어떠한 스트림을 사용 할 지를 설정  
`n번째 원본 파일:비디오 또는 오디오:n번째 스트림`을 의미, n은 zero-index로 0부터 시작  
`0:v:0`의 의미는 첫번째 입력 파일에서 비디오는 첫번째 스트림을 사용  
`0:a:0`의 의미는 첫번째 입력 파일에서 오디오는 첫번째 스트림을 사용  
[trac wiki - FFmpeg Map](https://trac.ffmpeg.org/wiki/Map)  
`map` 옵션이 대한 자세한 설명은 위 링크를 참조한다

-var_stream_map
: 재생 스트림의 목록을 지정  
재생 스트림은 공백으로 구분  
`v:n번째 재생 목록,a:n번째 재생 목록,name:재생 목록 이름`으로 구성 되며 v는 비디오, a는 오디오를 의미, n은 zero-index로 0부터 시작

-b:v:{n}
: n번째 재생 스트림의 비디오 평균 비트레이트

-maxrate:v:{n}
: n번째 재생 스트림의 비디오 최대 공차를 지정  
`-bufsize:v:{n}` 옵션과 같이 사용

-bufsize:v:{n}
: n번째 재생 스티림의 비디오 출력 비트레이트의 가변성을 설정하는 버퍼의 크기를 지정

-s:v:{n}
: n번째 재생 스트림의 비디오 해상도 설정

-crf:v:{n}
: n번째 재생 스트림의 비디오 품질  
0~51의 값을 가지며 0에 가까울수록 품질이 좋고 51에 가까울수록 품질이 낮음  
권장 사용 범위는 17에서 28 사이의 값

-b:a:{n}
: n번째 재생 스트림의 오디오 품질
