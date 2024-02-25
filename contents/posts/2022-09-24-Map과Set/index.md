---
title: 'Map과 Set'
date: 2022-09-24
slug: javascript-map-set
tags: [javascript]
---

Map과 Set은 자바스크립트의 자료구조로, map은 Object와 유사한 구조를 가지고, Set은 중복을 없애주는 등에 사용된다. 유용한 자료구조이기에 정리를 한번 해보려 한다.
## Set

Set 자료형은 위에 언급한대로 데이터의 중복을 없을 뿐 아니라 배열과 달리 순서를 가지지도 않는 특징을 가진다.

### 1. Set 만들기

set은 간단하게 class를 사용하듯이 new를 이용해서 만들 수 있고, 내부 값을 전해줄 수 있는데 이때 주의할 점은 iterable한 값을 전달해주어야 한다.

앞서 정리했던 iterable한 값으로, array, 문자열, Object의 keys, values등을 전달해줄 수 있다.

```javascript
const set = new Set('hello');
console.log(set); //Set(4) {'h', 'e', 'l','l', 'o'}

const set2 = new Set([1, 2, 3, 4]);
console.log(set2); //Set(4) {1, 2, 3, 4}
```

### 2. 추가,삭제

추가,삭제를 위해서는, Set의 instance레벨의 함수인 **add**와 **delete**를 이용해야한다. 내부 값을 모두 삭제하고 싶을 때는 **clear**를 사용할 수 있다.

```javascript
const set = new Set([1, 2, 3, 4]);
set.add(5); //Set(5) {1, 2, 3, 4, 5}
set.delete(3); //Set(4) {1, 2, 4, 5}
set.clear(); //Set(0) {size: 0}
```

### 3. 길이와 포함여부

Set의 길이는 배열의 length와 다르게 **size**를 이용해서 얻을 수 있으며, 특정 자료를 가지고 있는지 **has**를 통해 알 수 있다.

```javascript
const set = new Set([1, 2, 3, 4]);
set.size; //4
set.has(5); //false
set.has(4); //true
```

### 4. 활용

set는 유일한 데이터만을 가지기 때문에 자주 쓰이는 배열내부의 중복되는 데이터를 정리하는데 사용할 수 있다.

```javascript
const duplicatedItems = [1, 1, 2, 3, 4, 5, 5];
const uniqueItems = [...new Set(duplicatedItems)]; //(5) [1, 2, 3, 4, 5]
```

### 주의점

Set자료형이 유일한 데이터를 가지지만 주의할 점으로, set에 객체가 있다면 객체의 참조값을 가지고 있기에 주소값으로 동일한 데이터인지 확인한다. 그렇기에 객체의 내부 속성이 달라진다면 set내부의 객체의 값이 달라지고, 만약 속성이 동일한 두개의 객체가 있다면, 둘의 참조값이 다르기 때문에 모두 set에 추가된다.

```javascript
const obj = {
  name: 'choi',
};
const set = new Set();
set.add(obj);
obj.name = 'park';
console.log(set); //name이 park로 바뀌어있어

const obj2 = {
  name: 'park',
};

set.add(obj2); //obj와 obj2 모두 set에 포함되어 있어
```

# Map

map을 처음 배웠을 때 느꼈던 부분은 "object"와 너무 비슷하다는 점이었다. object와 동일하게 key와 value로 값을 저장해, 해당 key를 통해 값을 전달 받을 수 잇다. map은 앞서 정리한 set과 유사한 문법을 가지는 부분들이 있어 함께 정리했다.

### 1.map 만들기

set과 동일하게 new Map()을 통해서 새로 만들 수 있으며, 바로 값을 전달할 때는 배열의 형태로 [key,value]로 전달해주어야 한다.

```javascript
const map = new Map([
  ['1', 1],
  ['2', 2],
]); //Map(2) {'1' => 1, '2' => 2}
```

### 2. get,set,delete,clear

map은 추가를 위한 **set**, key를 이용해 값을 불러오는 **get**, 값을 삭제하기 위한 **delete**, 내부 값을 모두 삭제하고 싶을 때는 **clear**를 사용할 수 있다.

```javascript
const map = new Map();
map.set('name', 'choi'); //Map(1) {'name' => 'choi'}
map.get('name'); //choi
map.delete('name'); //Map(0) {size: 0}

const map2 = new Map([
  ['1', 1],
  ['2', 2],
]); //Map(2) {'1' => 1, '2' => 2}
map2.clear(); //Map(0) {size: 0}
```

### 3. 길이와 iterator

map의 길이는 set과 동일하게 **size**를 이용해서 얻을 수 있으며, object와 같이 **iterator**들을 이용할 수 있다.

```javascript
const map = new Map();
map.set('name', 'choi'); //Map(1) {'name' => 'choi'}
map.set('age', '26'); //Map(2) {'name' => 'choi', 'age' => '26'}

map.forEach((value, key) => console.log(value, key));
/*
choi name
26 age
*/

map.keys(); //MapIterator {'name', 'age'}
map.values(); //MapIterator {'choi', '26'}
map.entries(); //MapIterator {'name' => 'choi', 'age' => '26'}
```

### 4. Object와 다른점

앞서 언급한 대로 Key와 value로 이루어져 Object와 매우 유사한 구조를 가지지만 Object와는 다른 방식을 이용해 값을 추가하고 삭제한다. 한마디로 둘은 **Interface**가 다르다고 할 수 있다.
