---
title: '[나만의 웹팩 만들기] 11. eslint, prettier로 편안해지기'
date: 2019-2-12 23:02:30
category: webpack
---

> 코드를 보려면 [이동](https://github.com/hoilzz/create-react-packzz/tree/master)

velopert님이 굉장히 잘 [정리](https://velog.io/@velopert/eslint-and-prettier-in-react)해주셔서 링크 참고 부탁드립니다.

위 방식대로 하면 dynamic import에서 eslint가 다음과 같은 에러를 발생시킵니다.

```
Parsing error: Unexpected token import
```

[Eslint README의 What about experimental features?](https://github.com/eslint/eslint#what-about-experimental-features)를 보면 이유를 알 수 있습니다.

요약하자면 EsLint's parser는 공식적으로 최신 ES 표준만 지원합니다. experimental 이거나 비표준(ts) 은 지원하지 않습니다. experimental, non-standard에 대해서는 babel-eslint parser와 eslint-plugin-babel을 사용하세요. ES 표준 으로 (TC39 process에 따라 stage 4) 채택되면, 그 때 core rule에 추가합니다. 여튼 이외에 것들은 다른 parser를 이용하면 됩니다.

## babel-eslint

babel-eslint를 이용하면 eslint가 이해할 수 있도록 babel parser가 먼저 파싱을 진행합니다.

```
npm i -D eslint babel-eslint
```

```js
// .eslintrc.js
module.exports = {
  //...
  parser: 'babel-eslint',
}
```

이케하면 끝납니다.
