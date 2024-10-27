---
title: "BEM과 자바스크립트의 자료형"
date: 2022-09-06
slug: javascript-BEM-자료형
tags: [css, javascript, 문법]
---

5 개월 간의 학원에서 강사 생활을 끝내고 이번 주부터 좀 더 집중적으로 개발 공부를 할 수 있게 되었다.

틈틈이 쪼개서 공부하다 보니 공부해 온 것들이 산발 적으로 흩어져 있다는 생각이 들었다.

우선 가장 기본이 되는 HTML, CSS, Javascript에 대한 복습을 했다.

## HTML과 CSS

### BEM (Block-Element-Modifier)

HTML을 사용하는 데에는 큰 무리가 없었지만 여전히 부족한 부분은 class의 이름을 naming하는 부분이었다.

React를 사용하지 않고 CSS와 연결하기 위해 BEM을 다시 공부하고 정리해 보았다.

BEM은 Block-Element-Modifier를 줄인 CSS 방법론으로 React와 같이 모듈화가 가능한 경우에는 사용하지 않지만 모듈화를 할 수 없을 때에

class간의 충돌을 피하기 위해 사용한다.

![Inside a card component, buttons can be a BEM element](https://scalablecss.com/static/04158e912667d940eeb914f724379072/37523/nesting-elements-within-block.png)

[그림 출처, scalableCss](https://scalablecss.com/bem-blocks-within-blocks/)

위와 같은 그림에서 큰 component에 해당하는 card가 block, component를 이루고 있는 image, title, description,button이 element에 해당되게 된다.

그림을 이용한 코드는 다음과 같다.

```html
<div class="“card--dog”">
  <img class="“card__image”" />
  <h2 class="“card__title”">I am a card</h2>
  <p class="“card__description”">I am the card paragraph</p>
  <!-- The button is an element inside the block -->
  <a class="“card__button”">Learn more</a>
</div>

<div class="“card--cat”">
  <img class="“card__image”" />
  <h2 class="“card__title”">I am a card</h2>
  <p class="“card__description”">I am the card paragraph</p>
  <!-- The button is an element inside the block -->
  <a class="“card__button”">Learn more</a>
</div>
```

위 코드에서 component는 ".card"로, element들은 `.card__title` 과 같이 \_\_로 표현된다.

modifier는 강아지 사진을 담고 있는 카드의 경우 `.card--dog`으로 표현이 가능하고, 고양이 사진을 담고 경우 `.card--cat`과 같이 표현이 가능하다.

이러한 BEM의 장점은 구분이 잘된다는 점이지만 단점으로는 점점 길어지고 component가 복잡해 질 수록 작성해야 하는 코드 양이 증가하게 된다.

## Javascript

자바스크립트의 문법의 기초 중 변수, 자료형에 대해 공부했다.

변수를 선언하는 방법은 상수를 선언하는 'const'와 변할 수 있는 'let' 두 가지가 존재한다. (var는 문제가 많아서 제외)

언급한 두 가지 방법을 통해 메모리에 우리가 원하는 데이터를 임시로 저장하게 되는데 이때 다음과 같은 일이 일어난다.

1. 변수를 선언하면 (variable declaration) 메모리 cell의 특정 부분을 가리키게 된다.

2. 변수를 할당하면 (variable assignment) 메모리 cell에 특정 값을 저장한다.

#### Number

자바스크립트는 다른 언어들과 다르게 메모리 크기를 크게 고려하지 않고 저장할 수 있는데 Number 자료형은 실수, 음수 등 모두 담을 수 있고

`103/0` 이나 `23/-0` 과 같은 경우는 INFINITY로 표현한다.

문자열을 숫자로 간단하게 변환할 때 `+`를 이용할 수 있다.

#### String

문자열을 입력할 때 `""`, `''`와 함께 back tick을 이용한 template literals 세가지 방식이 존재한다.

```javascript
let string = "hello world!"
string = "halo world!"

const word = "world"
string = "hello " + word + "!"
string = `hello ${word}`
```

Back tick (``)은 큰따옴표 ("")와 작은 따옴표로 표현하기 힘든 표현들을 위한 문법으로 변수를 ${}에 넣어, 일일이 조합을 고민하지 않고 원하는 문자열을 만들 수 있는 편리함을 준다.

##### 이스케이프 문자

이스케이프 문자는 문자열 내 특정 기호를 의미하며 백스페이스(\\)와 조합해 사용된다.

1. `\n`: 띄어쓰기
2. `\t`: 탭
3. `\\`: 백스페이스
4. `\u`:유니코드

### Boolean

불리언은 간단하게 true와 false를 가진다. 이때 중요한 부분은 falshy와 truthy 값이다.

boolean으로 자료형을 변환할 때는 간단하게 `!!`을 이용할 수 있다.

##### Falshy

1. 0,-0
2. 빈 문자열("")
3. null
4. undefinded
5. NaN(not a number)

##### Truthy

1. 0이 아닌 숫자
2. 빈 문자열이 아닌 문자열 ("False"도 가능)
3. 객체 (비어있어도 true)
4. Infinity
5. 배열 (비어있어도 true)

### Empty 표현

자바스크립트 내 두 가지 비어있다는 표현, undefined와 null은 다음과 같은 차이점을 가지고 있다.

#### undefined

undefined의 경우에 말 그대로 아직 정해지지 않아 변수가 선언은 되었지만, 해당 메모리에 값이 할당되지 않은 상태를 가리킨다.

#### null

null의 경우에는 변수가 선언되고 메모리에 비어있는 object가 할당되어 있는 상태를 가리킨다.

- 여기서 특이한 점은 null의 type이 object라는 점이다. primitive 자료형이지만 type으로는 null을 가리키고 있는데 이러한 아이러니는 초기 자바스크립트를 개발하면서 생긴 오류라고 한다.

[참조]
[null===object에 대한 고찰](https://blog.naver.com/gofkdvjvl/222166705113)
