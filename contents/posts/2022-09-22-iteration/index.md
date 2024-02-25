---
title: 'Iteration'
date: 2022-09-22
slug: javascript-iteration
tags: [javascript, 문법]
---

Iteration은 순회라는 뜻으로, 순서대로 item에 접근할 수 있을 때를 의미한다. 앞서 정리한 **배열, 문자열**는 순회가 가능한데 이렇게 순회가 가능하기 위해서는 **iterable protocol**을 따라야 한다. Protocol이란 약속이나 규약을 의미하므로 이런 규칙을 따른다는 의미로 이해할 수 있다.

먼저 iterable protocol에 대해 정리해보자.

### 1. Iterable Protocol

iterable protocol을 따른다는 말은 "객체가 **[Symbol.iterator]**라는 메소드가 존재하고, [Symbol.iterator]는 **iterator protocol을 따르는 객체를 반환한다"**는 것을 의미한다. 그럼 iterator Protocol은 어떤 규칙일까?

## 2. Iterator Protocol

Iterator Protocol을 따른다는 것은 **next()**라는 메소드를 가지고 있는 객체로, next()는 다시 값을 담고 있는 **value**와 순회가 끝난지를 담고 있는 **done**이라는 두가지 속성을 가지고 있는 객체를 반환한다.

위에 헷갈리는 두가지 프로토콜을 정리하면 다음과 같이 정리할 수 있다.

1. 순회가 가능하려면 Iterable Protocol을 따라야한다.
2. Iterable Protocol은 [Symbol.iterator]메소드가 Iterator protocol을 따르는 객체를 반환한다는 뜻이다.
3. Iterator Protocol은 next라는 메소드가 value와 done이란 두가지 속성을 가지고 있는 객체를 반환한다.

Iteration procol을 따르는 iterator의 예로 배열이 있고, iterator를 반환하는 Object.values()나 Object.entries()가 있다.

직접 iteration이 가능한 객체를 만들어보면 다음과 같이 코드를 만들 수 있다.

```javascript
const iterableObj = {
  [Symbol.iterator]() {
    let num = 0;
    const end = 5;
    return {
      next() {
        return { value: num++, done: num > end };
      },
    };
  },
};

/*
0
1
2
3
4
*/
```

### 3. iteration이 가능하면 뭐가 좋을까?

이렇게 Iteration이 가능하면 뭐가 좋을까? 바로 for...of 구문을 사용할 수 있다. 그냥 순회하는 코드는 다음과 같이 작성할 수 있다.

```\javascript
const arr=[1,2,3]
const iterator=arr.values() //iterator
while(true){
    const item=iterator.next()
    if (item.done)break
    console.log(item.value)
}
/*
1
2
3
*/
```

for... of를 사용하면 보다 간단하게 사용할 수 있다. for... of 구문은 iteration이 가능한 경우에만 사용할 수 있기에 일반적인 객체는 for...of 구문은 사용하지 못하고 key값을 이용하는 for...in을 이용할 수 있다.

```javascript
const arr = [1, 2, 3];
for (const item of arr) {
  console.log(item);
}
/*
1
2
3
*/

const obj = {
  1: '1',
  2: '2',
  3: '3',
};

for (const key in obj) {
  console.log(key);
}

/*
"1"
"2"
"3"
*/
```

### 4. Generator

Generator는 iteration protocl을 따르는 iterator를 간단하게 만들 수 있는 방법으로 다음과 같이 만들 수 있다.

```javascript
function* iteratorGenerator(start, end) {
  for (let i = start; i < end; i++) {
    yield i;
  }
}

const generator = iteratorGenerator(0, 5);
let result = generator.next();
while (!result.done) {
  console.log(result.value);
  result = generator.next();
}

/*
0
1
2
3
4
*/
```

generator 함수에는 return 대신 **yield**를 사용해, 그때그때 값을 무조건 반환하는 게 아니라 사용자의 요구에 따라 값을 전달 할 수 있다. generator에는 yield로 전달받은 value와 done을 담고 있는 객체를 받는 **next()**, 에러를 던질 수 있는 **throw()**와 순회를 끝낼 수 있는 **return()** 메소드가 존재한다.

실제로는 generator를 직접만드는 경우는 드물지만, 조건에 맞는 객체를 만들고 호출하는 과정을 이해하는 것에 중요함을 느낄 수 있었다.
