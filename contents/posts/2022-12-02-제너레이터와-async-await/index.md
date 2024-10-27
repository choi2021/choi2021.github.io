---
title: "제너레이터와 Async-Await"
date: 2022-12-02
slug: javascript-generator-async-await
tags: [javascript, 문법]
---

# 💎 제너레이터와 Async-Await

EO 채널의 시니어 개발자분들의 이력서 관련 팁을 말씀해 주시는 영상을 본 적이 있다. 그때 한 시니어 개발자분께서 "skill란에 자바스크립트를 잘한다고 하면서 제너레이터를 물어보면 제대로 설명 못하는 사람을 있다"라고 말씀하셨다. 그 말씀에 "제너레이터? 이터러블 객체를 편하게 만들 수 있는 것 아닌가"라고 나는 알고 있다고 생각했다. 하지만 공부하면서 왜 제너레이터가 중요한 지, 어떻게 쓰이고 있는지를 보니 나는 제너레이터를 몰랐다는 사실을 깨달을 수 있었다.

이제는 제대로 이해하고 설명할 수 있게 제너레이터와 제너레이터가 어떻게 활용되고 있는지 정리해 보고자 한다.

## 🎈 제너레이터

제너레이터는 "이터러블 객체"를 만든다. 그러면 이터러블 객체는 무엇인지 먼저 정리해 보고자 한다.

### 이터러블 객체

이터러블 객체 (iterable object)는 이전에 iteration에 대해 공부하면서 정리한 적이 있지만, 아직 완전히 받아 들여지지 않아 다시 한 번 정리 해보려 한다.

이터러블 객체는 이터러블 프로토콜과 이터레이터 프로토콜 두 가지를 만족해야 한다.

<img src="https://poiemaweb.com/img/iteration-protocol.png" width="800px"/>

먼저 이터러블 프로토콜은 위 그림의 왼쪽의 조건으로 객체 내의 [Symbol.iterator]를 키로 한 메소드가 존재하는 것을 의미한다. 이때 [Symbol.iterator] 메소드의 반환값이 next를 메소드를 가지는 객체여야하는데 이것을 이터레이터 프로토콜이라고 부른다. next메소드는 순회하며 value,done 속성을 갖는 객체를 반환한다.

이렇게 두 가지 조건을 만족한 이터러블 객체는 for-of로 순회가 가능한 특징을 가지며, 대표적인 예로 배열이 있다.

```javascript
const fibonacci = {
  [Symbol.iterator]() {
    let [pre, cur] = [0, 1]
    const max = 10
    return {
      next() {
        ;[pre, cur] = [cur, pre + cur]
        return { value: cur, done: cur >= max }
      },
    }
  },
}

for (const num of fibonacci) {
  console.log(num)
}
```

### 이터러블 객체와 제너레이터

위의 언급한 대로 제너레이터는 이터러블 객체를 만들 수 있다. 제너레이터는 선언시, function\* 로 시작하고 yield 문을 하나 이상 포함한다. yield는 제너레이터로 만들어진 이터러블 객체의 next 메소드의 호출에 따라 **함수 내부의 실행을 중지하거나 반환한다**. 여기서 중요한 점은 제너레이터로 함수 내부 실행을 제어할 수 있다는 점이다.

앞서 만들었던 이터러블 객체 fibonacci를 제너레이터로 다시 만들면 다음과 같다.

```javascript
const fibonacciGen = (function* () {
  let [pre, cur] = [0, 1]
  const max = 10
  while (pre + cur <= max) {
    ;[pre, cur] = [cur, pre + cur]
    yield cur
  }
})()

for (const num of fibonacciGen) {
  console.log(num)
}
```

이제 제너레이터가 "어떻게 함수 내부 실행을 제어하는지" 두 가지 예제로 알아보자. 첫 번째 예제 코드를 보면 generator는 제너레이터로 만든 이터러블 객체를 반환 받는다. next 메소드를 이용하면 value,done을 key로 하는 객체가 반환되는데, 여기서 중요한 점은 next가 호출될 때마다 "yield문까지 진행되고 멈춘다는 점"이다. yield문이 더 이상 없을 때는 value로 undefined, done은 true로 반환해 준다.

```javascript
function* genFunc() {
  try {
    yield 1
    yield 2
    yield 3
  } catch (e) {
    console.error(e)
  }
}

const generator = genFunc()
console.log(generator.next()) // { value: 1, done: false }
console.log(generator.next()) // { value: 2, done: false }
console.log(generator.next()) // { value: 3, done: false }
console.log(generator.next()) // { value: undefined, done: true }
```

두번째 예제는 yield를 이용해 값을 할당한 예제다. 실행 과정을 정리하면 다음과 같다.

1. res 첫 번째 호출: yield 1까지 실행하고 진행을 멈춘다. `{ value:1, done:false }`를 반환한다.
2. res 두 번째 호출: next(10)으로 전달 받은 10을 x에 할당하고, x+10까지 실행하고 멈춘다. `{ value: 20, done: false }`를 반환한다.
3. res 세 번째 호출: next(20)으로 전달 받은 20을 y에 할당하고, `{ value: 30, done: true }`를 반환한다.

