---
title: concurrency model and event loop
date: 2017-8-1 22:20:00
category: javascript
---

- *Javascript는 싱글 스레드로 동작한다.*
- *node.js는 이벤트 루프 기반 싱글 스레드다*

등등 이런 말을 많이 들었다.

싱글 스레드는 동시에 하나의 작업만 처리 가능하다.
하지만 js가 실행되는 환경에서는 많은 작업이 동시에 처리되고 있다.

정답은 동시성(Concurrency) 모델과 이벤트 루프에 있다.
JS는 이벤트 루프를 통해 비동기 방식으로 동시성을 지원한다.

사실 무슨말인지 이해하기 힘들다. 먼저, JS가 동작하는 환경과 엔진을 살펴보자.

JS 엔진은 JS로 작성한 코드를 해석하고 실행하는 인터프리터다. 주로 웹 브라우저에서 이용된다.

JS엔진은 3가지 영역을 가진다.

![concurrency model 1](./images/concurrency_model_1.png)

## Call Stack (a.k.a. run to completion)

JS는 단 하나의 호출 stack을 사용한다. (그래서 Run To Completion)
1개 함수가 실행되면 call stack에 push되어 다른 함수는 실행되지 못한다.
실행이 끝나면 pop이 되고 다음 함수가 실행된다.

```javascript
function test1(){
	console.log("test1");
}
function test2(){
	test1();
	console.log("test2");
}

test2();
```

![concurrency model 2](./images/concurrency_model_5.png)

1. 먼저, 전역 환경에 대한 코드 블록이 먼저 스택에 쌓인다.
- 전역 환경은 가상의 익명함수로 감싸져 있다고 생각하자.
- 그래서, 마지막 라인까지 실행되어야 스택에서 제거 된다.
2. `test2()`라인을 만나고 `test2` 함수에 대한 코드 블록이 쌓인다.
3. 그 다음에는 `test1()`에 대한 코드 블록이 형성된다.
4. `test1()`에서 `"test1"`을 출력 후 함수 종료와 함께 스택에서 제거된다.
5. `test2()`로 돌아와서 `"test2"`를 출력 후 함수 종료와 함께 스택에서 제거된다.
6. 모든 전역환경에 대한 코드를 수행하였으므로 익명 함수도 제거된다.


## Heap

인스턴스와 같은 동적으로 생성된 객체가 Heap에 할당된다.

## Task Queue

Queue는 JS 런타임 환경에서 처리해야 하는 Task들을 임시 저장한다.
저장된 Task들은 `Call Stack`이 비었을 때 Queue의 Task를 순서대로 수행한다.

*누가 Queue의 task들을 Call Stack에 추가할까?*

`Event Loop`가 큐에 대기 중인 첫번째 태스크를 추가한다.

*그럼 어떤 task가 저장될까?*

비동기로 호출되는 함수들은 Task Queue에 쌓인다. 예륻 들어 이벤트에 의해 실행되는 함수(handler)들이 비동기로 실행된다. 또한, JS 엔진이 나닌 Web API영역에 따로 정의되어 있는 함수들(ajax, timer, DOM Events)의 콜백 함수들이 비동기로 실행된다.


위 예제에 몇가지를 더 추가해보자.

```javascript
function test1(){
	delay();
	console.log("test1");
}
function test2(){
	delay();
	test1();
	console.log("test2");
}
function delay(){
	for(var i=0; i<1000000;i++);
}

function asyncFunc(){
	console.log("asyncFunc");
}

setTimeout(asyncFunc, 10);

test2();
```

Call Stack에는 전역환경의 코드 순서대로 setTimeout - test2() 쌓인다.
하지만 출력 순서는 위와 같지 않다. 즉, `asyncFunc`가 백만번 도는 for문보다 먼저 실행되지 않는다. 즉, 10ms가 지나도 `test2`에서 `test1`을 실행하기 전까지는 `asyncFunc`가 절대 실행되지 않는다.

그러므로 위 예제는 `"asyncFunc"`가 가장 마지막에 출력된다.

그림으로 다시 이해해보자.

![concurrency model 4](./images/concurrency_model_4.png)

근데 여기서 궁금한 점은

- 이벤트 루프가 현재 실행 중인 태스크가 있는지 없는지 어떻게 **실시간으로** 확인하는가?

MDN의 이벤트 루프에 관한 글에 첨부된 가상의 코드를 보자.

```javascript
while(queue.waitForMessage()){
	queue.processNextMessage();
}
```

`waitForMessage()` 는 큐가 비어있을 때 태스크가 큐에 추가될 때까지 대기한다.
while문을 통해 태스크가 추가될 때까지 계속 반복한다.

이 때, 비동기 API들의 작업이 완료 되어 태스크가 추가되면
이벤트 루프는 콜 스택이 비었을 때, 태스크 큐의 첫번째 태스크를 Call Stack으로 가져와 가져와 실행한다. (이 때, 이벤트 큐에 대기하는 이벤트들은 **한 번에 하나씩** Call Stack으로 호출되어 처리된다.)


## Summary

- JS 엔진이 단일 Call Stack을 사용한다는 점에서 `싱글 스레드`로 동작한다는 말은 맞다.
- 하지만 `Event Loop`를 이용하여 비동기 방식으로 동시성(Concurrency)을 지원한다.
- Web API의 timer, ajax, DOM Events 등이 가진 콜백 함수들은 비동기로 실행된다. 태스크 큐는 콜백 함수들이 대기하는 큐다.
	- 이 때, Event Loop는 Call Stack이 비워질 때 큐에서 콜백 함수를 FIFO로 꺼내와서 Call Stack에서 처리한다.


## 참고

[자바스크립트와 이벤트 루프 nhn ent.](http://meetup.toast.com/posts/89)

[Concurrency model and Event Loop from mdn](https://developer.mozilla.org/ko/docs/Web/JavaScript/Memory_Management)