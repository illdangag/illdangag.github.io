---
title: "[HTML]팝업 윈도우 닫힘 이벤트"

classes:
- wide

categories:
- HTML

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

<div>
<iframe src="https://codesandbox.io/embed/popup-close-event-hl49zt?expanddevtools=1&fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="popup_close_event"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>
</div>

## 페이지 구성
- index.html
- popup.html


`show popup` 버튼을 누르면 팝업창이 나타나고 팝업창을 닫으면 index.html의 콘솔에 `popup closed!` 메시지가 출력된다

### 참고 페이지
- [developer.mozilla.org - setInterval](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setInterval)
- [developer.mozilla.org - window.open](https://developer.mozilla.org/en-US/docs/Web/API/Window/open)
