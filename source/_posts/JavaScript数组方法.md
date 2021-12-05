---
title: JavaScript数组方法
date: 2021/11/11
updated: 2021/11/12
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
console.log(array.isArray([1,2,3])); // true
console.log(array.isArray({foo: 123})); // false
console.log(array.isArray('123')); // false
console.log(array.isArray(undefined)); // false
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
const string = array.from("foo"); // [ "f", "o", "o" ]

// 从set生成数组，使用 array.from 转换Set为Array
const set = new Set(["foo", "bar", "baz", "foo"]);
console.log(array.from(set)); // [ 'foo', 'bar', 'baz' ]

// 从map生成数组,使用array.from()可以将一个map对象转换成一个二维键值对数组
const map = new Map([
	[1, "a"],
	[2, "b"],
]);
console.log(array.from(map)); // [ [ 1, 'a' ], [ 2, 'b' ] ]

// 使用mapFn，让你可以在最后生成的数组上再执行一次 map 方法后再返回。
// 也就是说 array.from(obj, mapFn, thisArg) 就相当于 array.from(obj).map(mapFn, thisArg)
const nums = array.from([1, 2, 3], (e) => e * 2);
console.log(nums); // [ 2, 4, 6 ]
```

注意： `array.from(null)`或者`array.from(undefined)`会抛出异常

### Array.of()

Array.of( element0[, element1[, ...[, elementN]]])

定义：返回一个由所有参数值组成的数组，如果没有参数，就返回一个空数组

参数：

​	elementN（可选）：任意个参数，将按顺序成为返回数组中的元素

返回值：新的Array实例

```
array.of() // []
array.of(undefined) // [ undefined ]
array.of(1) // [ 1 ]
```

## 遍历（迭代）方法

### array.forEach()

array.forEach( callback(element, index, array), thisArg)

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
- `forEach`会自动跳过已删除或者未初始化的元素

### array.map()

array.map( callback(element, index, array), thisArg)

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

  

### array.filter()

array.filter( callback(element, index, array), thisArg)

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

### array.some()

array.some( callback(element, index, array), thisArg)

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

### array.every()

array.every( callback(element, index, array), thisArg)

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

### array.find()

array.find( callback(element, index, array), thisArg)

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

### array.findIndex()

array.findIndex( callback(element, index, array), thisArg)

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

## 操作方法

### push() 向数组末尾添加元素

array.push(element1, ..., elementN)

定义：将一个或者多个元素添加到数组的末尾，并且返回该数组的新长度

参数：

​	elementN： 被添加到数组末尾的元素

返回值：数组的新长度

关于push()

- `push` 方法根据 `length` 属性来决定从哪里开始插入给定的值。如果 `length` 不能被转成一个数值，则插入的元素索引为 0，包括 `length` 不存在时。当 `length` 不存在时，将会创建它。
- 对象也可以使用`push`，其创建一个键为`length`、值为`element`的键值对

```javascript
// 对象中定义push方法
let obj = { 2: "a", length: 3, push: array.prototype.push };
obj.push("c");
// { '2': 'a', '3': 'c', length: 4, push: [Function: push] }
// 创建键为3、值为c的键值对
```

### pop() 删除数组末尾的元素

array.pop()

定义：从数组末尾删除最后一个元素，并返回该元素的值。此方法改变数组的长度

参数：无

返回值：从数组中删除的元素，当数组为空时返回`undefined`

关于pop()：

- `pop`方法根据 `length`属性来确定最后一个元素的位置。如果不包含`length`属性或`length`属性不能被转成一个数值，会将`length`置为0，并返回`undefined`。

```javascript
let myFish = ["angel", "clown", "mandarin", "surgeon"];
let popped = myFish.pop();

console.log(myFish);
// ["angel", "clown", "mandarin"]

console.log(popped);
// surgeon
```

### shift() 删除数组第一个元素

array.shift()

定义：从数组中删除第一个元素，并返回该元素的值。此方法改变数组的长度

参数：无

返回值：从数组中删除的元素，当数组为空时返回`undefined`

```javascript
let myFish = ['angel', 'clown', 'mandarin', 'surgeon'];
let shifted = myFish.shift();

console.log(myFish);
// [ 'clown', 'mandarin', 'surgeon' ]

