---
title: mvc를 이용하여 todolist 만들어보기
date: 2017-6-22 23:02:44
category: javascript
---

Javascript 기초 이론 공부 후, 잘 배웠나 테스트 하기 위해 TodoList(based on MVC with vanilla JS)를 만들어보았다.

```
npm install -g http-server
http-server -a 0.0.0.0 -p 3000
```

## 전체 구조

`app.js`를 통해 todolist MVC 구조를 나타내는 코드를 통해 구조를 알 수 있다.
DB는 영구 저장되는 localStorage를 이용하였다.

*app.js*
```javascript
(function(){
    'use strict';
    function App(){
        this.storage = new app.Storage("test9");
        this.model   = new app.Model(this.storage);
        this.template = new app.Template();
        this.view = new app.View(this.template);
        this.controller = new app.Controller(this.model, this.view);
    }
    var todoListApp = new App();
})();
```

## day1

[item list에 추가하기](https://github.com/seaunseen/todolist_mvc_vanila/tree/master/day1)

## day2

[X버튼 누르면 할 일 삭제하기](https://github.com/seaunseen/todolist_mvc_vanila/tree/master/day2)

## day3

[item 완료 체크 toggle](https://github.com/seaunseen/todolist_mvc_vanila/tree/master/day3)

## day4

[item 내용 수정하기](https://github.com/seaunseen/todolist_mvc_vanila/tree/master/day4)

## day5

[완료된 todo 삭제하기](https://github.com/seaunseen/todolist_mvc_vanila/tree/master/day5)