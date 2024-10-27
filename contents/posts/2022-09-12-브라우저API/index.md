---
title: "Browser API"
date: 2022-09-12
slug: browser-api
tags: [javascript, 브라우저]
---

이번주부터는 드림코딩의 브라우저101 강의를 통해 배운 API들과 자바스크립트 문법들을 함께 정리해나가려한다.

## API

API는 application programming interface의 약자로 소프트웨어 내부에 존재하는 기능과 규칙의 집합이다. (출처: [MDN API](https://developer.mozilla.org/ko/docs/Glossary/API))

API라는 단어는 너무 편하게 말하지만 사실 정확히 말하라고 했을 때 모호한 단어였다. 청원 사이트 프로젝트를 할 때, 백엔드 팀에서 만들어준 여러 API들을 받아와 프론트의 UI들과 연결해 여러 일들을 처리하고, 브라우저 API들 또는 Youtube와 같은
기업들의 API들을 이용해 개인 프로젝트들을 만들면서 자주 사용했지만 모호하게 느꼈던 API는 나에게 다음과 같이 정의할 수 있을 것 같다.

"클라이언트에서 요청하는 내용(Request)를 서버에서 요청에 맞게 제공하는 과정(response)"

이러한 과정은 내부에서 어떻게 설정되어있는지 모르지만 사용자가 편하게 사용할 수 있게 되어 있으며, 브라우저에는 정말 다양한 API들이 존재했다.

이렇게 많은 브라우저의 API들은 window를 전역 객체 (global)로 다음과 같이 분류 될 수 있다.

1. DOM (Document Object Model): HTML이 작성되어 렌더링 되는 영역

2. BOM (Browser Object Model): 브라우저 자체의 정보 (프로퍼티와 메소드)를 담고 있는 영역

![img](https://velog.velcdn.com/images%2Ftjdud0123%2Fpost%2F5fdd4197-125a-4790-b56f-5b0ef03fe7a0%2Fimage.png)

자바스크립트는 이러한 DOM과 BOM API를 이용해 다양한 기능을 구현하는데 사용된다. 이렇게 다양한 API들이 있지만 그중에서 자주 사용되는 API들을 정리해보려 한다.

### 브라우저의 크기

화면의 크기, 브라우저의 크기는 다음과 같이 4가지로 정의된다.

![img](https://blog.kakaocdn.net/dn/bZHQoM/btqT18YjBgT/VPYwdvpvjXA1fekKhFOU6k/img.png)

[출처:[디자이너의 기록-[web] window size 및 좌표 관련 속성](https://designer-ej.tistory.com/entry/Web-window-size-%EB%B0%8F-%EC%A2%8C%ED%91%9C-%EA%B4%80%EB%A0%A8-%EC%86%8D%EC%84%B1)]

- window.screen.width와 window.screen.height: 사용자가 보고있는 스크린 전체의 화면 크기
- window.outerWidth와 window.outerHeight: 브라우저의 Tab과 스크롤바를 모두 포함한 크기

- window.innerWidth와 window.innerHeight:브라우저의 Tab을 제외하고 스크롤바는 포함한 크기

- document.documentElement.clientWidth와 document.documentElement.clientHeight: 브라우저의 Tab과 스크롤바를 포함한 크기

위와 같이 총 4가지로 정리할 수 있다. window.screen.width와 window.screen.height은 사용자의 모니터 크기와 같으므로 브라우저의 크기를 작게 하거나

크게 만들어도 달라지지 않는다. 앞서 언급한 window는 브라우저의 전역객체이기 때문에 생략하고 사용이 가능하며, resize 이벤트는 window에 이벤트 리스너를

사용해야한다.

```javascript
window.addEventListener("resize", () => {
  console.log(window.screen.width)
  console.log(screen.width)
})
```

## 좌표

브라우저의 좌표는 왼쪽 상단을 기준점 (0,0)으로 두고 다음과 같이 축을 가진다.

![img](https://developer.mozilla.org/ko/docs/Web/API/Element/getBoundingClientRect/element-box-diagram.png)

[출처:[mdn Element.getBoundingClientRect()](https://developer.mozilla.org/ko/docs/Web/API/Element/getBoundingClientRect)]

위와 같이 아래로 갈수록 y좌표가 증가, 우측으로 갈수록 x좌표가 증가한다. 브라우저 위의 Element의 크기 정보는 Element.getBoundingClientRect()를 통해

알 수 있다. CSS와 달리 right와 bottom도 동일한 기준점 (0,0)에서 측정한 값이므로 주의해야 한다.

![pageY vs clientY](https://i.stack.imgur.com/4C3no.png)

[출처:[stack-overflow](https://stackoverflow.com/questions/6073505/what-is-the-difference-between-screenx-y-clientx-y-and-pagex-y)]

브라우저에서의 위치를 표시할 때 두가지 종류의 좌표가 존재한다.

1. Page X와 Page Y: 전체 페이지 크기에서의 좌표
2. Client X와 Client Y: 사용자가 보고있는 viewport를 기준에서의 좌표