console.log(shifted);
// angel
```

### unshift() 向数组头部添加元素

array.unshift(element1, ..., elementN)

定义：将一个或者多个元素添加到数组的开头，并返回该数组的新长度

参数：

​	elementN：被添加到数组开头的元素

返回值：数组的新长度

```javascript
let arr = [4,5,6];
arr.unshift(1,2,3);
console.log(arr); // [1, 2, 3, 4, 5, 6]
```

### concat() 合并数组

array.concat(value1[, value2[, ...[, valueN]]])

定义：用于合并两个或者多个数组，此方法**不会改变现有数组**，而是返回一个**新数组**

参数：

​	valueN： 数组/值，将被合并到一个新的数组中。如果省略了 valueN 参数，则concat会返回调用此方法的数组的一个浅拷贝

返回值：新的 Array 实例

关于concat()：

- `concat`方法创建一个新的数组，它由被调用的对象中的元素组成，每个参数的顺序依次是该参数的元素（如果参数是数组）或参数本身（如果参数不是数组）。它不会递归到嵌套数组参数中。
- `concat`方法不会改变`this`或者作为参数提供的数组，而且返回一个浅拷贝。
- 如果复制元素为对象引用，`concat`将对象引用复制到新数组中。 原始数组和新数组都引用相同的对象。 也就是说，如果引用的对象被修改，则更改对于新数组和原始数组都是可见的。

```javascript
// concat返回一个浅拷贝
const array1 = [{keyword: 'a'}];
const array2 = array1.concat();
array2[0].keyword = 1
console.log(array1);
// [ { keyword: 1 } ]

// 参数可以是数组，也可以是值
const array1 = [];
const array2 = array1.concat(1);
const array3 = array1.concat([1])
// array2: [ 1 ], array3: [ 1 ]
// 结果相同
```

### indexOf()

array.indexOf(searchElement[, fromIndex])

定义：返回在数组中可以找到一个给定元素的第一个索引，如果不存在，则返回 -1

参数：

​	searchElement（必选）：要查找的元素

​	fromIndex（可选）：从此索引开始顺向查找。如果该索引值大于或等于数组长度，意味着不会在数组里查找，返回-1。如果参数中提供的索引值是一个负值，则将其作为数组末尾的一个抵消，即-1表示从最后一个元素开始查找，-2表示从倒数第二个元素开始查找 ，以此类推。注意：如果参数中提供的索引值是一个负值，并不改变其查找顺序，查找顺序仍然是从前向后查询数组。如果抵消后的索引值仍小于0，则整个数组都将会被查询。其默认值为0

返回值：被找到的元素的索引，如果没找到则返回 -1

### lastIndexOf()

array.lastIndexOf(searchElement[, fromIndex])

定义：返回指定元素在数组中的最后一个的索引。如果不存在，则返回 -1

参数：

​	searchElement：要查找的元素

​	fromIndex：从此索引开始逆向查找。默认为数组的长度减 1(`arr.length - 1`)，即整个数组都被查找。如果该值大于或等于数组的长度，则整个数组会被查找。如果为负值，将其视为从数组末尾向前的偏移。即使该值为负，数组仍然会被从后向前查找。如果该值为负时，其绝对值大于数组长度，则方法返回 -1，即数组不会被查找。

返回值：数组中该元素最后一次出现的索引，如果没找到则返回 -1

### slice() 浅拷贝数组元素

array.slice( [begin[, end]])

定义：返回一个新的数组对象，对象由 begin 和 end 决定的原数组的**浅拷贝**（包括 begin ，不包括 end ），此方法不会改变原数组

参数：

​	begin（可选）：

- 默认值为0
- 提取起始处的索引（从 `0` 开始），从该索引开始提取原数组元素。
- 如果该参数为负数，则表示从原数组中的倒数第几个元素开始提取，`slice(-2)` 表示提取原数组中的倒数第二个元素到最后一个元素（包含最后一个元素）。
- 如果 `begin` 超出原数组的索引范围，则会返回空数组。

​	end（可选）：

- 默认值为 array.length
- 提取终止处的索引（从 `0` 开始），在该索引处结束提取原数组元素。`slice` 会提取原数组中索引从 `begin` 到 `end` 的所有元素（包含 `begin`，但不包含 `end`）。
- 如果该参数为负数， 则它表示在原数组中的倒数第几个元素结束抽取。 `slice(-2,-1)` 表示抽取了原数组中的倒数第二个元素到最后一个元素（不包含最后一个元素，也就是只有倒数第二个元素）。
- 如果 `end` 被省略，则 `slice` 会一直提取到原数组末尾。
- 如果 `end` 大于数组的长度，`slice` 也会一直提取到原数组末尾。

返回值：一个含有被提取元素的新数组

```javascript
var fruits = ['Banana', 'Orange', 'Lemon', 'Apple', 'Mango'];
var citrus = fruits.slice(1, 3);
console.log(citrus);
// [ 'Orange', 'Lemon' ]
```

### splice() 删除 / 插入 / 替换元素

array.splice( start[, deleteCount[, item1[, item2[, ...itemN]]]])

定义：向数组中**删除** / **插入** / **替换**元素，返回值是被删除的元素组成的数组，如果没有删除元素，则返回空数组。此方法会改变原数组

参数：

​	start（必选）：指定修改位置的索引，使用负数可从数组结尾处规定位置。

​	deleteCount（可选）：

- 表示要移除的数组元素的个数。
- 如果为 0 或者负数，则不删除元素。

​	itemN（可选）：要添加入数组中的元素，从 start 位置开始。如果不指定，则 `splice()`将只删除元素

返回值：由被删除的元素组成的一个数组。如果只删除了一个元素，则返回只包含一个元素的数组。如果没有删除元素，则返回空数组。

eg1：删除元素，从索引3的位置开始删除1个元素

```javascript
let myFish = ['angel', 'clown', 'drum', 'mandarin', 'sturgeon'];
let removed = myFish.splice(3, 1);

