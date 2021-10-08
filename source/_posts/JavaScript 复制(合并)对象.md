---
title: JavaScript 复制(合并)对象
date: 2021/10/9
updated: 2021/10/9
tags: JavaScript
categories: JavaScript
top_img: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/蝴蝶忍3.jpg
cover: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/蝴蝶忍3.jpg
---

# JavaScript 合并对象

简单介绍目前了解到的两种JavaScript 的合并对象的方法。

合并对象，简而言之，就是把多个对象的属性合并在一起，如果目标对象中的属性具有相同的键，则属性将被源对象中的属性覆盖。后面的源对象的属性将类似地覆盖前面的源对象的属性。

1. ## 利用`...`展开运算符

   ```javascript
   let o1 = { a: 1 };
   let o2 = { a: 2, b: 2 };
   let o3 = { c: 3 };
   
   let obj = { ...o1, ...o2, ...o3};
   
   console.log(obj); // { a: 2, b: 2, c: 3 }
   ```

   

2. ## 利用`Object.assign()`

   ```javascript
   let o1 = { a: 1 };
   let o2 = { a: 2, b: 2 };
   let o3 = { c: 3 };
   
   // 因为第一个参数会被覆盖，所以使用空对象避免
   let obj = Object.assign({}, o1, o2, o3);
   console.log(obj); // { a: 2, b: 2, c: 3 }
   ```
   
   

