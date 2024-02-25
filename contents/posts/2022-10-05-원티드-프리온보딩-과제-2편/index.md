---
title: '원티드 프리온보딩 사전과제-과제 2,3'
date: 2022-10-05
slug: 원티드-프리온보딩-사전과제-과제-2번-3번
tags: [원티드프리온보딩]
series: 원티드프리온보딩
---

## Assignment 2)

어제에 이어 API 에러를 해결하기 위해 다시 시도했지만 여전히 같은 에러가 보였다.

![image-20221005143256870](/assets/img/2022-10-04-원티드 프리온보딩 과제 1편/image-20221005143256870.png)

뭐가 잘못된건지 다시 한번 확인하러 안내페이지에 다시갔더니....

![image-20221005144450498](/assets/img/2022-10-05-원티드 프리온보딩 과제 2편/image-20221005144450498.png)

API 주소가 바뀌어 있었다....ㅜㅜㅜㅜ 어제 안된건 내가 잘못 입력한 게 아니라 진짜 서버가 꺼져있었거나, 주소를 잘못 제공받았던건가 보다...

[회원가입에 성공한 API결과]

![image-20221005150009567](/assets/img/2022-10-05-원티드 프리온보딩 과제 2편/image-20221005150009567.png)

API주소가 처음부터 맞게 제공되었다면 좋았겠지만, 이번 경험을 통해서 내가 너무 **REST API나 HTTP통신을 잘모르구나**를 많이 느꼈다. 우선 과제를 2번과 3번을 마무리하고, HTTP서버 통신, 토큰, 쿠키, JWT등 서버 통신을 한번 정리할 필요를 느꼈다.

API를 이제 사용할 수 있으니 각각 submit에 연결만 하면 쉽게 연결할 수 있었다. 위에 받아온 response를 이용해 로직을 작성했다. 과제 안내에 axios를 쓸 수 있다고 했지만 우선은 fetch만을 이용해서 작성한 후에 axios로 코드를 수정해보려고 계획했다.

```javscript
//api.js
const BASE_URL = 'https://pre-onboarding-selection-task.shop/';

export function postSignUp(data) {
  const { email, password } = data;
  return fetch(`${BASE_URL}auth/signup`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      email,
      password,
    }),
  });
}

export function postSignIn(data) {
  const { email, password } = data;
  return fetch(`${BASE_URL}auth/signin`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      email,
      password,
    }),
  });
}

```

api.js를 보면 url만 다르고 너무 동일한 게 보여서 코드 중복을 줄일 방법에 대해 고민하는 것도 너무 좋은 공부가 될 것 같다.

```javascript
//login.jsx
...

  const [message, setMessage] = useState({
    loginMessage: '',
    registerMessage: '',
    loginSuccess: false,
    registerSuccess: false,
  });

  const handleLoginSubmit = (e) => {
    e.preventDefault();
    const { email, password } = loginInfo;
    postSignIn({
      email,
      password,
    })
      .then((response) => response.json())
      .then((data) => {
        let loginMessage = '';
        if (data.statusCode >= 400) {
          loginMessage = data.message;
          setMessage((prev) => {
            return {
              ...prev,
              loginMessage,
              loginSuccess: false,
            };
          });
          return;
        }
        navigate('/todo');
        localStorage.setItem('loginToken', data.access_token);
        setMessage((prev) => {
          return {
            ...prev,
            loginMessage: '로그인에 성공했습니다',
            loginSuccess: true,
          };
        });
      });
  };

  const handleRegisterSubmit = (e) => {
    e.preventDefault();
    const { email, password } = registerInfo;
    postSignUp({
      email,
      password,
    })
      .then((response) => response.json())
      .then((data) => {
        if (data.statusCode >= 400) {
          setMessage((prev) => {
            return {
              ...prev,
              registerMessage: data.message,
              registerSuccess: false,
            };
          });
          return;
        }
        setMessage((prev) => {
          return {
            ...prev,
            registerMessage: '회원가입에 성공했습니다',
            registerSuccess: true,
          };
        });
      });
  };

```

fetch를 이용해 코딩했기 때문에 에러처리를 catch를 사용해도 해결하지 못했다. [참고:https://intrepidgeeks.com/tutorial/error-handling-with-fetch]

우선은 예외처리를 response에서 처리하려 했지만 response에서 처리할 경우에 서버에서 보내 준 에러 메시지를 이용하지 못했다. 그래서 data를 json으로 바꿔서 받아와 data에서 statusCode에 따라 400번 이상의 에러는 로그인과 회원가입 창에 해당 에러 메시지를 보여 주는 식으로 구성했다. 에러에 따라 달라지는 메시지를 담을 수 있게 message 상태를 추가했고 로그인과 회원가입이 성공했는가에 따라 메시지 보여주는 곳의 색깔을 다르게 전달해주기 위해 각각 success에 관한 내용도 message 상태에 포함시켰다.

![image-20221006011958282](/assets/img/2022-10-05-원티드 프리온보딩 과제 2편/image-20221006011958282.png)

로그인은 **이미 존재하는 이메일일 경우 400에러**를 보내줬고, 회원가입는 **존재하지 않는 이메일 경우에 404 not found에러**를, **이메일에 맞지 않는 비밀번호를 보낼 시에는 401 unauthorized 에러**를 보내주는 걸 확인했다. 로그인의 경우 401과 404에 따라 다른 메시지가 이미 서버로부터 받아오기 때문에, 바로 에러 메시지에 보여줄 수 있게 코드를 구성했다.

회원가입과 로그인 모두 성공할 경우에는 하얀색으로 성공했다고 보일 수 있게 했고, 로그인은 과제안내에서 성공시 받아오는 JWT 토큰을 로컬스토리지에 저장해달라 했기 때문에 로컬스토리지에 "loginToken"이라는 키로 저장했다. 저장후에는 useNavigate를 이용해 "/todo"로 페이지이동을 추가했다.

## Assignment 3)

세번째 과제는 다음과 같다.

![image-20221006012913151](/assets/img/2022-10-05-원티드 프리온보딩 과제 2편/image-20221006012913151.png)

전체적인 페이지 이동을 위해서 app.jsx에서 token의 유무에 따라 이동시켜주는 게 좋을 것 같아 useEffect로 코드를 짰다. 토큰을 "loginToken" key로 불러온 후에 존재한다면 token 상태에 저장한 후에 존재 유무에 따라 해당 url로 이동시켜주었다.

```javascript
//App.jsx

const navigate = useNavigate();
const [token, setToken] = useState('');
useEffect(() => {
  const prevToken = localStorage.getItem('loginToken');
  prevToken && setToken(prevToken);
  if (token) {
    navigate('/todo');
  } else {
    navigate('/');
  }
}, [token]);
```

생각보다 간단했지만, 회원가입과 로그인 부분에서 겹치는 부분과 분리가 필요해보여서 내일 4번과 5번 과제까지 마무리 한 후에 컴포넌트화 해볼 예정이다.
