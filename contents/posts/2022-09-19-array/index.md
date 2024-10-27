---
title: "Array"
date: 2022-09-19
slug: javascript-array
tags: [javascript, 문법]
---

자바스크립트에서의 배열은 객체로, index를 통해 데이터에 접근할 수 있게 정리해놓은 자료구조이다. 다른 언어들과는 다르게 메모리셀에서 변수들이 연속적으로 저장되어 있지는 않는다. 유용한 자료구조 배열에 대해 알아보자.

### 1. Array 만들기

배열을 만드는 방법에는 총 4가지 방법이 있다.

1. new Array(size): size만큼의 길이를 가진 배열을 만든다.

   new Array(data1,data2,...): 전달받은 데이터를 가진 배열을 만든다.

2. Array.of(data1,data2,...): Array 객체의 static함수인 of 를 이용해 전달받은 데이터를 가진 배열을 만든다
3. [data1,data2,...] : array literal로 간단하게 배열을 만들 수 있다.
4. Array.from(array) : Array 객체의 static 함수인 from을 이용해 전달받은 배열을 복사할 수 있다.

다음과 같이 정리된 배열을 이용해보면 다음과 같이 만들 수 있다.

```javascript
let array = new Array(3) //(3) [비어 있음 × 3]
array = new Array(1, 2, 3) //[1, 2, 3]
array = Array.of(1, 2, 3) //[1, 2, 3]
array = [1, 2, 3] //[1,2,3,]
arr2 = [1, 4, "A"]
array = array.from(arr2) //[1,4,'A']
```

### 2. Array의 요소 추가, 삭제

배열은 index를 이용해 데이터에 접근할 수 있는 자료구조로, 요소를 추가하고 삭제할 수 있는 함수들이 방법들이 존재한다.

#### 2.1 add와 delete (비추)

add와 delete는 배열의 index를 이용해서 요소를 추가 삭제하는 방법이지만 index를 직접 이용하기 때문에 배열의 크기나 위치에 따른 데이터 값을 모를 경우에 문제가 될 수 있으며 delete의 경우 삭제하고 빈 공간이 남아 있는 문제가 있다.

#### 2.2 push와 pop

**push**와 **pop**은 배열의 끝에 요소를 추가하고 삭제하는 방법이다. **push**는 요소를 추가하고 난 다음의 배열의 길이를 반환하고, **pop**은 제거한 요소를 반환한다.

맨뒤의 요소만 제거하고 추가하기 때문에 O(1)의 시간복잡도를 가진다.

#### 2.3 unshift와 shift

**unshift**와**shift** 는 배열의 처음에 요소를 추가하고 삭제하는 방법이다. **unshift**는 push와 같이 요소를 추가하고 난 다음의 배열의 길이를 반환하고, **shift** 는 제거한 요소를 반환한다. **unshift**와 **shift**는 **push**와 **pop**과는 다르게 배열의 맨앞의 요소를 제거하고 추가하기 때문에 전체적인 배열요소들의 이동이 있다. 그렇기 때문에 **unshift**와 **shift**의 시간 복잡도 O(n)로 더 오래걸린다.

```javascript
const alphabets = ["a", "b", "c"]
console.log(alphabets.push("d")) //4
console.log(alphabets) //['a','b','c','d']
console.log(alphabets.pop()) //'d'

const alphabets = ["a", "b", "c"]
console.log(alphabets.unshift("d")) //4
console.log(alphabets) //['d','a','b','c']
console.log(alphabets.shift()) //'d'
```

### 3. Array의 다양한 속성과 함수

배열의 매력은 다양한 내장함수를 지원한다는 점이라고 한다. 자주 사용했던 함수들을 정리해보려한다.

#### 3.1 배열의 정보를 반환

1. array.length: 배열의 길이를 반환
2. Array.isArray(arr): 배열인지 아닌지 True/False
3. array.indexof(value): value가 있는 인덱스를 반환, 없으면 -1
4. array.includes(value): value를 배열이 포함하는지 True/False

#### 3.2 배열을 수정하는 함수

앞서 정리한 추가,삭제 함수들과 함께 배열 자체를 수정하는 함수를 정리하면 다음과 같다.

1. array.splice(start,count,value):

   splice는 배열의 중간에서 값을 삽입하거나 삭제할 때 사용할 수 있는 함수로 start 인덱스로부터 count 갯수만큼 삭제하고, value를 삽입할 수 있다.

2. array.fill(value,start,end):

   fill는 배열에 start부터 end-1까지 value로 바꾸거나 삽입한다.

#### 3.3 새로운 배열을 반환하는 함수

배열을 직접 수정하는 게 아니라, 배열을 변환해서 새로운 배열을 반환하는 함수들은 다음과 같다.

