---
title: 关于 ES5 的六种继承
date: 2023/1/2
updated: 2023/1/2
tags: JavaScript
categories: JavaScript
top_img: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/taylor4.jpg
cover: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/taylor4.jpg
description: ' '
---
# 关于 ES5 的六种继承

## TL, DR

六种继承方式的总结

![ES5 的五种继承](https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/inherit.png)

## 原型链继承

```javascript
function Parent() {}
Parent.prototype.age = 18
Parent.prototype.getName = function () {
  return this.name
}

function Child(name) {
  this.name = name
}
// 实质：Child.prototype.__proto__ = Parent.prototype
// 原型链继承行为
Child.prototype = new Parent()

// 实质：child.__proto__ = Child.prototype
const child = new Child("leo")
console.log(child.age) // 18
console.log(child.getName()) // "leo"
```

### 分析：

​  所有的实例的原型链属性 `[[Prototype]]` 都指向同一个原型对象（`Parent.prototype`），而且这是一个**引用类型**。从而导致数据会被共享

## 构造函数继承

```javascript
function Parent(name) {
  this.name = name
  this.food = ["水果"]
}
Parent.prototype.getName = function () {
  return this.name
}
function Child(name) {
  // 构造函数继承行为
  Parent.call(this, name)
}

const child1 = new Child("leo")
child1.food.push("apple")
const child2 = new Child("bruce")

console.log(child1.name) // "leo"
console.log(child1.food) // ['水果', 'apple']
console.log(child2.food) // ['水果']
console.log(child1.getName()) // TypeError: child1.getName is not a function

```

### 分析：

​  使用构造函数继承，可以避免共享原型对象的情况，但是却不能继承父类的方法了。因为父类方法是挂载到原型对象上的，调用构造函数并不会发生原型链指向改动

## 组合继承（常用）

```javascript
function Parent(name) {
  this.name = name
  this.colors = ["res", "blue", "green"]
}
Parent.prototype.getName = function () {
  console.log(this.name)
}

// 组合继承行为
function Child(name) {
  Parent.call(this, name)
}
// 将 Child.prototype.__proto__ 指向 Parent.prototype，但是多执行了一次 Parent 的构造函数
Child.prototype = new Parent()
// 修正 constructor 以保证 child.constructor === Child
Child.prototype.constructor = Child

const child1 = new Child("foo")
child1.colors.push("black")
child1.getName() // foo
console.log(child1.name) // foo
console.log(child1.colors) // ["red", "blue", "green", "black"]

const child2 = new Child("bar")
child2.getName() // bar
console.log(child2.name) // bar
console.log(child2.colors) // ["red", "blue", "green"]
console.log(child2.__proto__.colors) // ["red", "blue", "green"]，__proto__ 和 实例上有重复的属性
```

### 分析：

​  解决了共享原型对象、不能继承父类方法的问题，但是由于 `call()` 和 `new Parent()` 调用了两次父构造函数，导致 `__proto__` 和 实例上有重复的属性

## 原型式继承

```javascript
function Parent(name) {
  this.name = name
  this.colors = ["res", "blue", "green"]
}
let Child = new Parent()

// 原型式继承行为
let child1 = Object.create(Child)
let child2 = Object.create(Child)
child1.colors.push("花椒")
console.log(child1.colors) //    ["水果", "鸡", "烤肉", "花椒"]
console.log(child2.colors) //    ["水果", "鸡", "烤肉", "花椒"]，共享同一个对象
```

### 分析：

​  原型式继承主要是通过 `Object.create()` 来将实例 `child1.__proto__` 指向 `Child` (即 `child1.__proto__ === Child`)

​  但这也会面临所有实例的原型链共享同一个对象（`Child`）的问题

## 寄生式继承

> ​  与原型式继承比较接近的一种继承方式是寄生式继承。寄生式继承背后的思路类似于寄生构造函数和工厂模式：创建一个实现继承的函数，以某种方式增强对象，然后返回这个对象。
>
> ——《JavaScript 高级程序设计》

```javascript
// 寄生继承
function parasiticInheritance(object) {
  const clone = Object.create(object) // 通过 Object.create 创建一个新对象
  clone.sayHi = function() { // 以某种方式增强对象
    console.log('hi')
  }
  return clone // 返回这个对象
}
```

### 分析：

​  寄生继承核心实现是完成继承 + 给实例添加方法

## 寄生式组合继承（常用）

> 组合继承其实也存在效率问题，最主要的效率问题就是父类构造函数始终会被调用两次：一次是在子类构造函数中调用，一次是创建子类原型时调用。本质上，子类原型最终是要包含超类对象的所有实例属性，子类构造函数只要在执行时重写自己的原型就行了。
>
> ——《JavaScript 高级程序设计》

```javascript
function inherit(child, parent) {
  let prototype = Object.create(parent.prototype)  // 创建对象
  prototype.constructor = child // 增强对象
  child.prototype = prototype // 赋值对象
}
```

这个 inherit 函数实现了寄生式组合继承的核心逻辑

`inherit()` 接收两个参数，子类构造函数和父类构造函数，在这个函数内部，第一步是创建父类原型的一个副本。然后，给返回的 `prototype` 对象设置 `constructor` 属性，解决由于重写原型导致默认 constructor 丢失的问题。最后将新创建的对象赋值给子类型的原型。

```javascript
function Parent(name) {
  this.name = name
  this.likeFood = ["水果", "鸡", "烤肉"]
}

// 寄生式组合继承行为
function Child(name) {
  Parent.call(this, name)
}
inherit(Child, Parent)
```

这里只调用了一次 Parent 构造函数，避免了在 `Child.prototype` 上绑定不必要的属性（像组合继承那样），而且原型链依然保持不变，因此 `instanceof` 和 `isPrototypeOf()` 正常有效，寄生式组合继承可以算是引用类型继承的最佳模式



引用文章

- [图解JS中的六种继承：原型链、盗用构造函数、组合继承、原型式继承、寄生继承、组合寄生继承](https://juejin.cn/post/7028960476025323551)
- JavaScript 高级程序设计 第四版

