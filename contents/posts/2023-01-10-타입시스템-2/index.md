---
title: "TS study: 타입 시스템 (2)"
date: 2023-01-10
slug: typescript-타입시스템-2
tags: [typescript]
series: "Typescript"
---

# 🎚 타입 시스템(2)

타입스크립트의 타입 시스템 중 **객체 래퍼 타입, 잉여 속성 체크, 함수 표현식, 타입과 인터페이스의 차이**에 대해서 정리해보자 한다.



## 📦 객체 래퍼 타입

문자열을 입력하고 `.`을 찍으면 객체처럼 우리는 다양한 메소드를 이용할 수 있다. 이렇게 가능한 것은 자바스크립트가 `.`을 찍는 순간 `string`에서 `String` 객체 래퍼로 타입변환이 이루어진다. 객체로 변환해 메소드를 사용한 후에는 객체 타입에서 다시 원시형으로 돌아간다. 

이렇게 유용한 객체 래퍼 타입이지만 정의된 타입을 보면 오타가 나기 쉽게 되어있다.  

- 원시형: `string`, 객체래퍼:`String`
- 원시형: `number`, 객체래퍼:`Number`
- 원시형: `boolean`, 객체래퍼:`Boolean`



```typescript
function getStringLen(foo: String) {
  return foo.length;
}

getStringLen('hello');
getStringLen(new String('hello'));

function isGreeting(phrases: String) {
  return ['hello', 'good day'].includes(phrases); // Argument of type 'String' is not assignable to parameter of type 'string'.
}
```

크게 문제가 되지 않을 것 같아 보이지만 원시형은 객체래퍼에 할당할 수 있는 반면 객체 래퍼는 원시형에 할당할 수 없기 때문에 오타로 인한 에러가 발생한 것을 볼 수 있다. 직접 객체 래퍼를 할당하지 않게 주의해야 함을 보여주는 예제다. 



## ✒ 잉여 속성 체크

잉여속성 체크는 우리가 정의해 준 타입의 속성들 외의 추가된 속성이 있는지 확인하는 과정을 의미한다.

```typescript
interface Dog {
  age: number;
  name: string;
}

const dog1: Dog = {
  age: 1,
  name: '바둑이',
  bark() { //  'bark' does not exist in type 'Dog'.
    console.log('짖기');
  },
};
```

위 예제에서 `Dog`타입에 bark 속성이 없기 때문에 에러가 발생했지만 앞서 배웠던 구조적 타이핑의 의미로 보았을 때는 필요한 속성인 age와 name이 있기 때문에 에러가 나지 않아야 할 것 같다.



```typescript
const dog2 = {
  age: 1,
  name: '바둑이',
  bark() {
    console.log('짖기');
  },
};
const r: Dog = dog2;
```

같은 조건이지만 앞선 예제와는 다르게 에러가 발생하지 않았다. 



```typescript
interface Options {
  title: string;
  darkMode?: boolean;
}

function createWindow(options: Options) {
  if (options.darkMode) {
    // setDarkMode()
  }
}
createWindow({
  title: 'Spider',
  darkmode: true,
}); // 'darkmode' does not exist in type 'Options'. Did you mean to write 'darkMode'?

const o1: Options = document; 

```

둘의 차이점은 뭘까?

앞선 에러가 발생한 예제들은 `잉여속성 체크`가 진행되었기 때문에 에러가 발생했다. `잉여속성 체크`가 발생하는 조건은 dog1처럼 `객체 리터럴`을 할당하거나 createWindow처럼 `함수에 매개변수를 전달할 때` 적용된다. 

에러가 발생하지 않은 경우는 변수를 통해 값을 전달했을 때로 임시 변수를 전달하면 `잉여 속성 체크`가 이루어지지 않는다. 우리가 정의한 속성만 추가되게 하는 경우에 `잉여속성 체크`를 적용해 오류를 찾을 수 있는 장점이 있다. 하지만 `구조적 타이핑`의 관점과 충돌하기 때문에 필요할 때에 적절하게 사용할 필요가 있어 보였다.



## 🎈 함수 표현식

 자바스크립트에서 함수를 사용하는 방법에는 함수 선언문과 함수 표현식이 있다. 타입스크립트는 함수 표현식일 때 `매개변수와 반환 값을 타입으로 선언`할 수 있는 장점을 갖고 있다.

``` typescript

function add(a: number, b: number) {
  return a + b;
}
function sub(a: number, b: number) {
  return a - b;
}
function mul(a: number, b: number) {
  return a * b;
}
function div(a: number, b: number) {
  return a / b;
}

type BinaryFn = (a: number, b: number) => number;

const add: BinaryFn = (a, b) => a + b;
const sub: BinaryFn = (a, b) => a - b;
const mul: BinaryFn = (a, b) => a * b;
const div: BinaryFn = (a, b) => a / b;
```

