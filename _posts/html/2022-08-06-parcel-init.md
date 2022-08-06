---
title: "[Parcel]Parcel 프로젝트 생성"

classes:
- wide

categories:
- HTML

tags:
- Parcel
- HTML
- CSS
- SASS
- SCSS
- Javascript
- Typescript

toc: false

toc_sticky: false

date: 2022-08-06T00:00:00+09:00

last_modified_at: 2022-08-06T00:00:00+09:00

---

<p align="center">
  <img src="/assets/images/thumbnails/2022-08-06-parcel-init.png"/>
</p>

# Parcel

- 다른 번들러들보다 빠름
- 설정 파일이 필요 없거나 최소한으로 설정

## 성능

| 번들러 | 시간 |
|:-:|:-:|
| browserify | 22.98s |
| webpack | 20.71s |
| parcel | 9.98s |
| parcel - 캐시 사용 | 2.64s |


# 환경
> nodejs: 16.16.0

# 프로젝트 생성

프로젝트를 생성할 디렉토리를 만들고 그 디렉토리에서 아래 명령어를 실행  

```shell
# package.json을 생성
yarn init

# parcel 설치
yarn global add parcel-bundler

# scss, sass를 사용하기 위해서는 sass 모듈을 설치
yarn add sass
```

# 프로젝트 구조

```shell
├── package.json
├── src
│   ├── images
│   │   └── html_logo.png
│   ├── index.html
│   ├── index.scss
│   ├── index.ts
│   └── styles
│       └── main.scss
```

## 파일

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Parcel Example</title>
    <link rel="stylesheet" href="./styles/main.scss"/>
    <link rel="stylesheet" href="./index.scss"/>
</head>
<body>
    <h1 class="main-title">
        Hello Parcel!
    </h1>
    <div>
        <img id="htmlImage" src="./images/html_logo.png"/>
    </div>
    <script src="./index.ts"></script>
</body>
</html>
```
./src/index.html  

```scss
.main-title {
    color: #748ffc;
}
```
./src/index.scss  

```typescript
console.log('index.ts');

const htmlImage = document.getElementById('htmlImage');
htmlImage?.addEventListener('click', () => {
    alert('CLICK!!');
});
```
./src/index.ts

```scss
h1 {
    font-size: 42px;
}
```
./src/styles/main.scss

```json
{
  "name": "parcel-init",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "scripts": {
    "dev": "parcel ./src/index.html"
  },
  "dependencies": {
    "parcel-bundler": "^1.12.5",
    "sass": "^1.54.3"
  },
  "devDependencies": {
    "typescript": "^4.7.4"
  }
}
```
./package.json

# 실행
```shell
yarn dev
```

`package.json` 파일에 scripts에 설정한 `dev`로 실행

## 실행 화면
![Result00](/assets/images/2022-08-06-parcel-init-00.png){: width="360"}


![Result01](/assets/images/2022-08-06-parcel-init-01.png){: width="360"}

이미지를 클릭하면 alert이 나타난다

### 참고 페이지
- [예제 프로젝트 github repository](https://github.com/illdangag/parcel-init/tree/37a4c87e1cea9be8e14e6ee21335310b0afb9986)
- [ko.parceljs.org](https://ko.parceljs.org)
