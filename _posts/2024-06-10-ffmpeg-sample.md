---
title: "FFmpeg 기본 사용법"
description: "FFmpeg의 설치 및 기본적인 사용법에 대한 예제이다"
author: illdangag

# permalink: docker
date: 2024-06-10T20:29:00+09:00
last_modified_at: 2024-06-10T20:29:00+09:00

categories: [FFmpeg]
tags: [FFmpeg]
---

FFmpeg는 모든 비디오, 오디오, 이미지 포멧에 대한 디코딩 인코딩을 목표로 진행되고 있는 오픈소스 포로젝트이다

- [FFmpeg](https://ffmpeg.org/)

FFmpeg의 구체적인 사용법이나 특정 인코딩 관련 링크는 아래 링크를 확인해 본다

- [FFmpeg 옵션](https://ffmpeg.org/ffmpeg.html)
- [trac wiki - FFmpeg mp3](https://trac.ffmpeg.org/wiki/Encode/MP3)
- [trac wiki - FFmpeg H.264](https://trac.ffmpeg.org/wiki/Encode/H.264)
- [trac wiki - FFmpeg AV1](https://trac.ffmpeg.org/wiki/Encode/AV1)

## 다운로드 및 설치

- [ffmpeg.org/download.html](https://ffmpeg.org/download.html)

<img src="/assets/img/post/2024-06-10/ffmpeg-sample-00.png" width="480px" alt="ffmpeg-sample-00">

위 페이지에 점속한 후 실행 환경의 OS에 마우스 오버하면 다운로드 링크가 나타난다

### macOS

<img src="/assets/img/post/2024-06-10/ffmpeg-sample-01.png" width="580px" alt="ffmpeg-sample-01">

FFmpeg의 최신 빌드 ZIP 파일을 다운로드 한다

현재 기준으로 `ffmpeg-115575-g4e120fbbbd.zip` 파일을 압축 해제 하여 압축이 해제된 `ffmpeg` 파일을 확인 할 수 있다

`ffmpeg` 파일이 위치한 디렉토리를 환경 변수에 등록 하여 경로 입력 없이 `ffmpeg` 파일을 호출 할 수 있다

## FFmpeg 옵션

### 기본 옵션

- `-i`  
  원본 파일

### 코덱

- `-c:v`  
  출력 파일의 비디오 코덱  
  값을 설정하지 않으면 원본 파일의 비디오 코덱을 사용하여 변환한다

- `-c:a`  
  출력 파일의 오디오 코덱  
  옵션의 값을 copy로 설정 하면 원본의 오디오를 수정하지 않고 출력 파일에 그대로 사용한다

### 품질

- `-b:v`  
  출력 파일의 비디오 비트레이트

- `-q:v`  
  출력 파일의 가변 비디오 비트레이트의 품질  
  libx265 (H.264) 코덱에서는 사용되지 않음  
  mpeg4, mpeg2video, libxvid 등에서는 사용 할 수 있음  

- `-b:a`  
  출력 파일의 오디오 비트레이트

- `-q:a`  
  출력 파일의 가변 오디오 비트레이트의 품질  
  값의 범위는 0에서 10, 0에 가까울수록 음원이 높은 품질
  기본값은 4

- `-s`  
  출력 파일의 비디오 해상도

- `-r`  
  비디오 프레임  
  원본 파일을 설정하는 `-i` 옵션보다 먼저 위치하는 경우 원본 파일의 프레임 속도를 설정  
  원본 파일을 설정하는 `-i` 옵션보다 나중에 위치하는 경우 출력 파일의 프레임 속도를 설정

- `-ar`
  옵션은 샘플레이트  
  사용 가능한 값은 44100, 48000, 32000, 22050, 24000, 16000, 11025, 12000, 8000  
  기본값은 44100

### 위치 및 시간

- `-ss`  
  원본 파일의 시작 시간 위치

- `-to`  
  원본 파일의 종료 시간 위치

- `-t`  
  `-ss` 옵션인 원본 파일의 시작 시간으로부터 지난 시간 만큼 종료 시간을 설정 

- `-frames:v`  
  출력 파일에 대한 프레임 수

## FFmpeg 예제

### 스크린샷 추출

```shell
./ffmpeg \
  -i sample.mp4 \
  -ss 00:00:00 \
  -frames:v 10 \
  thumnail_%d.png
```
{:file='screenshot png'}

`-ss` 옵션으로 시작 시간을 설정하고 `-frames:v` 옵션으로 10 프레임을 설정  
출력 파일의 확장자가 `png`이므로 영상의 시간이 00:00:00 부터 10 프레임을 png 파일이 10개가 생성된다


```shell
./ffmpeg \
  -i sample.mp4 \
  -ss 00:00:10 \
  -t 00:00:05 \
  thumnail.gif
```
{:file='screenshot gif'}

`-ss` 옵션으로 시작 시간을 설정하고 `-t` 옵션으로 시작 시간으로부터 5초 동안을 편집 구간으로 설정  
출력 파일의 확장자가 `gif`이므로 시간이 5초인 gif 파일이 1개 생성 된다

### 비디오 파일에서 오디오 음원 추출

#### 오디오 포멧

비디오 파일에서 mp3, wav, ogg 등 오디오 음원을 추출 할 수 있다

```shell
./ffmpeg \
  -i ./sample.mp4 \
  output.mp3
```
{:file='extract mp3 audio'}

출력 파일의 확장자가 `mp3`이므로 원본 영상의 오디오가 mp3 파일을 추출한다

```shell
./ffmpeg \
  -i ./sample.mp4 \
  output.wav
```
{:file='extract wav audio'}

출력 파일의 확장자가 `wav`이므로 원본 영상의 오디오가 wav 파일을 추출한다

```shell
./ffmpeg \
  -i ./sample.mp4 \
  output.ogg
```
{:file='extract ogg audio'}

출력 파일의 확장자가 `ogg`이므로 원본 영상의 오디오가 ogg 파일을 추출한다

#### 특정 구간의 오디오 추출

```shell
./ffmpeg \
  -i sample.mp4 \
  -ss 00:00:05 \
  -to 00:00:10 \
  output.mp3
```
{:file='start end'}

`-ss` 옵션으로 시작 시간을 설정  
`-to` 옵션으로 종료 시간을 설정  
원본 파일의 5초부터 10초 구간에 대한 5초의 길이를 가진 mp3 파일을 생성한다

#### 오디오 샘플레이트 및 비트레이트

```shell
./ffmpeg \
  -i sample.mp4 \
  -ar 44100 \
  -b:a 320k \
  -q:a 4 \
  output.mp3
```
{:file='sample rate, bit rate'}

`-ar` 옵션에 44100으로 설정하여 출력하는 오디오 셈플레이트를 44100Hz로 설정  
`-b:a` 옵션에 320k를 설정하여 오디오 비트레이는 320k로 설정  
`-q:a` 옵션에 4를 설정하여 오디오의 가변 비트레이트의 품질을 중간으로 설정하여 mp3 파일을 생성한다

### 비디오 파일 인코딩

#### 비디오 포맷

```shell
./ffmpeg \
  -i sample.mp4 \
  -c:v libx264 \
  output.mp4
```
{:file='video codec h264'}

`-c:v` 옵션에 libx264으로 설정하여 h264 포맷으로 인코딩 하도록 설정하여 mp4 파일을 생성한다

```shell
./ffmpeg \
  -i sample.mp4 \
  -c:v mpeg2video \
  output.avi
```
{:file='video codec mpeg2'}

`-c:v` 옵션에 mpeg2 설정하여 mpeg2 포맷으로 인코딩 하도록 설정하여 avi 파일을 생성한다

```shell
./ffmpeg \
  -i sample.mp4 \
  -c:v libxvid \
  output.avi
```
{:file='video codec xvid'}

`-c:v` 옵션에 libxvid 설정하여 xvid 포맷으로 인코딩 하도록 설정하여 avi 파일을 생성한다

#### 특정 기간의 비디오 추출

```shell
./ffmpeg \
  -i sample.mp4 \
  -c:v libx264 \
  -ss 00:00:05 \
  -to 00:00:10 \
  output.mp4
```
{:file='video codec h264 start end'}

`-c:v` 옵션에 libx264으로 설정하여 h264 포맷으로 인코딩 하도록 설정하고  
`-ss` 옵션으로 시작 시간을 설정  
`-to` 옵션으로 종료 시간을 설정  
원본 파일의 5초부터 10초 구간에 대한 5초의 길이를 가진 mp4 파일을 생성한다

#### 비디오의 비트레이트와 오디오 샘플레이트 및 비트레이트

```shell
./ffmpeg \
  -i sample.mp4 \
  -c:v libx264 \
  -b:v 2000k \
  -c:a aac \
  -ar 44100 \
  -b:a 128k \
  output.mp4
```
{:file='video codec h264 bitrate'}

`-c:v` 옵션에 libx264으로 설정하여 h264 포맷으로 인코딩 하도록 설정  
`-b:v` 옵션에 2000k으로 비트레이트가 2000k으로 설정하여 mp4 파일을 생성  
`-c:a` 옵션에 aac로 설정하여 aac 오디오 코덱을 사용하도록 설정  
`-ar` 옵션에 44100으로 설정하여 출력하는 오디오 셈플레이트를 44100Hz로 설정  
`-b:a` 옵션에 128k를 설정하여 오디오 비트레이는 128k로 설정하여 mp4 파일을 생성한다

#### h.264 포맷의 two pass 인코딩

- [trac wiki - FFmpeg h.264 two pass](https://trac.ffmpeg.org/wiki/Encode/H.264#twopass)

고정 비트레이트인 CBR, 가변 비트레이트인 VBR과 다르게 두번의 인코딩 과정으로 첫번째 인코딩 과정에서 원본 파일을 분석하고 두번째 인코딩 과정에서 분석 결과를 바탕으로 효율적으로 가변 비트레이트를 지정 할 수 있는 방법이다

```shell
./ffmpeg \
  -i sample.mp4 \
  -c:v libx264 \
  -b:v 2000k \
  -pass 1 \
  -an \
  -f null /dev/null \
  && \
./ffmpeg \
  -i sample.mp4 \
  -c:v libx264 \
  -b:v 2000k \
  -pass 2 \
  -c:a aac \
  -ar 44100 \
  -b:a 128k \
  output.mp4
```
{:file='video codec h264 two pass'}

**첫번째 ffmpeg 명령줄**

`-c:v` 옵션에 libx264으로 설정하여 h264 포맷으로 인코딩 하도록 설정  
`-b:v` 옵션에 2000k으로 비트레이트가 2000k으로 설정  
`-pass` 옵션에 1로 설정하여 two pass 설정의 첫번째 인코딩 과정임을 설정  
`-an` 옵션으로 오디오는 처리하지 않음을 설정, 비디오만 처리 하기 위함  
`-f` 옵션에 null /dev/null로 설정하여 출력 동영상 파일이 생성되지 않게 설정

첫번째 ffmpeg 명령줄을 실행하면 원본 파일의 분석 결과에 대한 log, mbtree 파일을 생성한다

**두번째 ffmpeg 명령줄**

`-c:v` 옵션에 libx264으로 설정하여 h264 포맷으로 인코딩 하도록 설정  
`-b:v` 옵션에 2000k으로 비트레이트가 2000k으로 설정  
`-pass` 옵션에 2로 설정하여 two pass 설정의 두번째 인코딩 과정임을 설정  
`-c:a` 옵션에 aac로 설정하여 aac 오디오 코덱을 사용하도록 설정  
`-ar` 옵션에 44100으로 설정하여 출력하는 오디오 셈플레이트를 44100Hz로 설정  
`-b:a` 옵션에 128k를 설정하여 오디오 비트레이는 128k로 설정하여 mp4 파일을 생성한다

두번째 ffmpeg 명령줄을 실행하면 첫번째 ffmpeg 명령줄에서 생성된 분석 결과를 토대로 mp4 파일을 생성한다
