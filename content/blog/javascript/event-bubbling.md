---
title: Event Bubbling
date: 2017-5-21 11:65:23
category: javascript
---

급하면 2번 부터 보자.

## 1. Event dispatch and DOM event flow

`event 전파 과정`과 `DOM tree를 통한 이벤트 propagate 방식`에 대해 알아보자. Application은 `dispatchEvent()`를 통해 이벤트 objects를 전파한다. 그리고 event object는 DOM event flow로 결정된 DOM tree를 통해 propogate한다.

![W3C 이벤트 전파 과정](https://www.w3.org/TR/DOM-Level-3-Events/images/eventflow.svg)

Event objects는 `event target`으로 전파된다. 하지만 전파되기 전에, event object의 `propagation path`가 먼저 결정되어야 한다.

`propogation path`는 이벤트가 통과하는 현재 이벤트 타겟들의 ordered list다. propagation path는 document의 계층 트리 구조의 영향을 받는다.(위 그림처럼) 리스트의 마지막 item은 `event target`이다. 그리고 리스트의 이전 아이템들은 `target`의 조상을 가리킨다.

`propagation path`가 결정되면, **이벤트 오브젝트는 1개 이상의 event 경로를 통과**한다. 3가지 이벤트 경로가 있다.

1. capture phase
2. target phase
3. bubble phase

이벤트 객체는 이 경로를 다음 설명처럼 완료한다. 만약 경로를 지원하지 않는다거나, 이벤트 객체 propagation이 멈추면 skip한다. 예를 들어 `bubble` attribute가 false라면, bubble phase는 skip된다. 만약 `stopPropagation()`이 `dispatch`보다 앞에 호출된다면, 모든 phases는 skip된다.

capture phase
: event object는 window 에서 target' parent로 target의 조상을 통해 propogate한다. (event는 엘리먼트로 내려간다.)

target phase
: event 객체는 event 객체의 target에 도착한다. event가 버블링 false로 명시하면, 이벤트 객체는 phase 완료 후 중지한다. (이벤트가 타겟 엘리먼트에 도착했다.)

bubble phase
: event 객체는 target의 조상을 통해 (capture와)반대로 propagate한다. target의 부모에서 시작하여 Window에서 끝이 난다. (이벤트는 위로 버블링 up 한다.)

## 2. Capture and Bubble

<p class="codepen" data-height="265" data-theme-id="dark" data-default-tab="js,result" data-user="cooking" data-slug-hash="wdQbJq" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid black; margin: 1em 0; padding: 1em;" data-pen-title="wdQbJq">
  <span>See the Pen <a href="https://codepen.io/cooking/pen/wdQbJq/">
  wdQbJq</a> by cooking (<a href="https://codepen.io/cooking">@cooking</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

`<em></em>` 안의 내용을 클릭했는데 어떻게 div가 실행될까?

### Bubbling

Bubbling은

이벤트가 엘리먼트에서 일어났을 때, 먼저 이벤트가 일어난곳의 핸들러를 실행한다. 그 다음엔 해당 엘리먼트의 부모, 그 부모의 부모.. 조상까지 DOM 트리를 타고 올라간다.

각각 클릭 이벤트가 바인딩 된 중첩된 엘리먼트 `form > div > p`를 보자.

<p class="codepen" data-height="265" data-theme-id="dark" data-default-tab="js,result" data-user="cooking" data-slug-hash="wdQbJq" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid black; margin: 1em 0; padding: 1em;" data-pen-title="wdQbJq">
  <span>See the Pen <a href="https://codepen.io/cooking/pen/wdQbJq/">
  wdQbJq</a> by cooking (<a href="https://codepen.io/cooking">@cooking</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

`<p>`를 클릭하면 다음 순서대로 실행된다.

1. `p`
2. `<div>`
3. `<form>`
4. `document` object까지 쭉 올라간다.

그래서 p,div,form 순으로 alert 메시지를 볼 수 있다. 그 이외의 부모 엘리먼트에 대한 alert가 안보이는 이유는 이벤트에 대한 핸들러가 없기 때문이다.

이와 같이 내부 엘리먼트(이벤트 타겟)에서 부모를 통해 DOM tree를 올라가는 과정을 `bubbling`이라고 한다.

> 거의 모든 이벤트는 버블링한다.
> `blur`이벤트는 버블링 안하는데 걍 문서 참고하면서 사용하자.

### event.target

부모 엘리먼트의 핸들러에서 이벤트가 실제로 일어난 곳에 대한 정보를 얻을 수 있다.

**이벤트를 발생시킨 깊게 중첩된 엘리먼트는 `target` element라고 한다. 이것은 `event.target`으로 접근가능하다**

`event.target`은 이벤트를 이니셜라이징한 `target`이고 그것은 버블링 과정동안 `불변`한다.

event handler에서 `this`는 `current` element다. 즉, 현재 핸들러를 실행하는 엘릴먼트다.

다음 코드는 `event.target`과 `this`를 알려주는 예제다.

가장 바깥쪽의 `form` 에 click event에 대한 핸들러를 바인딩 했다.

<p class="codepen" data-height="265" data-theme-id="dark" data-default-tab="js,result" data-user="cooking" data-slug-hash="wdQbJq" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid black; margin: 1em 0; padding: 1em;" data-pen-title="wdQbJq">
  <span>See the Pen <a href="https://codepen.io/cooking/pen/wdQbJq/">
  wdQbJq</a> by cooking (<a href="https://codepen.io/cooking">@cooking</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

### stop bubbling

bubbling event는 target element에서 부모의 부모로 계속 올라간다.
일반적으로 `<html>` 위쪽 까지 올라가고, 그리고나서 `document` object로 이동하고 일부 이벤트는 `window`까지 도달하여 모든 핸들러를 호출한다.

이 때, `event.stopPropagation()`을 통해 버블링을 중지할 수 있다.

> 필요한게 아니면 bubbling stop 하지말자.
> 버블링은 편리하다. 가끔 `stopPropogation()` 은 숨겨진 문제를 만든다.
> 예를 들어:

1. 중첩 메뉴 만들었다. 각 서브메뉴는 메뉴 아이템마다 클릭을 처리해야한다 그리고 `stopPropogation`을 호출해서 outer part에 trigger되지 않도록 한다.
2. 나중에 유저의 행위를 추적하기 위해 전체 윈도우 내부의 클릭 이벤트를 잡기로 결정했다. 가끔 counter만들 때 이렇게 만든다. `document.addEventListener('click'..)`
3. counter는 `stopPropogation`이 멈춘 지역에선 작동하지 않는다.

`Capture`는

- 이벤트 발생시, 해당 이벤트를 Capture하기 위해 이벤트가 발생한 요소를 포함하는 **부모 HTML로부터 이벤트의 근원지인 자식 요소까지** 이벤트를 검사한다.
- 이 때, **capture 속성의 이벤트 핸들러가 있다면 실행 시키면서 이벤트 요소로 접근**한다.

![W3C 이벤트 전파 과정](https://www.w3.org/TR/DOM-Level-3-Events/images/eventflow.svg)

`td` 클릭하며 이벤트는 조상을 통해 엘리먼트로 내려간다(capturing). 그러다가 target에 도착하고 다시 올라간다(bubbling).

capturing은 거의 사용되지 않는다.

> 보통 기본 이벤트 핸들러는 버블 속성이며 W3C 표준에서 이벤트를 묶을 때 캡처 핸들러인지 버블 핸들러인지 지정할 수 있다. 하지만 IE는 캡처 이벤트를 지원하지 않는다.

```javascript
EventTarget.addEventListener(type, listner[, userCapture]);
```

- type : event type을 나타내는 문자열
- listener : 특정 타입의 이벤트가 발생시 알림을 받을 객체.
- useCapture
  만약 true라면, capture 사용. capture를 초기화 한 후, 모든 지정된 타입의 이벤트는 DOM tree에서 하위의 어떤 EventTarget에 전달되기 전에 등록된 리스너에게 먼저 전달된다. tree의 상위로 버블링된 이벤트는 capture를 사용하기 위해 지정된 리스너를 동작 시키지 않는다.

## 참고

[quirkmode.org/events_order](https://www.quirksmode.org/js/events_order.html#link4)

[W3C event flow](https://www.w3.org/TR/DOM-Level-3-Events/#event-flow)
