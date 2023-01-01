---
title: 深入理解 TypeScript 常用操作符和高级概念
date: 2023/1/1
updated: 2023/1/1
tags: TypeScript
categories: TypeScript
---
# 深入理解 TypeScript 常用操作符和高级概念

## TS operator

### `keyof` 运算符

`keyof` 关键字表示它接受一个对象类型作为参数，并且返回该对象所有 `key` 值组成的联合类型

```typescript
interface Bar {
	name: string;
	count: number
}
type Foo = keyof Bar; // Foo = 'name' | 'count'
```

### extends 泛型约束

通过 `extends` 关键字可以约束泛型具有某些属性

```typescript
interface Lengthwise {
	length: number
}

// 表示传入的泛型 T 接受 Lengthwise 的约束
// T 必须实现 Lengthwise，换句话说 Lengthwise 这个类型是可以完全赋值给 T
function loggingIdentity<T extends Lengthwise>(arg: T): T {
	console.log(arg.length) // ok
	return arg
}
```

### `infer` 带推断类型

当我们类型定义是时不能立即确定某些类型，而是在使用类型时来根据条件推断对应的类型

需要注意的是，`infer` 关键字类型，必须结合 Conditional Types （extends）判断来使用

### `+` or `-` mapping modifiers

用于修饰 只读修饰符 `readonly` 和 可选修饰符 `?`

`+`: 表示添加修饰符

`-`: 表示移除修饰符

```typescript
// Removes 'readonly' attributes from a type's properties
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};
 
type LockedAccount = {
  readonly id: string;
  readonly name: string;
};
 
type UnlockedAccount = CreateMutable<LockedAccount>;
/** 
*	type UnlockedAccount = {
*   id: string;
*   name: string;
* }
*/
```

## TS 高级概念

### 分发（distributive conditional types）

分发指的是：使用联合类型做条件类型（conditional type）判断的时候，联合类型中的每一个成员都会参与条件类型判断

大多数联合类型的工具类型都是利用分发特性做的，举例：

```typescript
// Exclude<T, k>
type MyExclude<T, K extends T> = T extends K ? never : T

// test
type Bar = MyExclude<'a' | 'b' | 'c' | 'd', 'b'> 
// type Bar = "a" | "c" | "d"
```

分发的条件：

1. 必须是在 extends 的条件类型判断中才有分发
2. 只有联合类型才有分发



### 遍历

使用 `in` 操作符可以进行遍历操作，但遍历的对象必须是联合类型

```typescript
type Bar = ['a', 'b', 'c', 'd']
type Foo = 'a' | 'b' | 'c' | 'd'

type A = { [key in Foo]: string } // OK
type B = { [key in Bar]: string } // Error
```

遍历在构造对象类型的时候十分有效，比如

```typescript
// Partial<T>
type MyPartial = {
	[key in keyof T]?: T[p]
}
```



### 逆变（covariant）

如果有 A << B，而 F(A) >> F(B)，那么称 F 是逆变的（父子关系相反）



### 协变（contravariant）

如果有 A << B，而 F(A) << F(B)，那么称 F 是协变的（父子关系不变）



### 双向协变（bivariant）

既是协变又是逆变



### 不变（invariant）

不存在分配关系，或者说无法互相分配



### 函数参数类型是逆变的

在函数类型赋值兼容时，函数参数少的向下兼容函数参数多的



### 函数返回值类型是协变的

在函数类型赋值兼容时，返回值多的向下兼容返回值少的



### TypeScript 默认认为函数方法的入参是双向协变的