---
title: JavaScript数组方法
date: 2021/11/11
updated: 2021/11/11
tags: JavaScript
categories: JavaScript
top_img: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/蝴蝶忍4.jpg
cover: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/蝴蝶忍4.jpg
---

# JavaScript数组方法

## 检测方法

### Array.isArray()

Array.isArray(obj)

定义：判断传入的值是否是一个数组

参数：

​	obj：需要判断的值

返回值：如果 obj 是Array，则为 true ，否则为 false

```javascript
console.log(Array.isArray([1,2,3])); // true
console.log(Array.isArray({foo: 123})); // false
console.log(Array.isArray('123')); // false
console.log(Array.isArray(undefined)); // false
```

## 创建数组方法

### Array.from()

Array.from( arrayLike, mapFn, thisArg)

定义：用于将**类数组对象**和**可迭代对象**转为真正的数组（不改变原对象，返回新的数组）

参数：

​	arrayLike（必选）：需要转成数组的对象

​	mapFn（可选）：类似于数组的map方法，对每个元素进行处理，将处理后的值放入返回的数组

​	thisArg（可选）：用于绑定this

返回值：新的数组实例

```javascript
// 从String生成数组
const string = Array.from("foo"); // [ "f", "o", "o" ]

// 从set生成数组，使用 Array.from 转换Set为Array
const set = new Set(["foo", "bar", "baz", "foo"]);
console.log(Array.from(set)); // [ 'foo', 'bar', 'baz' ]

// 从map生成数组,使用Array.from()可以将一个map对象转换成一个二维键值对数组
const map = new Map([
	[1, "a"],
	[2, "b"],
]);
console.log(Array.from(map)); // [ [ 1, 'a' ], [ 2, 'b' ] ]

// 使用mapFn，让你可以在最后生成的数组上再执行一次 map 方法后再返回。
// 也就是说 Array.from(obj, mapFn, thisArg) 就相当于 Array.from(obj).map(mapFn, thisArg)
const nums = Array.from([1, 2, 3], (e) => e * 2);
console.log(nums); // [ 2, 4, 6 ]
```

注意： `Array.from(null)`或者`Array.from(undefined)`会抛出异常

### Array.of()

Array.of( element0[, element1[, ...[, elementN]]])

定义：返回一个由所有参数值组成的数组，如果没有参数，就返回一个空数组

参数：

​	elementN（可选）：任意个参数，将按顺序成为返回数组中的元素

返回值：新的Array实例

```
Array.of() // []
Array.of(undefined) // [ undefined ]
Array.of(1) // [ 1 ]
```

## 遍历（迭代）方法

### Array.forEach()

Array.forEach( callback(element, index, array), thisArg)

定义：对数组的每个元素执行一个回调函数

参数：

​	callback（必选）：数组中每个元素执行的回调函数，接受三个参数

​		element（必选）：数组当前元素

​		index（可选）：当前元素的索引

​		array（可选）：数组本身

​	thisArg（可选）：当执行回调函数 `callback` 时，用作 `this` 的值

返回值：undefined

关于forEach()：

- 无法中途退出循环，只能用`return`退出本次回调，进行下一次回调。
- 它总是返回一个undefined值，即使你return了一个值，并且不可链式调用
- `forEach` 不会直接改变调用它的对象，但是那个对象可能会被 `callback` 函数改变

### Array.map()

Array.map( callback(element, index, array), thisArg)

定义：创建一个新数组，其结果是该数组中的每个元素是调用回调函数后的返回值

参数：

​	callback（必选）：数组中每个元素执行的回调函数，接受三个参数

​		element（必选）：数组当前元素

​		index（可选）：当前元素的索引

​		array（可选）：数组本身

​	thisArg（可选）：当执行回调函数 `callback` 时，用作 `this` 的值

返回值：一个由原数组每个元素执行回调函数的结果组成的新数组

关于map()：

- `map`会产生一个新数组，当你不打算使用返回的新数组却使用`map`是违背设计初衷的，请用`forEach`或者`for-of`替代
- `map `不修改调用它的原数组本身（当然可以在 `callback` 执行时改变原数组）

应用场景：

- 格式化数组

  ```javascript
  let a = ['1','2','3','4'];
  let result = a.map(function (value, index, array) {
    return value + '新数组的新元素'
  });
  console.log(result, a); 
  // ["1新数组的新元素","2新数组的新元素","3新数组的新元素","4新数组的新元素
  // ["1","2","3","4"]
  ```

  

