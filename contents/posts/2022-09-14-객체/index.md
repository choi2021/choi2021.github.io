---
title: "객체"
date: 2022-09-14
slug: javascript-object
tags: [javascript, 문법]
---

어제 정리한 함수에 이어 이번엔 오브젝트를 정리해보고자 한다. 오브젝트는 이어질 클래스와도 많이 연결되어 있으므로 생성자함수 전까지만 정리해보고자 한다.

우선 정리하면서 자주 언급했던 오브젝트는 <b>서로 연관있는 속성(property)과 행동(method)들을 묶어놓은 변수</b>라고 할수있다.

```javascript
const Youngjun = {
  gender: "male",
  age: 26,
  smile() {
    console.log("웃음")
  },
}
```

다음과 같이 "나"라는 객체에 나에 대한 정보( property)와 내가 할 행동(method)을 정리했다. 이러한 오브젝트에 대해 알아보자.

## 1) Object 만들기

Object를 만들 수 있는 방법에는 크게 3가지가 있다.

1. `{ key: value}`
2. new Object 사용하기
3. object.create() 메소드 사용하기

오브젝트는 "key"와 "value"로 이루어져 있는 자료형으로 3가지 방법으로 key와 value가 같은 오브젝트를 다음과 같이 만들 수있다.

```javascript
const obj1 = { key: "any value" }
const obj2 = new Object()
obj2.key = "any value"

const obj3 = Object.create({}, { key: "any value" })
```

첫번째는 전달해준 key와 value를 가지고 있는 오브젝트를 바로 만드는 방법으로 가장 많이 사용하는 방법이라 생각된다. 두번째는 새로운 빈 객체를 만든 뒤에 key와 value를 전달해준다. 세번째 방법은 Object의 메소드를 이용하는 방법으로, 첫번째 인자로 우리가 만들 객체의 프로토타입을, 두번째 인자로 우리가 원하는 key와 value를 가진 오브젝트를 전달해 만들었다. 확실히 첫번째 방법이 간단하고 이해가 잘되는 방법인 것같다.

## 2) Object의 key와 value

오브젝트의 key와 value는 서로 사용될 수 있는 자료형이 다르다. key가 될 수 있는 자료형은 <b>문자,숫자,문자열,심볼</b>이고, value가 될 수 있는 자료형은 <b>primitive 자료형과 객체</b>, 모든 자료형이 다 가능하다.

```javascript
const Youngjun = {
  gender: "male",
  habits: ["music", "game", "coding"],
  0: 300,
  ["biggest-goal-of-life"]: "성공하기",
}
```

위의 예제 오브젝트에서 key에서 문자열과 숫자로 담았고 문자열중에서도 "-"로 연결되어있는 경우에는 "[]"로 감싸서 전달해주어야 한다. 그냥 "biggest-goal-of-life"로 전달할 시에는 에러가 발생한다. 그렇기에 긴 단어의 경우에 camel case를 이용하는 게 더 나은 것 같다. value로는 habits key에 배열을 전달했는데 배열 또한 javascript에서 객체이기 때문에 사용가능하다.

key와 value는 자물쇠처럼 맞는 열쇠를 객체에 전달하면 해당하는 값을 얻을 수 있다. 이러한 key를 전달하는 방식에는 다음과 같은 2가지 방법이 있다.

1. dot notation ( . )
2. bracket notation ([ ])

```javascript
const Youngjun = {
  gender: "male",
  age: 26,
  smile() {
    console.log("웃음")
  },
}
const age = "age"

console.log(Youngjun.gender) //"male"
console.log(Youngjun[age]) //26
```

위와 같이 첫번째 방시인 dot notation을 이용하면 문자열 key로 되어있는 값에 접근할 수있고, bracket notation의 경우에는 변수로 값을 받을 수도 있어 좀 더 동적으로 사용할 수 있다.

## 3. Object 추가 삭제

Object의 key와 value를 추가하는 방법은 간단하게 앞서 언급한 2가지 접근 방법을 사용해 값을 지정해줄 수 있고, 제거할 때는 해당하는 key 값을 삭제하면 된다.

```javascript
const Youngjun = {
  gender: "male",
  age: 26,
  smile() {
    console.log("웃음")
  },
}

Youngjun.height = 172
Youngjun["homeAddress"] = "Gwangju"
delete Youngjun.age
```

다음과 같이 사용해서 Youngjun 오브젝트에 키와 주소 값을 추가하고 나이 정보를 삭제할 수 있다.

Object의 추가 삭제 방법과 bracket notation을 이용해 오브젝트에 값을 동적으로 추가하고 삭제하면 다음과 같이 사용할 수 있다.

```javascript
function addProperty(obj, key, value) {
  obj[key] = value //obj.key=value 사용하면 안돼
}

const Youngjun = {
  gender: "male",
  age: 26,
  smile() {
    console.log("웃음")
  },
}

addProperty(Youngjun, "height", 172)
```

addProperty함수에 오브젝트와 key와 value를 전달해 속성을 추가 해주었다. 이때 앞서 설명한 두가지 접근 방법 중 dot notation을 사용하면 항상 "key"라는 key에만 접근해 value를 바꾸기 때문에 사용해선 안된다. bracket notation을 이용하면 전달받은 인자를 이용해 추가할 수 있다.

## 4) Object의 Method와 간단한 팁

가장 처음 만든 오브젝트를 보면 smile이라는 함수를 가지고 있다. 이러한 오브젝트 내부의 함수를 <b>Method</b>라고 하고 property에 접근할 때와 동일하게 접근해 함수를 호출 할 수 있다. 그리고 오브젝트의 key와 value가 key만 전달해도 된다.

```javascript
const gender = "male"
const age = 26
const Youngjun = {
  gender, //gender:gender와 같아
  age,
  smile() {
    console.log("웃음")
  },
}
Youngjun.smile() //"웃음"
```

예에서 gender와 age의 key와 value가 같으므로 축약해서 한번에 나타냈다. 이러한 특징을 이용해 좀 더 편하게 오브젝트에 값을 설정할 수 있다.
