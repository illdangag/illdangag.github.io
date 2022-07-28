---
title: "[HTML]팝업 윈도우 닫힘 이벤트"

classes:
  - wide

categories:
  - blog

tags:
  - HTML
  - CSS
  - Javascript

toc: false

toc_sticky: false

date: 2021-07-13T00:00:00+09:00

last_modified_at: 2021-07-13T00:00:00+09:00

---

팝업이 닫힘 이벤트를 등록하는 것이 아닌 팝업의 상태를 주기적으로 확인하는 방법이다

## index.html

```html
<!DOCTYPE html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, user-scalable=no">
  <title>Index</title>
</head>
<body>
  <h1>INDEX</h1>
  <button id="popupButton">show popup</button>
  <script type="text/javascript">
    const popupButton = document.getElementById('popupButton');
    popupButton.addEventListener('click', () => {
      const popupWindow = window.open('/popup.html', '', 'left=100,top=100,width=320,height=240');
      const popupInterval = setInterval(function() {
        if (popupWindow.closed) {
          clearInterval(popupInterval);
          console.log('popup closed!');
        }
      }, 500);
    });
  </script>
</body>
```
![index.html](/assets/images/2021-08-23-popup-close-event-01.png)

## popup.html
```html
<!DOCTYPE html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, user-scalable=no">
  <title>Popup</title>
</head>
<body>
  <h1>POPUP</h1>
</body>
```
![popup.html](/assets/images/2021-08-23-popup-close-event-02.png)

`show popup` 버튼을 누르면 팝업창이 나타나고 팝업창을 닫으면 index.html의 콘솔에 `popup closed!` 메시지가 출력된다

### 참고 페이지
- [developer.mozilla.org - setInterval](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setInterval)
- [developer.mozilla.org - window.open](https://developer.mozilla.org/en-US/docs/Web/API/Window/open)
