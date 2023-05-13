---
title: 理解 async 和 await
date: 2023/5/12
updated: 2023/5/13
tags: JavaScript,
categories: JavaScript
description: " "
---

# 理解 async 和 await

## async 关键字

async 函数会返回一个 promise，这个 promise 要么会被 async 函数返回的值解决，要么会通过一个从 async 函数中抛出的（或者其中 **没有被捕获到的**）异常拒绝

## await 关键字

`await`总会同步地对表达式求值并处理，处理的行为与 `Promise.resolve()` 一致。

<u>不属于原生 `Promise` 的值全都会被隐式地转换为 `Promise` 实例后等待</u>

async 函数内如果有一个 await 表达式， async 函数就一定会异步执行。（无论 await 任何表达式）

例如：

```js
async function foo() {
	await 1;
}
```

等价于

```js
function foo() {
	return Promise.resolve(1).then(() => undefined);
}
```

在 await 表达式之后的代码可以被认为是存在在链式调用的 then 回调中，多个 await 表达式都将加入链式调用的 then 回调中，返回值将作为最后一个 then 回调的返回值。

## async / await 对错误的处理

async 函数体中的 异常 会被 Promise.reject() 包裹，如果没有被 await 的话，属于异步抛出异常

<u>如果 await 表达式的 Promise 被拒绝，await 表达式会把 拒因 当作异常抛出</u>

在 JavaScript 中，当代码遇到 `throw` 语句时，会抛出一个异常，并立即终止当前的函数执行。但是，由于 `Promise` 构造函数是异步执行的，因此在 `throw` 抛出异常时，该函数并不会立即返回，而是返回一个 `Promise` 对象，并将异常保存在该对象中以供后续处理。

换句话说，`Promise` 构造函数中的 `throw` 语句并不会阻塞代码执行，因为它是在异步上下文中执行的。当你调用 `new Promise()` 时，JavaScript 引擎会创建一个新的 `Promise` 对象，并在调用构造函数时立即执行该函数，但是该函数内部的代码执行是在异步任务中完成的，因此不会阻塞代码执行。

由于 `Promise` 对象本身是异步的，它会在某个时间点调用 `then` 方法，并将捕获到的异常作为参数传递给该方法。这时候才会对异常进行处理，而不是在 `Promise` 构造函数中立即抛出异常并终止执行。这种方式使得我们能够更好地控制异步代码的执行流程，并在发生错误时进行处理。

## 全局异常

Promise 中抛出异常可以由两种方式处理

- Catch()
- Async / await 中的 try...catch...

如果这两种方式都没有出现，则异常将会被视为 “Uncaught (in promise)” 被抛出到全局去。 在 NodeJS 中，你可以通过`process.on('unhandledRejection', e => {...})`来捕获全局异常。 遗憾的是，在浏览器中，尚未发现有效的方式能捕获此类全局异常。 所以应当极力避免出现全局异常的情况，尤其是在前端，可能由异常导致流程中断。

**只要 Promise 的异常被方式一或方式二捕获，就不会抛出全局异常！**

## 看一个异常处理的例子

```js
async function fn() {
	throw new Error();
}

try {
	fn();
} catch (e) {
	console.log("捕获异常：", error);
}
```

例子中的异常并不会被捕获，而是以 `Uncaught (in promise) Error` 的形式抛出

异常未被正常捕获，是因为`promise`虽然出现在`try...catch...`中，但是并没有被`await`，如此将不进入上述的异常捕获流程，一旦出现异常并且没有其它有效的 catch 时，就将抛出至全局。
