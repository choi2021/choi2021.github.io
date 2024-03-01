---
title: '원티드 프리온보딩 1주차 첫째주 과제 3일차-best case 프로젝트 회고'
date: 2022-10-27
slug: 원티드-프리온보딩-1주차-첫째주-과제-3일차-best-case-프로젝트-회고
tags: [원티드프리온보딩]
series: 원티드프리온보딩
---

# 투두 best case 프로젝트

어제 회의를 통해 정한 best case들을 프로젝트에 적용하기 위해 협업 tool 세팅과 합치면서 생긴 시행착오득을 정리해보려 한다.

## 🧐 협업툴 설정

협업툴인 ESLint, prettier, husky를 이용한 git hook 세가지를 설정하면서 겪었던 에러들을 정리해보았다.

### 1. Husky Prettier 자동화

윈도우 OS를 사용하는 내가 세팅을 끝내고 레포를 올리고 팀원 분들이 받아서 사용할 때, Prettier가 작동하지 않는 문제가 생각했다.

여기서 문제점은 **윈도우에서 init 할때 리눅스에서 파일 권한과 충돌**로 터미널에 `chmod ug+x .husky/*` 을 추가해서 설정해줘야 정상적으로 작동할 수 있었다.

[husky 관련 이슈](https://github.com/typicode/husky/issues/1177)

### 2. ESlint Air-bnb style

엄격한 기준을 가지고 있는 ESLint Air-bnb 스타일을 적용해, 필요한 부분들만 설정을 꺼주는 방식으로 ESLint 설정을 했다. 우리가 프로젝트를 하면서 느꼈던 세팅변경은 다음과 같다.

- Air-bnb Eslint에서 화살표 함수를 함수 선언문으로 바꾸는 설정 off

```
react/function-component-definition : 0 // 컴포넌트 작성 시 화살표 함수 사용 가능
```

- 화살표 함수에서 단일 파라미터 일 때 소괄호 없어지는 설정 off

```
 'arrow-body-style' : 0
```

- 함수 내에서 return을 생략하는 설정 off

```
 'Consistent-return' : 0
```

- 사용하지 않는 변수 에러로 인식하는 설정 off

```
 'no-unused-vars' : 0
```

- object 안에서 comma를 무조건 써줘야하는 설정 off

이유 : prettier 설정에서 자동으로 마지막 키나 인덱스의 comma를 삭제시키기 떄문에 eslint와 충돌이 나 꺼주었다.

```
 'comma-dangle' : 0
```

팀 내의 convention으로 함수형 컴포넌트를 화살표함수로 작성하기로 해, 함수 선언문이 아니면 에러로 던지는 설정이나 prettier와 충돌해서 생기는 에러들을 꺼주었다. 처음 생각했던 설정들보다 더 많은 설정들이 필요했지만 보다 이번 프로젝트를 함께 하면서 불편한 점들을 같이 해결해나가는 과정이어서 더 의미있었다.

초기 세팅을 위한 boilerplate를 만들어서 보다 깔끔한 설정으로 앞으로 프로젝트를 진행해나갈 수 있을 것 같다.

## 📢 contextAPI Provider의 eslint에서 에러

context API를 이용해서 로그인 상태를 전역상태로 관리하려 했다. 원래 개인적인 프로젝트에서는 잘 작동하던 코드가 다음과 같은 에러를 던졌다.

```jsx
export const LoginProvider = ({ children }) => {
  const [isLoggedIn, setIsLoggedIn] = useState(
    !!getLocalStorage({ name: TOKEN_NAME })
  );

  return (
    <LoginContext.Provider value={% raw %}{{ isLoggedIn, setIsLoggedIn }}{% endraw %}>{children}</LoginContext.Provider>
  );
};
```

```
The object passed as the value prop to the Context provider changes every render
The object passed as the value prop to the Context provider
```

문제는 value로 전달되는 object가 계속해서 바뀌어서 생기는 에러로 ESLint에서 성능저하를 예상해 에러를 던져주었다. 이를 해결하기 위한 방법은 Context 데이터를 가지고 있는 컴포넌트에 useMemo hook을 이용하여 데이터를 캐싱하여 불필요한 리렌더링 방지하였다.

contextAPI의 단점인 전체적인 리렌더링을 마주할 수 있는 좋은 기회가 되었다.

```jsx
export const LoginProvider = ({ children }) => {
  const [isLoggedIn, setIsLoggedIn] = useState(
    !!getLocalStorage({ name: TOKEN_NAME })
  );
  const value = useMemo(() => ({ isLoggedIn, setIsLoggedIn }), [isLoggedIn]);

  return (
    <LoginContext.Provider value={value}>{children}</LoginContext.Provider>
  );
};
```

## 🎈 아쉬웠던 점

auth 페이지의 구조를 바꾸다 보니 디테일한 부분을 고민하지 못했다. 변수명, 함수명을 정할 때 보다 명료하게 정리하지 못했고, 불필요한 리렌더링을 막거나 내부적인 에러메시지를 좀 더 정리해서 보내는 부분이 아쉬웠다.

## 💡 프로젝트 진행방식에 대한 생각

팀원중 한명이 driver 나머지 팀원 7명이 navigator로 참여해 진행하다보니, 너무 오랜시간이 걸리는 단점을 발견했다. 팀원별로 기능을 담당해서 개발을 한다면 더빠르고 각자 일을 맡아서 하는 참여도가 높아지는 장점이 있지만, 개인과제들을 합치다 보니 맞춰가는 데 더 많은 시간이 필요했다.

이부분을 해결하기 위해서 과제 시작전에 폴더 구조들을 정리하고 거기서 출발하게 된다면 앞으로 과제를 할때 추가된 부분만 이야기하면 되기 때문에 더 생산성이 높아질 것이라는 생각이 들었다.
