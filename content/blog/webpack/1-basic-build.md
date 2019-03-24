---
title: '[나만의 웹팩 만들기] 1. basic build'
date: 2019-2-2 23:02:30
category: webpack
---

> 코드를 보려면 [이동](https://github.com/hoilzz/create-react-packzz/tree/1-basic-build)

웹팩에 대해 간단히 알아보기 위해 [concepts](https://webpack.js.org/concepts/)를 요약한 내용은 다음과 같다.

**webpack** 은 모던 자바스크립트 앱을 위한 **정적 모듈 번들러** 다. 웹팩이 앱을 처리할 때, 내부적으로 프로젝트에 필요한 모든 모듈을 mapping하는 [의존성 그래프](https://webpack.js.org/concepts/dependency-graph/)를 만들고 1게 이상의 번들을 생성한다.
