---
title: "connect의 mapStateToProps는 언제 실행될까. 그리고 리렌더링은 언제 어떻게 발생하는가."
date: 2018-09-01 21:20:00
category: react
---

## react-redux와 connect

[https://react-redux.js.org/using-react-redux/connect-mapstate](https://react-redux.js.org/using-react-redux/connect-mapstate)를 번역한 글이다.

## Connect: mapStateToProps로 데이터 가져오기

`mapStateToProps`는 스토어에서 데이터의 부분을 선택하여 사용한다. 즉, 연결된 컴포넌트가 필요한 데이터만 스토어에서 가져온다. 

- 이것은 스토어 상태가 변할 때마다 호출된다.
- 전체 스토어 상태를 받고 이 컴포넌트가 필요한 데이터 객체만 가져와야한다.

### mapStateToProps and Performance

__반환 값은 컴포넌트가 리렌더할지 결정한다.__

React Redux는 `shouldComponentUpdate(SCU)`를 내부적으로 구현하여, 컴포넌트가 필요한 데이터가 변경 될 때 wrapper component가 리렌더링되도록 한다. 기본적으로, React Redux는 `mapStateToProps`에서 리턴 받은 객체의 내용이 달라졌는지 판단하기 위해 리턴된 객체의 각 필드에 `===` 비교(shallow equality)를 이용한다. 만약 필드가 하나라도 변경되면, 컴포넌트가 리렌더링되고 update된 prop을 전달받을 수 있다. __동일한 참조 값의 변경된 오브젝트를 리턴하는 것은 컴포넌트가 예상대로 리렌더링 되지 않는 일반적인 실수다.__

`connect`로 래핑된 컴포넌트의 행동을 요약하자면,

| |state => stateProps | (state, ownProps) => stateProps|
|:--:|:--:|:--:|
|mapStateToProps사 실행될 떄: | store `state`가 변경될 때|store `state`가 변경되거나 `ownProps`의 필드가 하나라도 달라지면|
|component가 리렌더링 될 때| `stateProps`의 필드가 하나라도 달라지면 | `stateProps`의 필드가 하나라도 달라지거나 `ownProps`의 필드가 하나라도 달라지면|   
#

__only Return New Object References If Needed__

React-Redux는 `mapStateToProps`의 결과값의 변경여부를 체크하기 위해 shallow compare를 한다. 새로운 객체 혹은 새로운 배열 참조 값을 매번 리턴하는 것과 같은 실수는 하기 쉽다. (그리고 이것은 데이터가 실제로 같더라도 리렌더링을 유발한다.)

new object or array reference를 생성하는 일반적인 동작들
- `.map()`, `.filter()`로 새로운 배열 만들기
- .concat으로 배열 합치기
- .slice로 부분적 배열 선택하기
- .assign으로 값 복사하기
- spread oeprator {...oldState, ...newDate}로 값 복사하기

만약 input value가 변경된 경우에만 실행하도록 해주는 memoized selector 함수를 이용하자. value가 변경되지 않으면 `mapStateToProps`는 여전히 동일한 결과 값을 리턴할 거고 `connect`는 re-render를 스킵할 거다.

__Only Perform Expensive Operations When Data Changes__

데이터 변경(transformation)하는 것은 비용이 높다. (보통 새로운 객체 참조값을 리턴하는 경우) `mapStateToProps` 함수가 가능한 빨라지기 위해, 관련 데이터가 변경된 경우에만 복잡한 transformation을 재실행해야 한다.

이러한 접근에 몇가지 방법이 있다.

- 몇몇 transformation은 action creator나 reducer에서 계산되어야 한다. transformed data는 스토어에 유지되어야 한다.
- `render` 메서드에서 transformation이 되어야한다.
- 만약 transformation이 `mapStateToProps` 함수에서 해야한다면, 값이 변경될 때만 transformation 하기 위해 memoized selector function 이용하기를 권장한다.

## Behavior and Gotchas

__mapStateToProps는 Store state가 동일하다면 실행되지 않는다__

`connect`로 생성된 wrapper component는 Redux store를 구독한다. action이 dispatch 될 때마다, `store.getState()`를 호출하고 `lastState === currentState`인지 확인한다. 만약 2개의 state 참조 값이 같다면 `mapStateToProps` 함수를 재실행 하지 않는다. 왜냐하면 store state의 나머지도 변경되지 않았을거라고 판단하기 때문이다. 

`combineReducer` utility 함수는 이것을 최적화하려고 한다. 만약 reducer 조각이 new value를 반환하지 않는다면, combineReducers는 새 object 대신에 old state object를 리턴한다. 이것은 reducer에서 변경이 root state object가 업데이트 되지 않도록 할 수 있다. 그래서 UI는 리렌더링 되지 않는다.

## Summary

react-redux는 SCU를 내부적으로 구현
- 컴포넌트가 필요한 데이터가 변경 될 때 리렌더링 일으킴
- mapStateToProps에서 데이터 변경 유무를 판단하는 방법
  - 리턴된 객체의 각 필드에 대해 `===` 으로 shallow equality 비교
  - 여기서 필드가 하나라도 변경되면 리렌더링

__퍼포먼스__
- 동일한 input 값이더라도 .map, .filter .assign 등과 같이 새로운 참조 값을 리턴하는 것은 mapStateToProps가 shallow compare시 값이 달라졌다고 판단
  - 불필요한 리렌더링 유발
- memoized selector 함수를 이용하여 해결

- transformation은 mapStateToProps에서 하지 않는 것을 권장
  - 해당 연산의 input 값이 안바뀌더라도, state가 변경될 때마다 매번 불필요한 연산을 필요로함.
- 해결책
  - action creator나 reducer에서 연산하여 store에 유지
  - 굳이 mapStateToProps에서 해야한다면 memoized

- `lastState === curentSttate`를 확인하여 mapStateToProps 실행 여부 판단.
- mapStateToProps가 실행되야 한다면 실행 후에, 리턴된 객체에 대해 `===`으로 shallow equality 비교하여 리렌더링 판단.