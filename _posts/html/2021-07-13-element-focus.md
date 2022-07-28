---
title: "[HTML]tabindex를 이용하여 foucs/blur 이벤트 동작"

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

`focus`와 `blur` 이벤트는 `input` 또는 `button` 요소에서 동작하지만, 경우에 따라 일반적인 `span` 요소나 `div`, `p` 요소에서 사용 해야 하는 경우가 있다

그런 경우에는 요소에 `tabindex`를 추가하면 `focus`와 `blur` 이벤트를 사용 할 수 있다

<div>
<iframe src="https://codesandbox.io/embed/elementfocus-47dmp?fontsize=14&hidenavigation=1&theme=dark"
style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
title="element_focus"
allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe>
</div>

`tabindex`에 -1로 설정한 요소는 `focus`와 `blur` CSS 및 Javascript 이벤트가 동작하지만, `tabindex`에 값을 설정하지 않은 요소는 이벤트가 동작하지 않는다

### 참고 페이지
- [developers.google.com](https://developers.google.com/web/fundamentals/accessibility/focus/using-tabindex?hl=ko)
