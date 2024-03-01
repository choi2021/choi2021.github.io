---
title: '원티드 프리온보딩 사전과제 5편'
date: 2022-10-09
slug: 원티드-프리온보딩-과제-폴더-구조-수정
tags: [원티드프리온보딩]
series: 원티드프리온보딩
---

## 리액트 폴더, 파일 구조

어제 리팩토링에 이어 폴더와 파일구조에 대해 먼저 고민해보았다. 최근에 혼자 프로젝트를 하다보니 파일 구조에 대해서는 크게 생각해본 적이 없었다. 지스트 청원 서비스 같은 경우에는 친구들과 함께 의논하면서 convention을 정해서 했지만 혼자 할 때는 크게 고민을 안했던 부분이라 이번 기회에 한 번 정리하면 좋을 것 같다는 생각이 들었다.

### 1. 리액트 공식 홈페이지

리액트 공식홈페이지에서는 **파일 기능이나 라우트에 의한 분류**와 **파일 유형에 의한 분류** 두가지로 설명하고 있다.

#### 1.1 파일기능이나 라우트에 의한 분류

관련된 기능들을 함께 폴더로 보관하는 방법으로 하나의 컴포넌트에 연관된 것을 묶어서 찾을 수 있다. 복잡한 어플리케이션일 수록 기능별로 다 보관하기 어려워 큰 프로젝트에 참고해서 사용하면 좋겠다는 생각이 들었다.

```
common/
  Avatar.js
  Avatar.css
  APIUtils.js
  APIUtils.test.js
feed/
  index.js
  Feed.js
  Feed.css
  FeedStory.js
  FeedStory.test.js
  FeedAPI.js
profile/
  index.js
  Profile.js
  ProfileHeader.js
  ProfileHeader.css
  ProfileAPI.js
```

#### 1.2 파일유형에 의한 분류

내가 가장 많이 사용하는 방식이라 생각된다. Api는 api끼리 component들은 componet들끼리 묶어서 정리해두는 방식으로 개인 프로젝트와 같은 소규모 프로젝트에 유용하다고 생각이 되었다.

```
api/
  APIUtils.js
  APIUtils.test.js
  ProfileAPI.js
  UserAPI.js
components/
  Avatar.js
  Avatar.css
  Feed.js
  Feed.css
  FeedStory.js
  FeedStory.test.js
  Profile.js
  ProfileHeader.js
  ProfileHeader.css
```

## 2. Styled-components

폴더 구조에 대해서는 원래 고민을 하지 않았지만 styled-components를 사용하면서 고민이 시작되었다. styled-components는 CSS-in-javascript로 자바스크립트 파일 내에서 css를 사용하기 때문에, **스타일링을 위한 코드와 기능을 위한 코드를 어떻게 분리하면 좋을 까** 고민이 되었다. 찾아본 결과는 컴포넌트 폴더를 만들고 내부에 styles.js를 만들어서 모듈 시스템으로 전달해주는 방식이었다.

[사진 첨부: ghoon99님의 블로그](https://ghoon99.tistory.com/46)

![사진 첨부:ghoon99님의 블로그](https://blog.kakaocdn.net/dn/bl8El0/btrigf0IB7G/luN4wBBYzm4hcNBXsWc7sK/img.png)

위의 구조를 이용해 정리해 다음과 같은 폴더 구조로 수정했다. 스타일링을 위한 코드내용을 분리하니 훨씬 가독성이 좋아졌다.

```
src
├── components
│   ├── auth_form
│   │      auth_form.jsx
│   │      styles.js
│   └── todo_item
│          todo_item.jsx
│          styles.js
├── pages
│   ├── auth
│   │      auth.jsx
│   │      styles.js
│   └── todo
│          todo.jsx
│          styles.js
├── index.js
└── router.jsx
```

이렇게 코드를 정리하고 나니 styled-componets에서 스타일링을 위한 컴포넌트와 로직을 위한 컴포넌트들이 구분이 잘 안간다는 생각이 들었다. 구분하기 위해 styling을 위한 컴포넌트에 S라는 styling을 위한 오브젝트를 만들고 각 컴포넌트들을 property로 전달하는 방식을 도입하기로 했다. [medium Naming Styled Components](https://medium.com/inturn-eng/naming-styled-components-d7097950a245)

Todo를 예로, 모든 스타일링 component들을 다 styles.js에서 받아와 복잡했던 todo.jsx의 import가 S로 간단해지고, styles.js도 간단하게 object만 export하면 되기에 효율적인 방식이라 생각되었다.

```jsx
//todo/styles.js

const S = { TodoContent, TodoForm, TodoLayout, TodoList };

export default S;


//todo.jsx

import S from './styles';

function Todo() {
  return (
    <S.TodoLayout>
      <header>TO DO</header>
      <S.TodoContent>
        <S.TodoForm onSubmit={onSubmit}>
          <input
            ref={inputRef}
            type='text'
            id='todoInput'
            placeholder='오늘의 할 일을 입력해주세요'
          />
          <button>Add</button>
        </S.TodoForm>
        <S.TodoList>
          {todos.map((item) => (
            <TodoItem
              key={item.id}
              todoItem={item}
              onUpdate={onUpdate}
              onDelete={onDelete}
            ></TodoItem>
          ))}
        </S.TodoList>
      </S.TodoContent>
    </S.TodoLayout>
  );
}

export default Todo;


```
