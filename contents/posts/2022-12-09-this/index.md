---
title: 'this'
date: 2022-12-09
slug: javascript-this
tags: [javascript, 문법]
---

# 👉 This

자바스크립트에서 this는 "동적"으로 정해진다라고 한다. 기존 객체지향 언어들과는 다르게 작동하는 자바스크립트의 this에 대해 정리해 보려 한다.

## 🙋‍♂️ This란

this는 **자신이 속한 객체 또는 만들어 질 인스턴스를 가리키는 변수**이다. 이해를 위해 다음 코드를 보자

```javascript
const square = {
  distance: 5,
  getArea() {
    return square.distance ** 2;
  },
};

console.log(square.getArea());
```

square 객체의 getArea 메소드는 distance를 이용해 넓이를 계산한다. 이때 중요한 것은 square가 가지고 있는 속성인 `distance`값이 필요하다는 점이다. 객체를 선언한 변수 square를 입력하면 distance를 사용할 수는 있지만, 항상 객체를 선언한 식별자를 이용한다면 오직 square라는 이름의 객체에서만 사용이 가능해 재사용성이 떨어지게 된다.

이러한 재사용성을 고려해 객체가 어떻게 선언되는지에 상관없이, 어떤 이름의 인스턴스인지 상관없이 **항상 자기 자신을 참조할 수 있는 변수**가 바로 this라고 이해할 수 있다.

위를 객체 리터럴, 생성자함수, class로 다음과 같이 표현할 수 있다.

```javascript
const square1 = {
  distance: 5,
  getArea() {
    return this.distance ** 2;
  },
};

function ProtoSquare(distance) {
  this.distance = distance;
}
ProtoSquare.prototype.getArea = function () {
  return this.distance ** 2;
};

const square2 = new ProtoSquare(5);
console.log(square2.getArea());

class ClassSquare {
  constructor(distance) {
    this.distance = distance;
  }
  getArea() {
    return this.distance ** 2;
  }
}

const square3 = new ClassSquare(5);
console.log(square3.getArea());
```

이러한 this는 "어떻게" 결정되는 걸까?

## 👍 this가 결정되는 방식

this는 함수가 어떻게 `호출되느냐`에 따라 동적으로 결정되는데, this 바인딩에는 4가지 규칙이 있다. 4가지 규칙에 대해 알아보자.

### 1) 기본바인딩

객체의 메소드나 생성자 함수를 제외한 함수들, 일반함수로 함수 내부에서 this를 호출하면 `전역객체`를 얻을 수 있다. 전역객체는 브라우저냐 Node.js냐에 따라 각각 window와 global을 의미한다. 브라우저 기준으로 전역객체는 window로 설명을 진행하려 한다.

```javascript
function hi() {
  console.log(this); //window 또는 global
}
hi();
```

이때 메소드 내부 중첩 함수나 콜백함수가 일반함수로 호출될 경우에도 동일하게 전역 객체로 this 바인딩이 된다.

```javascript
var value = 1;

const obj = {
  value: 100,
  foo() {
    setTimeout(function () {
      console.log(this); // window
      console.log(this.value); // 1
    }, 100);
  },
  too() {
    function bar() {
      console.log(this); // window
      console.log(this.value); // 1
    }
    bar();
  },
};

var value = 1;

const obj = {
  value: 100,
  too() {
    'use strict';
    function bar() {
      console.log(this); // undefined
      console.log(this.value); // Uncaught TypeError: Cannot read properties of undefined
    }
    bar();
  },
};

obj.too();
```

이때 use strict로 엄격모드를 적용하면 전역객체는 기본 바인딩에서 제외되어 this.value는 에러를 던지게 된다.

### 2) 암시적 바인딩

호출할 때의 객체의 존재 여부에 따라, this는 만들어진 객체에 속해지지 않고, `호출한 객체, 컨텍스트 객체`에 바인딩된다. 이것을 **암시적 바인딩**이라 하는데 이해를 위해 다음 예제를 보자.

```javascript
const person = {
  name: 'lee',
  getName() {
    return this.name;
  },
};

console.log(person.getName()); //lee

const anotherPerson = {
  name: 'kim',
};

anotherPerson.getName = person.getName;
console.log(anotherPerson.getName()); //kim

const getName = person.getName;
console.log(getName()); // window
```

위의 코드를 보면 getName이 **함수가 사용되는 곳에 따라 this가 달라진다**. 객체에서 선언된 this라면 해당 객체와 binding이 되어야 하지만, 호출하는 객체가 누구냐에 따라 달라지는 결과를 보여준다. 그렇기 때문에 함수가 가리키는 객체는 독립적인 `컨텍스트 객체`라고 볼 수 있다. 이러한 결과는 함수의 컨텍스트는 함수가 실행될 때 만들어지고, 만들어지는 과정에서 this가 바인딩되기 때문에 그때의 컨텍스트 객체가 this에 바인딩된다.

