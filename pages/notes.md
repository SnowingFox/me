---
title: Notes - Snowingfox
display: Notes
subtitle: Quick notes / tips
description: Quick notes / tips
---

## TypeChallenges

### [Permutation](https://github.com/type-challenges/type-challenges/blob/main/questions/00296-medium-permutation/README.md)

Reference: [distributive-conditional-types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#distributive-conditional-types)

Issue: [never is the empty union so we map over nothing, returning never](https://github.com/microsoft/TypeScript/issues/27418)

```ts
type Permutation<T, U = T> = [T] extends [never] ? [] : T extends U ? [T, ...Permutation<Exclude<U, T>>] : nerver
```

why use `[T] extends [nerver]` not `T extends nerver`? Because in union type
```ts
type UnionType = 'A' | 'B' | 'C' extends U ? X : Y
// will be spread to (A extends U ? X : Y) | (B extends U ? X : Y) | (C extends U ? X : Y), you can see this example in reference
```


### [IsUnion](https://github.com/type-challenges/type-challenges/blob/main/questions/01097-medium-isunion/README.md)

Reference: [掘金](https://juejin.cn/post/7116327095118069773#heading-4)

```ts
type IsUnion<T, U = T> = [T] extends [never] ? false : 
  T extends U ? [U] extends [T] ? false : true : false
```

### [MinusOne](https://github.com/type-challenges/type-challenges/blob/main/questions/02257-medium-minusone/README.md)

Reference: [掘金(神仙解法，菜鸡只能感叹)](https://juejin.cn/post/7118913605835194398#heading-1)

```ts

type MinusOne<T extends number> = CountTo<`${T}`> extends [...infer T, 1] ? T['length'] : 0

type CountTo<T extends string, Count extends 1[] = []> =
  T extends `${infer First}${infer Rest}` ? (
    CountToT<Rest, N<Count>[keyof N & First]> // `keyof N & First` can ensure `First` is index signature of N
  ) : Count

type N<T extends 1[] = [], Rest extends 1[] = [...T, ...T, ...T, ...T, ...T, ...T, ...T, ...T, ...T, ...T]> = {
  '0': [...Rest],
  '1': [...Rest, 1],
  '2': [...Rest, 1, 1],
  '3': [...Rest, 1, 1, 1],
  '4': [...Rest, 1, 1, 1, 1],
  '5': [...Rest, 1, 1, 1, 1, 1],
  '6': [...Rest, 1, 1, 1, 1, 1, 1],
  '7': [...Rest, 1, 1, 1, 1, 1, 1, 1],
  '8': [...Rest, 1, 1, 1, 1, 1, 1, 1, 1],
  '9': [...Rest, 1, 1, 1, 1, 1, 1, 1, 1, 1],
}
```
