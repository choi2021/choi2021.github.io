---
title: "자바스크립트의 타입과 연산자"
date: 2022-09-07
slug: javascript-type-operator
tags: [javascript]
---

### 1. 자바스크립트의 타입

자바스크립트는 dynamic 프로그래밍 language로 동적으로 타입이 결정되는 언어이다. 자바스크립트는 컴파일러가 아니라 interpreter언어이기 때문이다. 위 두가지의 차이를 정리를 해보고자 한다.

##### 컴파일러와 인터프리터

우리가 쓰는 언어 (high-level)를 컴퓨터가 이해할 수 있게 변환해주는 방식에 따라 컴파일러와 인터프리터로 나뉘게 된다.

![컴파일러-인터프리터.png](..%2F2022-09-07-%EC%9E%90%EB%B0%94%EC%8A%A4%ED%8A%B8%EB%A6%BD%ED%8A%B8%EC%9D%98-%EB%91%90%EA%B0%80%EC%A7%80-%EB%B3%B5%EC%82%AC%EB%B0%A9%EB%B2%95%2F%EC%BB%B4%ED%8C%8C%EC%9D%BC%EB%9F%AC-%EC%9D%B8%ED%84%B0%ED%94%84%EB%A6%AC%ED%84%B0.png)

출처: [difference-compiler-vs-interpreter](https://www.guru99.com/difference-compiler-vs-interpreter.html)

##### 1) 컴파일러

컴파일러는 우리가 작성한 언어를 실행전에 모든 코드를 검사하고 machine code로 바꾸어주는 작업을 먼저한다. 이렇게 만들어진 machine code로 인해 타입이 정적으로 정해진다.

그렇기 때문에 machine code로 바꾸는 데 시간이 걸리지만 만들고 나서는 실행이 빠른 장점을 갖는다. 컴파일러 언어에는 C, Java와 같은 언어가 있다.

##### 2) 인터프리터

인터프리터는 우리가 작성한 언어를 한줄한줄 런타임에서 읽어 해석한다. 그렇기에 machine code로 바꾸는 작업이 없지만 실행과 함께 한줄 한줄 읽어 나가기 때문에 실행속도는 느린 단점이 있으며, 타입이 동적으로 정해진다.
인터프리터 언어에는 Python, javascript와 같은 언어가 있다.

내가 공부하고 있는 javascript는 인터프리터로 내가 쓴 코드를 javscript engine은 한 줄, 한 줄 읽으면서 해석하기 때문에
타입이 동적으로 결정되어 내가 처음에 number로 정한 변수가 나중에는 string으로 바뀌어도 에러를 던지지 않는다.

```javascript
let number = 3
number = "3"
console.log(number) //"3"
```

이렇게 유동적인 점은 빠르게 개발할 때는 좋지만, 복잡해질 수록 개발자에게는 오류를 만들 수 있는 부분이 되기 때문에 Typescript언어가 나오게 되었다.

### 연산자와 형변환

##### 1. 산술 연산자

+, -, \*, /, %, \*\* 와 같이 계산을 위한 기호를 의미한다.

여기서 주의할 점은 +인데 자바스크립트는 다른 type간의 +가 가능하다.

```javascript
const text = 1 + "1" //"11"
```

위와 같이 숫자와 string 타입간의 더하기를 통해 결과가 string으로 나오는 것을 볼 수 있다.

##### 2. 단항 연산자

단항연산자는 양수,음수를 표현하기 위한 +,-와 부정을 의미하는 !가 있다.

여기서 유용한 점은 +와 !!를 이용해 형변환이 가능하다는 점이다.

```javascript
;+false + //0
  null + //0
  true + //1
  "text" + //NaN
  undefined / NaN +
  "3" //3

!3 //false
!!3 //true
!!"" //false
```

위와 같은 방법으로 형변환을 할 때, Truthy와 falsy값을 생각하면 좀 더 이해하기 쉬웠다.

Truthy한 값을 숫자로 형변환할 때는 1로, falsy한 값을 숫자로 변환하면 0으로 변환되었다.

##### 3. 할당 연산자

할당연산자는 간단하게 연산식을 표현하기 위한 연산자다.

```javascript
let a = 3
a += 2 //a=a+2;
a -= 2 //a=a-2;
a *= 2 //a=a*2;
a /= 2 //a=a/2;
a %= 2 //a=a%2;
a **= 2 //a=a**2;
```

##### 4. 증감 연산자

증감 연산자는 간단하게 변수를 1 증가,1 감소를 표현하는 연산자다. 간편하지만 앞에 붙이냐 뒤에 붙이냐에 따라 달라질 수 있으니 주의하자

```javascript
let a = 100
const b = a++
console.log(b, a) //100,101

a = 100
const b = ++a
console.log(b, a) //101,101
```

++a 경우에 1을 증가시키는 걸 먼저한 후에 필요한 계산을 하라는 의미가 되며, a++ 경우에는 필요한 계산을 먼저한 후에 1을 증가시키라는 의미가 된다. 어떤 걸 사용해도 동일할 때도 있지만 상황에 따라 달라질 수 있으니 주의할 필요가 있다.

##### 5. 비교 연산자

비교 연산자는 `>,<,>=,<=` 4가지로 간단하다. `=<`와 같이 쓸 때가 있는데 인식하지 못하니 주의하자

##### 6. 연산자 우선순위

우선순위는 항상 괄호가 높으므로 필요한 연산에는 괄호를 꼭 사용하자. 기본적으로는 \*, / 가 + , - 보다 우선순위가 높다.

##### 7. 동등 비교 연산자

동등 비교연산자는 (==, !=) 와 (===,!==) 로 값이 같은 지를 비교할 수 있다. 이때 두가지의 차이점은 ==와 !=는 값만 비교하는 반면 ===와 !==는 값과 타입을 같이 비교한다. 실제 개발할 때는 타입까지 고려한 ===과 !==를 이용하는 게 좋다.

```javascript
const num1 = 2
const num2 = "2"
console.log(num1 == num2) //true
console.log(num1 === num2) //false
```

- key와 value가 같은 오브젝트 비교

```javascript
const obj1 = { firstname: "Youngjun" }
const obj2 = { firstname: "Youngjun" }
const obj3 = {}
const obj4 = {}
console.log(obj1 == obj2) //false
console.log(obj3 == obj4) //false
```

위에 키와 값이 같은 오브젝트 2가지를 비교했지만 결과는 false이다. 그이유는 어제 공부했던 오브젝트를 저장하는 변수에는 오브젝트 자체가 아닌 오브젝트를 보관하고 있는 곳의 메모리 주소가 저장되기 때문이다. 그렇기에 새롭게 만들어지는 오브젝트는 항상 서로 다르다고 할 수 있다.
