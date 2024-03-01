---
title: "TS study: 타입 시스템 (3)"
date: 2023-01-16
slug: typescript-타입시스템-3
tags: [typescript]
series: Typescript
---



# 🎚 타입 시스템(3)

타입시스템에 대한 정리 중 마지막으로 **제네릭, 인덱스 시그니처, Array 타입과 readonly**에 대해 정리해보려 한다.



## 🕹 제네릭

타입스크립트를 사용할 수록 느끼는 점은 타입을 다룬다는 것은 추가적인 변수와 함수를 다루는 것 같았다. 그 이유는 변수를 재 사용하고 반복되는 로직은 함수로 분리하듯 타입의 재사용성을 고려해야 했기 때문이다. 코드 반복을 줄이려는 노력은 **DRY(Don't Repeat Yourself)**로 불리는 프로그래밍의 기본 원칙으로 타입에도 적용된다.

### 1. 타입 네이밍

중복되는 변수가 있다면 하나의 상수로 정해서 사용하듯이 타입에 이름을 붙여서 사용해 중복을 제거할 수 있다.

```typescript
// 네이밍 전
function distance(a: { x: number; y: number }, b: { x: number; y: number }) {
  return Math.sqrt(Math.pow(a.x - b.x, 2) + Math.pow(a.y - b.y, 2));
}

// 네이밍 후
interface Point2D {
  x: number;
  y: number;
}

function distance(a: Point2D, b: Point2D) {
  return Math.sqrt(Math.pow(a.x - b.x, 2) + Math.pow(a.y - b.y, 2));
}
```

위 예제는 x, y의 타입을 정리한 오브젝트 타입이 반복되는 상황이다. 이점을 해결하기 위해 `Point2D`로 interface를 정의해 서로 다르게 정해져있던 타입을 하나로 정리해줄 수 있다. 이러한 방법은 앞서 정리한 함수 표현식으로 매개변수와 반환 값의 타입을 정리했던 것과 같다.

```typescript
// 네이밍 전
function get(url: string, opts: Options): Promise<Response>{ }
function post(url: string, opts: Options): Promise<Response>{ }

// 네이밍 후
type HTTPFunction = (url: String, opts: Options) => Promise<Response>;
const get:HTTPFunction=(url,opts)=>{}
const post:HTTPFunction=(url,opts)=>{}
```



### 2. extends와 intersection 연산자 (&)

하나의 타입이 있고 해당 타입의 속성에 다른 속성을 추가한 타입을 만들 때 새롭게 만들기 보다 앞서 배웠던 `extends`와 `&`를 이용해 속성을 추가할 수 있다.

```typescript
// 변경 전

interface Person {
  firstName: string;
  lastName: string;
}

interface PersonWithBirthDate {
  firstName: String;
  lastName: string;
  birth: Date;
}

// 변경 후

interface Person {
  firstName: string;
  lastName: string;
}

interface PersonWithBirthDate extends Person {
  birth: Date;
}

type PersonWithBirthDate=Person&{birth: Date}
```



### 3. 부분 타입

기존에 정의한 타입의 일부분 속성을 정의할 때 새롭게 정의하고 확장해서 사용했다. 하지만 논리적으로 맞지 않다고 느낀 적이 많았는데 책에서는 `인덱싱`을 이용해 중복을 제거하는 방법을 제시한다.

```typescript
interface State {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  pageContents: string;
}

type TopNavState = {
  userId: State['userId'];
  pageTitle: State['pageTitle'];
  recentFiles: State['recentFiles'];
};
```

위 방법을 이용하면 부분 타입을 정의할 수 있다. 하지만 일일이 속성을 나열해 반복되는 부분이 있다. 이점을 해결하기 위해 `mapping`을 이용할 수 있다,

```typescript
type TopNavState = {
  [k in 'userId' | 'pageTitle' | 'recentFiles']: State[k];
};

type TopNavState = Pick<State, 'userId' | 'pageTitle' | 'recentFiles'>;
//  type Pick<T, K extends keyof T> = {
//     [P in K]: T[P];
// };
```

위 예시의 첫 번째는 `mapping`을 이용해 반복되는 속성 key를 순회하며 State에 대입한 값의 타입을 받아와 코드 중복을 제거했다. 이러한 `mapping`은 유틸타입`Pick`으로 정의되어 있어 두 번째 예제로 좀 더 편하게 나타낼 수 있다. `Pick`을 사용할 때는 먼저 참조할 Type을 가져오고 해당 타입에서 가져올 key값을 두 번째 인자로 전달한 Generic을 이용해 나타낼 수 있다.   



`Pick` util type은 유용하지만 조심해야 할 부분은 객체 type을 정의한다는 점이다.

```typescript
interface SaveAction {
  type: 'save';
}

interface LoadAction {
  type: 'load';
}

type Action = SaveAction | LoadAction;
type ActionType = 'save' | 'load';
```

위 예제에서 `ActionType`은 이미 정의한 `'save'`와 `'load'`를 다시 적어 코드 중복이 발생했다. 이점을 해결하기 위해서 `Pick`을 이용하면 될 것 같지만 이때는 `mapping`을 이용하는 것이 의도에 더 맞다.

```typescript
type ActionType = Action['type']; //'save' | 'load';

type ActionRec = Pick<Action, 'type'>; // {type:'save' | 'load'}
```



### 4. keyof 와 Partial util 타입

```typescript
interface Options {
  width: number;
  height: number;
  color: string;
  label: string;
}

interface OptionUpdate {
  width?: number;
  height?: number;
  color?: string;
  label?: string;
}

class UIWidget{
  constructor(init: Options) { }
  update(options:OptionUpdate){}
}
```

위 예제는 `Options`와 동일한 key를 가진 `OptionUpdate`타입을 정의하는데 선택적으로 만들기 위해 새롭게 정의한 것을 볼 수 있다.



```typescript
type OptionsUpdate = { [k in keyof Options]?: Options[k] };

class UIWidget {
  constructor(init: Options) {}
  update(options: Partial<Options>) {}
}

// type Partial<T> = {
//   [P in keyof T]?: T[P];
// };

```

위 예제를 보면 `keyof`를 이용해 `Options`의 key들을 받아오고 새롭게 정의해 코드 중복을 제거했다. key들을 모두 선택적으로 만들기 위해 utilType `Partial`이 존재한다. `Partial`의 정의를 보면 앞서 `keyof`를 그대로 사용한 것을 볼 수 있다.



### 5. typeof

`typeof`는 원시값의 타입을 정의하고 재 사용할 때 요즘 가장 많이 사용하는 연산자인 것 같다. `typeof`는 원시 값 뿐만 아니라 값에 대해 알고 있을 때 해당 값의 타입을 정의할 때 간단하게 사용할 수 있다.

```typescript
// typeof 전
const INIT_OPTIONS = {
  width: 640,
  height: 40,
  color: 's',
  label: 's',
};

interface OPtions{
  width: number;
  height: number;
  color: string;
  label: string;
}

// typeof 후 
type OPtions = typeof INIT_OPTIONS; // { width: number; height: number; color: string;label: string; }

```



함수의 경우 반환 값의 타입을 정의하고 싶을 때 `typeof`와 utiltype `ReturnType`을 이용할 수 있다.

```typescript
function getUserInfo(userId: string) {
  return {
    userId,
    name,
    age,
    height,
    weight,
    favoriteColor,
  };
}

type UserInfo = ReturnType<typeof getUserInfo>;
// type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
```

`ReturnType`의 정의의 조건부 부분이 아직 이해가 되지 않아 이후에 한번 더 볼 필요가 있을 것 같다.



제네릭은 타입을 유연하게 사용할 수 있는 장점을 가지지만 유연성은 `범위`가 중요하다. 이러한 범위를 정의해주지 않는다면 타입을 정의해주기 전과 같아질 수 있으므로  `extends`와 `keyof`와 같은 연산자를 이용해 범위를 명시적으로 정해 줄 수 있다. 

```typescript
type Pick<T, K> = {
  [k in K]: T[k]; // This type parameter might need an `extends string | number | symbol` constraint.
};


type Pick<T, K extends keyof T> = {
  [k in K]: T[K];
};
```

위의 예제는 k의 범위를 몰라 에러가 발생했지만 `extends`와 `keyof`를 이용해 key값의 타입으로 정의해 해결할 수 있다.



## 😃 인덱스 시그니처

`인덱스 시그니처`는 사용하면서 어려운 부분 중 하나였다. 구체적인 타입을 지정하지 못하기 때문에 `Object.keys`와 같이 배열로 나열하는 방식에 어려움을 겪었다. 이번 기회를 통해 좀 더 제대로 이해하고 사용하고자 했다. 

```typescript
type Rocket = { [property: string]: string };
```

위 예제에서 key를 정확히 명시하지 않기 때문에 자동 완성의 도움을 받을 수 없고, value의 타입이 string이 아니라 다른 타입을 가질 수 없는 단점을 가진다. 그렇기 때문에 보다 정확하게 type을 정의할 필요가 있다. 

그러면 `인덱스 시그니처`는 **어디에** 쓰여야 할까? `인덱스 시그니처`는 동적데이터를 표현할 때 사용되어야 한다. 미리 알 수 없는 데이터를 받아 와야 할 때 사용할 수 있다.

```typescript

function parseCSV(input: string): { [columnName: string]: string }[] {
  const lines = input.split('\n');
  const [header, ...rows] = lines;
  const headerColumns = header.split(',');
  return rows.map((rowStr) => {
    const row: { [columnName: string]: string } = {};
    rowStr.split(',').forEach((cell, i) => {
      row[headerColumns[i]] = cell;
    });
    return row;
  });
}
```

실제로 사용할 값에 대해서 알 수 없을 때 사용하는 것이 `인덱스 시그니처`의 본질이므로 내가 사용했던 방식은 잘못된 방식이었음을 깨달을 수 있었다.

`인덱스 시그니처`는 string타입으로 광범위하므로 대체할 수 있는 방법들이 존재한다.

```typescript
type Vec3D = Record<"x" | "y" | "z", number>
// type Record<K extends keyof any, T> = {
//     [P in K]: T;
// };

```

`Record` utilType은 key의 타입을 유연하게 해주면서도 범위를 정해 줄 수 있다. 



## 🚅 Array, Tuple, ArrayLike

배열은 **오브젝트**다. 그렇기 때문에 오브젝트를 사용하듯 문자열 key로 배열 요소에 접근할 수 있다.

```typescript
const xs = [1, 2, 3];
const x0 = xs[0];
const x1 = xs['1'];
console.log(x1); // 2
```

타입스크립트의 `Array`는 숫자 키만을 허용하고 문자열 키는 다르게 인식해 key type을 하나로 정해 준다.



``` typescript
function get<T>(array: T[], k: string): T {
  return array[k]; // Element implicitly has an 'any' type because index expression is not of type 'number'.
}
```

실제 런타임은 문자열 키로 인식하지만 타입 체크를 통해 오류를 잡을 수 있는 장점을 가질 수 있다. 책을 읽으면서 어떻게 적용하면 좋을지 고민을 해봤지만, 일단은 이러한 부분이 있구나 이해하고 넘어가서 나중에 다시 보기로 생각했다.



## 😁 Readonly

`Readonly`는 말 그대로 읽기만 가능하게 해줄 수 있는 연산자로 해당 변수를 변화시키는 것에 에러를 던지게 해 원본을 보존하는 데 도움을 준다. 이러한 점이 중요한 것은 `call by reference`를 고려해 객체 타입의 매개변수를 다뤄야 함수로 인한 side-effect를 막을 수 있기 때문이다. 객체의 경우에 원본의 값이 들어오는 것이 아니라 reference값이 전달되기 때문에 함수 내부에서 변화시킨다면 원본에 변화가 생기는 문제점이 생긴다. 

이러한 문제점을 막을 수 있는 방법이 바로 `readonly`연산자다.

``` typescript
function arraySum(arr: readonly number[]) {
  let sum = 0,
    num;
  while ((num = arr.pop()) !== undefined) { // Property 'pop' does not exist on type 'readonly number[]'.
    sum += num;
  }
  return sum;
}
```

위의 예제는 `pop`과 같이 원본 요소를 바꾸는 메소드를 호출할 수 없기 때문에 생긴 에러로 함수 내에서 매개변수 내부의 변화가 생기지 않게 막는다. `readonly`배열에 변경 가능한 배열을 할당할 수 있지만, 반대는 되지 않는 특징을 가진다. 

 

```typescript
// 변경 전
function parseTaggedText(lines: string[]): string[][] {
  const paragraphs: string[][] = [];
  const currPara: readonly string[] = [];

  const addParagraph = () => {
    if (currPara.length) {
      paragraphs.push(currPara); //readonly string[]' is 'readonly' and cannot be assigned to the mutable type 'string[]'
      currPara.length = 0;
    }
  };

  for (const line of lines) {
    if (!line) {
      addParagraph();
    } else {
      currPara.push(line);
    }
  }

  addParagraph();
  return paragraphs;
}

```

앞서 말했던 대로 `readonly`배열은 변경 가능한 배열에 할당할 수 없어 에러가 발생한 것을 볼 수 있다. 이렇게 원본을 훼손하지 않고 사용하기 위해서는 앞서 리액트를 공부하며 정리한 `불변성`을 지켜주면 된다.



```typescript
function parseTaggedText(lines: string[]): string[][] {
  const paragraphs: string[][] = [];
  let currPara: readonly string[] = [];

  const addParagraph = () => {
    if (currPara.length) {
      paragraphs.push([...currPara]); // 얕은 복사로 새로운 배열을 만들어
      currPara = [];
    }
  };

  for (const line of lines) {
    if (!line) {
      addParagraph();
    } else {
      currPara = currPara.concat([line]);
    }
  }

  addParagraph();
  return paragraphs;
}
```



### Readonly util 타입

`Readonly` util 타입은 객체의 속성을 readonly로 만들어준다.

```typescript
const o: Readonly<Outer> = { inner: { x: 0 } };
o.inner = { x: 1 }; // Cannot assign to 'inner' because it is a read-only property.

// type Readonly<T> = {
//   readonly [P in keyof T]: T[P];
// };
```



## 😏 Mapping된 타입을 이용해 값 동기화하기

```typescript
interface ScatterProps {
  xs: number[];
  ys: number[];
  xRange: [number, number];
  yRange: [number, number];
  color: string;
  onClick: (x: number, y: number, index: number) => void;
}

function shouldUpdate(
  oldProps: ScatterProps,
  newProps:ScatterProps
) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k]) {
      if (k !== "onClick") return true;
    }
  }
  return false;
}
```

위 예제에서 `ScatterProps` interface를 이용해서 현재 Props와 newProps를 비교해 props가 변경될 경우 지도를 다시 그리는 예제를 담고 있다. 실제 리액트에서 컴포넌트가 re-rendering되는 조건 중 하나가 props나 상태가 바뀌는 경우로 기존 props와의 차이를 `얕은 비교`를 통해 비교한다.

현재 정의된 shouldUpdate에서 `onClick`함수 변화는 업데이트에 영향을 주지 않게 설정되어 있다. 하지만 만약 새로운 속성이 추가되는 경우에 항상 true이기 때문에 너무 자주 새로 그려지게 된다. 이점을 막기 위해서 다음과 같은 코드로 수정할 수 있다.

```typescript
function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  return (
    oldProps.xs !== newProps.xs ||
    oldProps.ys !== newProps.ys ||
    oldProps.xRange !== newProps.xRange ||
    oldProps.yRange !== newProps.yRange ||
    oldProps.color !== newProps.color
  );
}
```

위 코드는 일일이 속성들의 변화를 체크하는 방식으로 매번 새로운 속성이 추가될 때마다 작성해 주어야 하므로 비효율적이다. 

좀 더 이상적인 방법은 새로운 속성이 추가될 때 타입체커와 매핑된 타입,객체를 이용하는 코드다. 다음 예제를 보자.

```typescript
const REQUIRES_UPDATE: { [k in keyof ScatterProps]: boolean } = {
  xs: true,
  ys: true,
  xRange: true,
  yRange: true,
  color: true,
  onClick: false,
};

function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k] && REQUIRES_UPDATE[k]) {
      return true;
    }
  }
  return false;
}
```

keyof를 이용해 Update가 필요한 속성들에 대해 명시해두고, true로 설정해놓은 속성이 변화했을 때만 업데이트될 수 있게 정의했다. 타입과 값이 동기화 되어 있기 때문에 새로운 속성이 추가되어도 바로 에러가 발생해 알려줄 수 있다.

 
