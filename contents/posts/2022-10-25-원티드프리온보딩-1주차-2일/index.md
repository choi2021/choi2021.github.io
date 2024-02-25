---
title: '원티드 프리온보딩 1주차 첫째주 과제-best case찾기와 보충공부:context API'
date: 2022-10-25
slug: 원티드-프리온보딩-1주차-첫째주-과제-best-case찾기와-보충공부-context-API
tags: [원티드프리온보딩]
series: 원티드프리온보딩
---

# 사전 과제 정리

팀원들의 사전 과제들을 서로 발표하고 어떤 점이 좋은지, 왜 이렇게 했는지 서로 의논하고 피드백을 하는 시간을 삼았다. 발표를 위해 먼저 내가 한 사전 과제를 정리해보고 내가 어떤 점이 부족했는지 보완해서 발표하는 시간을 가졌다.

### 🔨 수정사항

먼저 수정한 부분은 api들을 클래스로 주입해주는 방식이었다. 단순히 함수를 불러와서 쓰는 것이 아니라 index.js에서 instance를 만든 후에 필요한 페이지에 주입하는 방법으로 수정했다. 원래 존재했던 함수들은 method들로 수정했다.

```javascript
export class AuthService {
  constructor(baseURL) {
    this.baseURL = baseURL;
  }
export async function postSignUp(email, password) {
  try {
    const res = await fetch(`${BASE_URL}/auth/signup`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        email,
        password,
      }),
    });
    if (!res.ok) {
      console.log(`${res.status}에러가 발생했습니다`);
      throw new HTTPError(res.status, res.statusText);
    } else {
      return await res.json();
    }
  } catch (e) {
    return e.codeToErrorMessage;
  }
}

}
```

두 번째로 수정한 부분은 httpClient 클래스를 추가했다. httpClient는 반복되는 fetch의 헤더와 바디 부분의 코드 중복을 제거하고, baseURL에 전달받은 url을 더한 url에 option에 따라 fetch를 수행 해주는 클래스다.

#### httpClient

httpClient 클래스는 가지고 있는 baseURL에 전달받은 url과 option에 따라 fetch를 실행해주는 클래스입니다.

fetch는 성공했다면 JWT가 담긴 객체를 반환하고 실패했다면 커스텀 에러 HTTPError를 던지고 메시지를 전달합니다.

```javascript
export class HttpClient {
  constructor(baseURL) {
    this.baseURL = baseURL;
  }

  async fetch(url, options) {
    const res = await fetch(`${this.baseURL}${url}`, {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        ...options.headers,
      },
    });
    try {
      if (!res.ok) {
        console.error(`${res.status}에러가 발생했습니다`);
        throw new HTTPError(res.status, res.statusText);
      } else {
        return await res.json();
      }
    } catch (e) {
      return e.codeToErrorMessage;
    }
  }
}
```

HTTPClient class를 이용함으로써 authService와 todoService는 훨씬 간결한 코드를 가질 수 있다.

```javascript
export class AuthService {
  constructor(httpClient) {
    this.httpClient = httpClient;
  }
  async postSignUp(email, password) {
    return this.httpClient.fetch('/auth/signup', {
      method: 'POST',
      body: JSON.stringify({
        email,
        password,
      }),
    });
  }

  async postSignIn(email, password) {
    return this.httpClient.fetch('/auth/signin', {
      method: 'POST',
      body: JSON.stringify({
        email,
        password,
      }),
    });
  }
}
```

#### Auth의 함수

함수들의 promise들의 체이닝으로 복잡하게 보여 async-await으로 보다 간결한 코드로 수정했다.

```javascript
const handleLoginSubmit = useCallback(
  async (info) => {
    const { email, password } = info;
    const response = await authService.postSignIn(email, password);
    if ('access_token' in response) {
      navigate('/todo');
      localStorage.setItem('access_token', response.access_token);
    } else {
      setLoginMessage((prev) => {
        return {
          ...prev,
          ...response,
        };
      });
    }
  },
  [navigate]
);

const handleRegisterSubmit = useCallback(async (info) => {
  const { email, password } = info;
  const response = await authService.postSignUp(email, password);
  let message = response;
  if ('access_token' in response) {
    message = {
      message: `회원가입에 성공했습니다`,
      success: true,
    };
  }
  setRegisterMessage((prev) => {
    return {
      ...prev,
      ...message,
    };
  });
}, []);
```

### 투두 페이지

todo도 동일하게 함수상태로 api호출을 하는 게 아니라 todoService 클래스를 만들었고 httpClient를 이용해 api 호출 해 기존의 코드중복을 제거했다.