```javascript
function* genFunc() {
  const x = yield 1
  console.log("x", x) // x 10
  const y = yield x + 10
  console.log("y", y) // y 20
  return x + y
}

const generator = genFunc(0)

let res = generator.next()
console.log(res) // { value: 1, done: false }
res = generator.next(10)
console.log(res) // { value: 20, done: false }
res = generator.next(20)
console.log(res) // { value: 30, done: true }
console.log(generator.next()) // { value: undefined, done: true }
```

제너레이터가 어떻게 함수 내부 실행을 제어하는지에 대해 알아봤다. 그러면 제너레이터를 <u>어디에</u> 사용해야 할까?

## 🧨 제너레이터의 활용: Async-Await

제너레이터는 사실 이미 많이 사용되고 있었다. 바로 async- await 구문이다. async-await은 먼저 정리했던 프로미스를 동기적으로 코드를 작성할 수 있게 해주고 반환할 때는 프로미스로 반환해주는 기능을 가진다. 이렇게 가능했던 것은 async-await이 내부적으로 **제너레이터를 이용해 구현되어** 있었기 때문이다.

아래는 간단한 async-await을 구현한 코드이다. 제너레이터가 가지는 특징, next와 yield로 함수 호출자와 함수의 상태를 알 수 있는 점을 이용했다. 함수 동작을 간단하게 정리하면 다음과 같다.

1. async함수는 전달 받은 generator함수로 이터러블 객체를 등록한 후에, generator의 next의 값을 반환하는 클로저 함수 onResolved를 반환한다.
2. 즉시 실행 함수로, 반환된 onResolved를 실행하고, 첫번째 yield문의 fetch(url)을 진행해 result로 결과를 반환받는다.
3. 아직 fetchTodo함수의 끝까지 진행되지 않아 result의 done이 false이므로 다시 onResolved를 실행한다.
4. 두 번째 onResolved실행으로 fetchTodo의 response값을 할당하고, 두 번째 yield문의 response.json()을 진행해 result로 결과를 반환받는다.
5. 아직 fetchTodo함수의 끝까지 진행되지 않아 result의 done이 false이므로 다시 onResolved를 실행한다.
6. 세 번째 onResolved실행으로 fetchTodo의 todo값을 할당하고 console에 호출한 뒤에 result로 결과를 반환받는다.
7. fetchTodo가 끝났기 때문에 result.done은 true가되어 undefined를 반환하며 종료한다.

```javascript
const async = generatorFunc => {
  const generator = generatorFunc()
  const onResolved = arg => {
    const result = generator.next(arg)
    return result.done
      ? result.value
      : result.value.then(res => onResolved(res))
  }
  return onResolved
}

async(function* fetchTodo() {
  const url = "https://jsonplaceholder.typicode.com/todos/1"
  const response = yield fetch(url)
  const todo = yield response.json()
  console.log(todo)
})()
```

### Async- Await

async,await은 위의 예제보다 훨씬 가독성이 좋게 이용될 수 있게 추가된 문법으로 비동기 로직을 동기적으로 작성이 가능하게 도와준다. 주의할 점은 **항상 반환값은 Promise**라는 점이다. Typescript로 작업하면서 async-await을 이용하면 동기적으로 작성하다 보니 잘못 타입을 전달해 준 경우가 있었다. async- await의 장점은 promise chaining에서 then,catch,finally로 연결하다 보면 callback hell처럼 가독성이 떨어지게 될 수 있을 때, 가독성을 높여줘 실행 순서를 예측할 수 있는 장점을 가지게 된다. 또 다른 장점으로 동기적 코드처럼 try-catch구문으로 에러핸들링이 가능하다.

위의 예제를 실제 async-await으로 고쳐보면 다음과 같다. 훨씬 간단하게 구현된 것을 알 수 있다.

```javascript
async function fetchTodo() {
  const url = "https://jsonplaceholder.typicode.com/todos/1"
  const response = await fetch(url)
  const todo = await response.json()
  console.log(todo)
}
```

await은 promise가 settled된 상태(성공,실패와 상관없이 처리가 끝난 상태)가 됐을 때 resolve한 결과를 반환한다. 그렇기 때문에 연관되지 않은 여러 promise를 사용해야 할 때 모두 await을 붙여서 사용하면 시간이 오래걸리는 단점을 갖는다. 앞서 정리한 Promise.all을 이용해 병렬 처리로 한번만 await 사용해 해결가능하다.

```javascript
//수정 전
async function foo() {
  const a = await new Promise(resolve => setTimeout(() => resolve(1), 3000))
  const b = await new Promise(resolve => setTimeout(() => resolve(2), 2000))
  const c = await new Promise(resolve => setTimeout(() => resolve(3), 1000))
  console.log([a, b, c])
}

foo() // 6초 뒤 [1,2,3]

async function foo() {
  const res = await Promise.all([
    new Promise(resolve => setTimeout(() => resolve(1), 3000)),
    new Promise(resolve => setTimeout(() => resolve(2), 2000)),
    new Promise(resolve => setTimeout(() => resolve(3), 1000)),
  ])
  console.log(res)
}

foo() // 3초 뒤 [1,2,3]
```

[참고]

[모던 자바스크립트 딥다이브](http://www.yes24.com/Product/Goods/92742567)
[제너레이터와 async/await](https://poiemaweb.com/es6-generator)
