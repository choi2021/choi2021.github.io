---
title: "원티드 프리온보딩 사전과제 8편"
date: 2022-10-12
slug: 원티드-프리온보딩-과제-8
tags: [원티드프리온보딩]
series: 원티드프리온보딩
---

## 최적화를 위한 todo Btn 컴포넌트 분리

TodoItem은 memo를 통해 최적화를 해두었기 때문에 투두 아이템에 클릭을 하면 다른 투두아이템들이 re-rendering되는 것은 막았다. 하지만 같은 TodoItem 내부에서 클릭되지 않은 버튼들이 다시 re-rendering이 되는 것을 보고, TodoBtn 컴포넌트로 분리하고 memo로 감싸준 다음, 전달해주는 함수들은 useCallback을 통해 불필요한 re-rendering을 막았다.

```jsx
//TodoBtn.jsx

function TodoBtn({ name, clicked, onClick, text }) {
  return (
    <S.Btn name={name} clicked={clicked} onClick={onClick}>
      {text}
    </S.Btn>
  );
}

export default memo(TodoBtn);

//TodoItem.jsx

function TodoItem({
  todoItem: { isCompleted, id, todo },
  todoItem,
  onDelete,
  onUpdate,
}) {
	...

  const onClick = useCallback((e) => {
    const { name } = e.currentTarget;
    if (name === 'cancel') {
      inputRef.current.value = ``;
    }
    setOnModifyMode((prev) => !prev);
  }, []);

  const handleDelete = useCallback(() => {
    onDelete(id);
  }, [id, onDelete]);

  const handleCompleteUpdate = useCallback((e) => {
    const { name } = e.currentTarget;
    if (name === 'complete') {
      setUpdated((prev) => {
        return { ...prev, isCompleted: true };
      });
    } else {
      setUpdated((prev) => {
        return { ...prev, isCompleted: false };
      });
    }
  }, []);

  const handleSubmit = useCallback(() => {
    const todo = inputRef.current.value;
    if (!todo) {
      setIsBlank(true);
      return;
    }
    onUpdate({ ...updated, todo });
    inputRef.current.value = ``;
    setOnModifyMode((prev) => !prev);
    setIsBlank(false);
  }, [onUpdate, updated]);

  return (
    <S.TodoLayout>
     ...
      <S.RightBox>
        {onModifyMode ? (
          <>
            <div>
              <TodoBtn
                name='complete'
                clicked={updated.isCompleted}
                onClick={handleCompleteUpdate}
                text='Completed🙆‍♀️'
              ></TodoBtn>
              <TodoBtn
                name='not yet'
                clicked={!updated.isCompleted}
                onClick={handleCompleteUpdate}
                text='Not yet 🙅‍♂️'
              ></TodoBtn>
            </div>
            <TodoBtn name='cancel' onClick={onClick} text='취소하기'></TodoBtn>
            <TodoBtn
              name='submit'
              onClick={handleSubmit}
              text='제출하기'
            ></TodoBtn>
          </>
        ) : (
          <>
            <TodoBtn
              text={isCompleted ? 'Completed🙆‍♀️' : 'Not yet 🙅‍♂️'}
            ></TodoBtn>
            <TodoBtn
              name='modify'
              onClick={onClick}
              text={'수정하기'}
            ></TodoBtn>
          </>
        )}
        <TodoBtn onClick={handleDelete} text={'삭제하기'}></TodoBtn>
      </S.RightBox>
    </S.TodoLayout>
  );
}

export default memo(TodoItem);

```

## fetch를 이용한 에러핸들링

#### 문제점

​ 기존 코드에서 exceptionTest라는 함수는 에러가 발생했던, 성공을 했던 data를 받아서, 성공과 실패를 함께 처리했다. 하지만 이렇게 처리하게 되면 성공과 실패가 로그인과 회원가입 각각에 맞게 엉켜있고 너무 많은 일을 하다 보니, 받아야할 변수가 많아 이해하기 어렵고 수정하기도 어려웠다.

```javascript
const exceptionTest = useCallback(
  (data, setMessage, process) => {
    let result = { message: "", success: false }
    if (data.statusCode >= 400) {
      if (data.statusCode === 401) {
        result = {
          message: "이메일 혹은 비밀번호를 확인해주세요.",
          success: false,
        }
      } else {
        result = {
          message: data.message,
          success: false,
        }
      }
    } else {
      if (process === "login") {
        navigate("/todo")
        localStorage.setItem("access_token", data.access_token)
      }
      result = {
        message: `${
          process === "login" ? "로그인" : "회원가입"
        }에 성공했습니다`,
        success: true,
      }
    }

    setMessage(prev => {
      return {
        ...prev,
        ...result,
      }
    })
  },
  [navigate]
)
```

#### 해결방법

