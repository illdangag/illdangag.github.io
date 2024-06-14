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

## 다운로드 및 설치

- [ffmpeg.org/download.html](https://ffmpeg.org/download.html)

<img src="/assets/img/post/2024-06-10/ffmpeg-sample-00.png" width="480px">

위 페이지에 점속한 후 실행 환경의 OS에 마우스 오버하면 다운로드 링크가 나타난다

### macOS

<img src="/assets/img/post/2024-06-10/ffmpeg-sample-01.png" width="580px">

FFmpeg의 최신 빌드 ZIP 파일을 다운로드 한다

현재 기준으로 `ffmpeg-115575-g4e120fbbbd.zip` 파일을 압축 해제 하여 압축이 해제된 `ffmpeg` 파일을 확인 할 수 있다

`ffmpeg` 파일이 위치한 디렉토리를 환경 변수에 등록 하여 경로 입력 없이 `ffmpeg` 파일을 호출 할 수 있다

## FFmpeg 사용 예제

### 비디오 파일에서 오디오 음원 추출

`sample.mp4` 이름의 샘플 비디오 파일을 준비 한다

#### 오디오 포멧

비디오 파일에서 mp3, wav, ogg 등 오디오 음원을 추출 할 수 있다

```shell
# mp3 오디오 음원 추출
./ffmpeg \
  -i ./sample.mp4 \
  -f mp3 \
  -vn output.mp3
```
{:file='extract mp3 audio'}

```shell
# wav 오디오 음원 추출
./ffmpeg \
  -i ./sample.mp4 \
  -f wav \
  -vn output.wav
```
{:file='extract wav audio'}

```shell
# ogg 오디오 음원 추출
./ffmpeg \
  -i ./sample.mp4 \
  -f ogg \
  -vn output.ogg
```
{:file='extract ogg audio'}

각각 mp3, wav, ogg 포멧으로 음원을 추출한다

- `-i` 옵션은 작업 대상 파일
- `-f` 옵션은 오디오 음원 포멧
- `-vn` 옵션은 결과 파일의 이름이다

#### 오디오 샘플레이트 및 비트레이트

샘플레이트 및 가변 또는 고정 비트레이트 설정 할 수 있다

```shell
# 44100 샘플레이트
# 320k 가변 비트레이트
./ffmpeg \
  -i sample.mp4 \
  -f mp3 \
  -ar 44100 \
  -b:a 320k \
  -q:a 4 \
  -vn output.mp3
```
{:file='sample rate, bit rate'}

- `-ar` 옵션은 샘플레이트
  - 사용 가능한 값은 44100, 48000, 32000, 22050, 24000, 16000, 11025, 12000, 8000
  - 기본값은 44100
- `-b:a` 옵션은 가변 비트레이트의 비트레이트 기준
- `-q:a` 옵션은 가변 비트레이트의 품질
  - 값의 범위는 0에서 10
  - 0에 가까울수록 음원이 높은 품질
  - 기본값은 4

### 스크린샷 추출

```shell
./ffmpeg \
  -i sample.mp4 \
  -ss 00:00:00 \
  -vframes 10 \
  thumnail_%d.png

```
{:file='screenshot png'}

- `-ss` 옵션은 스크린샷의 기준 시간
- `-vframes` 옵션은 스크린샷 개수

```shell
./ffmpeg \
  -i sample.mp4 \
  -ss 00:00:00 \
  -vframes 10 \
  thumnail_%d.gif

```
{:file='screenshot gif'}

스크린샷을 gif 형식으로 추출하며 `-vframes` 옵션에 따라 10 프레임으로 구성된 gi가 생성된다

### 비디오 인코딩

