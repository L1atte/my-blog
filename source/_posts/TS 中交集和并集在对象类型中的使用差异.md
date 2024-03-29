---
title: TS 中交集和并集在对象类型中的使用差异
date: 2023/1/1
updated: 2023/1/1
tags: TypeScript
categories: TypeScript
top_img: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/taylor4.jpg
cover: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/taylor4.jpg
description: ' '
---
# TS 中交集和并集在对象类型中的使用差异

## TL, DR

- 对象类型中使用 `|` ，效果等同于取交集
- 对象类型中使用 `&` ，效果等同于取并集

## 对象类型中使用联合类型 `|`

使用联合类型时，官网有以下解释 [Working with Union Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#working-with-union-types)

> It might be confusing that a *union* of types appears to have the *intersection* of those types’ properties. This is not an accident - the name *union* comes from type theory.

```typescript
type Foo = {
  name: string
  age: string
}
type Bar = {
  name: string
  age: string
  gender: number
}

type result = keyof (Foo | Bar) // "name" | "age"
```

## 对象类型中使用交集类型 `&`

使用交集类型时，可以看 [Intersection Types](https://www.typescriptlang.org/docs/handbook/2/objects.html#intersection-types) 的 demo

```typescript
interface Colorful {
  color: string;
}
interface Circle {
  radius: number;
}

type ColorfulCircle = keyof (Colorful & Circle) // "color" | "radius"
```

