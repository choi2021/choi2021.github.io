---
title: 'Intersection observer'
date: 2022-09-10
slug: intersection-observer
tags: [javascript, 브라우저, web]
---

추석에 휴일이라 많이 할 수 있을 것이라 생각했지만 그렇게 많은 것은 하지 못했다.

간단히 포트폴리오 강의를 다시 들으면서 intersection observation API를 사용했다.

여전히 어렵고 이해가 잘 안되는 부분이 많아, 예전 TIL을 참고해 다시 한 번 정리해 보고자 한다.

### 1) API의 배경

Intersection Observer API는 브라우저 API로 이전에 사용했던 방식 (getBoundingClientRect와 스크롤과 연결)을 사용할 때

scroll 이벤트의 경우에 eventListener와 연결하게 되었을 때, 자주 발생하는 event인데, 그때마다 우리가 원하는 요소의

크기를 불러오기 위한 getBoundingClientRect로 layout을 계속 불러오게 된다.

이로인한 성능저하를 해결하기 위한 API가 바로 <strong>Intersection Observer</strong>이다.

### 2) API의구성

intersectionObserver 오브젝트에는 observer가 observe하고 있는 DOM 요소에 intersection이 발생했을 때 실행할 **callback함수**와 Observer의 요소, 크기, 기준등을 어떻게 설정할지를 담고있는 **option**이다.

#### Option

```javascript
const option={
 root: document.querySelector("div")
 rootMargin:"0px",
 threshold:0
}
```

root는 관찰하는 영역으로 null로 전달시 사용자의 화면, viewport를 의미한다. 위 코드와 같이 DOM요소를 전달하면 DOM요소의 영역을

기준으로 관찰한다.

rootMargin은 내가 전한 영역에 margin을 주어서 주어진 margin을 포함한 만큼을 기준으로 삼는다. 예시로 사용자가 봐야할 화면을 미리 준비할 때 크기를 넓혀 화면을 미리 준비 할 수 있다.

threshold는 관찰하는 영역에 어느정도 들어왔을 때, callback함수를 실행할지를 담고 있다. 0~1까지 숫자로 전달할 때 들어올 때는 값을 그대로 사용하고, 반대로 나가는 상태에서는 1-threshold값을 사용한다.

#### callback function

```javascript
const callback = (entries, observer) => {
  entries.forEach((entry) => {
    // observer과 관찰하는 대상의 정보
  });
};
```

callback 함수는 entries와 observer를 인자로 받는 함수로, entries는 관찰하고 있는 대상의 정보를 담고 있는 배열이다. 배열 내에 관찰하는 item들의 객체에는 **boundingClientRect**로 관찰하고 있는 요소의 위치,크기와 같은 정보를 주고, **intersectionRatio**로 현재 영역에 얼마만큼 들어와 있는지, **isIntersecting**으로 앞으로 들어올 대상인지 이미 보이고 있어서 사라질 지를 알려준다. 이외에도 대상의 다양한 정보들을 담고 있다.

observer를 쓰는 예제는 아직 보지 못해 사용하지 않았다.

### Observer

```javascript
const observer = new IntersectionObserver(callback, option);
const sections = document.querySelectorAll('.section');
sections.forEach((section) => {
  observer.observe(section);
});
```

앞서 알아본 callback과 option을 observer에 전달하면 observer.observe()의 인자로 대상을 전달해 관찰할 수 있다.

위와 같은 3가지 요소를 이용해 API를 사용할 수 있다. 이때 활용할 수 있을 요소는 다양하지만 아직 부족해 다시 포트폴리오 페이지를

React를 이용해 만들 때, 더 자세히 공부해보고자 한다.