앞선 예제는 함수 선언문을 이용해 매개변수의 타입을 정해 준 경우이고, 아래는 함수 표현식에 `BinaryFn`타입을 이용해 매개변수와 반환 값의 타입을 한번에 정의한 경우이다. 함수 선언문의 경우 일일이 매개변수의 타입을 정해 주어야 하고, 정한 타입을 재 사용할 수 없는 반면, `함수 표현식`의 경우 함수에 필요한  매개변수와 반환 값의 타입을 한번에 표현해 훨씬 간결하면서도 재 사용성도 높이는 것을 볼 수 있다.



```typescript
async function checkedFetch(input: RequestInfo, init?: RequestInit) {
  const response = await fetch(input, init);
  if (!response.ok) {
    throw new Error(`${response.status}`);
  }
  return response;
}

const checkedFetch: typeof fetch = async (input, init) => {
  const response = await fetch(input, init);
  if (!response.ok) {
    throw new Error(`${response.status}`);
  }
  return response;
};

```



위의 예제는 내가 주로 쓰듯이 각각의 매개변수의 타입을 정해 준 모습이고, 아래는 내장된 `fetch` 타입으로 훨씬 간결하게 나타낸 모습이다. 이렇게 동일한 매개변수와 반환 값의 타입을 가지는 함수의 경우 함수 전체의 타입을 정해 재 사용하는 것이 효율적인 것을 새롭게 알게 되었다.



## 🥊 타입 VS Interface

타입과 interface는 항상 고민되는 문제다. 공통점이 많기 때문에 어떤 점을 기준으로 사용해야 할까 고민되는 경우가 많았다. 이러한 고민은 둘 다 가능한 공통점에서 시작되었다.

### 공통점

#### 1) 타입 정의

```typescript
type TState = {
  name: String;
  capital: string;
};

interface IState {
  name: string;
  capital: string;
}
```

둘 다 동일하게 커스텀 타입을 정의할 수 있다.



#### 2) Index와 함수 정의

```typescript
type TDict = { [key: string]: string };
interface IDict {
  [key: string]: string;
}

type TFn = (x: number) => string;
interface IFn {
  (x: number): string;
}
```

둘 다 동일하게 index와 함수를 정의할 수 있다.



#### 3) Generic과 확장

```typescript
type TPair<T> = {
  first: T;
  second: T;
};

interface IPair<T> {
  first: T;
  second: T;
}

interface IStateWithPop extends TState {
  population: number;
}

type TStateWithPop = IState & { population: number };
```

둘 다 Generic을 사용할 수 있고 확장도 가능하다.



#### 4) 클래스 구현

```typescript
class StateT implements TState {
  name: string = '';
  capital: string = '';
}

class StateI implements IState {
  name: string = '';
  capital: string = '';
}
```

클래스를 구현하는 것도 둘 다 가능하다.





### 차이점

대부분이 둘 다 가능하기 때문에 차이점이 없어보이지만 interface만 가능한 것과 type만이 가능한 역할이 있다.

#### 복잡한 type

```typescript
type AorB = 'a' | 'b';
```

union type이나 조건부 타입과 같이 좀 더 복잡한 type을 위해서는 interface가 사용될 수 없다.  type은 활용성이 interface보다 높다고 할 수 있다.

#### 보강

```typescript
interface IState {
  name: string;
  capital: string;
}

interface IState {
  population: number;
}
```

interface는 type과는 다르게 속성을 같은 이름의 interface을 선언해 확장할 수 있는 특징을 가진다. 이것을 통해서 실제로 우리가 사용하는 내장 메소드들의 정의가 버전 별로 확장되어 적용되고 있다.

 

### 🤔 그래서 기준은 어떤 것일까?

사용할 때 기준은 먼저 사용할 때 `일관성을 유지해야 한다`는 점이다. type과 interface는 공통점이 많기 때문에 둘 다 가능한 경우가 많다. 그렇지만 type을 쓰다가 interface를 쓰는 것이 아니라 하나로 정해서 일관되게 작성하는 코드스타일이 중요하다.

각각의 차이점을 고려해서 복잡한 타입은 `type`을 사용하고 보강이 필요한 경우에는 `interface`를 이용해 API를 정의할 때 사용할 수 있다. 



[참조]

[이펙티브 타입스크립트](https://search.shopping.naver.com/book/catalog/32473346832)