```javascript
function foo() {
  console.log(this.text);
}

var name = {
  text: '영준',
  foo: foo,
};

var person = {
  text: '가영',
  name: name,
};

person.name.foo();
```

만약 중첩된 상태에서 호출하게 된다면 가장 가까운 객체, name을 암시적으로 컨텍스트 객체로 바인딩한다.

이렇게 어디서 호출하냐에 따라 달라지면 예측하기 어렵고 에러가 발생할 수 있다. this binding을 우리가 정하는 방법은 없을까?

### 3) 명시적 바인딩

this binding을 명시적으로 정해주는 방식에는 apply, call, bind 메소드를 사용하는 방법이 있다.

#### apply와 call

두 가지 메소드는 사용 시 전달한 객체를 this에 바인딩하고, 호출한다. 두가지 메소드의 차이점은 함수에 인자를 전달하는 방식에 있다. apply는 배열로 인자를 전달하고 call은 하나 하나씩 전달하고 호출한다.

```javascript
function getThisBinding() {
  console.log(arguments);
  return this;
}

const thisArg = { a: 1 };
console.log(getThisBinding());
console.log(getThisBinding.apply(thisArg, [1, 2, 3])); //[1, 2, 3]
console.log(getThisBinding.call(thisArg, 1, 2, 3)); //[1,2,3]
```

#### bind

bind는 this만 바인딩하고 호출은 하지 않는다. 콜백함수나 중첩함수가 일반함수로 호출되었을 때, this가 달라지는 문제를 명시적으로 해결해 줄 수 있다.

```javascript
const person = {
  name: 'lee',
  foo(callback) {
    setTimeout(callback.bind(this), 100);
  },
};

person.foo(function () {
  console.log(this.name); //lee
});
```

### 4) new 바인딩

생성자함수 내부의 this는 인스턴스가 바인딩된다.

```javascript
function foo() {
  this.name = '영준';
  this.callName = function () {
    console.log(this.name);
  };
}

const bar = {
  name: '주희',
};

const x = new foo();
x.callName.bind(bar);
x.callName(); // 영준
```

생성자 함수를 이용해 만든 x 객체에 명시적 바인딩인 bind로 bar를 바인딩하려했지만, new 바인딩이 우선순위가 높아 "영준"으로 나온 것을 볼 수 있다.

### 🔼 예외적인 바인딩: 화살표 함수

앞서 설명한 this 바인딩과는 다르게 작용한다.화살표 함수는 함수 자체가 this를 갖지 않아 함수 선언 시의 **상위 스코프의 this를 바인딩**한다. 항상 this를 보장하기 때문에 callback함수에서 주로 사용된다.

```javascript
function foo() {
  setTimeout(() => {
    console.log(this.name);
  }, 1000);
}

const bar = {
  name: '영준',
};

foo.call(bar); // 영준
```

위의 예시를 보면 setTimeOut 콜백함수의 this는 foo의 this를 참조하는데 현재 call을 이용해 bar로 바인딩되어 결과가 철수로 나타난 것을 볼 수 있다.

```javascript
const foo = {
  name: '영준',
  bar: () => console.log(this.name),
};

foo.bar(); // undefined

const boo = {
  name: '영준',
  far() {
    console.log(this.name);
  },
};

boo.far(); //영준
```

하지만 주의할 점은 메소드를 화살표함수로 만들게 되면 foo 객체가 아니라 상위 스코프의 this인 전역객체를 binding하게 된다. 그렇기 때문에 메소드는 메소드 축약으로 정의하는 게 낫다는 것을 알 수 있다. 여기서 헷갈렸던 것은 bar라는 메소드를 함수 스코프로 생각해야 하는지 여부였다. 메소드는 결국 객체의 키와 연결된 함수의 참조값이므로 이것 자체가 함수 스코프를 가지진 않아 위와 같은 결과를 갖는 것을 볼 수 있다.

## 마치며

this 바인딩은 예상치 못하는 에러를 만들 수 있기 때문에 꼭 한번 정리를 하고 넘어가야 했다. 아직 화살표 함수에서 마지막 부분이 왜 전역객체를 바인딩하는지 잘 이해가 되지 않아 생성자함수와 prototype에 대해서 정리하고 다시한번 확인을 해보려고 한다.

[참고]

[모던 자바스크립트 딥다이브](http://www.yes24.com/Product/Goods/92742567)
[this 바인딩](https://doqtqu.tistory.com/307#2.2.%20Binding%20%EA%B7%9C%EC%B9%99)
