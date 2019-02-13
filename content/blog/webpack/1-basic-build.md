---
title: '[나만의 웹팩 만들기] 1. basic build'
date: 2019-02-02 23:02:30
category: webpack
---

> 코드를 보려면 [이동](https://github.com/hoilzz/create-react-packzz/tree/1-basic-build)

웹팩에 대해 간단히 알아보기 위해 [concepts](https://webpack.js.org/concepts/)를 요약 번역해보겠습니다.

**webpack** 은 모던 자바스크립트 앱을 위한 **정적 모듈 번들러** 다. 웹팩이 앱을 처리할 때, 내부적으로 프로젝트에 필요한 모든 모듈을 mapping하는 [의존성 그래프](https://webpack.js.org/concepts/dependency-graph/)를 만들고 1게 이상의 번들을 생성한다.

> Dependency Graph
> 한 파일이 다른 파일에 의존할 때마다, webpack은 이것을 *dependency*로 생각한다. 이렇게 하면, 웹팩은 non-code 자원(image, web fonts와 같은 것)을 가지고 application의 _dependency_ 로 제공할 수 있다.
> entry point(시작점)에서 시작하여, 웹팩은 재귀적으로 앱에 필요한 모든 모듈을 포함하는 의존성 그래프를 빌드한다.
> 그리고 나서 모든 모듈을 여러개의 작은 번들(대부분 1개)로 만든다.
