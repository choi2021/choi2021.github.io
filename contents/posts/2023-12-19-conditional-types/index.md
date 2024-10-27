---
title: "Typescript: Conditional Types"
slug: typescript-conditional-types
date: 2023-12-19
tags: [typescript]
series: "Typescript"
---

## Conditional Types 🤩

타입스크립트에서 조건부 타입은 `T extends U ? X : Y`와 같은 형태로 사용된다. 이는 `T`가 `U`에 할당 가능한지에 따라서 `X`와 `Y`중 하나의 타입을 선택하게 된다.

이러한 모습은 기존 javascript의 삼항연산자와 유사하게 느껴진다. `(condition ? trueExpression : falseExpression)`

```typescript
interface Animal {
  live(): void
}
interface Dog extends Animal {
  woof(): void
}

type Example1 = Dog extends Animal ? number : string // type Example1 = number

type Example2 = RegExp extends Animal ? number : string // type Example2 = string
```

### Generic과 함께 사용하기

Generic과 함께 사용하면 더 유용하게 사용할 수 있다. 다음 예제를 보자.

```typescript
// 예시 1
type MessageOf<T> = T extends { message: unknown } ? T["message"] : never

interface Email {
  message: string
}

interface Dog {
  bark(): void
}

type EmailMessageContents = MessageOf<Email> // string

type DogMessageContents = MessageOf<Dog> // never

// 예시 2
type Flatten<T> = T extends any[] ? T[number] : T

// Extracts out the element type.
type Str = Flatten<string[]> // type Str = string

// Leaves the type alone.
type Num = Flatten<number> // type Num = number
```

### Infer와 함께 사용하기

`infer`는 타입스크립트에서 타입을 추론하는 키워드이다. `infer`를 사용하면 조건부 타입을 사용해 동적으로 타입을 추론할 때 유용하게 사용할 수 있다.

```typescript
// 예시1
type Flatten<Type> = Type extends Array<infer Item> ? Item : Type

//예시2
type GetReturnType<Type> = Type extends (...args: never[]) => infer Return
  ? Return
  : never

type Num = GetReturnType<() => number> // number

type Str = GetReturnType<(x: string) => string> // string

type Bools = GetReturnType<(a: boolean, b: boolean) => boolean[]> // boolean[]
```

예시 2를 보면 함수의 반환타입을 동적으로 변경할 수 있게 infer를 이용해 추론한 예제다.

### Distributive Conditional Types

Distributive Conditional Types는 조건부 타입이 사용될 때 전달되는 타입이 `union`으로 구성되어 있으면 각각의 타입에 조건부 타입을 적용한 후 `union`으로 다시 합쳐진다.

```typescript
type ToArray<Type> = Type extends any ? Type[] : never

type StrArrOrNumArr = ToArray<string | number> // string[] | number[]
```