// 运算后的 myFish: ["angel", "clown", "drum", "sturgeon"]
// 被删除的元素: ["mandarin"]
```

eg2：插入元素，从索引2的位置开始删除0个元素，插入“drum”

```javascript
let myFish = ["angel", "clown", "mandarin", "sturgeon"];
let removed = myFish.splice(2, 0, "drum");

// 运算后的 myFish: ["angel", "clown", "drum", "mandarin", "sturgeon"]
// 被删除的元素: [], 没有元素被删除
```

eg3：替换元素，从索引2的位置开始删除1个元素，插入”trumpet“

```javascript
let myFish = ['angel', 'clown', 'drum', 'sturgeon'];
let removed = myFish.splice(2, 1, "trumpet");

// 运算后的 myFish: ["angel", "clown", "trumpet", "sturgeon"]
// 被删除的元素: ["drum"]
```

### copyWithin() 指定位置的成员复制到其他位置

array.copyWithin( target[, start[, end]])

定义：浅复制数组的一部分到同一数组中的另一个位置，并返回它。该方法会改变原数组，但不会改变数组长度

参数：

​	target（必选）：从该索引位置开始替换位置。如果 `target` 大于等于 `arr.length`，将会不发生拷贝

​	start（可选）：从该索引位置开始复制元素，默认为 0

​	end（可选）：停止复制的索引，不包含自身，默认为 array.length

返回值：改变后的数组

```javascript
let arr = [1, 2, 3, 4, 5, 6];
arr.copyWithin(0,1,4)
// 从索引位置0开始替换，取索引位置1到索引为3(不包括4)的元素
// [ 2, 3, 4, 4, 5, 6 ]
```

### fill() 填充数组

array.fill( value[, start[, end]])

定义：用一个固定值填充一个数组中从起始索引到终止索引内的全部元素。不包括终止索引。

参数：

​	value（必选）：用来填充数组的元素

​	start（可选）：起始索引，默认值为 0

​	end（可选）：终止索引，默认值为 array.length

返回值：修改后的数组

```javascript
[1, 2, 3].fill(4);
// [4, 4, 4]
```

### flat() 扁平化数组

let newArray = array.flat([depth])

定义：将嵌套的数组，变成一维的数组。返回一个新数组。该方法不会改变原数组

参数：

​	depth（可选）：

- 指定药提取嵌套数组的结构深度，默认值为 1
- 使用 Infinity，可展开任意深度的嵌套数组（Infinity是表示正无穷大的数值）

返回值：扁平化后的数组

关于flat()：

- 使用`Infinity`作为`flat`的参数，可以不需要知道数组的维度来进行扁平化
- `flat()` 方法会移除数组中的空项

```javascript
//使用 Infinity，可展开任意深度的嵌套数组
let arr4 = [1, 2, [3, 4, [5, 6, [7, 8, [9, 10]]]]];
let afterFlat = arr4.flat(Infinity);
// [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

