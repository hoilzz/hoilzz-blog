---
title: "[번역] Normalizing State Shape"
date: 2018-9-20 14:31:39
category: react
---

회사에서 최근본 / 찜목록 상품에 대해 추가/삭제 기능을 가진 페이지네이션 페이지를 작업했었다. 이 때, 서버에서 내려주는 데이터가 중첩되고 복잡하고 그대로 아무생각없이 그대로 사용하게 됐을 때, 데이터 구조를 개선할 수 없을까라는 생각에 찾은 [포스팅](https://redux.js.org/recipes/structuring-reducers/normalizing-state-shape#normalizing-state-shape)이 있다. 이 포스팅을 읽고 나서 state 구조 개선에 큰 도움을 얻어서 번역을 블로그에 직접 작성해보았다.

중첩된 데이터에 대한 처리. 블로그 에디터는 많은 포스트와 해당 포스트는 많은 댓글을 가질 수 있다. 이 포스트와 댓글은 유저에 의해 작성된다. 아래 예제 데이터를 보자.

```javascript
const blogPosts = [
  {
    id: "post1",
    author: {username: "user1", name:"u1"},
    body: ";..",
    commnets: [
      {
        id: "comment1",
        author: {..},
        comment: ".."
      },
      {
        ...
      }
    ]
  },
  {
    id: "post2",
    ...
  }
]
```

위 데이터 구조는 꽤 복잡하고 몇몇 데이터는 반복된다. 이는 몇 가지 이유에서 우려된다.

- 데이터 조각이 여러 곳에서 중복될 때, 제대로 업데이트가 됐는지 확인하기 어렵다
- 데이터의 중첩은 리듀서의 로직 또한 중첩되고 복잡해짐을 의미한다. 특히 깊은 곳의 데이터를 업데이트하는 것은 매우 지저분하다.
- 불변 데이터의 업데이트는 모든 조상들이 카피되고 갱신되어야 하고 새로운 객체 참조는 연결된 UI 컴포넌트를 리렌더링한다. 그래서 깊이 중첩된 데이터 업데이트시 관계없는 UI 컴포넌트가 강제 리렌더링될 수 있다.

위와 같은 이유 때문에, **스토어의 중첩되거나 연관된 데이터 처리시 스토어를 데이터베이스의 일부처럼 정규화된 형태로 유지하는 방법**을 권장한다.

## 정규화된 상태 설계

정규화

- 각 데이터 타입은 자신의 테이블을 가진다
- 데이터 테이블은 항목의 아이디를 Key로, 항목들을 값으로 가지는 개별 항목 아이템을 저장한다
- 개별 항목에 대한 참조는 항목의 ID를 저장하여 수행한다.
- 배열의 ID는 순서를 나타낸다.

```javascript
{
    posts : {
        byId : {
            "post1" : {
                id : "post1",
                author : "user1",
                body : "......",
                comments : ["comment1", "comment2"]
            },
            ...
        }
        allIds : ["post1", "post2"]
    },
    comments : {
        byId : {
            "comment1" : {
                id : "comment1",
                author : "user2",
                comment : ".....",
            },
            "comment2" : {
                id : "comment2",
                author : "user3",
                comment : ".....",
            },
            ...
        },
        allIds : ["comment1", "comment2", "comment3", "commment4", "comment5"]
    },
    users : {
        byId : {
            "user1" : {
                username : "user1",
                name : "User 1",
            },
            ...
        },
        allIds : ["user1", "user2", "user3"]
    }
}
```

첫 코드와 비교해서 구조가 전체적으로 flat하다. 초기 구조에 비해 개선된 점은

- 각 아이템은 한곳에서 정의되있기 때문에, 아이템이 업데이트되면 여러곳에 반영해줘야한다.
- 리듀서 로직은 깊은 레벨의 중첩을 처리하지 않아도 되기때문에 더 심플해졌다.
- 특정 아이템을 컴포넌트에 가져오거나, 업데이트하는 로직이 꽤 심플하고 일관성 있다. 특정 아이템을 찾기위해 다른 객체를 통해 디깅할 필요 없이, 주어진 아이템의 type과 id를 통해 몇가지 간단한 단계에서 직접 접근할 수 있다. 
- 각 데이터 타입은 분리되었기 때문에, comment 텍스트를 변경하는 것과 같은 업데이트는 comments.byId.comment의 새로운 카피본만 필요로 한다. 이것은 UI의 작은 부분만 업데이트하면 된다.(전체 UI 업데이트할 필요가 없다.) 반대로, 원래 중첩된 모양의 comment를 업데이트 하면, comment 객체, 부모인 post 객체, 모든 포스트 객체를 값으로 가진 배열을 업데이트를 해야한다. 이것은 모든 Post Component, Comment component의 불필요한 렌더링을 유발한다.

__normalized state 구조는 일반적으로 더 많은 컴포넌트에 연결되고, 각 컴포넌트는 자기가 가진 데이터에 대해서만 책임을 갖는다.__
반대로 기존의 중첩된 데이터 구조의 경우, 극소수의 connected components들이 엄청 큰 데이터에 접근하고 그 데이터를 아래(하위 컴포넌트)로 전달해야한다. 

연결된 부모 컴포넌트를 여러개 가져서, 단순히 아이템의 IDs를 연결된 자식 컴포넌트에 전달하도록 하는것

이것은 React Redux application에서 UI 성능을 최적화하는 좋은 패턴이다. 그래서 state를 normalized하게 유지하는 것은 퍼포먼스를 개선하는 핵심이다.


## state에서 정규화된 데이터 구성하기

```javascript
{
    simpleDomainData1: {....},
    simpleDomainData2: {....}
    entities : {
        entityType1 : {....},
        entityType2 : {....}
    }
    ui : {
        uiSection1 : {....},
        uiSection2 : {....}
    }
}
```

관계있는 데이터와 그렇지 않은 데이터가 공존. 다른 데이터 타입이 어떻게 구성되어야 하는지에 대한 규칙이 정확히 하나만 있는것이 아니다. 하지만, 이 중 하나는 관계있는 테이블을 `entities`와 같은 일반적인 부모 키 아래에 넣는 패턴이다.

위 예제는 여러가지 방법으로 확장될 수 있다. 예를 들어 많은 수정이 일어나는 애플리케이션은 2가지 상태 "테이블"을 유지하고 싶을거다. "현재"항목 값과 "진행단계" 항목 값이다. UI의 다른 부분이 원래 버전을 참조하는 동안 해당 데이터로 편집 폼을 제어할 수 있다. 편집 폼을 재지정하는 것은 그저 "진행 단계"의 항목을 지우고 "현재"섹션의 이전 데이터를 "진행단계"로 다시 복사하기만 하면 된다. 편집을 "적용"하는 동안 "진행 단계"섹션의 값을 "현재"섹션으로 복사해야 한다.

## Relationships and Tables

왜냐하면 Redux store를 `database`로 다루기 때문에, database 설게의 많은 원칙이 동일하게 적용된다.
예컨대, 만약 다대다 관게를 가진다면, 각 테이블의 기본키 ID를 저장하는 `연결 테이블`을 모델링 할 수 있다.(`join table` or `associatvie tabl`로 알려져있다). 일관성을 위헤, 실제 테이블에 사용한 것과 동일한 `byId`와 `allIds`를 사용한다.

```ts
{
    entities: {
        authors : { byId : {}, allIds : [] },
        books : { byId : {}, allIds : [] },
        authorBook : {
            byId : {
                1 : {
                    id : 1,
                    authorId : 5,
                    bookId : 22
                },
                2 : {
                    id : 2,
                    authorId : 5,
                    bookId : 15,
                },
                3 : {
                    id : 3,
                    authorId : 42,
                    bookId : 12
                }
            },
            allIds : [1, 2, 3]
        }
    }
}
```

`look up all books by this author`와 같은 operation은 join table의 single loop으로 쉽게 가져올 수 있다. 

## Normalizing Nested Data

API는 데이터를 nested form으로 보내기 때문에, state tree에 포함되기 전에 normalized shape으로 변경되야 한다. 스키마 타입과 relation을 정의하고, Normalizr에게 response data와 schema를 전달하여 정규화된 응답 데이터를 얻을 수 있다. 

