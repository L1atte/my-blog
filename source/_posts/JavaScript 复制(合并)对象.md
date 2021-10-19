---
title: JavaScript 合并数据
date: 2021/10/9
updated: 2021/10/19
tags: JavaScript
categories: JavaScript
top_img: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/蝴蝶忍3.jpg
cover: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/蝴蝶忍3.jpg

---

# JavaScript 合并数据

## 合并对象

合并对象，简而言之，就是把多个对象的属性合并在一起，如果目标对象中的属性具有相同的键，则属性将被源对象中的属性覆盖。后面的源对象的属性将类似地覆盖前面的源对象的属性。

1. ### 利用`...`展开运算符

   ```javascript
   let o1 = { a: 1 };
   let o2 = { a: 2, b: 2 };
   let o3 = { c: 3 };
   
   let obj = { ...o1, ...o2, ...o3};
   
   console.log(obj); // { a: 2, b: 2, c: 3 }
   ```

   

2. ### 利用`Object.assign()`

   ```javascript
   let o1 = { a: 1 };
   let o2 = { a: 2, b: 2 };
   let o3 = { c: 3 };
   
   // 因为第一个参数会被覆盖，所以使用空对象避免
   let obj = Object.assign({}, o1, o2, o3);
   console.log(obj); // { a: 2, b: 2, c: 3 }
   ```

   

## 合并数组

合并数组之前，一定要注意对数组进行去重。因为合并数组的 concat 方法是不去重的。

对于数组的去重，有很多种方法，但个人最喜欢的还是使用 es6 的 Set，利用 Set 中的元素是唯一的特性进行去重。

```javascript
// 数组去重
return Array.from(new Set(array))
// 进一步简化
return [...new Set(array)]
```

1. ### 利用`...`展开运算符

   ```javascript
   const a = [1, 2, 3];
   const b = [1, 5, 6];
   
   const c = [...new Set([...a, ...b])];
   console.log(c); // [ 1, 2, 3, 5, 6 ]
   ```

   

2. ### 利用`Array.concat()`

   ```javascript
   const a = [1, 2, 3];
   const b = [1, 5, 6];
   
   const c = [...new Set(a.concat(b))];
   console.log(c); // [ 1, 2, 3, 5, 6 ]
   ```

   

## 总结

​	在 JavaScript 中合并数据的时候，个人认为最优雅的方法是使用展开运算符，因为无论是合并对象还是合并数组的情况，都能使用展开运算符来解决。