1. array.concat(arr): 전달받은 배열의 요소들을 array에 추가한 새로운 배열을 반환한다.
2. array.reverse(): 배열의 순서를 뒤집은 배열을 반환한다.
3. array.flat(value): 배열 내 요소로 또다른 배열이 있을 때, 또다른 배열의 요소들을 요소로 가지고 있는 새로운 배열을 반환한다.
4. arr.join(string): 새로운 배열은 아니지만 배열을 문자열로 변환해서 반환한다.
5. arr.slice(start,end): start 인덱스부터 end-1 인덱스까지 배열을 자른 새로운 배열을 반환한다.

```javascript
const alphabets = ["a", "b", "c"]
alphabets.splice(1, 1)
console.log(alphabets) //["a","c"]
alphabets.splice(1, 0, "d")
console.log(alphabets) //["a",'d','c']

const alphabets1 = ["a", "b", "c"]
const alphabets2 = ["d", "e", "f"]
const alphabets3 = alphabets1.concat(alphabets2)
console.log(alphabets3) // ['a', 'b', 'c', 'd', 'e', 'f']

let arr = [[1, 2, 3], 4, 5]
const flatArr = arr.flat(1)
console.log(flatArr) //[1,2,3,4]
const text = arr.join(" ") //"1,2,3 4"
console.log(arr.slice(1, 3)) //[4,5]
```

### 4. 배열과 Shallow Copy

자바스크립트에서의 복사는 항상 **shallow copy**를 통해 진행된다. **shallow copy**는 객체를 복사할 때, 새로운 객체를 만든 후에 내부의 속성, 함수들을 모두 새로 만드는 것이 아니라, 원본 객체의 내부의 값들을 가져온다. 그렇기 떄문에 아래 그림과 같이 새로운 객체를 만들었지만 같은 reference값을 가지고 있다.

이에 반해 **Deep copy**는 객체뿐아니라 내부 값들 모두다 새롭게 만들어 추가한다.

![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--CjdqwIq1--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/llosmmb3rzbq5ravmfcp.jpg)

이점이 중요한 이유는 원본 객체 내부의 값의 변화가 생기면 **shallow copy**로 만든 객체에도 영향을 주기 때문이다.

```javascript
const person1 = { firstName: "Youngjun", lastName: "Choi" }
const person2 = { firstName: "Youngjun", lastName: "Park" }
const contact1 = [person1, person2]
const contact2 = [...contact1]

contact1.pop()
console.log(contact1) //[person1]
console.log(contact2) //[person1,person2]

person1.lastName = "Son"
console.log(contact1) //[{firstName:"Youngjun", lastName:"Son"}]
console.log(contact2) //[{firstName:"Youngjun", lastName:"Son"},person2]
```

위 코드를 보면 contact2는 contact1을 복사해서 만든 배열이다. contact1의 값의 변화가 있더라도 contact2는 서로 다른 배열이기 때문에 영향을 받지않는다. 하지만 person1의 속성이 바뀌면 person1을 가지고 있는 contact1과 contact2 모두 변화가 생기는 것을 볼 수 있다. 그렇기 때문에 배열이나 오브젝트 내부의 값을 바꿀 때 사이드 이펙트가 생길 수도 있다는 것을 알고 있어야 한다.

### 5. Array의 고차함수

자바스크립트는 일급함수를 다루기 때문에 고차함수를 이용한 함수간의 chaining, 함수형 프로그래밍이 가능하다. 배열 내부에는 좀 더 편리하게 사용할 수 있는 고차함수가 있는데 이에 대해 정리해보려한다.

1.  `array.forEach( (item,index,array ) => { sth })`: array 각 요소에 전달한 callback 함수를 실행 할 수 있다.

2.  `array.find( (item) => condition )` : array 요소중에서 처음으로 조건에 맞는 요소를 반환한다.
3.  `array.findIndex( (item) => condition )` : array 요소중에서 처음으로 조건에 맞는 요소의 index를 반환한다.
4.  `array.some( (item) => condition )` : array 요소중에서 하나라도 조건을 만족하는지 True/False
5.  `array.every( (item) => condition )` : array 요소중에서 모두 조건을 만족하는지 True/False
6.  `array.filter( (item) => condition )`: array 요소중에서 조건을 만족하는 요소들만을 모은 배열을 반환한다.
7.  `array.map( (item) => sth )` : array 요소들을 다른 값으로 변환해
8.  `array.flatMap( (item) => sth )` : array 요소들을 배열을 제거하고 다른 값으로 변환해서 반환한다.
9.  `array.sort((a,b)=> condition)` : condition이 a-b는 내림차순, b-a는 오름차순으로 정렬한다.
10. `array.reduce((prev,curr)=>(sth), initialVal)`: 초기값을 시작으로 curr값을 callback함수에 맞게 prev값에 더해간다.