// flat() 方法会移除数组中的空项
let arr4 = [1, 2, , 4, 5];
let afterFlat = arr4.flat();
// [1, 2, 4, 5]
```

手动实现 flat()

```javascript
// forEach+isArray+push+recursivity
function flatDeep(arr, depth) {
	let result = [];
	(function flat(arr, depth) {
		// forEach 会自动去除数组空位
		arr.forEach((item) => {
			// 控制递归深度
			if (Array.isArray(item) && depth > 0) {
				// 递归数组
				flat(item, depth - 1);
			} else {
				// 缓存元素
				result.push(item);
			}
		});
	})(arr, depth);
	return result;
}
```

### flatMap()

flatMap( callback(element, index, array), thisArg)

定义：首先使用映射函数映射每个元素，然后将结果压缩成一个新数组。它与 [map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) 连着深度值为1的 [flat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/flat) 几乎相同，即 array.map().flat(1)，但 `flatMap` 通常在合并成一种方法的效率稍微高一些。

参数：

​	callback（必选）：数组中每个元素执行的回调函数，接受三个参数

​		element（必选）：数组当前元素

​		index（可选）：当前元素的索引

​		array（可选）：数组本身

​	thisArg（可选）：当执行回调函数 `callback` 时，用作 `this` 的值

返回值：一个新的数组，其中每个元素都是回调函数的结果，并且结构深度 `depth` 值为1。

```javascript
var arr = [1, 2, [3]];
const newArr = arr.flatMap(el => el);
console.log(newArr);
// [ 1, 2, 3 ]
```

### includes() 查找数组是否包含某个元素 返回布尔

array.includes(value[, fromIndex])

定义：返回一个布尔值，表示数组是否包含给定的值

参数：

​	value（必选）：需要查找的值

​	fromIndex（可选）：从 fromIndex 索引处开始查找

返回值：如果找到返回 true ，否则返回 false

关于 includes()

- includes() 可以识别NaN

应用场景：优化 if 中的判断条件

```javascript
if(
    type == 1 ||
    type == 2 ||
    type == 3 ||
    type == 4 ||
){
   //...
}
优化后
const condition = [1,2,3,4];
if( condition.includes(type) ){
   //...
}
```

## 排序方法

### sort() 数组排序

array.sort([compareFunction ( a, b) ])

定义：对数组元素进行排序，并返回这个数组。此方法会改变原数组

参数：

​	compareFunction（可选）：比较函数。如果省略，元素按照转换为的字符串的各个字符的Unicode位点进行排序。

​	element1, element2:比较函数的参数

返回值：排序后的数组

关于sort()：

- sort的比较函数有两个默认参数，要在函数中接收这两个参数，这两个参数是数组中两个要比较的元素，通常我们用 a 和 b 接收两个将要比较的元素
- 若比较函数返回值<0，那么a将排到b的前面;
- 若比较函数返回值=0，那么a 和 b 相对位置不变；
- 若比较函数返回值>0，那么b 排在a 将的前面

对于sort()方法更深层级的内部实现以及处理机制可以看一下这篇文章[深入了解javascript的sort方法](https://juejin.cn/post/6844903507439419399)

sort() 排序常见用法

1. 数组元素为数字的升序、降序:

   ```javascript
   var array = [10, 1, 3, 4, 20, 4, 25, 8];
   // 升序 a-b < 0   a将排到b的前面，按照a的大小来排序的
   // 比如被减数a是10，减数是20  10-20 < 0   被减数a(10)在减数b(20)前面
   array.sort(function (a, b) {
   	return a - b;
   });
   console.log(array); // [1,3,4,4,8,10,20,25];
   // 降序 被减数和减数调换了  20-10>0 被减数b(20)在减数a(10)的前面
   array.sort(function (a, b) {
   	return b - a;
   });
   console.log(array); // [25,20,10,8,4,4,3,1];
   ```

   

2. 数组多条件排序

   ```javascript
   var array = [
   	{ id: 10, age: 2 },
   	{ id: 5, age: 4 },
   	{ id: 6, age: 10 },
   	{ id: 9, age: 6 },
   	{ id: 2, age: 8 },
   	{ id: 10, age: 9 },
   ];
   array.sort(function (a, b) {
   	if (a.id === b.id) {
   		// 如果id的值相等，按照age的值降序
   		return b.age - a.age;
   	} else {
   		// 如果id的值不相等，按照id的值升序
   		return a.id - b.id;
   	}
   });
   // [{"id":2,"age":8},{"id":5,"age":4},{"id":6,"age":10},{"id":9,"age":6},{"id":10,"age":9},{"id":10,"age":2}]
   
   ```

   

3. 自定义比较函数

   ```javascript
   var array = [
   	{ name: "Latte" },
   	{ name: "Latte" },
   	{ name: "OB" },
   	{ name: "Latte" },
   	{ name: "OB" },
   	{ name: "OB" },
   ];
   array.sort(function (a, b) {
   	if (a.name === "Latte") {
   		// 如果name是'Latte' 返回-1 ，-1<0 a排在b的前面
   		return -1;
   	} else {
   		// 如果不是的话，a排在b的后面
   		return 1;
   	}
   });
   // [{"name":"Latte"},{"name":"Latte"},{"name":"Latte"},{"name":"OB"},{"name":"OB"},{"name":"OB"}]
   ```

### reverse() 颠倒数组中元素的顺序

array.reverse()

定义：用于颠倒数组中元素的顺序，此方法会改变原数组

参数：无

返回值：颠倒后的数组

## 转换方法

### join() 数组转字符串

array.join([ separator ])

定义：用于把数组中的所有元素通过指定的分隔符进行分隔放入一个字符串，返回生成的字符串。

参数：

​	separator（可选）：分隔符，默认用逗号(,)分隔

返回值：转换后的字符串

关于join()：

- **<u>如果一个元素为 `undefined` 或 `null`，它会被转换为空字符串。</u>**

```javascript
let a = ["hello", "world"];
let str = a.join(); // 'hello,world'
let str2 = a.join(""); // 'helloworld'

