---
title: "원티드 프리온보딩 사전과제 4편"
date: 2022-10-08
slug: 원티드-프리온보딩-과제-4
tags: [원티드프리온보딩]
series: 원티드프리온보딩
---

어제 과제에서 요구하는 전반적인 요구사항에 대한 기능은 개발했지만 리팩토링을 할 부분을 세부적으로 많이 보였다. 리팩토링 할 부분들을 정리해보면 다음과 같다.

1. 회원가입과 로그인의 코드중복: 만들다 보니 너무 똑같은 styling과 api 사용이 반복되어서 수정이 필요하다.
2. 폴더나 파일 구조 설계: 리액트를 개인프로젝트를 주로하다보니 폴더 설계나 어떤 곳에 어떤 파일을 두어야 좀 더 좋을지에 대한 고민이 부족했다.
3. axios사용: fetch로 기능이 다 구현했으니 axios로 구현해보면 좋은 경험이 될 것 같다.
4. api 예외사항 체크: 미처 발견하지 못한 에러처리에 대해 고민해보고 해결이 필요하다.

## login.jsx 컴포넌트화

### 1. styling 코드중복 제거

기능구현을 위해 우선은 한번에 코딩을 하다보니 login.jsx에 많은 코드중복이 보였다. 우선 /경로에 로그인과 회원가입을 동시에 보여주다보니 login.jsx보다는 **auth.jsx**로 이름을 바꾸고, authForm.jsx로 컴포넌트를 추가해 로그인과 회원가입 모두 같은 구성을 갖게 만들었다.

```jsx
///이전 코드
function Login(props) {
  return (
    <LoginLayout>
      <LoginForm onSubmit={handleLoginSubmit}>
        <h1>Sign In</h1>
        <div>
          <label htmlFor='login-email'>Email</label>
          <LoginInput
       		...
          ></LoginInput>
        </div>
        <div>
          <label htmlFor='login-password'>Password</label>
          <LoginInput
			...
          ></LoginInput>
        </div>
        {message.loginMessage && (
          <Message success={message.loginSuccess}>
            {message.loginMessage}
          </Message>
        )}
        <SubmitBtn
        	...
        >
          Login
        </SubmitBtn>
      </LoginForm>
      <RegisterForm onSubmit={handleRegisterSubmit}>
        <h1>Sign Up</h1>
        <div>
          <label htmlFor='register-email'>Email</label>
          <RegisterInput
        		...
            name='email'
          ></RegisterInput>
        </div>
        <div>
          <label htmlFor='register-password'>Password</label>
          <RegisterInput
            	...
          ></RegisterInput>
        </div>
        {message.registerMessage && (
          <Message success={message.registerSuccess}>
            {message.registerMessage}
          </Message>
        )}
        <SubmitBtn
          disabled={
            !(registerInfo.isEmailValid && registerInfo.isPasswordValid)
          }
        >
          Submit
        </SubmitBtn>
      </RegisterForm>
    </LoginLayout>
  );
}

export default Login;

//수정한 코드
//auth.jsx
function Auth() {
 	...
  return (
    <AuthLayout>
      <AuthForm
       	...
      ></AuthForm>
      <AuthForm
       	...
      ></AuthForm>
    </AuthLayout>
  );
}

export default Auth;

//authForm.jsx
function AuthForm({
  process,
  onSubmit,
  onChange,
  message,
  setMessage,
  info,
  setInfo,
}) {
  	...
  return (
    <AuthFormLayout onSubmit={handleSubmit}>
      <h1>{process === 'login' ? '로그인' : '회원가입'}</h1>
      <div>
        <label htmlFor={`${process}_email`}>Email</label>
        <AuthInput
      		...
        ></AuthInput>
      </div>
      <div>
        <label htmlFor={`${process}_password`}>Password</label>
        <AuthInput
         	...
        ></AuthInput>
      </div>
      {message.loginMessage && (
        <Message success={message.loginSuccess}>{message.loginMessage}</Message>
      )}
      <SubmitBtn
       	...
      >
        Login
      </SubmitBtn>
    </AuthFormLayout>
  );
}

export default AuthForm;


```

### 2. 함수 코드중복 제거

login.jsx내부의 함수들에도 코드중복이 많이 존재했다. handleLoginChange,handleRegisterChange 등 이름과 내용만 조금씩 다른 함수들이 하나의 함수로 최대한 합쳤다.

#### 2.1 handleChange

​ 회원가입과 로그인의 input 값들의 유효성검사를 실행하고 통과했을 때 제출버튼을 활성화하는 함수로 아예 똑같은 코드를 반복하고 있어 하나로 합쳤다. 둘의 다른 점은 다른 setState함수를 사용한다는 점이었는데, 이부분은 authForm으로 컴포넌트로 분리하면서 각각 다른 상태와 setState함수를 전달하기 때문에 컴포넌트에 handleChange를 prop으로 전달 후에, 컴포넌트 내부에서 setState를 함수로 전달해 연결했다.