이런 문제를 해결하고 싶어서 에러핸들링에 대한 자료를 계속해서 찾고 있었지만 정확히 이해가 되지 않아서 정리하지 못하고 있었다가, 타입스크립트에서 fetch를 이용해 에러 핸들링을 하신 글을 보게 되었다. ( https://www.rinae.dev/posts/how-to-handle-errors-2)

코드는 다음과 같았다.

```typescript
class CustomError extends Error {
  name: string
  constructor(message?: string) {
    super(message)
    this.name = new.target.name
    Object.setPrototypeOf(this, new.target.prototype)
    Error.captureStackTrace && Error.captureStackTrace(this, this.constructor) // 하는 김에 스택트레이스도 바로잡아줍시다
  }
}
class HTTPError extends CustomError {
  constructor(
    private statusCode: number,
    private errorData: Record<string, any>,
    message?: string
  ) {
    super(message)
    this.name = ‘HTTPError’
  }
  get rawErrorData() {
    return this.errorData
  }
  get codeToErrorMessage() {
    switch (this.statusCode) {
      case 401:
        return `You don’t have a permission.`
      // ...
    }
  }
}

```

Error 객체를 상속하는 CustomError 클래스를 만들고, CustomError클래스를 상속하는 HTTPError클래스를 만들어서 해당 statusCode에 따른 결과를 전달해주는 방식이었다.

HTTPError클래스는 getter를 이용해 codeToErrorMessage라는 함수를 만들었는데 getter를 이용한 이유는 클래스의 statusCode값이 바뀔 때 같이 업데이트 될 수 있게 하기 위해서라고 생각되었고, 직접적인 수정을 막기위해서 getter로 읽을 수만 있게 설정했다고 이해했다.

#### 적용해보기

이 글을 제대로 이해하고 이렇게 한번 해보기로 마음먹었을 때 시간이 저녁 11시여서, 12시가 되기 전 제출해야했기 때문에 우선 HTTPError클래스만 Error를 상속해 만들었다.

Error객체를 상속할 때는 반드시 message를 전달해주어야하기에 super를 통해 상속한 Error객체에 전달하고, statusCode를 constructor함수를 이용해서 저장한다. constructor를 통해 저장한 statusCode에 따라 message상태에 전해줄 객체를 반환하게 함수를 만들었다.

클래스로 분리함으로써 HTTP에러가 발생했을 때의 로직을 분리해서 관리할 수 있기 때문에 더 가독성이 좋아졌다.

```javascript
//auth.js

class HTTPError extends Error {
  constructor(statusCode, message) {
    super(message)
    this.name = `HTTPError`
    this.statusCode = statusCode
  }

  get codeToErrorMessage() {
    let result = { message: "", success: false }
    switch (this.statusCode) {
      case 400:
        result = {
          message: "동일한 이메일이 이미 존재합니다.",
          success: false,
        }
        break
      case 401:
        result = {
          message: "이메일 혹은 비밀번호를 확인해주세요.",
          success: false,
        }
        break
      case 404:
        result = {
          message: "해당 사용자가 존재하지 않습니다.",
          success: false,
        }
        break
      default:
        throw new Error("Unknown Error")
    }
    return result
  }
}

export async function postSignUp(email, password) {
  try {
    const res = await fetch(`${BASE_URL}/auth/signup`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        email,
        password,
      }),
    })
    if (!res.ok) {
      console.log(`${res.status}에러가 발생했습니다`)
      throw new HTTPError(res.status, res.statusText)
    } else {
      return await res.json()
    }
  } catch (e) {
    return e.codeToErrorMessage
  }
}
```

반환한 객체를 받았던 handleSubmit함수는 각각 login과 회원가입으로 2개의 함수로 분리했다.

분리 후에 성공한 경우는 둘 다 access_token이 들어있는 객체를 전달받기 때문에 access_token이라는 프로퍼티의 유무에 따라 성공과 에러 처리를 하게 했다.

```jsx
//auth.jsx

const handleLoginSubmit = useCallback(
  info => {
    const { email, password } = info
    postSignIn(email, password) //
      .then(data => {
        if ("access_token" in data) {
          navigate("/todo")
          localStorage.setItem("access_token", data.access_token)
        } else {
          setLoginMessage(prev => {
            return {
              ...prev,
              ...data,
            }
          })
        }
      })
  },
  [navigate]
)

const handleRegisterSubmit = useCallback(info => {
  const { email, password } = info
  postSignUp(email, password) //
    .then(data => {
      if ("access_token" in data) {
        data = {
          message: `회원가입에 성공했습니다`,
          success: true,
        }
      }
      setRegisterMessage(prev => {
        return {
          ...prev,
          ...data,
        }
      })
    })
}, [])
```

이렇게 에러와 성공, 로그인과 회원가입을 로직을 분리하니까 더 보기 좋고 이해가 편해졌다.

너무 과도하게 합치는 게 더 이해하기 어렵다는 것을 느꼈고, 에러핸들링을 할 때 customError클래스를 만들어서 처리하는 방법에 대해 배울 수 있었다.