// undefined, null会被转换为空字符串
let arr = [undefined, null, 2]
console.log(arr.join());
// ,,2
```



### toLocaleString() 数组转字符串

array.toLocaleString([ locales[, options]]);

定义：返回一个表示数组元素的字符串。该字符串由数组中的每个元素的 `toLocaleString()` 的返回值调用 `join()` 方法连接（由逗号隔开）

参数：

​	locales（可选）：区域名称

​	options（可选）：元素调用的`toLocaleString()`的options

返回值：转换后的字符串

```javascript
let a=[{name:'OBKoro1'},23,'abcd',new Date()];
let str=a.toLocaleString(); 
// [object Object],23,abcd,2018/5/28 下午1:52:20 
```

如上述栗子：调用数组的`toLocaleString`方法，数组中的每个元素都会调用自身的`toLocaleString`方法，对象调用对象的`toLocaleString`,Date调用Date的`toLocaleString`。

### toString() 数组转字符串 不推荐

array.toString()

定义: toString() 方法可把数组转换为由逗号链接起来的字符串。

参数：无

返回值：转换后的字符串

不推荐理由：该方法的效果和join方法一样，都是用于数组转字符串的，但是与join方法相比没有优势，也不能自定义字符串的分隔符，因此不推荐使用。

值得注意的是：**当数组和字符串操作的时候，js 会调用这个方法将数组自动转换成字符串**

```
let b = ["toString", "演示"].toString(); // toString,演示
let a = ["调用toString", "连接在我后面"] + "啦啦啦"; // 调用toString,连接在我后面啦啦啦
```

## 归并方法（迭代数组的所有项，然后构建一个最终返回的值）

### reduce() 为数组提供累加器，合并为一个值

array.reduce(function(total, currentValue, currentIndex, arr), initialValue)

定义：对累加器和数组中的每个元素（从左到右）应用一个函数，最终合并为一个值。

参数：

​	function（必选）：执行函数，包括四个参数

- total（必选）：初始值, 或者上一次调用回调返回的值
- currentValue（必选）：数组当前的元素
- index（可选）：当前元素的索引
- arr：数组对象本身

​	initialValue（可选）: 指定第一次回调 的第一个参数，即total的初始值，如果没有提供初始值，则使用数组中的第一个元素

回调第一次执行时:

- 如果 initialValue 在调用 reduce 时被提供，那么第一个 total 将等于 initialValue，此时 currentValue 等于数组中的第一个值；

- 如果 initialValue 未被提供，那么 total 等于数组中的第一个值，currentValue 等于数组中的第二个值。此时如果数组为空，那么将抛出 TypeError。

- 如果数组仅有一个元素，并且没有提供 initialValue，或提供了 initialValue 但数组为空，那么回调不会被执行，数组的唯一值将被返回。

  

注意：如果没有提供`initialValue`，reduce 会从索引1的地方开始执行 callback 方法，跳过第一个索引。如果提供`initialValue`，从索引0开始。

也就是说，如果没有提供 `initiaValue`，则 `reduce()`的调用次数为 length - 1

```javascript
// 数组求和
let sum = [0, 1, 2, 3].reduce(function (a, b) {
	return a + b;
}, 0);
// 6
// 将二维数组转化为一维 将数组元素展开
let flattened = [
	[0, 1],
	[2, 3],
	[4, 5],
].reduce((a, b) => a.concat(b), []);
// [0, 1, 2, 3, 4, 5]
```



### reduceRight() 从右至左累加

这个方法除了与reduce执行方向相反外，其他完全与其一致，请参考上述 reduce 方法介绍。