클래스로 감싸주었기 때문에 간단하게 method이름들도 수정했다.

```javascript
export class TodoService {
  constructor(httpClient) {
    this.httpClient = httpClient;
  }
  async create(todo) {
    return this.httpClient.fetch('/todos', {
      method: 'POST',
      headers: {
        Authorization: `Bearer ${localStorage.getItem('access_token')}`,
      },
      body: JSON.stringify({
        todo,
      }),
    });
  }

  async get() {
    return this.httpClient.fetch('/todos', {
      method: 'GET',
      headers: {
        Authorization: `Bearer ${localStorage.getItem('access_token')}`,
      },
    });
  }

  async update(obj) {
    const { todo, isCompleted } = obj;
    return this.httpClient.fetch(`/todos/${obj.id}`, {
      method: 'PUT',
      headers: {
        Authorization: `Bearer ${localStorage.getItem('access_token')}`,
      },
      body: JSON.stringify({
        todo,
        isCompleted,
      }),
    });
  }

  async delete(id) {
    return this.httpClient.fetch(`/todos/${id}`, {
      method: 'DELETE',
      headers: {
        Authorization: `Bearer ${localStorage.getItem('access_token')}`,
      },
    });
  }
}
```

## 2. 피드백

피드백을 주고 받으면서 느꼈던 건 사람들마다 너무 다르게 코드를 구성하고, 그렇기 때문에 초기 설정이 많이 중요하구나라는 부분과 의존적이지 않은 컴포넌트를 만드는 것의 중요성이었다. 각자의 로직에 맞게 제작된 컴포넌트이다 보니 거기에 너무 특화되어 있어 취합하기가 어려울 것 같다는 이야기가 많이 나왔다.

그중에서도 좋은 포인트들로 의논되었던 점들은 독립적으로 사용될 수 있어서 어디에나 적용될 수 있는 로직들이었다.

axios의 interceptor를 이용해 httpClient 클래스 역할을 대신해서 적용하신 분도 계셨고, react 커스텀 hook을 이용해서 httpClient 클래스와 같이 옵션들만 잘 전달하면 어떤 api든 호출할 수 있게 만드신 분도 계셨다. 단순히 토큰을 추가하고 페이지이동을 시켜었던 나와 달리 context API를 이용해서 상태로 보관하고 상태에 따라 로직을 처리해주신 분도 계셨다.

나보다 뛰어난 사람들, 나보다 넓은 관점들을 가지신 분들이 너무 많기 때문에 더 많이 배울 수 있겠다 생각도 들었지만, 정말 가만히 수동적으로 있다면 아무것도 안 늘고 잘해 주시는 분들에게 **묻혀갈 수도 있는 위험성도 느꼈다**. 더 적극적으로 배우고, 더 적극적으로 만들어서 **간절하게 해야 한다**는 점을 더 많이 느꼈다.

발표를 하면서도 내 코드를 **왜 이렇게 짰는지** 설명하는 부분, 누군가가 이해할 수 있게 **정리되어있는 상태**가 된다는 것의 중요성도 느꼈다.

## 3. 공부가 필요한 부분들

피드백을 하면서 리액트를 잘한다고 느꼈던 부분들은 context API와 custom hook이었다. 내 프로젝트에서도 분명 쓸 수 있는 부분들이 있었지만 놓쳤다. 그렇기 때문에 공부가 필요하다고 느껴서 정리를 해보고자 한다.

### 3.1 context API

![post-thumbnail](https://velog.velcdn.com/images/devjade/post/c155b13a-e1b7-4205-9f39-875e1c284454/contextAPI.png)

context API는 prop-drilling (상태를 전달하기 위해 여러 컴포넌트를 거쳐서 전달해주어야 하는 경우)을 막기 위한 API로 전역상태를 만들 수 있다.

흔히 사용되는 경우는 로그인, 언어, 다크모드 등이 있다. 이번에 우리 과제에서 사용할 경우는 로그인으로 직접 코드를 만들어 보았다.

기존 코드에서는 App.jsx에서 상태로 관리되고 있었고 signIn, signUp, todo모두에서 useEffect에서 상태에 따라 redirect하는 기능이 있었다. context API를 이용한다면 반복되는 로직을 없앨 수 있어서 더 좋은 코드가 될 수 있다고 생각되었다.

```react
import { createContext } from 'react';
import { useState } from 'react';

export const LoginContext = createContext();
export function LoginProvider({ children }) {
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  return (
    <LoginContext.Provider value={% raw %}{{ isLoggedIn, setIsLoggedIn }}{% endraw %}>
      {children}
    </LoginContext.Provider>
  );
}

```