```jsx
//이전 코드
const handleLoginChange = e => {
  const { name, value } = e.currentTarget
  setLoginInfo(prev => {
    return {
      ...prev,
      [name]: value,
      isEmailValid: name == "email" ? value.includes("@") : prev.isEmailValid,
      isPasswordValid:
        name == "password" ? value.length >= 8 : prev.isPasswordValid,
    }
  })
}

const handleRegisterChange = e => {
  const { name, value } = e.target
  setRegisterInfo(prev => {
    return {
      ...prev,
      [name]: value,
      isEmailValid: name == "email" ? value.includes("@") : prev.isEmailValid,
      isPasswordValid:
        name == "password" ? value.length >= 8 : prev.isPasswordValid,
    }
  })
}

//수정 후 코드

const handleChange = (e, setInfo) => {
  const { name, value } = e.currentTarget
  setInfo(prev => {
    return {
      ...prev,
      [name]: value,
      isEmailValid: name == "email" ? value.includes("@") : prev.isEmailValid,
      isPasswordValid:
        name == "password" ? value.length >= 8 : prev.isPasswordValid,
    }
  })
}
```

#### 2.2 상태와 상태관리 함수

​ 상태도 유사한점이 많아 하나로 합치고 싶었지만 각각 다른 데이터를 보관하기 때문에 오히려 합쳐져있던 message상태를 각각 loginMessage와 registerMessage로 분리해, 컴포넌트에 상태들과 함수를 prop으로 전달했다.

```jsx
//이전코드
const [loginInfo, setLoginInfo] = useState({
  email: "",
  password: "",
  isEmailValid: false,
  isPasswordValid: false,
})

const [registerInfo, setRegisterInfo] = useState({
  email: "",
  password: "",
  isEmailValid: false,
  isPasswordValid: false,
})

const [message, setMessage] = useState({
  loginMessage: "",
  registerMessage: "",
  loginSuccess: false,
  registerSuccess: false,
})

//수정 후 코드
const [loginInfo, setLoginInfo] = useState({
  email: "",
  password: "",
  isEmailValid: false,
  isPasswordValid: false,
})

const [registerInfo, setRegisterInfo] = useState({
  email: "",
  password: "",
  isEmailValid: false,
  isPasswordValid: false,
})

const [registerMessage, setRegisterMessage] = useState({
  registerMessage: "",
  registerSuccess: false,
})

const [loginMessage, setLoginMessage] = useState({
  loginMessage: "",
  loginSuccess: false,
})
```

#### 2.3 handleSubmit

예외처리와 함께 각각 다른 api를 불러야 하기 때문에 가장 까다로운 부분이었다. 로그인의 경우 성공할 경우에 "/todo"로 연결해주고, 토큰을 저장하는 부분을 추가적으로 갖고 있지만, 그외의 예외처리는 message에 errorMessage를 저장해 UI로 에러를 보여주게 구성했기 때문에 유사했기에 하나의 함수로 수정했다.

예외 검사 부분은 따로 exceptionTest란 함수로 만들었고, api.js에서는 data를 return해주는 부분까지 하는 것으로 수정해 response를 response.json으로 반환하는 부분을 제거했고, 컴포넌트에 전달한 process ("login" 또는 "register")를 이용해 process가 login일 때만 /todo로 페이지 이동하고 토큰을 저장하게 했다.

수정하고 나니 좀 더 가독성이 좋고, 이후에 예외처리를 수정할 때 좀 더 편하게 할 수 있게 따로 exceptionTest 함수를 만들기 잘했다는 생각이 들었다.

```jsx
//수정 전 코드
const handleLoginSubmit = e => {
  e.preventDefault()
  const { email, password } = loginInfo
  postSignIn({
    email,
    password,
  })
    .then(response => response.json())
    .then(data => {
      let loginMessage = ""
      if (data.statusCode >= 400) {
        loginMessage = data.message
        setMessage(prev => {
          return {
            ...prev,
            loginMessage,
            loginSuccess: false,
          }
        })
        return
      }
      navigate("/todo")
      localStorage.setItem("access_token", data.access_token)
      setMessage(prev => {
        return {
          ...prev,
          loginMessage: "로그인에 성공했습니다",
          loginSuccess: true,
        }
      })
    })
}

const handleRegisterSubmit = e => {
  e.preventDefault()
  const { email, password } = registerInfo
  postSignUp({
    email,
    password,
  })
    .then(response => response.json())
    .then(data => {
      if (data.statusCode >= 400) {
        setMessage(prev => {
          return {
            ...prev,
            registerMessage: data.message,
            registerSuccess: false,
          }
        })
        return
      }
      setMessage(prev => {
        return {
          ...prev,
          registerMessage: "회원가입에 성공했습니다",
          registerSuccess: true,
        }
      })
    })
}

//수정 후 코드

const exceptionTest = (data, setMessage, process) => {
  if (data.statusCode >= 400) {
    setMessage(prev => {
      return {
        ...prev,
        message: data.message,
        success: false,
      }
    })
    return
  }
  if (process == "login") {
    navigate("/todo")
    localStorage.setItem("access_token", data.access_token)
  }
  setMessage(prev => {
    return {
      ...prev,
      message: `${"login" ? "로그인" : "회원가입"}에 성공했습니다`,
      success: true,
    }
  })
}

const handleSubmit = (info, process, setMessage) => {
  const { email, password } = info
  const obj = { email, password }
  if (process == "login") {
    postSignIn(obj).then(data => {
      exceptionTest(data, setMessage, process)
    })
  } else {
    postSignUp(obj).then(data => {
      exceptionTest(data, setMessage, process)
    })
  }
}
```
