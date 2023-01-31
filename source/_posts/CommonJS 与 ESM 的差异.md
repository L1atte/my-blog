---
title: CommonJS 与 ES6 Module 的差异
date: 2022/10/11
updated: 2021/10/11
tags: JavaScript
categories: JavaScript
top_img: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/蝴蝶忍4.jpg
cover: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/蝴蝶忍4.jpg
description: ' '
---
# CommonJS 与 ES6 Module 的差异

## 历史背景

* CommonJS 来自于社区开发者的设计的规范，CJS 模块是 Node.js 专用的，与 ES6 模块不兼容。但是 Node.js 在 v13.2 版本后打开了 ES6 模块支持（需要采用 .mjs 后缀）
* ES6 Module 来自ES6的新规范，即TC39 制定的新的 ECMAScript 版本。

## 差异

### 语法差异

#### CommonJS

```javascript
/* 导出 */
exports.fs = fs;      // 单个特性导出，可导出多个
module.exports = fs;   // 整个模块导出，每个模块只包含一个

/* 引入 */
const fs = require('fs');   // 引入整个模块
```

#### ES6 Module

```javascript
/* 导出 */
export default fs;     // 默认导出 每个模块包含一个 每次导出都会覆盖前一个导出
export const fs;       // 导出单个特性 每个模块包括多个
export function readFile;    // 导出单个特性 每个模块包括多个
export { readFile, read };   // 导出列表
export * from 'fs';     // 导出模块合集

/* 引入 */
import fs from 'fs';       // 引入整个模块的内容
import '/fs.js';          // 仅为副作用而引入一个模块 不导入模块中的任何接口
import * as fs from 'fs';    // 引入整个模块的内容
import { readFile } from 'fs';   // 引入readFile单个接口
import { readFile as read } from 'fs';   // 引入模块中read接口，并重命名为readFile
import fs , { readFile } from 'fs';   // 引入整个模块的内容和readFile接口
```

### 输出差异

#### CommonJS 模块输出一个值的拷贝

也就是说，一旦输出一个值，模块内部的变化就影响不带这个值，请看下面的例子

```javascript
// test.js
let num = 0
function addNum() {
  num++
}
module.exports = {
  num: num,
  addNum: addNum
}

// main.js
const test = require('./test.js')

console.log(test.num) // output: 0
test.addNum()
console.log(test.num) // output: 0
```

#### ES6 Module 模块输出一个值的引用

也就是说，该引用其实是一个动态引用，当模块内部发生变化，引入值也会随之更新

```javascript
// test.js
export let num = 0
export function addNum() {
  num++
}

// main.js
import { num, addNum } from './test.js'

console.log(num) // output: 0
addNum()
console.log(num) // output: 1
```

### 运行差异

#### CommonJS 模块是运行时加载

因为 CommonJS 模块加载的是一个对象，即 module.exports 属性，它的模块解析发生在 **执行阶段**，也因此被称作 **运行时加载**

#### ES6 Module 模块是编译时加载

`ES6`模块不是对象，它对外接口只是一个静态定义，在代码静态解析阶段才会生成，因此效率要比 `CommonJS`模块加载高。ES6 Modul

### 异步差异

CommonJS 模块的 require() 是 **同步加载** 模块

ES6 Module 模块的 import 命令是 **异步加载** 模块（等同于打开了 `<script>` 的 defer 属性），有一个独立的模块依赖分析阶段
