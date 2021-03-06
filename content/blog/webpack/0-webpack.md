---
title: '[나만의 웹팩 만들기] 0. webpack 개념'
date: 2019-2-1 23:02:30
category: webpack
---

회사에 fe 환경 구축 cli가 있어서 편하게 사용하고 있었다. 이 cli는 웹팩 v3 환경으로 세팅해주었다. 해당 cli 관련하여 전부 세팅해준 사수님은 이직하고.. 내가 이 업무를 어쩌다 맡게되었습니다. 하고싶은 업무를 하게 되어서 좋았지만 굉장히 어려웠다. 이 업무에 관련하여 공부하기 위해 나만의 웹팩 환경을 구축할 수 있는 cli를 만들었는데, 이것과 관련된 포스트를 작성하려한다.

>나는 이해하기 귀찮고 빨리 써볼래! 하시는 분은 제가 만든 패키지 이용 부탁드립니다. (제발)[create-react-packzz](https://github.com/hoilzz/create-react-packzz)

일단 웹팩에 대해 간단히 알아보기 위해 [concepts](https://webpack.js.org/concepts/)를 요약한 내용은 다음과 같다.

**webpack** 은 모던 자바스크립트 앱을 위한 **정적 모듈 번들러** 다. 웹팩이 앱을 처리할 때, 내부적으로 프로젝트에 필요한 모든 모듈을 mapping하는 [의존성 그래프](https://webpack.js.org/concepts/dependency-graph/)를 만들고 1게 이상의 번들을 생성한다.

### Dependency Graph

한 파일이 다른 파일에 의존할 때마다, webpack은 이것을 *dependency*로 생각한다. 이렇게 하면, 웹팩은 non-code 자원(image, web fonts와 같은 것)을 가지고 application의 _dependency_ 로 제공할 수 있다.

entry point(시작점)에서 시작하여, 웹팩은 재귀적으로 앱에 필요한 모든 모듈을 포함하는 의존성 그래프를 빌드한다.

그리고 나서 모든 모듈을 여러개의 작은 번들(대부분 1개)로 만듭니다.