### Array.filter()

Array.filter( callback(element, index, array), thisArg)

定义：创建一个新数组， 其包含通过所提供函数实现的测试的所有元素。

参数：

​	callback（必选）：数组中每个元素执行的回调函数，接受三个参数

​		element（必选）：数组当前元素

​		index（可选）：当前元素的索引

​		array（可选）：数组本身

​	thisArg（可选）：当执行回调函数 `callback` 时，用作 `this` 的值

返回值：一个新的、由通过测试的元素组成的数组，如果没有任何数组元素通过测试，则返回空数组。

关于filter()：

- `filter `不修改调用它的原数组本身（当然可以在 `callback` 执行时改变原数组）

- 与`map()`的区别，在回调函数里，`filter`返回的是通过表达式的元素，而`map`返回的是表达式的布尔值

  ```javascript
  // 回调函数
  function callback(element) {
    return element > 20
  }
  // map 会返回相应的布尔值，比如false/true
  // filter 会返回通过表达式的元素
  ```

### Array.some()

Array.some( callback(element, index, array), thisArg)

定义：测试数组中是不是至少有1个元素通过了回调函数的测试

参数：

​	callback（必选）：数组中每个元素执行的回调函数，接受三个参数

​		element（必选）：数组当前元素

​		index（可选）：当前元素的索引

​		array（可选）：数组本身

​	thisArg（可选）：当执行回调函数 `callback` 时，用作 `this` 的值

返回值：数组中有**至少一个元素**通过回调函数的测试就会返回**`true`**；**所有元素**都没有通过回调函数的测试返回值才会为**`false`**

```javascript
[2, 5, 8, 1, 4].some(x => x > 10);  // false
[12, 5, 8, 1, 4].some(x => x > 10); // true
```

### Array.every()

Array.every( callback(element, index, array), thisArg)

定义：测试数组中是不是所有元素通过了回调函数的测试

参数：

​	callback（必选）：数组中每个元素执行的回调函数，接受三个参数

​		element（必选）：数组当前元素

​		index（可选）：当前元素的索引

​		array（可选）：数组本身

​	thisArg（可选）：当执行回调函数 `callback` 时，用作 `this` 的值

返回值：数组中**所有元素**通过回调函数的测试就会返回**`true`**，**至少一个元素**没有通过回调函数的测试就返回**`false`**

```javascript
[12, 5, 8, 130, 44].every(x => x >= 10); // false
[12, 54, 18, 130, 44].every(x => x >= 10); // true
```

### Array.find()

Array.find( callback(element, index, array), thisArg)

定义：用于找出第一个符合条件的数组成员，并返回该成员，如果没有符合条件的成员，则返回undefined

参数：

​	callback（必选）：数组中每个元素执行的回调函数，接受三个参数

​		element（必选）：数组当前元素

​		index（可选）：当前元素的索引

​		array（可选）：数组本身

​	thisArg（可选）：当执行回调函数 `callback` 时，用作 `this` 的值

返回值：数组中第一个满足所提供测试函数的元素的值，否则返回`undefined`

关于find()：

- 注意 `callback `函数会为数组中的每个索引调用即从 `0 `到 `length - 1`，而不仅仅是那些被赋值的索引，这意味着对于稀疏数组来说，该方法的效率要低于那些只遍历有值的索引的方法

```javascript
[2, 5, 8, 1, 4].find(x => x > 7); // 8
```

### Array.findIndex()

Array.findIndex( callback(element, index, array), thisArg)

定义：返回第一个符合条件的数组成员的索引，如果所有成员都不符合条件，则返回-1。

参数：

​	callback（必选）：数组中每个元素执行的回调函数，接受三个参数

​		element（必选）：数组当前元素

​		index（可选）：当前元素的索引

​		array（可选）：数组本身

​	thisArg（可选）：当执行回调函数 `callback` 时，用作 `this` 的值

返回值：数组中通过回调函数的第一个元素的**索引**，否则返回-1

```javascript
[2, 5, 8, 1, 4].findIndex(x => x > 7); // 2
```

### keys()&values()&entries() 遍历键名、遍历键值、遍历键名+键值

定义：遍历数组，返回一个遍历器**Array Iterator**对象，区别是`keys()`是对键名的遍历、`values()是对键值的遍历`，`entries()`是键值对的遍历

参数：无

返回值：一个新的 [`Array`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array) 迭代器对象

### 未完待续

