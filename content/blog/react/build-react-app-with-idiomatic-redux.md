---
title: "dan의 build-react-app-with-idiomatic-redux 강의 정리"
date: 2018-09-10 21:20:00
category: react
---

[dan의 building-react-applications-with-idiomatic-redux](https://egghead.io/courses/building-react-applications-with-idiomatic-redux)을 보며 새롭게 배운것들만 따로 정리.

전체 코드 및 해석된 대본 정리는 [advanced Redux](https://github.com/hoilzz/advancedredux) 에서 참고 가능.

## 1. Colocating Selectors with Reducer

```js
// visibleTodoList.js
...
import { getVisibleTodos } from 'Reducers/todos/'

const mapStateToProps = (state, { params }) => {
  todos: getVisibleTodos(state.todos, params.filter || 'all');
};
```

visibleTodos 를 가져오기 위해, getVisibleTodos라는 셀렉터를 사용한다.
이 때 전달되는 파라미터는 state.todos라는 조각이다.

이 때, state 구조가 바뀌면(정확히는 todos), getVisibleTodos함수를 사용하는 곳의 코드를 전부 변경해야한다.

그래서 public API와, private API로 나눠보자.

public API는

- 외부에서(컴포넌트에서) 접근 가능한 API
- 셀렉터를 사용하는쪽은 state만 전달해주면 된다. 즉, selector를 사용하는 컴포넌트는 selector를 사용하기 위해, 아무것도 몰라도됨. 걍 state만 전달하면된다.
  - 이렇게 하면, state 구조가 바뀌더라도 코드 변경 불필요하다.
  - 컴포넌트 측은 필요한 것을 얻기위해 어떠한 정보(어떤 state 조각을 넘겨야 할지)도 알필요 없이 state만 전달하면됨. (connect의 mapStateToProps 함수를 통해 전체 app 상태 정보를 알고있음)
- 이것은 rootReducer. 즉, 서브 리듀서들이 최종 통합되는 곳(`reducer/index`)에서 작성한다.

private API는

- 외부에서(컴포넌트에서) 접근 불가한 API
- 복잡한 로직(반복문을 통해 visible한 todo 찾기)은 private API에서 책임진다.

위와 같은 규칙을 통해 selector를 다시 작성해보자.

```js
// visibleTodoList.js
...
import { getVisibleTodos } from 'Reducers/'

const mapStateToProps = (state, { match: { params } }) => {
  const filter = params.filter || 'all';
  return {
    todos: getVisibleTodos(state, filter),
    filter,
  };
};
```

```js
// reducer/index.js
const todoApp = combineReducers({
  todos,
  visibilityFilter,
})

export default todoApp

// public API
export const getVisibleTodos = (state, filter) => {
  return fromTodos.getVisibleTodos(state.todos, filter)
}
```

```js
// reducer/todos.js
// private API
export const getVisibleTodos = (state, filter) => {
  switch (filter) {
    case 'all':
      return state
    case 'completed':
      return state.todos.filter(t => t.completed)
    case 'active':
      return state.todos.filter(t => !t.completed)
    default:
      throw new Error(`Unknown filter: ${filter}.`)
  }
}
```

`todos: getVisibleTodos(state, params.filter || 'all')` 이 구조를 만들어서

- state 구조가 변경되더라도 UI 를 사용하는 곳은 state 만 넣어주면 된다.
  - 왜냐하면 해당함수는 셀렉터 함수이기 때문이다.
  - 셀렉터 함수는 전체 어플리케이션 상태에 대한 정보를 알고 있고 해당 로직에 따라 state 만 넣어주면 select 해주기 때문이다.

## 2. Nomarlizing the state shape

State tree 내부에서 `todo` objects의 배열을 todo로 나타낸다.

하지만 real world 에서는

- single array 이상일 수 있고,
- 다른 배열에서 동일한 id를 가진 todo와 sync가 맞지 않을 수도 있다.

```javascript
const todos = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, todo(undefined, action)]
    case 'TOGGLE_TODO':
      return state.map(t => todo(t, action))
    default:
      return state
  }
}
```

### byId 리듀서 추가하기

**state를 db로 다루기, todos를 id에 의해 인덱싱된 obejct로 유지하기**

- reducer 를 `byID`로 리네이밍하기
- todo 추가 할 때 마지막 index에 추가하는 로직, todo toggle시에 map을 통해 상태 변경하는 로직을 사용하지 않고,

  - lookup table 내부의 값을 변경할 것이다.

`TOGGLE_TODO`와 `ADD_TODO`는 동일한 로직을 가진다.

(둘 다 룩업 테이블에서 key 값에 id 를 찾거나 추가, value 는 action 에서 태운 값)

`action.id`의 값이 이전 `action.id`값(`state[action.id]`)과

`action` 으로 todo reducer 를 호출한 결과(`todo(state[action.id], action)`)가 되도록 하는 new lookup table 을 리턴할거다.

아래 코드 중 이부분을 말한다. `[action.id]: todo(state[action.id], action),`

```javascript
const byId = (state = {}, action) => {
  switch (action.type) {
    case 'ADD_TODO':
    case 'TOGGLE_TODO':
      return {
        ...state,
        [action.id]: todo(state[action.id], action),
      }
    default:
      return state
  }
}
```

### allIds reducer 추가하기

위에서 작성한 byId todos를 id를 키로 가지고 todo에 대한 정보를 값으로 가지는 map으로 유지한다.

allIds 리듀서는 ADD_TODO만 신경쓴다.

그 이유는 new todo가 추가되면 new id를 가진 새로운 ids 배열을 리턴해야한다.(물론, 삭제 기능도 추가하면 얘도 포함..)

```js
const allIds = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.id]
    default:
      return state
  }
}
```

2가지 리듀서 byId, allIds 를 생성했다.
이 리듀서는 todos.js에서 다시 comebine 된다.

### Updating getVisible Todos selector

reducer에서 state shape을 변경했다. 변경된 state shape을 의존하는 selector를 업데이트해보자.

todos의 배열을 더이상 사용하지 않기 때문에(위에서 우리가 normalize해서 state.todos 배열이 없어짐), 배열을 생성하는 `getAllTodos` 셀렉터를 만들자.

`getAllTodos`는 todos.js 에서만 사용하기 때문에 `allIds`와 `byId` lookup table 을 매핑한 결과를 리턴하자.

```js
const getAllTodos = state => state.allIds.map(id => state.byId[id])
```

`getVisibleTodo` 셀렉터도 수정하자.

```js
export const getVisibleTodos = (state, filter) => {
  const allTodos = getAllTodos(state)
  switch (filter) {
    case 'all':
      return allTodos
    case 'completed':
      return allTodos.filter(t => t.completed)
    case 'active':
      return allTodos.filter(t => !t.completed)
    default:
      throw new Error(`Unknown filter: ${filter}.`)
  }
}
```

`allTodos`는 컴포넌트가 예측하는 todos 의 배열이다. getVisibleTodos를 사용하는 측에서는 동일한 결과를 리턴해주기 때문에 바뀐 리듀서 및 state 구조에 대해 대응할 필요가 없다.

### 더 나아가서, todo라는 서브 리듀서로 리듀서 쪼개기

todos.js는 갱장히 커졌다. single todo를 관리하는 파일을 생성하자.

todo.js는 todos.js의 서브 리듀서다.

---

## 3. 서버에서 내려준 데이터로 state 업데이트하기.

지금까지는 서버에서 내려준 데이터 없이 브라우저단에서만 데이터를 관리했다.

```js
export const getVisibleTodos = (state, filter) => {
  const allTodos = getAllTodos(state)
  switch (filter) {
    case 'all':
      return allTodos
    case 'completed':
      return allTodos.filter(t => t.completed)
    case 'active':
      return allTodos.filter(t => !t.completed)
    default:
      throw new Error(`Unknown filter: ${filter}.`)
  }
}
```

서버에서 todos를 내려준다고하자. 위 리듀서 구조는 todos를 전부 내려줄 때만 사용가능하다. 왜냐하면 모든 구조가 allTodos.으로 되있으니까.. 또, **만약 todos가 천단위로 내려온다고 할 때 전부 가져와서 필터링하는 것은 실용적이지 않다.**

### getVisibleTodos 리팩토링

엄청나게 큰(1000개가 들어있는) ids를 유지하는 것보다,

- every filter's tab에 대해 `id`s 리스트를 유지하는 것이 낫다.
- 그래서 그것들은 분리되서 저장되고 fetched된 데이터와 함께 action에 따라 채워질 것이다.

위에서 작성한 `getAllTodos` 셀렉터를 지우자. `allTodos`는 더이상 사용하지 않는다.

```js
export const getVisbileTodos = (state, filter) => {
  const ids = state.idsByFilter[filter]
  return ids.map(id => state.byId[id])
}
```

### todos 리팩토링

기존 todos 구조

```js
const todos = combineReducers({
  byId,
  allIds,
})
```

allIds는 더 이상 사용하지 않기 떄문에, 다음과 같이 변경

todos.allIds가 아닌, todos.idsByFilter[all, active, completed]로 분리.

```js
const todos = combineReducers({
  byId,
  idsByFilter,
})

const idsByFilter = combineReducers({
  all: allIds,
  active: activeIds,
  completed: completedIds,
})
```

### allIds 리듀서 업데이트하기.

```js
const allIds = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.id]
    default:
      return state
  }
}
```

allIds 리듀서는 `ADD_TODO` action과 ids 배열을 관리했다.

이제 ADD는 서버에 todo를 추가하라는 요청을 보내기 때문에, 직접 추가할 필요는 없다.

```js
const allIds = (state = [], action) => {
  switch (action.type) {
    case 'RECEIVE_TODOS':
      return action.response.map(todo => todo.id)
    default:
      return state
  }
}
```

그리고 ADD_TODO는 서버에서 수행하고 수행 결과를 받기 때문에 action명을 `RECEIVE_TODOS`로 변경하자.

서버에서 todos 배열을 받은 후, todo에서 id만 셀렉트해서 추가할거다.

### activeIds reducer 생성하기

각 filter의 ids 리듀서를 추가할거다. (activeIds, completedIds)
allIds와 동일한 역할을 한다. `RECEIVE_TODOS` 액션 디스패치될 때, filter 인자를 추가로 전달해줄거다.
이 filter와 일치한 ids reducer만 업데이트 될 것이다.

_activeIds.js_

```js
const activeIds = (state = [], action) => {
  if (action.filter !== 'active') {
    return state
  }
  switch (action.type) {
    case 'RECEIVE_TODOS':
      return action.response.map(todo => todo.id)
    default:
      return state
  }
}
```

_completedIds.js_

```js
const completedIds = (state = [], action) => {
  if (action.filter !== 'completed') {
    return state
  }
  switch (action.type) {
    case 'RECEIVE_TODOS':
      return action.response.map(todo => todo.id)
    default:
      return state
  }
}
```

### byId reducer도 리팩토링

_before byId.js_

```js
const byId = (state = {}, action) => {
  switch (action.type) {
    case 'ADD_TODO':
    case 'TOGGLE_TODO':
      return {
        ...state,
        [action.id]: todo(state[action.id], action),
      }
    default:
      return state
  }
}
```

서버의 응답값의 todos를 처리하기 위해 byId reducer도 업데이트하자.

```js
const byId = (state = {}, action) => {
  switch (action.type) {
    case 'RECEIVE_TODOS':
      const nextState;
      action.response.forEach(todo => {
        nextState[todo.id] = todo;
      });
      return nextState;
    default:
      return state;
  }
};
```

## 4. reducer 반복 코드 factory 패턴으로 줄이기

아래 코드를 잘 보면 반복되는 코드가 많다.

```js
const allIds = (state = [], action) => {
  if (action.filter !== 'all') {
    return state
  }
  switch (action.type) {
    case 'RECEIVE_TODOS':
      return action.response.map(todo => todo.id)
    default:
      return state
  }
}

const activeIds = (state = [], action) => {
  console.log('activeIds action: ', action)
  if (action.filter !== 'active') {
    return state
  }
  switch (action.type) {
    case 'RECEIVE_TODOS':
      return action.response.map(todo => todo.id)
    default:
      return state
  }
}

const completedIds = (state = [], action) => {
  if (action.filter !== 'completed') {
    return state
  }
  switch (action.type) {
    case 'RECEIVE_TODOS':
      return action.response.map(todo => todo.id)
    default:
      return state
  }
}
```

반복되는 ids 리듀서 코드 factory 패턴을 이용해 줄이기.

```js
// createList.js
const createList = filter => {
  return (state = [], action) => {
    if (action.filter !== filter) {
      return state
    }

    switch (action.type) {
      case 'RECEIVE_TODOS':
        return action.response.map(todo => todo.id)
      default:
        return state
    }
  }
}

// todos.js -> 이제는 index.js
const listByFilter = combineReducers({
  all: createList('all'),
  active: createList('active'),
  completed: createList('completed'),
})
```

요청완료 전까지 loading 을 보여주기 위해 각 ids에 isfetching 추가

```js
const createList = filter => {
  const ids = (state = [], action) => {
    // ...
  }

  const isFetching = (state = false, action) => {
    if (action.filter !== filter) {
      return state
    }
    console.log(action.type)
    switch (action.type) {
      case 'REQUEST_TODOS':
        return true
      case 'RECEIVE_TODOS':
        return false
      default:
        return state
    }
  }

  return combineReducers({
    ids,
    isFetching,
  })
}
```

## 5. avoiding-race-condition

요청 전에 isFething flag를 통해 요청 중일 경우 요청하지 않고 즉시 return;

```js
export const fetchTodos = (filter) => (dispatch, getState) => {
  if (getIsFetching(getState(), filter)) {
    return Promise.resolve();
  }
```

## 요약본

1. Colocating Selectors with Reducer

component에서 state의 값들을 가져올 때 selector를 적극 활용하자.

- 재사용 가능 및 유지보수 용이
  - component에서 직접 state접근하여 가져올 경우, state 구조가 변경될 때마다 다 수정해줘야함.
- 캡슐화
  - state의 구조와 state에 접근하기 위한 로직을 selector 함수에 은닉
  - state를 사용하는 component 입장에서는 단순히 호출만 하면됨.
    - 즉, 컴포넌트는 todos를 달라고할 때, state.todos를 전달(**"state안에있는 todos라는거 있는데 그거 줘"**)
    - 가 아닌, **todos 내놔** 라고 하면 selector 함수 내부에서 ("todos는 state.todos에 있으니까 이거 줄께")가 되야함. (selector에 대한 책임을 컴포넌트가 나눠갖는게 아닌 selector에게 일방적으로 강요 하면서 의존성 약하게 할 수 있음)

fake API 에서 0.5 -> 5 초 delay 를 줬을 때, 문제가 있었다. 요청 시작하기 전에 탭 로딩 여부를 체크하지 않았다. 그래서 `receiveTodos` action 이 다시 시작되고 잠재적으로 경쟁 조건이 발생할 수 있다.

이걸 고치기 위해, **주어진 필터의 todos 를 이미 fetching 중이라는걸 알게 되면 `fetchTodos` action creator 에서 조기 종료**할 수 있다.

`fetchTodos` 내부에서, 현재 store state 와 filter 를 인자로 받는 `getIsFetching` selector 를 이용하여 fetching 여부를 판단하기 위해 `if`를 추가 할 것이다. 만약 `true`를 리턴하면, action dispatching 없이 thunk 에서 조기 종료할 것이다.

```js
export const fetchTodos = filter => (dispatch, getState) => {
  if (getIsFetching(getState(), filter)) {
    return Promise.resolve()
  }
  dispatch(requestTodos(filter))
  return api
    .fetchTodos(filter)
    .then(response => dispatch(receiveTodos(filter, response)))
}
```

State Shape의 변화

1. API 통신 없이, 단일 todos

```js
const todos = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, todo(undefined, action)]
    case 'TOGGLE_TODO':
      return state.map(t => todo(t, action))
    default:
      return state
  }
}
```

2. byId, allIds를 통해 분리해서 관리.

- byId는 todo의 id를 key로 가지고 todo에 대한 정보를 값으로 갖는 lookup table
- allIds는 id들의 배열

```js
const todos = combineReducers({
  byId,
  allIds,
})

const allIds = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.id]
    default:
      return state
  }
}

const byId = (state = {}, action) => {
  switch (action.type) {
    case 'ADD_TODO':
    case 'TOGGLE_TODO':
      return {
        ...state,
        [action.id]: todo(state[action.id], action),
      }
    default:
      return state
  }
}
```

3. server API 통신을 통해 todo 관리

- 서버에서 1000개이상의 todo를 내려받을 경우 1개의 ids를 유지하는 것보다, 필터별로 관리하는게 낫다.

```js
const todos = combineReducers({
  byId,
  idsByFilter,
});

const idsByFilter = combineReducers({
  all: allIds,
  active: activeIds,
  completed: completedIds,
});

const allIds = (state = [], action) => {
  switch (action.type) {
    case 'RECEIVE_TODOS':
      return action.response.map(todo => todo.id);
    default:
      return state;
  }
};

const activeIds = (state = [], action) => {
  if (action.filter !== 'active') {
    return state;
  }
  switch (action.type) {
    case 'RECEIVE_TODOS':
      return action.response.map(todo => todo.id);
    default:
      return state;
  }
};

const completedIds = (state = [], action) => {
  if (action.filter !== 'completed') {
    return state;
  }
  switch (action.type) {
    case 'RECEIVE_TODOS':
      return action.response.map(todo => todo.id);
    default:
      return state;
  }
};

const byId = (state = {}, action) => {
  switch (action.type) {
    case 'RECEIVE_TODOS':
      const nextState;
      action.response.forEach(todo => {
        nextState[todo.id] = todo;
      });
      return nextState;
    default:
      return state;
  }
};
```

4. filter별 ids에 반복되는 패턴이 있음. factory 패턴으로 줄이기.

```js
const createList = filter => {
  return (state = [], action) => {
    if (action.filter !== filter) {
      return state
    }

    switch (action.type) {
      case 'RECEIVE_TODOS':
        return action.response.map(todo => todo.id)
      default:
        return state
    }
  }
}

// todos.js -> 이제는 index.js
const listByFilter = combineReducers({
  all: createList('all'),
  active: createList('active'),
  completed: createList('completed'),
})
```

5. API요청시 응답이 늦게 들어올 경우 race-condition 발생.

```js
// action.js
export const fetchTodos = (filter) => (dispatch, getState) => {
  if (getIsFetching(getState(), filter)) {
    return Promise.resolve();
  }

// reducer/createList.js
const createList = filter => {
  const ids = (state = [], action) => {
    // ...
  };

  const isFetching = (state = false, action) => {
    if (action.filter !== filter) {
      return state;
    }
    console.log(action.type);
    switch (action.type) {
      case 'REQUEST_TODOS':
        return true;
      case 'RECEIVE_TODOS':
        return false;
      default:
        return state;
    }
  };

  return combineReducers({
    ids,
    isFetching,
  });
};
```
